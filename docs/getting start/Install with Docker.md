# Install with Docker
# 使用 Docker 安装
从`docker run`开始是启动和运行 Dragonfly 的最简单方法。

如果您的计算机上没有 docker，请先[安装 Docker](https://docs.docker.com/get-docker/) ，然后再继续。

* 至少 4GB RAM 才能获得 Dragonfly 的优势
* 至少 1 个 CPU 核心
* Linux 内核 4.19 或更高版本



## 步骤1
### On linux
```bash
docker run --network=host --ulimit memlock=-1 docker.dragonflydb.io/dragonflydb/dragonfly
```
### On macOS
*network=host**在 macOS 上运行不佳，请参阅**此问题*

```bash
docker run -p 6379:6379 --ulimit memlock=-1 docker.dragonflydb.io/dragonflydb/dragonfly
```
Dragonfly 将会立即响应 `http` 和 `redis` 请求!

你可以使用 `redis-cli` 连接 `localhost:6379` 或者打开浏览器访问 `http://localhost:6379`

注意：在某些配置上，使用该 `docker run --privileged ...` 标志运行可以修复一些初始化错误.

## 步骤 2
使用redis客户端连接

```bash
redis-cli
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> keys *
1) "hello"
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379>
```
## 步骤 3
继续努力，利用 Dragonfly 的力量构建您的应用程序！

















