# 高可用
# 高可用性
高可用性 (HA) 是一种系统设计方法，可确保高水平的操作性能和正常运行时间。它通常是通过运行系统的多个实例并确保至少一个实例始终可用来实现的。借助 Dragonfly，您可以通过[复制](https://www.dragonflydb.io/docs/managing-dragonfly/replication)实现高可用性，并拥有可在主服务器发生故障时用作故障转移机制的只读副本。

使用 Dragonfly 实现高可用性取决于底层部署方法。在本节中，我们将介绍如何在以下场景中使用 Dragonfly 实现高可用性：

## Dragonfly [Operator](https://www.dragonflydb.io/docs/managing-dragonfly/high-availability#high-availability-with-dragonfly-operator "通过 Dragonfly Operator 直接链接到高可用性")
Dragonfly Operator 是一个管理 Dragonfly 实例的 Kubernetes Operator。它可以在[GitHub](https://github.com/dragonflydb/dragonfly-operator)上找到 ，并且可以安装在任何 Kubernetes 集群上。

Operator 的主要功能之一是开箱即用的高可用性。它允许您以最小的努力在高度可用的配置中运行 Dragonfly。默认情况下，当您将该`replicas`字段设置为大于 1 的值时，Operator 会自动将 Dragonfly 配置为以 HA 模式运行，其中一个实例为主实例，其余实例为副本实例。当这些 Pod 出现故障时，Operator 将自动重新配置复制，以维持所需数量的副本，并且一个主服务器始终可用。

应用程序客户端可以继续使用相同的 Dragonfly 服务，无需任何更改，并且 Operator 将自动更新 pod 选择器以指向正确的主节点。

让我们看看这在实践中是如何运作的。

按照[安装说明](https://www.dragonflydb.io/docs/getting-started/kubernetes-operator#installation)在 Kubernetes 集群上安装 Operator。

### 创建 Dragonfly[实例](https://www.dragonflydb.io/docs/managing-dragonfly/high-availability#creating-a-dragonfly-instance "直接链接到创建 Dragonfly 实例")
要创建示例 Dragonfly 实例，您可以运行以下命令：

```bash
kubectl apply -f https://raw.githubusercontent.com/dragonflydb/dragonfly-operator/main/config/samples/v1alpha1_dragonfly.yaml
```
这将创建一个具有 2 个副本的 Dragonfly 实例。您可以通过运行来检查实例的状态

```bash
kubectl describe dragonflies.dragonflydb.io dragonfly-sample
```
将创建以下`<dragonfly-name>.<namespace>.svc.cluster.local`形式的服务，用于选择主实例。您可以使用此服务连接到集群。添加/删除 pod 时，服务将自动更新以指向新的 master。

#### 使用 `redis-cli` 连接
要使用连接到集群`redis-cli`，您可以运行：

```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-sample.default
```
`redis-cli`这将创建一个运行并连接到集群的临时 Pod 。按 后`shift + R`，您就可以照常运行 Redis 命令。例如，要设置一个Key并尝试使用GET将其取回，您可以运行

```bash
If you don't see a command prompt, try pressing enter.
dragonfly-sample.default:6379> GET 1
(nil)
dragonfly-sample.default:6379> SET 1 2
OK
dragonfly-sample.default:6379> GET 1
"2"
dragonfly-sample.default:6379> exit
pod "redis-cli" deleted
```
您可以通过运行来检查哪个 pod 是 master

```bash
kubectl get pods -l role=master
NAME                 READY   STATUS    RESTARTS   AGE
dragonfly-sample-0   1/1     Running   0          2m
```
让我们看看Operator如何在面对故障时保持高可用性。

删除主 Pod：

```bash
kubectl delete pod -l role=master
```
Operator 将自动创建一个新的 master pod 并更新服务以指向新的 master。您可以通过运行来检查 Dragonfly 实例的状态

```bash
kubectl get pods -l role=master
NAME                 READY   STATUS    RESTARTS   AGE
dragonfly-sample-1  1/1     Running   0          2m
```
数据也应该保存。您可以通过运行来检查这一点

```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-sample.default
If you don't see a command prompt, try pressing enter.
dragonfly-sample.default:6379> GET 1
"2"
dragonfly-sample.default:6379> exit
pod "redis-cli" deleted
```
正如我们所看到的，数据仍然可用，并且服务指向新的主服务器。这样，Operator 可确保 Dragonfly 实例始终可用且零用户干预。

