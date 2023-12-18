# 集群模式
Dragonfly 有 2 种集群模式：

1. 模拟集群（Emulated模式）：单个 Dragonfly 节点，通常在从 Redis 集群迁移时很有帮助
2. 多节点集群：将多个 Dragonfly 服务器加入到单个数据存储中

## 模拟[集群](/docs/managing-dragonfly/Cluster-Mode.md#模拟集群emulated模式)（Emulated模式）
单个 Dragonfly 实例可以达到与多节点 Redis 集群相同的容量。为了帮助将应用程序从 Redis 集群迁移到 Dragonfly，Dragonfly 可以模拟 Redis 集群。

```bash
# Running Dragonfly instance in cluster mode
$ dragonfly --cluster_mode=emulated
$ redis-cli 
# See which cluster commands are supported
127.0.0.1:6379> cluster help
```
现在您可以使用 Redis 集群客户端连接到 Dragonfly 实例，这是一个 Python 示例：

```python
>>> import redis
>>> r = redis.RedisCluster(host='localhost', port=6379)
>>> r.set('foo', 'bar')
True
>>> r.get('foo')
b'bar'
```
请注意，如果您的应用程序代码可能使用不需要集群命令的 Redis 客户端，在这种情况下，您不需要指定该`--cluster_mode`标志。

如果您需要将数据从 Redis 集群迁移到 Dragonfly，您可能会发现[Redis Riot](https://developer.redis.com/explore/riot/)很有帮助。

## 多节点[集群](/docs/managing-dragonfly/Cluster-Mode.md#多节点集群 "直接链接到多节点集群")
有时垂直缩放是不够的。在这些情况下，您需要设置 Dragonfly 集群。

Dragonfly 集群类似于 Redis 集群：

* 多个 Dragonfly 服务器参与单个逻辑数据存储
* 它提供了Redis客户端库所需的所有集群相关命令
* 它以与 Redis 相同的方式[分发key](https://redis.io/docs/reference/cluster-spec/#key-distribution-model)
* 它支持[哈希标签](https://redis.io/docs/reference/cluster-spec/#hash-tags)（hash tags）

**任何使用 Redis 集群的客户端代码都应该能够无需更改地迁移到 Dragonfly 集群。**Dragonfly 集群在所有面向客户端的行为中与 Redis 集群类似，但它不像 Redis 集群那样进行自我管理，在 Redis 集群中，节点相互通信以发现集群设置和状态。

设置和管理 Dragonfly 集群与管理 Redis 集群不同。与 Redis 不同，Dragonfly 节点之间不进行通信（复制除外）。节点不知道其他节点不可用，并且集群配置是针对每个节点单独完成的。

Dragonfly仅提供 ***数据平面***（即Dragonfly服务器），但我们不**提供**控制 ***平面 ***来管理集群部署。

请按照以下步骤设置 Dragonfly 集群

### 启动Dragonfly[节点](/docs/managing-dragonfly/Cluster-Mode.md#启动dragonfly节点)
启动您需要的 Dragonfly 节点。使用您原本会使用的标志，但请确保添加以下内容（同时添加到主副本和副本）：

* `--cluster_mode=yes`让节点知道它们是集群的一部分。
* `--admin_port=X`，其中`X`是您将用于运行 ***管理命令的 ***某个端口。该端口通常不应暴露给用户。

注意：以集群模式启动时，节点将对任何用户请求回复错误，直到正确配置为止。

### 配置[复制](/docs/managing-dragonfly/Cluster-Mode.md#配置复制 "直接链接到配置复制")
复制的配置方式与非集群复制类似，使用`REPLICAOF`命令进行配置。请注意，即使此信息存在于集群配置中（见下文），这也是必要的步骤。

### 配置[节点](/docs/managing-dragonfly/Cluster-Mode.md#配置节点 "直接链接到配置节点")
注意：配置集群节点是通过 ***管理端口***** **发送管理命令来完成的。Dragonfly 将拒绝通过常规端口处理集群管理命令。

集群配置包括集群节点运行所需的信息，例如哪些节点参与集群（以及扮演什么角色）、哪个节点拥有哪些槽等。

您需要做的第一件事是获取每个节点的 ***id***，它是标识节点的唯一字符串。`DFLYCLUSTER MYID`可以通过在每个节点上发出命令来检索 ID 。

获得所有 ID 后，构建配置字符串（见下文）并使用命令将其传递到所有节点`DFLYCLUSTER CONFIG <str>`。

一旦配置了服务器，它将只处理它拥有的插槽。这意味着任何不属于它的密钥都将被删除，并且任何访问此类密钥的尝试都将被拒绝。

Dragonfly 服务器不与其他服务器通信，以确保所有配置都相同。将完全相同的配置传递给所有节点非常重要。

配置是一个 JSON 编码的字符串，格式如下：

```json
[  // Array of node-entries
  {  // Node #1
    "slot_ranges": [
      { "start": X1, "end": Y1 },  // Slots [X1, Y1) owned by node 1
      { "start": X2, "end": Y2 }   // Slots [X2, Y2) are also owned by node 1
    ],
    "master": {
      "id": "...",  // Returned from DFLYCLUSTER MYID
      "ip": "...",  // The IP or hostname that *clients* should use to access this node
      "port": ...   // The port that *clients* should use to access this node
    },
    "replicas": [  // Replicas use the same fields as the master config
      { "id": "...", "ip": "...", port: ... },  // First replica for node
      { "id": "...", "ip": "...", port: ... }   // Second replica for node
    ]
  },
  { ... }, // Node #2
  { ... }, // Node #3, etc
]
```
以下是具有 2 个主节点的集群配置示例，每个主节点有 1 个副本：

```json
[
   {
      "slot_ranges": [
         {
            "start": 0,
            "end": 8191
         }
      ],
      "master": {
         "id": "...",
         "ip": "...",
         "port": ...
      },
      "replicas": [
         {
            "id": "...",
            "ip": "...",
            "port": ...
         }
      ]
   },
   {
      "slot_ranges": [
         {
            "start": 8192,
            "end": 16383
         }
      ],
      "master": {
         "id": "...",
         "ip": "...",
         "port": ...
      },
      "replicas": [
         {
            "id": "...",
            "ip": "...",
            "port": ...
         }
      ]
   }
]
```
重新启动后，您需要将配置重新发送到节点，并在集群发生任何更改（例如添加/删除节点、更改主机名等）时使用新配置更新所有节点。

### 结尾的[笔记](/docs/managing-dragonfly/Cluster-Mode.md#结尾的笔记)
* 集群模式仍在开发中。如果怀疑您发现了错误，请提交问题。
* 您可以将其 `cluster_mgr.py`作为如何设置和配置集群的参考。***该脚本在本地 ***启动集群，但其大部分逻辑也可以重用于远程计算机上的节点。
* Dragonfly尚不支持节点之间槽数据的迁移。更改槽分配将导致数据删除。



