# Feast
## [简介](https://www.dragonflydb.io/docs/integrations/feast#introduction "Direct link to Introduction")
Feast 是一个独立的开源特征存储，可以一致地存储和提供特征以用于离线训练和在线推理。Feast 使用在线商店以低延迟提供功能。特征值通过物化从数据源加载到在线商店，可以使用`feast materialize`命令触发。

Redis 是 Feast 支持的在线商店之一。由于 Dragonfly 与 Redis 高度兼容，因此它可以用作 Feast 的替代在线商店，在您的应用程序中实现零代码更改和最少的配置更改。

## 搭配[Dragonfly](https://www.dragonflydb.io/docs/integrations/feast#running-feast-with-dragonfly%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E3%80%8A%E8%9C%BB%E8%9C%93%E5%A5%94%E8%B7%91%E7%9B%9B%E5%AE%B4%E3%80%8B%22) 运行Feast
请按照以下步骤配置 Dragonfly 并在您的应用程序中使用 Feast。

### 1.[先决条件](https://www.dragonflydb.io/docs/integrations/feast#1-prerequisites "直接链接至 1. 先决条件")
确保已安装 Python 和 pip。然后您可以安装 Feast SDK 和 CLI：

```bash
$> pip install feast
```
为了使用 Dragonfly 作为在线商店，您需要为 Feast 安装 Redis 额外组件：

```bash
$> pip install 'feast[redis]'
```
### 2\. 特征库[创建](https://www.dragonflydb.io/docs/integrations/feast#2-feature-repository-creation "直接链接到 2. 特征存储库创建")
引导一个新的功能存储库：

```bash
$> feast init feast_dragonfly
$> cd feast_dragonfly/feature_repo
```
更新`feature_repo/feature_store.yaml`以下内容：

```yaml
project: feast_dragonfly
registry: data/registry.db
provider: local
online_store:
  type: redis
  connection_string: "localhost:6379"
```
### 3.Dragonfly[初始化](https://www.dragonflydb.io/docs/integrations/feast#3-dragonfly-initialization%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%203.%20Dragonfly%20%E5%88%9D%E5%A7%8B%E5%8C%96%22)
有多种选项可用于[快速启动并运行 Dragonfly](https://www.dragonflydb.io/docs/getting-started)。假设您有本地 Dragonfly 二进制文件，则可以使用以下标志运行 Dragonfly。确保使用`feature_store.yaml`上面文件中指定的相同地址和端口。

```bash
$> ./dragonfly --bind localhost --port 6379
```
### 4\. Feast[使用](https://www.dragonflydb.io/docs/integrations/feast#4-feast-usage%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E8%87%B3%204.%20%E7%9B%9B%E5%AE%B4%E4%BD%BF%E7%94%A8%22)
至此，您已成功配置 Feast 以使用 Dragonfly 作为在线商店。要了解有关如何实际使用 Feast 的更多信息，请参阅[Feast 快速入门指南](https://docs.feast.dev/getting-started/quickstart)。

## 有用的[资源](https://www.dragonflydb.io/docs/integrations/feast#useful-resources "直接链接到有用资源")
* 盛宴[主页](https://feast.dev/)、[GitHub](https://github.com/feast-dev/feast)和[文档](https://docs.feast.dev/)。
* [在此处](https://docs.feast.dev/reference/online-stores/dragonfly)阅读有关如何配置 Feast 以使用 Dragonfly 的更多信息。
* [请阅读我们关于使用 Dragonfly 运行 Feast Feature Store 的](https://www.dragonflydb.io/blog/running-the-feast-feature-store-with-dragonfly)博客文章。



