# 命令行参数（标志）


Dragonfly 可以通过一组配置标志进行调整和配置。这些标志可以是：

1. 作为命令行参数传递给 Dragonfly。例如，`dragonfly --port=6379`。
2. 从文件加载。例如`dragonfly --flagfile=path_to_flags/flags.txt`
3. 可以通过环境变量进行设置，方法是添加前缀`DFLY_`后跟标志名称。例如`export DFLY_port=6379`（注意区分大小写）
4. 在运行时通过`CONFIG SET`命令。并非所有标志都可以在运行时配置。

您可以尝试`dragonfly --helpfull`获取所有标志的列表，或`--help=substring`显示名称、描述或路径中包含指定子字符串的标志的帮助。

## 可用[Flags](https://www.dragonflydb.io/docs/managing-dragonfly/flags#available-flags "直接链接到可用标志")
### `--port`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--port "直接链接到--port")
Redis 端口。0 禁用端口，-1 将绑定到随机可用端口。

```Plain Text
default: 6379
```
### `--cache_mode`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--cache_mode "直接链接到--cache_mode")
如果为 true，则后端的行为类似于缓存，在接近最大内存限制时逐出条目

```Plain Text
default: false
```
### `--cluster_mode`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--cluster_mode "直接链接到--cluster_mode")
设置集群模式。可用选项有：`yes`、`emulated`和 空`""`。

```Plain Text
default: ""
```
### `--maxmemory`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--maxmemory "直接链接到--maxmemory")
数据库使用的最大内存限制。0 - 表示程序将自动确定其最大内存使用量。

```Plain Text
default: 0
```
### `--dbnum`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--dbnum "直接链接到--dbnum")
数据库数量。

```Plain Text
default: 16
```
### `--bind`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--bind "直接链接到--bind")
绑定地址。如果为空 - 绑定在所有接口上。出于安全考虑，不建议这样做。

```Plain Text
default: ""
```
### `--requirepass`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--requirepass "直接链接到--requirepass")
AUTH 身份验证的密码。

```Plain Text
default: ""
```
### `--dbfilename`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--dbfilename "直接链接到--dbfilename")
保存/加载数据库的文件名。

```Plain Text
default: "dump-{timestamp}"
```
### `--snapshot_cron`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--snapshot_cron "直接链接到--snapshot_cron")
cron 表达式用于保存快照的时间，crontab 样式。

```Plain Text
default:
```
### `--use_set2`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--use_set2 "直接链接到--use_set2")
如果为 true，则使用 DenseSet 来优化集合数据结构。

```Plain Text
default: true
```
### `--use_zset_tree`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--use_zset_tree "直接链接到--use_zset_tree")
如果 true 使用 b+tree 进行 zset 实现。

```Plain Text
default: true
```
### [\`](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--admin_bind "Direct link to --admin_bind")`--admin_bind`
如果设置，管理控制台 TCP 连接将绑定到给定地址。它支持 HTTP 和 RESP 协议。

```Plain Text
default: ""
```
### `--admin_port`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--admin_port "直接链接到--admin_port")
如果设置，将允许管理员访问指定端口上的控制台。它支持 HTTP 和 RESP 协议。

```Plain Text
default: 0
```
### `--max_client_iobuf_len`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--max_client_iobuf_len "直接链接到 --max_client_iobuf_len")
用于读取客户端请求的最大 io 缓冲区长度。

```Plain Text
default: 65536
```
### `--max_multi_bulk_len`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--max_multi_bulk_len "直接链接到--max_multi_bulk_len")
解析RESP协议时允许接受的最大多块（数组）长度

```Plain Text
default: 65536
```
### `--migrate_connections`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--migrate_connections "直接链接到 --migrate_connections")
启用后，Dragonfly 将尝试将连接迁移到它们运行的​​目标线程。目前，这仅支持 Lua 脚本调用，并且每个连接最多发生一次。

```Plain Text
default: true
```
### `--no_tls_on_admin_port`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--no_tls_on_admin_port "直接链接到 --no_tls_on_admin_port")
允许管理端口上的非 tls 连接。

