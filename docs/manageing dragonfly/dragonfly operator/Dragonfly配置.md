# Dragonfly配置
Dragonfly Operator 使用专用的 Dragonfly CRD 来创建和管理 Dragonfly 资源。CRD 允许各种配置来定义Dragonfly控制器和Dragonfly吊舱的行为。下面是 Dragonfly CRD 字段的表。

|领域|类型|描述|
| ----- | ----- | ----- |
|affinity|[亲和力](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#affinity-v1-core)|Dragonfly荚亲和力（可选）  <br>spec:<br>  affinity: <br>    nodeaffinity:<br>      ...<br>您可以[在此处](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)了解有关亲和力的更多信息。|
|replicas|整数|包括主实例在内的 Dragonfly 实例总数。|
|image|string|要使用的Dragonfly图像。默认为`docker.dragonflydb.io/dragonflydb/dragonfly:v1.12.0`|
|args|\[\]string|（可选）传递给容器的 Dragonfly 容器参数。请参阅 Dragonfly 文档以获取支持的参数列表。例子 -  <br>spec:<br>  args:<br>   - "--cluster_mode=emulated"|
|annotations|object	|（可选）添加到 Dragonfly pod 的注释。请参阅[注释](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)以了解有关注释的更多信息。|
|env|array|添加到 Dragonfly pod 的环境变量。例子 -  <br>spec:<br>  env:<br>   - name: DEBUG<br>     value: true|
|resources|[资源需求](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#resourcerequirements-v1-core)|（可选）Dragonfly 容器资源限制。可以指定任何容器限制。|
|tolerations|\[ \][耐受性](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#toleration-v1-core)|（可选）Dragonfly pod 耐受性。请参阅[k8s 文档](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)以了解有关容忍度的更多信息|
|serviceAccountName|string|（可选）Dragonfly Pod 服务帐户名称|
|serviceSpec.type|string|（可选）Dragonfly服务类型|
|serviceSpec.annotations|object	|（可选）Dragonfly 服务注解|
|authentication.passwordFromSecret|[SecretKeySelector](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#secretkeyselector-v1-core)|（可选）来自 Secret 的 Dragonfly 密码作为对特定密钥的引用。例子 -<br>spec:<br>  authentication:<br>    passwordFromSecret:<br>      name: dragonfly-auth-secret<br>      key: password<br>|
|authentication.clientCaCertSecret|[SecretReference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#secretreference-v1-core)|（可选）如果指定，Dragonfly 实例将检查客户端证书是否由该 CA 之一签名。必须为此启用服务器 TLS。可以使用不同的密钥名称指定多个 CA。例子 -<br>spec:<br>  authentication:<br>    clientCaCertSecret:<br>      name: dragonfly-client-ca<br>|
|tlsSecretRef|[SecretReference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#secretreference-v1-core)|（可选）用于与 Dragonfly 进行 TLS 连接的 Dragonfly TLS 密钥。Dragonfly 实例必须有权访问此密钥并且位于同一命名空间中。例子 -<br>spec:<br>  tlsSecretRef:<br>    name: dragonfly-secret|
|snapshot.cron|string|（可选）Dragonfly 快照计划|
|snapshot.persistentVolumeClaimSpec|[持久卷声明规范](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#persistentvolumeclaimspec-v1-core)|（可选）DragonflyPVC规格|