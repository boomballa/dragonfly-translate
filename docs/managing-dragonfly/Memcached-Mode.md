# Memcached Mode模式
# Memcached模式
使用该`memcached_port`标志选择用于公开 Memcached 兼容接口的端口。默认情况下，它是禁用的。例如，要在 Memcached 的默认端口上运行，请使用：

```bash
$> ./dragonfly --logtostderr --memcached_port 11211
```
通过运行上面的命令，Dragonfly 允许您同时使用 Redis 和 Memcached API：

```Plain Text
AcceptServer - listening on port 11211
AcceptServer - listening on port 6379
```
## Keyspace[共享](/docs/managing-dragonfly/Memcached-Mode.md#keyspace共享 "直接链接到 keyspace 共享")
值得注意的是，**Memcached API 与 Redis API 逻辑数据库共享相同的键空间**`0`。这是一种设计行为，它允许您的后端服务使用不同的 API 来操作同一组数据。通过这样做，可以实现软迁移策略 - 如果需要，您可以逐渐将各个服务从 Memcached API 迁移出来。

但是，Memcached 不支持复杂的数据类型（列表、集合、排序集、哈希等），并且不建议混合使用除字符串数据类型之外的两个 API 的操作。

```bash
# === Redis API === #
# Set a key in database '0', which is the default database.
dragonfly$> SET my_key_db_ZERO 1000
OK

# Switch to database '1'.
dragonfly$> SELECT 1
OK

# Database '1' has no keys.
dragonfly$> KEYS *
(empty array)

# Set a key in database '1'.
dragonfly$> SET my_key_db_ONE 1000
OK
```
```bash
# === Memcached API === #
# Try to read the key 'my_key_db_ZERO' via the Memcached API.
# This works since the key exists in database '0'.
dragonfly-memcached$> get my_key_db_ZERO
VALUE my_key_db_ZERO 0 4
1000
END

# Try to read the key 'my_key_db_ONE' via the Memcached API.
# The key is not found since it only exists in database '1'.
dragonfly-memcached$> get my_key_db_ONE 
END
```
