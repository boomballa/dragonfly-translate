<p align="center">
  <a href="https://dragonflydb.io">
    <img src="https://raw.githubusercontent.com/dragonflydb/dragonfly/main/.github/images/logo-full.svg"
      width="284" border="0" alt="Dragonfly">
  </a>
</p>


# 快速入门

从 “docker run” 开始是启动和运行 DragonflyDB 的最简单方法。

如果你的电脑上面没有 docker， 先 [安装 Docker](https://docs.docker.com/get-docker/) ，然后再继续。

## 步骤 1

### 在 linux 上

```bash
docker run --network=host --ulimit memlock=-1 docker.dragonflydb.io/dragonflydb/dragonfly
```

### 在 macOS 上

_`network=host` 在 macOS上面运行不是很友好， 查看 [这个问题](https://github.com/docker/for-mac/issues/1031)_

```bash
docker run -p 6379:6379 --ulimit memlock=-1 docker.dragonflydb.io/dragonflydb/dragonfly
```

Dragonfly 数据库会立即响应 `http` 和 `redis` 这两种请求。

你可以使用 `redis-cli` 去连接 `localhost:6379` 或者打开浏览器访问 `http://localhost:6379`

**注意**: 在某些配置上， 运行 `docker run --privileged ...` 命令使用此参数可以修复一些初始化错误。

## 步骤 2

与redis客户端连接

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

继续加油，利用 DragonflyDB 的强大功能构建您的应用程序！

## 已知的问题

## 更多构建选项
- [Docker Compose部署](/contrib/docker/)
- [使用 Helm Chart进行 Kubernetes 部署](/contrib/charts/dragonfly/)
- [通过源代码构建](./build-from-source.Zh_CN.md)
