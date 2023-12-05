<p align="center">
  <a href="https://dragonflydb.io">
    <img src="https://raw.githubusercontent.com/dragonflydb/dragonfly/main/.github/images/logo-full.svg"
      width="284" border="0" alt="Dragonfly">
  </a>
</p>



# Dragonfly DB 与 Docker Compose

本指南将帮助您通过使用 `docker-compose`  在短短的几分钟内运行DragonflyDB。

| 指南假设您已将 `docker` 和 `docker-compose` 安装在您的计算机上。 如果没有，先[安装 Docker](https://docs.docker.com/get-docker/) 和 [安装 Docker Compose](https://docs.docker.com/compose/install/) ，然后再继续。

## 步骤 1

```bash
# Download Official Dragonfly DB Docker Compose File
wget https://raw.githubusercontent.com/dragonflydb/dragonfly/main/contrib/docker/docker-compose.yml

# Launch the Dragonfly DB Instance
docker-compose up -d

# Confirm image is up
docker ps | grep dragonfly
# ac94b5ba30a0   docker.dragonflydb.io/dragonflydb/dragonfly   "entrypoint.sh drago…"   45 seconds ago   Up 31 seconds         0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   docker_dragonfly_1

# Log follow the dragonfly container
docker logs -f docker_dragonfly_1
```

Dragonfly 数据库将立即响应 `http` 和 `redis` 两种请求。

您可以使用 `redis-cli` 来连接 `localhost:6379`或者打开浏览器访问 `http://localhost:6379`

## 步骤 2

与 Redis 客户端连接。

开启一个新的终端：

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

## 调试 Dragonfly 数据库
如果你尝试去调试 Dragonfly 数据库的性能， 请考虑 `NAT` 与容器化相关的性能成本。
> ## 性能调优
> ---
> 在 `docker-compose`中， 一个 `overlay` 网络(每个请求都在依赖 docker `NAT` 遍历) 和一个使用 `host` 网络(请参阅 [`docker-compose.yml`](https://github.com/dragonflydb/dragonfly/blob/main/contrib/docker/docker-compose.yml))，两者之间存在着意义上的不同。
> &nbsp;  
> 更多信息，请参阅 [official docker-compose network_mode Docs](https://docs.docker.com/compose/compose-file/compose-file-v3/#network_mode)  
> &nbsp;  

### 更多构建选项
- [Docker 快速入门](/docs/quick-start/)
- [ 使用 Helm Chart部署 Kubernetes](/contrib/charts/dragonfly/)
- [从源代码开始构建](/docs/build-from-source.md)