# 快照与恢复
## [简介](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#简介 "直接链接到简介")
快照和恢复是一种广泛采用的用于跨不同数据库实例迁移数据的技术。为了促进从 Redis 设置到 Dragonfly 环境的过渡，该过程涉及配置 Redis 以在磁盘上生成快照，称为 RDB（Redis 数据库）文件。或者，用户可以手动触发`SAVE`或`BGSAVE`命令来创建这些快照。

要将 Redis 数据迁移到 Dragonfly 实例，第一步是获取由 RDB 文件表示的时间点快照。作为 Dragonfly 初始化过程的一部分，系统会定位指定`dir`路径，这是一个可配置标志，指​​向 Dragonfly 管理其磁盘数据的目录。这是 RDB 文件应放置的位置，以便在启动期间自动加载。

Dragonfly 的无缝迁移通过其对 RDB 文件的自动识别而得到进一步增强。在识别指定目录路径中的快照后，Dragonfly 可以有效加载其中封装的数据，从而简化迁移过程。

为了更深入地了解快照文件的复杂性以及 Dragonfly 快照机制的复杂性，您可以深入研究[Dragonfly 时间点快照设计](/docs/managing-dragonfly/Dragonfly-Point-in-Time-Snapshotting-Design.md)文档页面中提供的综合资源。此外，要了解有关在 Dragonfly 中管理备份的更多信息，[“保存备份”](/docs/managing-dragonfly/Saving-Backups.md)部分提供了有关配置关键标志（如`dir`和 `dbfilename`）的宝贵见解。

## 迁移[步骤](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#迁移步骤 "直接链接到迁移步骤")
快照和恢复技术通过使用快照简化了数据从正在运行的 Redis 实例到 Dragonfly 环境的迁移。下面概述的步骤将指导您完成过渡过程：

### 1\. 创建Redis[快照](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#1-创建redis快照 "直接链接到 1. 创建 Redis 快照")
首先使用`SAVE`或`BGSAVE`命令启动 Redis 数据的快照：

```bash
redis> SAVE
OK
```
### 2.找到RDB[文件](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#2找到rdb文件 "直接链接到 2. 找到 RDB 文件")
发出快照命令后，您将发现生成的名为`dump.rdb`. 如果您使用 Docker，请注意 Redis 映像配置可能会将 RDB 文件定向到`/data`容器内的目录。默认情况下，Redis将此文件保存在`/var/lib/redis/`Linux系统的目录中：

```bash
$> ls /var/lib/redis/
dump.rdb
```
### 3.配置[Dragonfly](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#3配置dragonfly)
通过为快照管理配置适当的目录和文件名设置，为 Dragonfly 准备迁移过程。出于说明目的，我们假设 Dragonfly 二进制文件位于当前目录中`./`，并且 RDB 文件`dump.rdb`被复制到`./data`当前目录的子目录中：

```bash
$> tree
.
├── data
│   └── dump.rdb # the Redis snapshot file
└── dragonfly    # the Dragonfly binary
```
### 4.启动[Dragonfly](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#4启动dragonfly)
要使用 Redis 快照成功启动 Dragonfly 实例，请执行以下命令。确保 Dragonfly 二进制文件存在于目录中：

```bash
$> ./dragonfly --logtostderr --dir ./data --dbfilename dump
```
在上面的命令中，`dir`标志指定加载和保存快照文件的位置，而`dbfilename`标志指定数据库使用的文件名。

### 5\. 重新配置并重新启动[应用程序](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#5-重新配置并重新启动应用程序 "直接链接到 5. 重新配置和重新启动应用程序")
作为迁移过程的一部分，更新服务器应用程序用于连接到 Dragonfly 的连接字符串或凭据。此步骤涉及修改应用程序的配置以指向 Dragonfly 实例而不是之前的 Redis 实例。

## 停机[注意事项](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md#停机注意事项 "直接链接到停机注意事项")
请务必注意，此迁移技术可能会涉及应用程序的停机时间窗口。过渡期间，旧Redis实例和新Dragonfly实例之间的数据可能无法同步。请务必计划并传达停机时间，以尽量减少对用户或服务的干扰。

