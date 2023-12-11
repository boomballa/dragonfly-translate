# 哨兵晋升角色
# 哨兵促销
## [简介](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#introduction "直接链接到简介")
在上一节中，我们深入研究了[复制](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/replication)技术，这是一种使用`REPLICAOF`命令进行可靠数据传输的强大方法。然而，虽然这种技术具有显着的优点，但仍然存在一个小缺点：需要一个停机时间窗口，尽管可能很短暂。这需要重新配置并重新启动服务器应用程序。

在本节中，我们将通过介绍一项先进技术提升到新的复杂程度：利用 Redis Sentinel 的功能。***该技术有望超越停机时间的限制，使您能够以几乎不间断的**** *服务连续性执行迁移。

## Redis[哨兵](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#redis-sentinel "直接链接到 Redis Sentinel")
Redis Sentinel 是一个分布式系统，旨在监控和管理 Redis 实例，主要用于实现高可用性和自动故障转移。通过协调多个 Redis 实例的部署，Sentinel 可确保您的应用程序能够在节点故障中正常运行，从而保持系统的稳健性。本质上，Sentinel实例只是一个以特殊模式运行的Redis实例。自动处理故障转移的能力也可用于执行迁移。

## 迁移[步骤](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#migration-steps "直接链接到迁移步骤")
概括地说，涉及的步骤如下：

* 启动一个新的 Dragonfly 实例并将其配置为源（主）Redis 实例的副本。
* 将数据从源（主）Redis 实例复制到新的 Dragonfly 实例。
* 允许复制达到稳定状态并使用`INFO replication`命令进行监控。
* 让 Sentinel 提升 Dragonfly 实例成为新的主实例。

如您所见，这些步骤与[复制](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/replication)技术类似。这里的主要区别是利用 Sentinel 进行自动且可靠的转换，将 Dragonfly 实例提升为新的主实例。

在以下示例中，我们将假设：

* 源 Redis 实例使用主机名redis-source、IP 地址200.0.0.1和端口运行6379。
* 新的 Dragonfly 实例使用主机名dragonfly、IP 地址200.0.0.2和端口运行6380。
* Sentinel 实例使用主机名sentinel、IP 地址200.0.0.3和端口运行5000。

### 1\. 源[实例](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#1-source-instance "直接链接到 1. 源实例")
假设原始源Redis实例正在运行，您可以查看其复制信息：

```bash
redis-source:6379$> INFO replication
# Replication
role:master
connected_slaves:0
```
**假设源 Redis 实例已由 Sentinel 实例或 Sentinel 集群管理。否则，仍然需要重新配置服务器应用程序，这意味着潜在的停机时间窗口。** 但是，如果您的应用程序在 Kubernetes 等容器编排器上运行，则滚动更新机制可以帮助最大限度地减少停机时间。

