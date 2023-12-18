# 命令行参数（标志）


Dragonfly 可以通过一组配置标志进行调整和配置。这些标志可以是：

1. 作为命令行参数传递给 Dragonfly。例如，`dragonfly --port=6379`。
2. 从文件加载。例如`dragonfly --flagfile=path_to_flags/flags.txt`
3. 可以通过环境变量进行设置，方法是添加前缀`DFLY_`后跟标志名称。例如`export DFLY_port=6379`（注意区分大小写）
4. 在运行时通过`CONFIG SET`命令。并非所有标志都可以在运行时配置。

您可以尝试`dragonfly --helpfull`获取所有标志的列表，或`--help=substring`显示名称、描述或路径中包含指定子字符串的标志的帮助。

## 可用[flags](/docs/managing-dragonfly/Command-line-arguments.md#可用flags "直接链接到可用标志")
### `--port`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--port "直接链接到--port")
Redis 端口。0 禁用端口，-1 将绑定到随机可用端口。

```Plain Text
default: 6379
```
### `--cache_mode`[​](/docs/managing-dragonfly/Command-line-arguments.md#cache_mode​ "直接链接到--cache_mode")
如果为 true，则后端的行为类似于缓存，在接近最大内存限制时逐出条目

```Plain Text
default: false
```
### `--cluster_mode`[​](/docs/managing-dragonfly/Command-line-arguments.md#cluster_mode​ "直接链接到--cluster_mode")
设置集群模式。可用选项有：`yes`、`emulated`和 空`""`。

```Plain Text
default: ""
```
### `--maxmemory`[​](/docs/managing-dragonfly/Command-line-arguments.md#maxmemory​ "直接链接到--maxmemory")
数据库使用的最大内存限制。0 - 表示程序将自动确定其最大内存使用量。

```Plain Text
default: 0
```
### `--dbnum`[​](/docs/managing-dragonfly/Command-line-arguments.md#dbnum​ "直接链接到--dbnum")
数据库数量。

```Plain Text
default: 16
```
### `--bind`[​](/docs/managing-dragonfly/Command-line-arguments.md#bind​ "直接链接到--bind")
绑定地址。如果为空 - 绑定在所有接口上。出于安全考虑，不建议这样做。

```Plain Text
default: ""
```
### `--requirepass`[​](/docs/managing-dragonfly/Command-line-arguments.md#requirepass​ "直接链接到--requirepass")
AUTH 身份验证的密码。

```Plain Text
default: ""
```
### `--dbfilename`[​](/docs/managing-dragonfly/Command-line-arguments.md#dbfilename​ "直接链接到--dbfilename")
保存/加载数据库的文件名。

```Plain Text
default: "dump-{timestamp}"
```
### `--snapshot_cron`[​](/docs/managing-dragonfly/Command-line-arguments.md#snapshot_cron​ "直接链接到--snapshot_cron")
cron 表达式用于保存快照的时间，crontab 样式。

```Plain Text
default:
```
### `--use_set2`[​](/docs/managing-dragonfly/Command-line-arguments.md#use_set2​ "直接链接到--use_set2")
如果为 true，则使用 DenseSet 来优化集合数据结构。

```Plain Text
default: true
```
### `--use_zset_tree`[​](/docs/managing-dragonfly/Command-line-arguments.md#use_zset_tree​ "直接链接到--use_zset_tree")
如果 true 使用 b+tree 进行 zset 实现。

```Plain Text
default: true
```
### `--admin_bind`[](/docs/managing-dragonfly/Command-line-arguments.md#admin_bind "Direct link to --admin_bind")
如果设置，管理控制台 TCP 连接将绑定到给定地址。它支持 HTTP 和 RESP 协议。

```Plain Text
default: ""
```
### `--admin_port`[​](/docs/managing-dragonfly/Command-line-arguments.md#admin_port​ "直接链接到--admin_port")
如果设置，将允许管理员访问指定端口上的控制台。它支持 HTTP 和 RESP 协议。

```Plain Text
default: 0
```
### `--max_client_iobuf_len`[​](/docs/managing-dragonfly/Command-line-arguments.md#max_client_iobuf_len​ "直接链接到 --max_client_iobuf_len")
用于读取客户端请求的最大 io 缓冲区长度。

```Plain Text
default: 65536
```
### `--max_multi_bulk_len`[​](/docs/managing-dragonfly/Command-line-arguments.md#max_multi_bulk_len​ "直接链接到--max_multi_bulk_len")
解析RESP协议时允许接受的最大多块（数组）长度

```Plain Text
default: 65536
```
### `--migrate_connections`[​](/docs/managing-dragonfly/Command-line-arguments.md#migrate_connections​ "直接链接到 --migrate_connections")
启用后，Dragonfly 将尝试将连接迁移到它们运行的​​目标线程。目前，这仅支持 Lua 脚本调用，并且每个连接最多发生一次。

```Plain Text
default: true
```
### `--no_tls_on_admin_port`[​](/docs/managing-dragonfly/Command-line-arguments.md#no_tls_on_admin_port​ "直接链接到 --no_tls_on_admin_port")
允许管理端口上的非 tls 连接。

```Plain Text
default: false
```
### `--pipeline_queue_limit`[​](/docs/managing-dragonfly/Command-line-arguments.md#pipeline_queue_limit​ "直接链接到--pipeline_queue_limit")
每个 IO 线程用于存储流水线命令的内存量（以字节为单位）。

```Plain Text
default: 134217728
```
### `--pipeline_squash`[​](/docs/managing-dragonfly/Command-line-arguments.md#pipeline_squash​ "直接链接到--pipeline_squash")
启用压缩的排队流水线命令数，0 表示禁用。

