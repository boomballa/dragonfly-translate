# 具有服务器 TLS 的 Dragonfly
本指南将引导您完成使用 TLS 设置 Dragonfly。启用 TLS 后，您可以在客户端和 Dragonfly 实例之间进行加密通信。

## [先决条件](https://www.dragonflydb.io/docs/managing-dragonfly/operator/server-tls#prerequisites "Direct link to Prerequisites")
* 配置了 kubectl 来访问它的 Kubernetes 集群。
* [Dragonfly Operator](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation)安装在您的集群中。

## 使用证书[管理器](https://www.dragonflydb.io/docs/managing-dragonfly/operator/server-tls#generate-a-cert-with-cert-manager "使用证书管理器生成证书的直接链接")
Cert Manager 是一个 Kubernetes 插件，用于自动管理和颁发来自各种颁发源的 TLS 证书。在本指南中，我们将使用自签名颁发者来生成证书。

您也可以跳过此步骤并使用手动生成的自签名证书。`tls.crt`操作员期望相关密钥与,密钥一起出现`tls.key`。

### 安装证书[管理器](https://www.dragonflydb.io/docs/managing-dragonfly/operator/server-tls#install-cert-manager "直接链接到安装证书管理器")
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```
创建自签名发行者：

```bash
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: ca-issuer
spec:
  selfSigned: {}
EOF
```
向自签名颁发者请求证书：

```bash
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: dragonfly-sample
spec:
  # Secret names are always required.
  secretName: dragonfly-sample
  duration: 2160h # 90d
  renewBefore: 360h # 15d
  subject:
    organizations:
      - dragonfly-sample
  # The use of the common name field has been deprecated since 2000 and is
  # discouraged from being used.
  commonName: example.com
  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048
  # At least one of a DNS Name, URI, or IP address is required.
  dnsNames:
    - dragonfly-sample.com
    - www.dragonfly-sample.com
  # Issuer references are always required.
  issuerRef:
    name: ca-issuer
    kind: Issuer
    group: cert-manager.io
EOF
```
## 具有[TLS](https://www.dragonflydb.io/docs/managing-dragonfly/operator/server-tls#dragonfly-instance-with-tls "使用 TLS 直接链接到 Dragonfly 实例")
由于至少需要一种身份验证机制，我们将使用密码身份验证机制：

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: dragonfly-password
type: Opaque
stringData:
  password: dragonfly
EOF
```
启用 TLS 时，根据需要创建一个具有至少一种身份验证机制的 Dragonfly 实例。

```bash
kubectl apply -f - <<EOF
apiVersion: dragonflydb.io/v1alpha1
kind: Dragonfly
metadata:
  name: dragonfly-sample
spec:
    authentication:
      passwordFromSecret:
        name: dragonfly-password
        key: password
    replicas: 2
    tlsSecretRef:
      name: dragonfly-sample
EOF
```
检查 Dragonfly 实例的状态：

```bash
kubectl describe dragonflies.dragonflydb.io dragonfly-sample
```
## 连接到[蜻蜓](https://www.dragonflydb.io/docs/managing-dragonfly/operator/server-tls#connecting-to-dragonfly "直接链接到“连接到 Dragonfly”")
CA 证书存储在命名空间`dragonfly-sample`中命名的机密中`default`。我们用它来连接到 Dragonfly。这是必需的，因为我们使用默认情况下不受信任的自签名证书。

使用 ca.crt 创建 redis-cli 容器

```bash
kubectl run -it --rm redis-cli --image=redis:7.0.10 --restart=Never --overrides='
{
    "spec": {
        "containers": [
            {
                "name": "redis-cli",
                "image": "redis:7.0.10",
                "tty": true,
                "stdin": true,
                "command": [
                    "redis-cli",
                    "-h",
                    "dragonfly-sample.default",
                    "-a",
                    "dragonfly",
                    "--tls",
                    "--cacert",
                    "/etc/ssl/ca.crt"
                ],
                "volumeMounts": [
                    {
                        "name": "ca-certs",
                        "mountPath": "/etc/ssl",
                        "readOnly": true
                    }
                ]
            }
        ],
        "volumes": [
            {
                "name": "ca-certs",
                "secret": {
                    "secretName": "dragonfly-sample",
                    "items": [
                        {
                            "key": "ca.crt",
                            "path": "ca.crt"
                        }
                    ]
                }
            }
        ]
    }
}'
```
您应该会看到 redis-cli 提示符，并且可以运行 redis 命令。

