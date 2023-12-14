# 快速安装入门

## 步骤 1
使用以下选项之一可以快速启动并运行 Dragonfly：

* [使用 Docker 安装 Dragonfly](/docs/getting-start/install-with-docker.md)
* [使用 Docker Compose 安装 Dragonfly](/docs/getting-start/Install-with-Docker-Compose.md)
* [安装 Dragonfly Kubernetes Operator](/docs/getting-start/Install-Dragonfly-Kubernetes-Operator.md)
* [使用 Helm Chart 在 Kubernetes 上安装](/docs/getting-start/Install-on-Kubernetes-with-Helm-Chart.md)
* [从二进制文件安装](/docs/getting-start/Install-from-Binary.md)
* [从源代码构建 DragonflyDB](/docs/getting-start/build-from-source.Zh_CN.md)

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


# 硬件兼容性
Dragonfly 经过专门优化，可在云环境中运行。它得到官方支持和认证，可在 x86\_64 和 arm64 架构上使用。

对于 x86\_64 架构，Dragonfly 需要最低限度的\_sandybridge\_架构才能正常运行。为了确保兼容性和性能，Dragonfly 在各种云平台上进行持续测试。其中包括 AWS 云中的 Graviton2 实例，以及 AWS 和 GCP 云中基于 x86\_64 的实例。

此外，Dragonfly 的回归测试（可以[在此处找到](https://github.com/dragonflydb/dragonfly/actions/workflows/regression-tests.yml)）通过 GitHub Actions 在 Azure 云上持续运行。这有助于保证软件的可靠性和稳定性。

# 操作系统兼容性
Dragonfly 与 Linux 版本 4.14 或更高版本兼容。但是，为了获得最佳性能，建议在内核版本 5.10 或更高版本上运行 Dragonfly。Dragonfly 构建环境基于 Ubuntu 20.04。