```Plain Text
default: false
```
### `--pipeline_queue_limit`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--pipeline_queue_limit "直接链接到--pipeline_queue_limit")
每个 IO 线程用于存储流水线命令的内存量（以字节为单位）。

```Plain Text
default: 134217728
```
### `--pipeline_squash`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--pipeline_squash "直接链接到--pipeline_squash")
启用压缩的排队流水线命令数，0 表示禁用。

```Plain Text
default: 10
```
### `--primary_port_http_enabled`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--primary_port_http_enabled "直接链接到 --primary_port_http_enabled")
如果 true 允许在主 TCP 端口上访问 http 控制台。

```Plain Text
default: true
```
### `--request_cache_limit`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--request_cache_limit "直接链接到--request_cache_limit")
每个 IO 线程用于请求缓存的内存量（以字节为单位）。

```Plain Text
default: 67108864
```
### `--tcp_nodelay`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tcp_nodelay "直接链接到--tcp_nodelay")
使用套接字选项 TCP\_NODELAY 配置蜻蜓连接。

```Plain Text
default: true
```
### `--conn_io_thread_start`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--conn_io_thread_start "直接链接到--conn_io_thread_start")
用于处理服务器连接的启动线程 ID。

```Plain Text
default: 0
```
### `--conn_io_threads`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--conn_io_threads "直接链接到--conn_io_threads")
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
内存页低于利用率阈值。已使用大小和已提交大小之间的比率，低于此值，该页面中的内存将进行碎片整理。

```Plain Text
default: 0.8
```
### `--mem_defrag_threshold`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--mem_defrag_threshold "直接链接到 --mem_defrag_threshold")
运行碎片整理之前已用内存相对于最大内存上限的最小百分比。

```Plain Text
default: 0.7
```
### `--mem_defrag_waste_threshold`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--mem_defrag_waste_threshold "直接链接到 --mem_defrag_waste_threshold")
浪费/已提交内存的比率，高于该比率我们运行碎片整理。

```Plain Text
default: 0.2
```
### `--shard_round_robin_prefix`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--shard_round_robin_prefix "直接链接到--shard_round_robin_prefix")
当非空时，带有哈希标签（其哈希标签以此前缀开头）的键不会根据其值分布在分片上，而是通过循环方式分布。谨慎使用！这可以有效地支持多达数百个主题标签。

```Plain Text
default: ""
```
### `--spill_file_prefix`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--spill_file_prefix "直接链接到--spill_file_prefix")
如果设置，则启用分层存储。该字符串表示与分层存储关联的文件的路径和前缀。例如，`spill_file_prefix=/path/to/file-prefix`.

```Plain Text
default: ""
```
### `--keys_output_limit`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--keys_output_limit "直接链接到--keys_output_limit")
keys 命令输出的最大按键数。

```Plain Text
default: 8192
```
### `--backing_file_direct`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--backing_file_direct "直接链接到--backing_file_direct")
如果 true 用于`O_DIRECT`打开备份文件。

```Plain Text
default: false
```
### `--list_compress_depth`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--list_compress_depth "直接链接到--list_compress_depth")
压缩列表的深度。默认是不压缩。

```Plain Text
default: 0
```
### `--list_max_listpack_size`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--list_max_listpack_size "直接链接到--list_max_listpack_size")
最大列表包大小，默认为 8kb。

```Plain Text
default: -2
```
### `--admin_nopass`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--admin_nopass "直接链接到--admin_nopass")
如果设置，将启用对分配端口上控制台的开放管理访问，无需授权。

```Plain Text
default: false
```
### `--memcached_port`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--memcached_port "直接链接到 --memcached_port")
内存缓存端口：

```Plain Text
default: 0
```
### `--multi_eval_squash_buffer`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--multi_eval_squash_buffer "直接链接到--multi_eval_squash_buffer")
每个脚本压缩命令的最大缓冲区：

```Plain Text
default: 4096
```
### `--multi_exec_mode`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--multi_exec_mode "直接链接到--multi_exec_mode")
设置多执行原子性模式：1为全局，2为提前锁定，3为非原子。

```Plain Text
default: 2
```
### `--multi_exec_squash`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--multi_exec_squash "直接链接到--multi_exec_squash")
多重执行是否会压缩单个分片命令以优化性能。

