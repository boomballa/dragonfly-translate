#  通过PVC进行快照
本指南提供了通过 PVC 设置 Dragonfly 和快照的分步说明。[Dragonfly 已经支持快照](https://www.dragonflydb.io/docs/managing-dragonfly/backups)，它允许您在任何时间点创建数据库快照并在以后恢复。通过这种 PVC 集成，您现在可以将快照存储在[持久卷](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)中 ，并在 Dragonfly Pod 重新启动或跨节点重新安排时自动恢复它们。这是可能的，因为每个 Dragonfly 都附加了一个特定的 PVC `statefulset`，并且快照存储在该 PVC 中。持久卷可以由 Kubernetes 支持的任何存储提供商提供支持。

虽然此功能可以帮助您从故障中恢复，但它并不能替代复制。如果您想保持高可用性，您仍然应该使用复制。

## [先决条件](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-pvc#prerequisites "Direct link to Prerequisites")
* [安装了 Dragonfly](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation)的 Kubernetes 集群。
* 相关[存储提供程序](https://kubernetes.io/docs/concepts/storage/storage-classes/)已在集群中安装并可用。

## 通过[PVC](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-pvc#deploying-dragonfly-with-snapshots-through-pvc "通过 PVC 直接链接到使用快照部署 Dragonfly")
在默认storage class上应用启用 PVC 的 Dragonfly 清单：

```bash
kubectl apply -f - <<EOF
apiVersion: dragonflydb.io/v1alpha1
kind: Dragonfly
metadata:
  name: dragonfly-pvc
spec:
  replicas: 1
  snapshot:
    cron: "*/5 * * * *"
    persistentVolumeClaimSpec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
EOF
```
这将创建具有给定 PVC 规范的 Dragonfly Statefulset。该字段`persistentVolumeClaimSpec`与[Kubernetes PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)中使用的字段相同 ，可用于根据您的要求配置 PVC。

等待 Dragonfly 实例准备就绪：

```bash
kubectl describe dragonflies.dragonflydb.io dragonfly-pvc
```
## [测试](https://www.dragonflydb.io/docs/managing-dragonfly/operator/snapshot-pvc#testing "Direct link to Testing")
连接到 Dragonfly 实例并添加一些数据：

```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-pvc.default SET foo bar
If you don't see a command prompt, try pressing enter.
pod "redis-cli" deleted
```
删除 Dragonfly pod：

```bash
kubectl delete pod dragonfly-pvc-0
```
等待 pod 重新创建：

```bash
kubectl get pods -w
```
连接到 Dragonfly 实例，并检查数据是否仍然存在：

```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-pvc.default GET foo
"bar"
pod "redis-cli" deleted
```
您应该看到key`bar` 的值`foo`。这意味着数据是从 PVC 中存储的快照恢复的。

