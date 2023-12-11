# 将 Prometheus 与 Dragonfly Operator 集成
本指南提供了使用 Dragonfly Operator 设置 Prometheus 的分步说明。在本指南中，您将使用[Promethues Operator](https://github.com/prometheus-operator/prometheus-operator) 和 PodMonitor 来管理 Prometheus 资源。

## [先决条件](https://www.dragonflydb.io/docs/managing-dragonfly/operator/prometheus-guide#prerequisites "Direct link to Prerequisites")
* [安装了 Dragonfly](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation)的 Kubernetes 集群

## 安装 Prometheus [Operator](https://www.dragonflydb.io/docs/managing-dragonfly/operator/prometheus-guide#install-prometheus-operator "直接链接到安装 Prometheus Operator")
首先确保集群中安装了所有 Prometheus CRD。运行以下命令来安装 Prometheus Operator。

```Plain Text
LATEST=$(curl -s https://api.github.com/repos/prometheus-operator/prometheus-operator/releases/latest | jq -cr .tag_name)
curl -sL https://github.com/prometheus-operator/prometheus-operator/releases/download/${LATEST}/bundle.yaml | kubectl create -f -
```
## 创建 PodMonitor [资源](https://www.dragonflydb.io/docs/managing-dragonfly/operator/prometheus-guide#create-the-podmonitor-resource%20%22%E5%88%9B%E5%BB%BA%20PodMonitor%20%E8%B5%84%E6%BA%90%E7%9A%84%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%22)
PodMonitors 允许 Prometheus 监控具有目标标签的特定 pod。这是监控 Dragonfly pod 的最简单方法。您可以创建自己的 PodMonitor 配置文件，也可以使用我们的示例[podMonitor.yaml](https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/podMonitor.yaml) 文件创建 PodMonitor 对象。

```Plain Text
kubectl apply -f https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/podMonitor.yaml
```
请注意，您必须在 PodMonitor中用`app: <dragonfly-name>`作为selector label来定位正确的 Dragonfly 实例。标签值是 Dragonfly 资源的名称（在本例中为`dragonfly-sample`）。

Dragonfly Pod 公开一个名为`admin`的端口，您可以将其用作 PodMonitor 中的endpoint。

## 创建 Promethues[资源](https://www.dragonflydb.io/docs/managing-dragonfly/operator/prometheus-guide#create-promethues-resources "直接链接到创建 Promethues 资源")
现在您已经安装了 Operator 和 PodMonitor，是时候创建 `prometheus`资源了。如果您启用了 RBAC，请首先创建必要的 `serviceaccount`、`clusterrole`和`clusterrolebinding`资源。

```Plain Text
$ kubectl apply -f https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/promServiceAccount.yaml
$ kubectl apply -f https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/promClusterRole.yaml
$ kubectl apply -f https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/promClusterBinding.yaml
```
这将允许 Prometheus 从 Dragonfly 资源中抓取数据。运行以下命令来创建 Prometheus 对象。它将创建一个名为 `prometheus-prometheus-0`的 pod 。

```Plain Text
$ kubectl apply -f https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/prometheus-config.yaml
```
您还可以创建一个指向 Prometheus Pod 的服务。

```Plain Text
$ kubectl apply -f https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/prometheus-service.yaml
```
运行`kubectl get all`检查是否所有资源都已成功创建。

```Plain Text
$ kubectl get all
NAME                                       READY   STATUS    RESTARTS         AGE
pod/dragonfly-sample-0                     1/1     Running   4 (3h50m ago)    12d
pod/dragonfly-sample-1                     1/1     Running   4 (3h50m ago)    12d
pod/prometheus-operator-744c6bb8f9-vnxw4   1/1     Running   16 (3h49m ago)   19d
pod/prometheus-prometheus-0                2/2     Running   6 (3h50m ago)    11d

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/dragonfly-sample      ClusterIP   10.96.2.149     <none>        6379/TCP   12d
service/kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP    45d
service/prometheus-operated   ClusterIP   None            <none>        9090/TCP   11d
service/prometheus-operator   ClusterIP   None            <none>        8080/TCP   19d
service/prometheus-svc        ClusterIP   10.96.8.22      <none>        9090/TCP   11d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-operator   1/1     1            1           19d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-operator-744c6bb8f9   1         1         1       19d

NAME                                     READY   AGE
statefulset.apps/dragonfly-sample        2/2     12d
statefulset.apps/prometheus-prometheus   1/1     11d
```
## 访问普罗米修斯[UI界面](https://www.dragonflydb.io/docs/managing-dragonfly/operator/prometheus-guide#access-prometheus-ui "直接链接到访问 Prometheus UI")
Prometheus 有一个漂亮的 UI，您可以使用它来查询某些指标。使用该 `port-forward`命令可以直接暴露 Prometheus pod（端口 9090），也可以暴露 Prometheus 服务。

```Plain Text
kubectl port-forward prometheus-prometheus-0 9090:9090
```
现在去`localhost:9090`. 恭喜！您刚刚将 Prometheus 与 Dragonfly 集成！