```Plain Text
default: true
```
### `--num_shards`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--num_shards "直接链接到--num_shards")
数据库分片数量，0 - 自动选择。

```Plain Text
default: 0
```
### `--oom_deny_ratio`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--oom_deny_ratio "直接链接到--oom_deny_ratio")
当 maxmemory 和已用内存之间的比率高于此值时，带有标志 denyoom 的命令将返回 OOM。

```Plain Text
default: 1.1
```
### `--masterauth`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--masterauth "直接链接到--masterauth")
用于与 master 进行身份验证的密码。

```Plain Text
default: ""
```
### `--replicaof`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--replicaof "直接链接到--replicaof")
指定指向要复制的目标主机的主机和端口。格式应为`<IPv4>:<PORT>`或`host:<PORT>`或`[<IPv6>]:<PORT>`。

```Plain Text
default:
```
### `--tls_replication`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tls_replication "直接链接到--tls_replication")
在复制时启用 TLS。

```Plain Text
default: false
```
### `--compression_level`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--compression_level "直接链接到--compression_level")
zstd/lz4 压缩使用的压缩级别。

```Plain Text
default: 2
```
### `--compression_mode`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--compression_mode "直接链接到--compression_mode")
设置 0 表示无压缩，设置 1 表示单条目 lzf 压缩，设置 2 表示 df 快照上的多条目 zstd 压缩和 rdb 快照上的单条目，设置 3 表示 df 快照上的多条目 lz4 压缩和 rdb 快照上的单条目。

```Plain Text
default: 3
```
### `--enable_multi_shard_sync`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--enable_multi_shard_sync "直接链接到--enable_multi_shard_sync")
在同步副本上执行多分片命令。

```Plain Text
default: false
```
### `--master_connect_timeout_ms`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--master_connect_timeout_ms "直接链接到--master_connect_timeout_ms")
与复制主机建立连接超时。

```Plain Text
default: 20000
```
### `--master_reconnect_timeout_ms`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--master_reconnect_timeout_ms "直接链接到--master_reconnect_timeout_ms")
重新建立与复制主机的连接超时。

```Plain Text
default: 1000
```
### `--replica_partial_sync`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--replica_partial_sync "直接链接到--replica_partial_sync")
当副本连接中断时，使用部分同步重新连接。

```Plain Text
default: true
```
### `--replication_acks_interval`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--replication_acks_interval "直接链接到--replication_acks_interval")
ack 之间的时间间隔（以毫秒为单位）。

```Plain Text
default: 3000
```
### `--default_lua_flags`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--default_lua_flags "直接链接到--default_lua_flags")
配置运行 Lua 脚本的默认标志：

* 用于`allow-undeclared-keys`允许访问未声明的密钥，
* 用于`disable-atomicity`允许非原子地运行脚本。

指定由空格分隔的多个值，例如`allow-undeclared-keys disable-atomicity` 以非原子方式运行脚本并允许访问未声明的键。

```Plain Text
default: ""
```
### `--lua_auto_async`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--lua_auto_async "直接链接到--lua_auto_async")
如果启用，带有丢弃值的 call/pcall 将自动替换为 acall/apcall。

```Plain Text
default: false
```
### `--df_snapshot_format`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--df_snapshot_format "直接链接到--df_snapshot_format")
如果为 true，则以蜻蜓特定的快照格式保存

```Plain Text
default: true
```
### `--dir`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--dir "直接链接到--dir")
工作目录

```Plain Text
default: ""
```
### `--epoll_file_threads`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--epoll_file_threads "直接链接到--epoll_file_threads")
以 epoll 模式运行时文件工作线程的线程大小，默认为硬件并发线程。

```Plain Text
default: 0
```
### `--maxclients`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--maxclients "直接链接到--maxclients")
允许的最大并发客户端数。

```Plain Text
default: 64000
```
### `--s3_ec2_metadata`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--s3_ec2_metadata "直接链接到--s3_ec2_metadata")
是否从 EC2 元数据加载凭证和配置。

```Plain Text
default: false
```
### `--s3_endpoint`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--s3_endpoint "直接链接到--s3_endpoint")
s3 快照的端点，默认使用 aws 区域端点。

