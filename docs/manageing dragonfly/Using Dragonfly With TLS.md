# 将 Dragonfly 与 TLS 结合使用


如果您在连接到互联网的端点上部署 Dragonfly，您可能需要保护您的数据免遭泄露或被拦截。为了支持这一点，蜻蜓可以通过 TLS 为您提供数据，TLS 是一种提供加密和身份验证的 ISO 标准化协议。

在本教程中，我们将展示配置 Dragonfly 以使用 TLS 是多么容易！为了获得有效的 TLS 证书，我们假设您在 Ubuntu 服务器上打开了一个 shell，并且有一个指向该服务器的 DNS 地址。然后，该 DNS 地址将用于为我们的数据存储提供服务器。

## 获得[证书](https://www.dragonflydb.io/docs/managing-dragonfly/using-tls#getting-a-certificate "获取证书的直接链接")
`certbot`我们将使用由 Let's Encrypt 制作的名为 Let's Encrypt 的 Python 程序来提供 TLS 证书。让我们使用它吧！

1. 安装 python 和 nginx： `sudo apt install -y nginx-light python3 python3-venv libaugeas0`
2. 创建虚拟环境并安装certbot：

```bash
$ python3 -m venv certbot

$ certbot/bin/pip install certbot certbot-nginx

```
3. 现在`certbot`以 root 身份运行： `sudo ./certbot/bin/certbot` 您必须输入我们要使用的 URL，并指向服务器（在我们的例子中）`dfly.scalable-meteorite-collections.com`。

如果一切顺利，私钥和证书现在位于`/etc/letsencrypt/live/`.

## 运行的[Dragonfly](https://www.dragonflydb.io/docs/managing-dragonfly/using-tls#running-dragonfly%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E3%80%8A%E5%A5%94%E8%B7%91%E7%9A%84%E8%9C%BB%E8%9C%93%E3%80%8B%22)
选择用于验证数据存储用户身份的密码：

```Plain Text
export DFLY_PASSWORD=<???>
```
我们现在可以在启用 TLS 的服务器上启动 Dragonfly：

```bash
$ sudo ./dragonfly \
  --tls \
  --tls_key_file=/etc/letsencrypt/live/dfly.scalable-meteorite-collections.com/privkey.pem \
  --tls_cert_file=/etc/letsencrypt/live/dfly.scalable-meteorite-collections.com/fullchain.pem \
  --requirepass=${DFLY_PASSWORD}
```
并连接以查看数据存储是否正常工作：

```bash
$ redis-cli --tls -h dfly.scalable-meteorite-collections.com -a ${DFLY_PASSWORD}
dfly.scalable-meteorite-collections.com:6379> set anchdrite 33.7
OK
dfly.scalable-meteorite-collections.com:6379> set pallasite 19.1
OK
dfly.scalable-meteorite-collections.com:6379> get anchdrite
"33.7"
dfly.scalable-meteorite-collections.com:6379> 
```
## 使用[TLS](https://www.dragonflydb.io/docs/managing-dragonfly/using-tls#client-authentication-with-tls "使用 TLS 直接链接到客户端身份验证")
除了使用密码之外，还可以使用**客户端证书对客户端进行身份验证。**由于在这种情况下您将管理证书，因此您不需要使用 Let's Encrypt。假设您拥有所需的证书，您可以将 Dragonfly 配置为需要客户端证书，如下所示：

```bash
$ sudo ./dragonfly \
  --tls \
  --tls_key_file=/etc/mycerts/server_key.pem \
  --tls_cert_file=/etc/mycerts/server_cert.pem \
  --tls_ca_cert_file=/etc/mycerts/clients_root_ca.pem
```
并像这样连接客户端证书：

```bash
redis-cli -h dfly.scalable-meteorite-collections.com \
  --key /etc/mycerts/client_priv.pem
  --cert /etc/mycerts/client_cert.pem
```
在这种情况下，`client_cert.pem`需要由服务器信任的根进行签名 `clients_root_ca.pem`。