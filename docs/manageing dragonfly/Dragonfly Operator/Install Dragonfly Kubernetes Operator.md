# 安装 Dragonfly Kubernetes Operator
Dragonfly Operator 是一个 Kubernetes Operator，用于在 Kubernetes 集群中部署和管理[Dragonfly实例。](https://dragonflydb.io/)

主要特点包括：

* 自动故障转移
* 水平和垂直扩展（使用自定义推出策略）
* 自定义配置选项
* 身份验证和服务器 TLS
* 持久卷声明 (PVC) 和 S3 兼容云存储的快照
* 使用 Prometheus 和 Grafana 进行监控

## [先决条件](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation#prerequisites "Direct link to Prerequisites")
* 一个工作的 Kubernetes 集群（使用 Kubernetes 1.19+ 进行测试）
* [kubectl](https://kubernetes.io/docs/tasks/tools/)安装并配置为连接到您的集群

## [安装](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation#installation "Direct link to Installation")
确保您的 Kubernetes 集群已启动并正在运行。要安装 Dragonfly Operator，请运行：

```bash
# Install the CRD and Operator
kubectl apply -f https://raw.githubusercontent.com/dragonflydb/dragonfly-operator/main/manifests/dragonfly-operator.yaml
```
默认情况下，该operator将安装在`dragonfly-operator-system`namespace中。

## [用途](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation#usage "Direct link to Usage")
### 创建带有[副本的Dragonfly实例](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation#create-a-dragonfly-instance-with-replicas "直接链接到使用副本创建 Dragonfly 实例")
1. 要创建具有一个主实例和三个副本的示例 Dragonfly 实例，请运行以下命令：

```bash
kubectl apply -f https://raw.githubusercontent.com/dragonflydb/dragonfly-operator/main/config/samples/v1alpha1_dragonfly.yaml
```
2. 要检查实例的状态，请运行：

```bash
kubectl describe dragonflies.dragonflydb.io dragonfly-sample
```
3. 连接到服务的主实例：
`<dragonfly-name>.<namespace>.svc.cluster.local`。

随着 Pod 的添加或删除，该服务会自动更新以指向新的 master。

#### 使用`redis-cli`客户端连接
要使用 连接到实例`redis-cli`，请运行：

```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-sample.default
```
该命令创建一个运行`redis-cli`并连接到实例的临时 Pod。要运行 Redis 命令，请按`shift + R`，然后输入 Redis 命令。例如，要设置和检索密钥，您可以运行：

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
### 更改副本[实例数量](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation#change-the-number-of-replica-instances "更改副本实例数量的直接链接")
要更改副本实例的数量，请编辑`spec.replicas`Dragonfly 实例中的字段。例如，要扩展到 5 个副本，请运行：

```bash
kubectl patch dragonfly dragonfly-sample --type merge -p '{"spec":{"replicas":5}}'
```
### 传递自定义 Dragonfly[参数](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation#pass-custom-dragonfly-arguments "传递自定义 Dragonfly 参数的直接链接")
要将自定义参数传递给 Dragonfly，请编辑`spec.args`Dragonfly 实例中的字段。例如，要将 Dragonfly 配置为需要密码，请运行：

```bash
kubectl patch dragonfly dragonfly-sample --type merge -p '{"spec":{"args":["--requirepass=supersecret"]}}'
```
### 垂直扩容[实例](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation#vertically-scale-the-instance%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E5%9E%82%E7%9B%B4%E7%BC%A9%E6%94%BE%E5%AE%9E%E4%BE%8B%22)
要垂直扩容实例，请编辑`spec.resources`Dragonfly 实例中的字段。例如，要将 CPU 请求增加到 2 个内核，请运行：

```bash
kubectl patch dragonfly dragonfly-sample --type merge -p '{"spec":{"resources":{"requests":{"cpu":"2"}}}}'
```
要了解如何配置高可用性，请参阅[高可用性](https://www.dragonflydb.io/docs/managing-dragonfly/high-availability#high-availability-with-dragonfly-operator)部分。