```Plain Text
default: 10
```
### `--primary_port_http_enabled`[​](/docs/managing-dragonfly/Command-line-arguments.md#primary_port_http_enabled​ "直接链接到 --primary_port_http_enabled")
如果 true 允许在主 TCP 端口上访问 http 控制台。

```Plain Text
default: true
```
### `--request_cache_limit`[​](/docs/managing-dragonfly/Command-line-arguments.md#request_cache_limit​ "直接链接到--request_cache_limit")
每个 IO 线程用于请求缓存的内存量（以字节为单位）。

```Plain Text
default: 67108864
```
### `--tcp_nodelay`[​](/docs/managing-dragonfly/Command-line-arguments.md#tcp_nodelay​ "直接链接到--tcp_nodelay")
使用套接字选项 TCP\_NODELAY 配置蜻蜓连接。

```Plain Text
default: true
```
### `--conn_io_thread_start`[​](/docs/managing-dragonfly/Command-line-arguments.md#conn_io_thread_start​ "直接链接到--conn_io_thread_start")
用于处理服务器连接的启动线程 ID。

```Plain Text
default: 0
```
### `--conn_io_threads`[​](/docs/managing-dragonfly/Command-line-arguments.md#conn_io_threads​ "直接链接到--conn_io_threads")
用于处理服务器连接的线程数。

```Plain Text
default: 0
```
### `--conn_use_incoming_cpu`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--conn_use_incoming_cpu "直接链接到--conn_use_incoming_cpu")
如果为 true，则使用套接字的传入 cpu 来分配传入连接。

```Plain Text
default: false
```
### `--tcp_keepalive`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tcp_keepalive "直接链接到--tcp_keepalive")
触发保持活动之前的不活动时间（以秒为单位），直到不活动连接终止的持续时间是指定时间的两倍。

```Plain Text
default: 300
```
### `--tls`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tls "直接链接到--tls")
启用 tls。

```Plain Text
default: false
```
### `--tls_ca_cert_dir`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tls_ca_cert_dir "直接链接到--tls_ca_cert_dir")
认证机构签署的证书目录。

```Plain Text
default: ""
```
### `--tls_ca_cert_file`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tls_ca_cert_file "直接链接到--tls_ca_cert_file")
认证机构签署的证书用于验证 tls 连接。

```Plain Text
default: ""
```
### `--tls_cert_file`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tls_cert_file "直接链接到--tls_cert_file")
用于 tls 连接的证书文件（公钥）。

```Plain Text
default: ""
```
### `--tls_key_file`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tls_key_file "直接链接到--tls_key_file")
tls 连接的私钥文件。

```Plain Text
default: ""
```
### `--rename_command`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--rename_command "直接链接到--rename_command")
更改命令名称，格式为：<cmd1\_name>=<cmd1\_new\_name>,<cmd2\_name>=<cmd2\_new\_name>）

```Plain Text
default:
```
### `--restricted_commands`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--restricted_commands "直接链接到 --restricted_commands")
命令仅限于管理端口上的连接。

```Plain Text
default:
```
### [\--](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--lock_on_hashtags "直接链接到--lock_on_hashtags")`lock_on_hashtags`
如果为 true，则锁定在 {hashtag} 级别而不是密钥级别完成。仅将此与`--cluster_mode=emulated|yes`.

```Plain Text
default: false
```
### `--enable_heartbeat_eviction`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--enable_heartbeat_eviction "直接链接到--enable_heartbeat_eviction")
当内存面临压力时，在心跳期间启用驱逐。

```Plain Text
default: true
```
### `--max_eviction_per_heartbeat`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--max_eviction_per_heartbeat "直接链接到 --max_eviction_per_heartbeat")
在内存压力下触发基于心跳的驱逐时，每次驱逐中将删除的键值对的最大数量。

```Plain Text
default: 100
```
### `--max_segment_to_consider`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--max_segment_to_consider "直接链接到 --max_segment_to_consider")
当在内存压力下触发基于心跳的驱逐时，每次驱逐中要扫描的破折号表段的最大数量。

```Plain Text
default: 4
```
### `--force_epoll`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--force_epoll "直接链接到--force_epoll")
如果为 true - 使用下面的 linux epoll 引擎。适合 5.10 之前的内核。

```Plain Text
default: false
```
### `--pidfile`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--pidfile "直接链接到--pidfile")
如果不为空 - 服务器将其 pid 写入文件）。

```Plain Text
default: ""
```
### `--unixsocket`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--unixsocket "直接链接到--unixsocket")
如果不为空 - 指定将用于侦听传入连接的 Unix 套接字的路径。

```Plain Text
default: ""
```
### `--unixsocketperm`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--unixsocketperm "直接链接到--unixsocketperm")
设置 unixsocket 的权限，以八进制值表示。

```Plain Text
default: ""
```
### `--version_check`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--version_check "直接链接到--version_check")
如果属实，将每天监控 Dragonfly 服务器上的新版本一次。

```Plain Text
default: true
```
### `--hz`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--hz "直接链接到--hz")
服务器执行其他后台任务的基本频率。警告：不建议减产。

```Plain Text
default: 100
```
### `--mem_defrag_page_utilization_threshold`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--mem_defrag_page_utilization_threshold "直接链接到 --mem_defrag_page_utilization_threshold")
import re

# Update links after each heading
code = re.sub(r'\[(.*?)\]\((.*?)\)', r'[\1](/docs/managing-dragonfly/Command-line-arguments.md#\2)', code)

print(code)
