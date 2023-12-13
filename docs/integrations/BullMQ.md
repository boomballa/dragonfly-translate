# BullMQ
## [简介](https://www.dragonflydb.io/docs/integrations/bullmq#introduction "Direct link to Introduction")
BullMQ 是一个 Node.js 库，它实现了一个构建在 Redis 之上的快速且强大的队列系统，有助于解决许多现代微服务架构的问题。

由于 Dragonfly 与 Redis 高度兼容，因此 BullMQ 可以与 Dragonfly 一起使用，只需零或最少的代码更改。通过使用 Dragonfly 替换 Redis，您可以为 BullMQ 应用程序实现卓越的性能和可扩展性。

## 使用[Dragonfly](https://www.dragonflydb.io/docs/integrations/bullmq#running-bullmq-with-dragonfly "直接链接到使用 Dragonfly 运行 BullMQ")
然而，Dragonfly 与 BullMQ 的集成涉及一些特定的配置步骤，以确保最佳性能以及与 BullMQ 内部的兼容性。

BullMQ 广泛使用 Lua 脚本（服务器端脚本）在 Redis 中执行命令。在 Redis 中运行 Lua 脚本时，必须显式指定脚本将访问的所有键。然而，BullMQ 的设计不允许它提前预测其 Lua 脚本将需要哪些键。尽管 Redis 不支持访问未声明的密钥，但它仍然有效。然而，在 Dragonfly 中，由于我们的多线程事务框架，它无法开箱即用。

因此，可以使用 Lua 全局事务模式运行 Dragonfly：

```bash
$> ./dragonfly --default_lua_flags=allow-undeclared-keys
```
该模式会锁定每个 Lua 脚本的整个数据存储。换句话说，它很慢，但是很安全，并且不需要对使用 BullMQ 的代码进行任何更改。

## 高级和优化的[配置](https://www.dragonflydb.io/docs/integrations/bullmq#advanced--optimized-configurations "直接链接到高级和优化配置")
为了利用 Dragonfly 的多线程功能并为您的应用程序实现卓越的性能，我们引入了一种允许对主题标签而不是单个密钥进行锁定的模式。在这种模式下，每个 BullMQ 队列将由单个线程独占，并且可以并行访问多个队列。要在此模式下使用 Dragonfly，请按照以下步骤操作。

### 1\. 标签[锁定](https://www.dragonflydb.io/docs/integrations/bullmq#1-hashtag-locking "直接链接到 1. 标签锁定")
使用以下标志运行 Dragonfly：

```bash
$> ./dragonfly --cluster_mode=emulated --lock_on_hashtags
```
* `--cluster_mode=emulated`指示 Dragonfly 在单个实例上模拟 Redis 集群。
* `--lock_on_hashtags`启用主题标签锁定。[下一步](https://www.dragonflydb.io/docs/integrations/bullmq#2-queue-naming-strategies)将详细介绍主题标签。

### 2\. 队列命名[策略](https://www.dragonflydb.io/docs/integrations/bullmq#2-queue-naming-strategies "直接链接到 2. 队列命名策略")
设置应用程序时，请在队列名称中使用主题标签。简而言之，当密钥中存在主题标签以及标志时`--lock_on_hashtags`，Dragonfly 将根据主题标签将密钥锁定到线程。[在此处](https://redis.io/docs/reference/cluster-spec/#hash-tags)阅读有关 Redis 集群规范的原始主题标签设计以及[此处的](https://www.dragonflydb.io/docs/managing-dragonfly/cluster-mode)Dragonfly 模拟集群模式的更多信息。

为了在队列名称中使用主题标签，您可以按如下方式初始化队列：

```javascript
import { Queue } from 'bullmq';
// Note the wrapping curly brackets in the queue name string.
//
// Dragonfly assigns this queue to a specific thread based on only the substring "myqueue",
// instead of the full queue name string "{myqueue}". Despite how your queue name is formatted,
// Dragonfly will only consider the substring "myqueue" as the hashtag.
//
// Also, do not confuse the curly brackets here with JavaScript template literals.
// These curly brackets are part of the queue name string that will be used by Dragonfly.
const queue = new Queue("{myqueue}");
```
或者，您可以利用 BullMQ`prefix`选项进行队列初始化：

```javascript
import { Queue } from 'bullmq';

const queue = new Queue("myqueue", {
  prefix: "{myprefix}",
});
```
通过采用上述队列命名策略，共享相同主题标签的队列将被分配到同一个 Dragonfly 线程。这确保了分片一致性和高效的硬件资源利用。

但是，正因为如此，如果您使用前缀主题标签，则**使用唯一的每个队列前缀非常重要，这样并非所有队列都由同一 Dragonfly 线程处理**。有关更多详细信息，请参阅下面的[线程平衡。](https://www.dragonflydb.io/docs/integrations/bullmq#3-thread-balancing)

您应该始终避免对所有队列使用相同的主题标签。但有时，有策略地将某些队列放置在同一个 Dragonfly 线程上是很好的。有关更多详细信息，请参阅下面的[队列依赖性。](https://www.dragonflydb.io/docs/integrations/bullmq#4-queue-dependencies)

### 3\. 线程[平衡](https://www.dragonflydb.io/docs/integrations/bullmq#3-thread-balancing "直接链接到 3. 线程平衡")
为了使您的应用程序获得卓越的性能，请考虑使用大量队列，每个队列具有不同的主题标签。通过将队列分布在不同的 Dragonfly 线程中，您可以有效地优化 Dragonfly 架构的利用率。

### 4\. 队列[依赖](https://www.dragonflydb.io/docs/integrations/bullmq#4-queue-dependencies "直接链接到 4. 队列依赖项")
如果您具有队列依赖性，尤其是父子关系，则为两个队列使用相同的主题标签非常重要。这可确保它们在同一 Dragonfly 线程中进行处理并保持依赖关系的完整性。

## 有用的[资源](https://www.dragonflydb.io/docs/integrations/bullmq#useful-resources "直接链接到有用资源")
* BullMQ[主页](https://bullmq.io/)、[GitHub](https://github.com/taskforcesh/bullmq)和[文档](https://docs.bullmq.io/)。