```Plain Text
default: ""
```
### `--s3_sign_payload`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--s3_sign_payload "直接链接到--s3_sign_payload")
上传快照时是否对 s3 请求负载进行签名。

```Plain Text
default: true
```
### `--s3_use_https`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--s3_use_https "直接链接到--s3_use_https")
是否对 s3 端点使用 https。

```Plain Text
default: true
```
### `--slowlog_log_slower_than`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--slowlog_log_slower_than "直接链接到--slowlog_log_slower_than")
将慢于此阈值的命令添加到慢日志中。该值以微秒表示，如果为负数则禁用慢日志。

```Plain Text
default: 10000
```
### `--slowlog_max_len`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--slowlog_max_len "直接链接到--slowlog_max_len")
慢日志最大长度。

```Plain Text
default: 20
```
### `--interpreter_per_thread`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--interpreter_per_thread "直接链接到 --interpreter_per_thread")
每个线程的 Lua 解释器

```Plain Text
default: 10
```
### `--enable_top_keys_tracking`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--enable_top_keys_tracking "直接链接到 --enable_top_keys_tracking")
启用/禁用跟踪热键调试功能。

```Plain Text
default: false
```
### `--tiered_storage_max_pending_writes`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--tiered_storage_max_pending_writes "直接链接到--tiered_storage_max_pending_writes")
每个线程的最大挂起写入数。

```Plain Text
default: 32
```
### `--aclfile`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--aclfile "直接链接到--aclfile")
acl 文件的路径和名称。

```Plain Text
default: ""
```
### `--acllog_max_len`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--acllog_max_len "直接链接到--acllog_max_len")
指定日志条目的数量。每个线程的日志都保存在本地，因此条目总数为`acllog_max_len * threads`

```Plain Text
default: 32
```
### `--cluster_announce_ip`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--cluster_announce_ip "直接链接到--cluster_announce_ip")
集群命令向客户端公布的 IP。

```Plain Text
default: ""
```
### `--shard_repl_backlog_len`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--shard_repl_backlog_len "直接链接到--shard_repl_backlog_len")
每个分片的循环复制日志的长度。

```Plain Text
default: 1024
```
### `--proactor_affinity_mode`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--proactor_affinity_mode "直接链接到--proactor_affinity_mode")
可以打开、关闭或自动。

```Plain Text
default: "on"
```
### `--proactor_threads`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--proactor_threads "直接链接到--proactor_threads")
池中的 io 线程数。

```Plain Text
default: 0
```
### `--proactor_register_fd`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--proactor_register_fd "直接链接到--proactor_register_fd")
如果 true 尝试注册文件描述符。

```Plain Text
default: false
```
### `--flagfile`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--flagfile "直接链接到--flagfile")
要从中加载标志的以逗号分隔的文件列表。

```Plain Text
default:
```
### `--alsologtostderr`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--alsologtostderr "直接链接到--alsologtostderr")
除了日志文件之外，日志消息也会发送到 stderr。

```Plain Text
default: false
```
### `--log_dir`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--log_dir "直接链接到--log_dir")
如果指定，日志文件将写入此目录而不是默认日志记录目录。

```Plain Text
default: ""
```
### `--logtostderr`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--logtostderr "直接链接到--logtostderr")
日志消息转到 stderr 而不是日志文件。

```Plain Text
default: false
```
### `--logtostdout`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--logtostdout "直接链接到--logtostdout")
日志消息转到标准输出而不是日志文件。

```Plain Text
default: false
```
### `--max_log_size`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--max_log_size "直接链接到--max_log_size")
大约。最大日志文件大小（以 MB 为单位）。值 0 将被静默覆盖为 1。

```Plain Text
default: 1800
```
### `--minloglevel`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--minloglevel "直接链接到--minloglevel")
在低于此级别记录的消息实际上不会记录在任何地方。

```Plain Text
default: 0
```
### `--stderrthreshold`[​](https://www.dragonflydb.io/docs/managing-dragonfly/flags#--stderrthreshold "直接链接到--stderrthreshold")
除了日志文件之外，处于或高于此级别的日志消息也会复制到 stderr。该标志已过时 --alsologtostderr

```Plain Text
default: 2
```