### 2.配置[哨兵](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#2-configure-sentinel "直接链接到2.配置Sentinel")
**如果您的源 Redis 实例尚未由 Sentinel 管理，请按照以下步骤操作。**否则，继续[配置复制](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#3-configure-replication)。

要在 Sentinel 模式下启动 Redis 实例，您可以使用`sentinel.conf`如下所示的最小文件：

```Plain Text
# sentinel.conf
port 5000
sentinel monitor master-instance 200.0.0.1 6379 1
sentinel down-after-milliseconds master-instance 5000
sentinel failover-timeout master-instance 60000
```
配置标志的通用形式`sentinel monitor`如下。上面的示例非常小，仅用于说明目的。`quorum`在生产环境中，选择合理的值和其他可配置值至关重要。[在此处](https://github.com/redis/redis/blob/unstable/sentinel.conf)阅读有关 Sentinel 配置的更多信息。

```Plain Text
sentinel monitor <master-name> <ip> <redis-port> <quorum>
```
以Sentinel模式运行Redis实例：

```bash
$> redis-server sentinel.conf --sentinel
```
一旦 Sentinel 实例和 Sentinel 管理的 Redis 实例（在本例中为源 Redis 实例）运行，确认应用程序使用正确的客户端连接到它们非常重要。以[go-redis](https://github.com/redis/go-redis)库为例：

```go
import "github.com/redis/go-redis/v9"

// The <master-name> configuration has the value 'master-instance' in the 'sentinel.conf' file.
//
// There is only one Sentinel instance in our setup.
// In a production environment, multiple Sentinel instances with a reasonable 'quorum' value can be very important to achieve high availability.
client := redis.NewFailoverClient(&redis.FailoverOptions{
    MasterName:    "master-instance",
    SentinelAddrs: []string{"200.0.0.3:5000"},
})
```
如上面的代码片段所示，客户端不是直接连接到 Sentinel 管理的 Redis，而是通过 Sentinel 实例连接。原因是在故障转移过程中，Sentinel 需要通知客户端各种事件和信息，例如：

* 主实例离线。
* Sentinel 提升了一个新的主实例，这里是新主实例的网络信息。

Sentinel 到客户端的通知机制由 Pub/Sub 提供支持，您可以[在此处](https://redis.io/docs/management/sentinel/)阅读有关 Sentinel 内部结构的更多信息。 **使用与 Sentinel 兼容的客户端对于实现最短停机迁移目标至关重要。**

### 3\. 配置[复制](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#3-configure-replication "直接链接到 3. 配置复制")
全面介绍了Sentinel，目标仍然是将Redis实例迁移到Dragonfly实例。启动一个新的 Dragonfly 实例并使用`REPLICAOF`命令指示自身从源复制数据：

```bash
dragonfly:6380$> REPLICAOF 200.0.0.1 6379
"OK"
```
建立主副本关系后，您应该看到数据复制到新的 Dragonfly 实例中。我们可以再次检查两个实例上的复制信息：

```bash
redis-source:6379$> INFO replication
# Replication
role:master
connected_slaves:1
slave0:ip=200.0.0.2,port=6380,state=online,offset=15693,lag=2
# ... more
# ... output
# ... omitted
```
```bash
dragonfly:6380$> INFO replication
# Replication
role:replica
master_host:200.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:8
master_sync_in_progress:0
```
Sentinel 还应该了解主 Redis 实例以及作为副本加入的新 Dragonfly 实例：

```bash
sentinel:5000$> SENTINEL GET-MASTER-ADDR-BY-NAME master-instance

1) "200.0.0.1"
2) "6379"
```
```bash
sentinel:5000$> SENTINEL REPLICAS master-instance
1)  1) "name"
    2) "200.0.0.2:6380"
    3) "ip"
    4) "200.0.0.2"
    5) "port"
    6) "6380"
# ... more
# ... output
# ... omitted
```
### 4\. 将副本提升为[主要](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#4-promote-replica-as-primary "直接链接到 4. 将副本升级为主副本")
现在您已完成设置。让我们回顾一下当前的拓扑：

* 有一个主 Redis 实例 ( `redis-source`, `200.0.0.1:6379`)。
* 新的 Dragonfly 实例 ( `dragonfly`, `200.0.0.2:6380`) 是主 Redis 实例的副本。
* 还有一个 Sentinel 实例 ( `sentinel`, `200.0.0.3:5000`) 监视主节点和副本节点。

服务器应用程序通过 Sentinel 实例连接到主副本。如果您强制 Sentinel 进行故障转移，**这意味着将 Dragonfly 实例提升为新的主实例**，那么这将成功迁移到 Dragonfly。

有多种方法可以触发故障转移/升级，例如：

* 直接关闭主Redis实例。Sentinel 会将 Dragonfly 实例提升为主实例。
* 使用该`SENTINEL FAILOVER <master-name>`命令强制进行故障转移，就好像主服务器无法访问一样，然后关闭 Redis 实例。

**请注意，上述方法基于这样的假设：Dragonfly 实例是唯一可以升级的候选者，因为拓扑中没有其他副本。** 我们使用第二种方法：

```bash
sentinel:5000$> SENTINEL FAILOVER master-instance
OK

sentinel:5000$> SENTINEL GET-MASTER-ADDR-BY-NAME master-instance
1) "200.0.0.2"
2) "6380"

sentinel:5000$> SENTINEL REPLICAS master-instance
1)  1) "name"
    2) "200.0.0.1:6379"
    3) "ip"
    4) "200.0.0.1"
    5) "port"
    6) "6379"
# ... more
# ... output
# ... omitted
```
正如您所看到的，现在 Dragonfly 实例是主实例，我们可以安全地关闭作为副本的 Redis 实例。关闭 Redis 实例后，Sentinel 将无法再次将其提升为主实例。您已成功迁移到 Dragonfly。

## [注意事项](https://www.dragonflydb.io/docs/migrating-to-dragonfly/from-redis-instance/sentinel-promotion#considerations "直接链接到注意事项")
使用 Redis Sentinel 执行迁移引入了一系列对于确保平稳成功过渡至关重要的注意事项。以下是一些需要记住的关键注意事项：

* **停机时间：**虽然 Redis Sentinel 旨在最大限度地减少停机时间，但在故障转移/升级期间仍然可能会出现轻微的中断。评估您的应用程序对停机的容忍度并做出相应的计划。
* **兼容性和版本控制：**确保您使用的 Redis 和 Sentinel 版本彼此兼容并与 Dragonfly 兼容。版本不匹配可能会导致迁移过程中出现意外行为或问题。
* **网络延迟：**由于需要在源 Redis 主实例和 Dragonfly 副本之间同步数据，因此请考虑这些实例之间的网络延迟和带宽。连接速度慢或不稳定可能会影响数据复制速度和稳定性。
* **数据量：**评估数据集的大小以及数据更改的速率。较大的数据集或较高的写入速率可能需要更多时间来进行数据复制和同步。
* **故障转移测试：**在受控环境中测试故障转移场景，以确保 Sentinel 的行为符合预期。这有助于在迁移到生产环境之前识别并解决任何潜在问题。
* **监控和警报：**对源 Redis 实例和 Dragonfly 实例实施强大的监控。设置警报以通知您任何异常或潜在问题。
* **负载平衡：**考虑如何配置应用程序的负载平衡器或 DNS。Sentinel 驱动的迁移可能需要调整负载平衡设置，以确保流量定向到新的主节点。
* **备份策略：**在开始迁移之前制定可靠的备份策略。虽然 Redis Sentinel 有助于故障转移/升级，但将备份作为安全网至关重要。
* **回滚计划：**尽管进行了仔细的计划，但仍可能会出现不可预见的问题。制定回滚计划，以便您在迁移遇到重大问题时恢复到以前的设置。
* **文档和培训：**确保您的团队熟悉 Redis Sentinel 概念、配置和故障转移过程。适当的文档和培训可以减少迁移过程中的混乱。
* **测试环境：**只要有可能，请在密切模仿您的生产设置的临时或测试环境中测试迁移。这有助于在执行实际迁移之前识别并解决任何问题。

通过考虑这些因素并相应地定制迁移计划，您可以有效地利用 Redis Sentinel 执行从 Redis 到 Dragonfly 的无缝迁移，同时最大限度地降低风险并确保成功过渡。



