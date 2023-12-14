# Install on Kubernetes with Helm Chart
# 使用 Helm Chart 在 Kubernetes 上安装
本指南介绍如何使用 Helm 在 Kubernetes 集群上部署 Dragonfly（请参阅[安装 Helm](https://helm.sh/docs/intro/install/)）。

## [先决条件](https://www.dragonflydb.io/docs/getting-started/kubernetes#prerequisites "Direct link to Prerequisites")
* Kubernetes 集群（如果您想在本地进行实验，请参阅[Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)或[Minikube ）。](https://minikube.sigs.k8s.io/docs/start/)
* 选择蜻蜓版本：
* 对于最新版本，设置： `VERSION=v1.13.0`
* [从这里](https://github.com/dragonflydb/dragonfly/pkgs/container/dragonfly%2Fhelm%2Fdragonfly)选择一个版本

## 安装独立[主控](https://www.dragonflydb.io/docs/getting-started/kubernetes#install-a-standalone-master "直接链接安装独立主站")
运行这个命令：

```Plain Text
helm upgrade --install dragonfly oci://ghcr.io/dragonflydb/dragonfly/helm/dragonfly --version $VERSION
```
## 安装一个独立的 master，每[分钟](https://www.dragonflydb.io/docs/getting-started/kubernetes#install-a-standalone-master-with-snapshot-taken-every-minute "直接链接到安装独立主服务器，每分钟拍摄一次快照")
1. 将以下内容添加到`myvals.yaml`值文件中（如果不存在则创建一个新文件）：

```yaml
storage:

  enabled: true

  requests: 128Mi # Set as desired



extraArgs:

  - --dbfilename=dump.rdb

  - --save_schedule=*:* # HH:MM glob format



podSecurityContext:

  fsGroup: 2000



securityContext:

  capabilities:

    drop:

      - ALL

  readOnlyRootFilesystem: true

  runAsNonRoot: true

  runAsUser: 1000

```
2. 运行这个命令： `helm upgrade -f myvals.yaml --install dragonfly oci://ghcr.io/dragonflydb/dragonfly/helm/dragonfly --version $VERSION`

## 与 Kube-Prometheus[监控](https://www.dragonflydb.io/docs/getting-started/kubernetes#integrate-with-kube-prometheus-monitoring "与 Kube-Prometheus 监控集成的直接链接")
如果您的集群中安装了[Kube-Prometheus](https://github.com/prometheus-operator/kube-prometheus)，则可以通过在值文件中启用`serviceMonitor`和来将其设置为监视 Dragonfly 部署`prometheusRule`。例如：

```yaml
serviceMonitor:

  enabled: true



prometheusRule:

  enabled: true

  spec:

    - alert: DragonflyMissing

      expr: absent(dragonfly_uptime_in_seconds) == 1

      for: 0m

      labels:

        severity: critical

      annotations:

        summary: Dragonfly is missing

        description: "Dragonfly is missing"

```
## 更多[定制](https://www.dragonflydb.io/docs/getting-started/kubernetes#more-customization "直接链接到更多定制")
如需更多定制，请参阅[蜻蜓图](https://github.com/dragonflydb/dragonfly/tree/main/contrib/charts/dragonfly)

