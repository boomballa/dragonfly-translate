# 将 Grafana 与 Dragonfly operator集成
在本指南中，我们将了解如何设置 Grafana 仪表板来监控 kubernetes 集群中的 Dragonfly Pod。

## [先决条件](https://www.dragonflydb.io/docs/managing-dragonfly/operator/grafana-guide#prerequisites "Direct link to Prerequisites")
* [安装了 Dragonfly](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation)的 kubernetes 集群
* Prometheus 在集群中的设置。请参阅[Prometheus 集成指南](https://www.dragonflydb.io/docs/managing-dragonfly/operator/prometheus-guide)
* 您已安装[Helm](https://helm.sh/docs/intro/install/)。

如果您的集群中已有 Grafana 实例，请使用[grafana-dashboard.json](https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/grafana-dashboard.json) 导入仪表板。

## 安装 Grafana Helm [Chart](https://www.dragonflydb.io/docs/managing-dragonfly/operator/grafana-guide#install-grafana-helm-chart%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E5%AE%89%E8%A3%85%20Grafana%20helm%20%E5%9B%BE%E8%A1%A8%22)
我们将使用[Grafana Helm Chart](https://github.com/grafana/helm-charts)来设置 Grafana。因此请确保您已安装[Helm](https://helm.sh/docs/intro/install/)。

```Plain Text
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana-ui grafana/grafana
```
运行该命令后，`grafana`将创建设置 grafana 所需的所有资源。运行以下命令检查所有资源是否创建成功。

```Plain Text
$ kubectl get all
NAME                                       READY   STATUS    RESTARTS        AGE
pod/dragonfly-sample-0                     1/1     Running   1 (171m ago)    23h
pod/dragonfly-sample-1                     1/1     Running   1 (171m ago)    23h
pod/grafana-ui-785f79fb65-rmk52            1/1     Running   1 (171m ago)    23h
pod/prometheus-operator-744c6bb8f9-vnxw4   1/1     Running   10 (170m ago)   8d
pod/prometheus-prometheus-0                2/2     Running   0               167m

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/dragonfly-sample      ClusterIP   10.96.2.149     <none>        6379/TCP   23h
service/grafana-ui            ClusterIP   10.96.146.235   <none>        80/TCP     23h
service/kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP    33d
service/prometheus-operated   ClusterIP   None            <none>        9090/TCP   167m
service/prometheus-operator   ClusterIP   None            <none>        8080/TCP   8d

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana-ui            1/1     1            1           23h
deployment.apps/prometheus-operator   1/1     1            1           8d

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-ui-785f79fb65            1         1         1       23h
replicaset.apps/prometheus-operator-744c6bb8f9   1         1         1       8d

NAME                                     READY   AGE
statefulset.apps/dragonfly-sample        2/2     23h
statefulset.apps/prometheus-prometheus   1/1     167m
```
## 创建普罗米修斯[服务](https://www.dragonflydb.io/docs/managing-dragonfly/operator/grafana-guide#create-prometheus-service "直接链接到创建 Prometheus 服务")
创建一个prometheus Service，让Grafana通过它访问prometheus。如果您已有 Prometheus 服务对象，请转到下一步。

```Plain Text
kubectl apply -f https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/prometheus-service.yaml
```
请注意，需要使用`prometheus: prometheus`标签作为选择器（如果没有为现有 Prometheus 对象指定标签）。prometheus Operator 暴露了一个名为 `web` 的端口，因此您可以在服务中使用该端口作为 targetPort。

## 创建 Grafana[仪表板](https://www.dragonflydb.io/docs/managing-dragonfly/operator/grafana-guide#create-grafana-dashboard "直接链接到创建 Grafana 仪表板")
Helm Chart 还创建了一项`grafana-ui`服务，您可以使用该服务来转发 Grafana 端口。

```Plain Text
kubectl port-forward services/grafana-ui 3000:80
```
现在去`localhost:3000`. 您将看到在本地主机中运行的 grafana 仪表板。添加 Prometheus 数据源。用作`http://prometheus-svc:9090`数据源 url。之后，将[grafana-dashboard.json](https://github.com/dragonflydb/dragonfly-operator/blob/main/monitoring/grafana-dashboard.json)导入仪表板。