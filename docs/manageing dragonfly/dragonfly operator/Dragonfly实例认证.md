# Dragonfly实例认证
本指南提供了使用身份验证设置 Dragonfly 的分步说明。目前，Dragonfly 支持两种类型的身份验证：

* 通过秘密进行[基于密码的身份验证](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#password-based-authentication)
* 通过密钥进行[基于 TLS 的身份验证](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#tls-based-authentication)

## [先决条件](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#prerequisites "Direct link to Prerequisites")
* [安装了 Dragonfly](https://www.dragonflydb.io/docs/managing-dragonfly/operator/installation)的 Kubernetes 集群

## 基于密码的[身份验证](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#password-based-authentication "直接链接到基于密码的身份验证")
基于密码的身份验证是保护 Dragonfly 实例的最简单方法。在此方法中，您可以通过密钥为 Dragonfly 实例设置密码。然后使用该密码对客户端进行身份验证。

### 创建一个[密码](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#create-a-secret "直接链接到创建秘密")
```bash
kubectl create secret generic dragonfly-auth --from-literal=password=dragonfly
```
### 通过[验证](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#deploy-dragonfly-with-authentication%20%22%E9%80%9A%E8%BF%87%E8%BA%AB%E4%BB%BD%E9%AA%8C%E8%AF%81%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%20Deploy%20Dragonfly%22)部署Dragonfly
```bash
kubectl apply -f - <<EOF
apiVersion: dragonflydb.io/v1alpha1
kind: Dragonfly
metadata:
  name: dragonfly-auth
spec:
    authentication:
      passwordFromSecret:
        name: dragonfly-auth
        key: password
    replicas: 2
EOF
```
### 检查 Dragonfly[实例](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#check-the-status-of-the-dragonfly-instance%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E6%A3%80%E6%9F%A5%20Dragonfly%20%E5%AE%9E%E4%BE%8B%E7%9A%84%E7%8A%B6%E6%80%81%22)状态
```bash
kubectl describe dragonflies.dragonflydb.io dragonfly-auth
```
### 连接到[Dragonfly](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#connecting-to-dragonfly "直接链接到“连接到 Dragonfly”")
```bash
kubectl run -it --rm --restart=Never redis-cli --image=redis:7.0.10 -- redis-cli -h dragonfly-auth.default
if you don't see a command prompt, try pressing enter.
dragonfly-auth.default:6379> GET 1
(error) NOAUTH Authentication required. 
dragonfly-auth.default:6379> AUTH dragonfly
OK
dragonfly-auth.default:6379> GET 1
(nil)
dragonfly-auth.default:6379> SET 1 2
OK
dragonfly-auth.default:6379> GET 1
"2"
dragonfly-auth.default:6379> exit
```
## 基于 TLS 的[身份验证](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#tls-based-authentication "直接链接到基于 TLS 的身份验证")
基于 TLS 的身份验证是保护 Dragonfly 实例的更安全的方法。首先，您需要在 Dragonfly 实例上配置 TLS。然后，您可以指定 Dragonfly 实例信任的 CA 证书列表。客户端必须提供由受信任的 CA 之一签名的证书才能连接到 Dragonfly 实例。

### 通过 cert- [manager](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#create-a-tls-secret-for-dragonfly-through-cert-manager "通过 cert-manager 为 Dragonfly 创建 TLS 密钥的直接链接")
#### 安装证书[管理器](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#install-cert-manager "直接链接到安装证书管理器")
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```
#### 创建自签名[证书](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#create-a-self-signed-certificate "创建自签名证书的直接链接")
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
#### 请求 TLS[证书](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#request-a-tls-certificate "请求 TLS 证书的直接链接")
```bash
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: dragonfly-sample
spec:
  secretName: dragonfly-sample
  duration: 2160h # 90d
  renewBefore: 360h # 15d
  subject:
    organizations:
      - dragonfly-sample
  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048
  dnsNames:
    - dragonfly-sample.com
    - www.dragonfly-sample.com
  issuerRef:
    name: ca-issuer
    kind: Issuer
    group: cert-manager.io
EOF
```
### 生成由客户端[CA](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#generate-a-client-certificate-signed-by-a-client-ca "生成由客户端 CA 签名的客户端证书的直接链接")
#### 创建客户端[CA](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#create-a-client-ca "直接链接到创建客户端 CA")
```bash
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: client-ca-issuer
spec:
  selfSigned: {}
EOF

```
#### 请求客户[证书](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#request-a-client-certificate%20%22%E8%AF%B7%E6%B1%82%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AF%81%E4%B9%A6%E7%9A%84%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%22)
```bash
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: dragonfly-client-ca
spec:
    secretName: dragonfly-client-ca
    duration: 2160h # 90d
    renewBefore: 360h # 15d
    subject:
        organizations:
        - dragonfly-client-ca
    privateKey:
        algorithm: RSA
        encoding: PKCS1
        size: 2048
    dnsNames:
        - dragonfly-client-ca.com
        - www.dragonfly-client-ca.com
    usages:
        - client auth
    issuerRef:
        name: client-ca-issuer
        kind: Issuer
        group: cert-manager.io
EOF
```
### 使用[TLS](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#create-a-dragonfly-instance-with-tls "使用 TLS 创建 Dragonfly 实例的直接链接")
```bash
kubectl apply -f - <<EOF
apiVersion: dragonflydb.io/v1alpha1
kind: Dragonfly
metadata:
  name: dragonfly-sample
spec:
    authentication:
      clientCaCertSecret:
        name: dragonfly-client-ca
        key: ca.crt
    replicas: 2
    tlsSecretRef:
      name: dragonfly-sample
EOF
```
### 验证 Dragonfly 实例是否已[准备好](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#verify-the-dragonfly-instance-is-ready "验证 Dragonfly 实例是否准备就绪的直接链接")
```bash
kubectl describe dragonflies.dragonflydb.io dragonfly-sample
```
### 使用[TLS](https://www.dragonflydb.io/docs/managing-dragonfly/operator/authentication#connecting-to-dragonfly-with-tls "直接链接到使用 TLS 连接到 Dragonfly")
仅当您拥有客户端 CA 签名的客户端证书时，您才应该能够连接到 Dragonfly 实例。

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
                    "--tls",
                    "--cacert",
                    "/etc/ssl/ca.crt",
                    "--cert",
                    "/etc/tls/tls.crt",
                    "--key",
                    "/etc/tls/tls.key"
                ],
                "volumeMounts": [
                    {
                        "name": "ca-certs",
                        "mountPath": "/etc/ssl",
                        "readOnly": true
                    },
                    {
                        "name": "client-certs",
                        "mountPath": "/etc/tls",
                        "readOnly": true
                    }
                ]
            }
        ],
        "volumes": [
            {
                "name": "ca-certs",
                "secret": {
                    "secretName": "dragonfly-sample"
                }
            },
            {
                "name": "client-certs",
                "secret": {
                    "secretName": "dragonfly-client-ca"
                }
            }
        ]
    }
}'
```
