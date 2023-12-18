# 副本
## 副本管理
Dragonfly 支持主/从复制模型，类似于 Redis 的[复制](https://redis.io/topics/replication)。使用复制时，Dragonfly 创建主实例的精确副本。正确配置后，secondary实例在连接中断时会重新连接到主实例，并且始终努力保持与主要副本完全一致。

Dragonfly 复制管理 API 与 Redis API 兼容，由两个面向用户的命令组成：ROLE 和 REPLICAOF (SLAVEOF)。

如果您不确定当前连接的 Dragonfly 实例是主实例还是副本实例，可以通过运行以下命令进行检查 `role` ：

```bash
role
```
该命令将返回 `master` 或 `replica` 。

## Redis/Dragonfly复制[](/docs/managing-dragonfly/Replication.md#redisdragonfly复制 "直接链接到使用 redis/dragonfly复制")
Dragonfly 支持 Redis -> Dragonfly 复制，以便将 Redis 工作负载方便地迁移到 Dragonfly。目前我们支持Redis OSS的数据结构和复制协议，最高版本为6.2。以下说明也适用于这种类型的复制，唯一的区别是主实例是正在运行的 Redis 服务器。

## Dragonfly/Dragonfly复制[](/docs/managing-dragonfly/Replication.md#dragonflydragonfly复制 "直接链接到使用 dragonfly/dragonfly复制")
这个复制过程在内部与原来的Redis复制算法有很大不同，但从外部来看，API保持相同，以使其与当前的生态系统兼容。

要将 Dragonfly 实例指定为另一个动态实例的副本，请运行以下 `replicaof` 命令。此命令采用预期的主服务器的主机名或 IP 地址和端口作为参数：

```bash
replicaof hostname_or_IP port
```
如果该服务器已经是另一台主服务器的副本，它将停止复制旧服务器并立即开始与新服务器同步。它还将丢弃旧的数据集。

要将副本提升回master副本，请运行以下`replicaof`命令：

```bash
replicaof no one
```
这将阻止实例复制master服务器，但不会丢弃已复制的数据集。当原始主数据库发生故障时，此语法非常有用。在发生故障的主副本的副本上运行后`replicaof no one`，以前的副本可以用作新的master副本，并拥有自己的副本作为故障保护。

## 使用 TLS 进行安全复制——[先决条件](/docs/managing-dragonfly/Replication.md#使用-tls-进行安全复制先决条件 "直接链接到使用 TLS 进行安全复制 — 先决条件")
Dragonfly 支持通过 TLS 进行复制。为了使其发挥作用，您需要：

1. 将 Dragonfly 实例配置为使用 TLS。这包括获取或生成服务器证书。
2. 选择副本和master服务器之间的身份验证方法。这可以使用密码或客户端证书。

另请注意，我们仅支持`.pem`文件，任何其他格式都不适用于Dragonfly。

您可以使用它`OpenSSL`来生成所需的证书。例如，生成服务器证书并使用自我管理的CA对其进行签名的过程如下：

生成 Dragonfly 的私钥和证书签名请求（CSR）：

```bash
openssl req -newkey rsa:4096 -nodes -keyout server-key.pem -out server-req.pem
```
这将提示您为主题添加其他详细信息，其中包含与证书相关的信息。主题的属性是键值对，如下：

* CN: 通用名称
* OU：组织单位
* O：组织
* L：地点
* S：州或省名称
* C: 国家名称

该`-nodes`部分的意思`no DES`是，即不加密私钥。对于生产用例，您应该始终考虑加密私钥。

最后，您应该使用CA的私钥对Dragonfly的CSR进行签名并取回签名的证书

```bash
openssl x509 -req -in server-req.pem -days 365 -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem
```
和可以像上述第一步一样生成`ca-cert.pem`。`ca-key.pem`相同的过程适用于生成客户端密钥。

生成私钥时，请始终注意安全地存储和配置它们！

## 使用[TLS](/docs/managing-dragonfly/Replication.md#使用tls "直接链接到使用 TLS 进行安全复制")
要为 Dragonfly 启用 TLS，您必须提供以下参数：

```bash
dragonfly --tls --tls_key_file=server-key.pem --tls_cert_file=server-cert.pem --tls_ca_cert_file=ca-cert.pem
```
在不同的端口上启动副本（在本示例中，端口为 6380，主机为本地主机）：

```bash
dragonfly --tls --tls_key_file=replica-client-key.pem --tls_cert_file=replica-client-cert.pem --tls_ca_cert_file=ca-cert.pem --tls_replication=true --port 6380
```
最后，连接到副本并发出命令`REPLICAOF`：

```bash
redis-cli --tls --key ./client-key.pem --cert ./client-cert.pem --cacert ./ca-cert.pem -p 6380

REPLICAOF LOCALHOST 6379
```
现在，复制通过 TLS 运行并且是安全的。此示例假设这`ca-cert.pem`是一个用于验证服务器和客户端身份的根证书。例如，如果服务器使用由公共 CA 签名的证书，则不应将此标志作为参数传递给副本。

## 配置 TLS 的[Dragonfly](/docs/managing-dragonfly/Replication.md#配置-tls-的dragonfly "通过 TLS 配置的 Dragonfly 上的管理端口直接链接到非 TLS 连接")
有时，主节点和副本节点可能位于同一专用网络上。对于这种情况，您可能需要 TLS 用于专用网络之外的连接，但通过更安全的专用网络进行明文通信。在这种情况下，您可以专门在管理端口上禁用 tls 复制：

开始大师：

```bash
dragonfly --tls --tls_key_file=server-key.pem --tls_cert_file=server-cert.pem --tls_ca_cert_file=ca-cert.pem -admin_port=6380 --no_tls_on_admin_port=true
```
启动副本：

```bash
dragonfly --tls --tls_key_file=replica-client-key.pem --tls_cert_file=replica-client-cert.pem --tls_ca_cert_file=ca-cert.pem --port 6381 -admin_port=6382
```
然后`REPLICAOF`通过管理端口发出命令：

```bash
redis-cli -p 6382

REPLICAOF HOST 6380
```
从现在开始，副本\_不再\_通过 TLS 进行通信，但端口上的传入连接`6379, 6381`需要 TLS，并且被认为是安全的。

## 监控[延迟](/docs/managing-dragonfly/Replication.md#监控延迟 "直接链接到监控延迟")
Dragonfly 将复制延迟定义为所有分片中未确认数据库的最大数量。[该指标由主实例计算，并且作为prometheus 指标](/docs/managing-dragonfly/Monitoring.md)`dfly_connected_replica_lag_records`中的字段以及通过命令可见。`INFO REPLICATION`

