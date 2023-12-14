# 使用二进制文件构建 DragonflyDB
以下是从二进制文件运行 Dragonfly 的分步说明。

## [先决条件](#先决条件 "Direct link to Prerequisites")
* 您必须运行基于 Linux 的操作系统（如果使用 Mac，请使用[Docker](/docs/getting-start/install-with-docker.md)运行，Windows WSL 也可以工作）。
* 网络接入
* 至少 4GB RAM 才能获得 Dragonfly 的优势
* 至少 1 个 CPU 核心
* Linux 内核 4.19 或更高版本

## 第 1 步：下载首选[版本](#第-1-步下载首选版本 "直接链接到步骤 1：下载首选版本")
从以下链接之一下载最新的 Dragonfly 版本。您还可以[在此处查看所有版本](https://github.com/dragonflydb/dragonfly/releases)。

* [下载最新版本 (Aarch64)](https://dragonflydb.gateway.scarf.sh/latest/dragonfly-aarch64.tar.gz)
* [下载最新版本 (X86-64)](https://dragonflydb.gateway.scarf.sh/latest/dragonfly-x86_64.tar.gz)
* [下载最新版本（ARM64 Debian）](https://dragonflydb.gateway.scarf.sh/latest/dragonfly_arm64.deb)
* [下载最新版本 (AMD64 Debian)](https://dragonflydb.gateway.scarf.sh/latest/dragonfly_amd64.deb)

## 第 2 步：解压并重命名[文件](#第-2-步解压并重命名文件 "直接链接到步骤 2：解压缩并重命名文件")
解压下载的文件并将其重命名为`dragonfly`：

```Plain Text
tar zxf {file_name}
mv {file_name} dragonfly
```
## 第 3 步：运行[Dragonfly](#第-3-步运行dragonfly "直接链接到步骤 3：运行 Dragonfly")
使用以下命令运行 Dragonfly：

```Plain Text
./dragonfly --logtostderr
```
## 带[标志](#带标志 "直接链接到带有标志的 Run Dragonfly")
通过运行该选项可以看到 Dragonfly 支持的标志列表`--help`，并且可以使用该选项查看完整的选项列表`--helpfull`。

[可以在此处](https://github.com/boomballa/dragonfly-translate/blob/main/README_Official.md#%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95)找到最有用的标志的列表。

例如，要运行蜻蜓：

* 标准错误处的日志消息
* 密码（设置为`youshallnotpass`）
* 缓存模式已启用
* 数据库数量设置为`1`
* 仅监听端口`6379`和`localhost`流量
* `30`以分钟为间隔保存数据
* 最大内存设置为 12GB
* 命令返回的键数`KEY`（设置为12288）
* 设置转储文件名（为`dump.db`）

你会运行：

```Plain Text
dragonfly --logtostderr --requirepass=youshallnotpass --cache_mode=true -dbnum 1 --bind localhost --port 6379  --save_schedule "*:30" --maxmemory=12gb --keys_output_limit=12288 --dbfilename dump.rdb
```
#### 标记[文件](#标记文件 "直接链接到标志文件")
要从配置文件添加标志，请使用`--flagfile <filename>`flag. 该文件必须每行列出一个标志。对于键值标志，请使用`--<flag_name>=<flag_value>`格式（注意等于字符，不能使用空格字符）。

