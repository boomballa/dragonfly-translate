# HTTP
Dragonfly 支持 HTTP 的某些操作。默认情况下，`HTTP`可以在与主端口相同的端口上进行访问`TCP`。但是，可以通过命令关闭此功能 `--primary_port_http_enabled=false`。此外，如果`--admin_port`设置了标志，它也支持`HTTP`管理端口。

## [认证](https://www.dragonflydb.io/docs/managing-dragonfly/using-http#authentication "Direct link to Authentication")
默认情况下，HTTP 不需要身份验证。但是，如果`requirepass`设置了，则 HTTP 请求应使用用户名进行授权`default`，密码将为 的值`requirepass`。

请注意，`default`不应与 ACL 的`default`用户混淆。`requirepass`即使设置了ACL 用户密码，这两者也不总是共享完全相同的密码，`default`因为反之亦然，即 `ACL SETUSER default >newpass`不会更改 中指定的当前密码`requirepass`。

此外，如果`admin_nopass`设置了，它会绕过`requirepass`管理端口上的选项，允许所有请求发送到该端口而无需任何身份验证。

## 指标[页面](https://www.dragonflydb.io/docs/managing-dragonfly/using-http#metrics-page "直接链接到指标页面")
请记住，该`/metrics`页面始终绕过身份验证。这意味着用户可以自由查看该页面，无需任何身份验证。