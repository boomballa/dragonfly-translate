# Install with Docker Compose
# 使用 Docker Compose 安装
本指南将帮助您`docker-compose`在短短几分钟内运行 Dragonfly。

如果您的计算机上尚未`docker`安装，请在继续安装[Docker](https://docs.docker.com/get-docker/)和[Docker Compose](https://docs.docker.com/compose/install/)之前。`docker-compose`

## 步骤[1](https://www.dragonflydb.io/docs/getting-started/docker-compose#step-1 "直接链接到步骤 1")
```bash
# Download Official Dragonfly Docker Compose File
wget https://raw.githubusercontent.com/dragonflydb/dragonfly/main/contrib/docker/docker-compose.yml

# Launch the Dragonfly Instance
docker compose up -d

# Confirm image is up
docker ps | grep dragonfly
# ac94b5ba30a0   docker.dragonflydb.io/dragonflydb/dragonfly   "entrypoint.sh drago…"   45 seconds ago   Up 31 seconds         0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   docker_dragonfly_1

# Log follow the dragonfly container
docker logs -f docker_dragonfly_1
```
Dragonfly 将立即响应这两种`http`请求！`redis`

您可以使用`redis-cli`连接`localhost:6379`或打开浏览器并访问`http://localhost:6379`

## 步骤[2](https://www.dragonflydb.io/docs/getting-started/docker-compose#step-2 "直接链接到步骤 2")
与 Redis 客户端连接。

从新终端：

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
## 步骤[3](https://www.dragonflydb.io/docs/getting-started/docker-compose#step-3 "直接链接到步骤 3")
继续努力，利用 Dragonfly 的力量构建您的应用程序！

## 调整[Dragonfly](https://www.dragonflydb.io/docs/getting-started/docker-compose#tuning-dragonfly "直接链接到调整蜻蜓")
如果您尝试调整 Dragonfly 的性能，请考虑`NAT`与容器化相关的性能成本。

### 性能[调整](https://www.dragonflydb.io/docs/getting-started/docker-compose#performance-tuning "直接链接到性能调整")
在 中，使用网络（每个请求都依赖 docker遍历）和网络（请参阅 参考资料）`docker compose`之间存在显着差异。`overlay``NAT``host``docker-compose.yml`

有关更多信息，请参阅 Docker 撰写文件[network\_mode docs](https://docs.docker.com/compose/compose-file/compose-file-v3/#network_mode)。

