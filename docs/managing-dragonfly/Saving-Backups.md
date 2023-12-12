# 备份保存
Dragonfly 实例可以自动和手动生成磁盘备份。许多标志会影响备份机制的行为。可以通过命令行或通过[标志文件](https://www.dragonflydb.io/docs/getting-started/binary#flag-files)给出标志。

## 自动[备份](https://www.dragonflydb.io/docs/managing-dragonfly/backups#automatic-backups "直接链接到自动备份")
`dbfilename`只要标志不为空，Dragonfly 就会在关闭时创建备份。此外，Dragonfly 可以配置为使用该`snapshot_cron`标志执行计划备份。注意：

* 在 Dragonfly >= 1.7.1 中，`snapshot_cron`引入了该标志。
* 在 Dragonfly < 1.7.1 中，只能`save_schedule`使用标志。
* 该`save_schedule`标志已被弃用，并将在未来的版本中完全删除。

[了解下面有关备份标志](https://www.dragonflydb.io/docs/managing-dragonfly/backups#flags)的详细用法的更多信息。

## 手动[备份](https://www.dragonflydb.io/docs/managing-dragonfly/backups#manual-backups "直接链接到手动备份")
`SAVE`可以使用或命令手动触发备份`BGSAVE`。

## 自动[装载](https://www.dragonflydb.io/docs/managing-dragonfly/backups#automatic-loading "直接链接到自动加载")
当 Dragonfly 实例启动时，它将尝试在其配置的路径中查找快照文件`dir`并自动加载它。`dbfilename`与自动备份一样，可以通过配置空值来禁用此功能。

## 参数flags
* **dir**-- A path to the directory where the backup snapshot files will be saved.
* df\_snapshot\_format -- Set to true to save snapshots in Dragonfly file format, true by default.
* dbfilename -- The filename to save and load the database. See more details about this flag below.
* snapshot\_cron -- Generate snapshots based on a cron schedule. See more details about this flag below.
* save\_schedule (deprecated) -- Generate snapshots periodically. The argument is a HH:MM format that supports [globbing](https://en.wikipedia.org/wiki/Glob_(programming)) (i.e., `23:45`, `*:15`, `*:*`). **This flag is deprecated, and the support will be completely removed in a future release.**

\**-- 根据 cron 计划生成快照。**请参阅下面**有关此标志的更多详细信息。* **save\_schedule****（已弃用）** ——定期生成快照。参数是`HH:MM`支持[通配符](https://en.wikipedia.org/wiki/Glob_(programming))的格式（即，`23:45`, `*:15`, `*:*`）。 **该标志已弃用，并且在未来版本中将完全删除支持。**

### `参数_dbfilename`
该`dbfilename`标志控制 Dragonfly 用于加载和保存备份快照的文件名。值得注意的是，传递的参数应该只包含文件名，不包含任何文件扩展名。例如，如果所需的文件名是`dump.dfs`，其中`.dfs`Dragonfly 快照文件格式的扩展名是 ，则该标志应设置为`dump`。

该标志支持自动将时间戳添加到快照文件名。要生成带有时间戳的文件名，请在传递给此标志的参数中使用宏`{timestamp}`（即`dump-{timestamp}`）。每次保存快照时，宏将被替换为本地服务器时间的时间戳，采用字典顺序排序的格式。

**该标志的默认值为****dump-{timestamp}****。**

让我们看一个使用该`dbfilename`标志的示例。使用以下命令启动 Dragonfly 实例：

```bash
$> ./dragonfly --logtostderr --dir my-snapshot-dir --dbfilename my-snapshot-file-{timestamp}
```
当 Dragonfly 实例运行时，发出命令`SAVE`两次，间隔大约 5 秒：

```bash
dragonfly$> SAVE
OK
# 5 seconds later...

dragonfly$> SAVE
OK
```
随后，大约 5 秒后停用 Dragonfly 实例。当 Dragonfly 实例关闭时，此操作将立即启动自动备份快照。此时，检查`my-snapshot-dir`目录的内容，该目录最初被指定为标志的输入参数`dir`。在此目录中，您现在应该观察到三个快照的存在，每个快照都带有预期的文件名`my-snapshot-file`，通过替换`{timestamp}`宏的生成时间戳来区分，间隔大约 5 秒。

```bash
$> ls my-snapshot-dir
my-snapshot-file-2023-08-10T07:23:02-0000.dfs
my-snapshot-file-2023-08-10T07:23:02-summary.dfs
my-snapshot-file-2023-08-10T07:23:07-0000.dfs
my-snapshot-file-2023-08-10T07:23:07-summary.dfs
my-snapshot-file-2023-08-10T07:23:12-0000.dfs
my-snapshot-file-2023-08-10T07:23:12-summary.dfs
```
### 参数[\_](https://www.dragonflydb.io/docs/managing-dragonfly/backups#the-snapshot_cron-flag%20%22Direct%20link%20to%20the-snapshot_cron-flag%22)`snapshot_cron`
在 Dragonfly >= 1.7.1 中，`snapshot_cron`引入了该标志。如果可用，强烈建议优先考虑该`snapshot_cron`标志而不是已弃用的`save_schedule`标志。顾名思义，`snapshot_cron`为 Dragonfly 实例建立一个 cron 计划，从而启用自动备份快照。

Cron（或 crontab）作为类 Unix 操作系统上广泛使用的作业调度程序：

* 如果您想深入研究 cron，可以浏览其[维基百科页面](https://en.wikipedia.org/wiki/Cron)。
* crontab [guru](https://crontab.guru/)网站是一个有用的在线工具，用于翻译和验证您的 cron 计划。

cron调度的一般结构如下：

```Plain Text
┌───────────────────── minute (0 - 59)
│ ┌─────────────────── hour (0 - 23)
│ │ ┌───────────────── day of the month (1 - 31)
│ │ │ ┌─────────────── month (1 - 12)
│ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
│ │ │ │ │
│ │ │ │ │
│ │ │ │ │
* * * * *
```
以下是一些 cron 计划示例：

|克朗|描述|
| ----- | ----- |
|* * * * *|每分钟|
|*/5 * * * *|每 5 分钟一次|
|*/30 * * * *|每 30 分钟一班|
|0 */2 * * *|每 2 小时 0 分|
|5 */2 * * *|每 2 小时 5 分|
|0 0 * * *|每天 00:00（午夜）|
|0 0 * * 1-5|周一至周五 00:00（午夜）|
|0 6 * * 1-5|周一至周五 06:00（黎明）|

让我们看一个使用该`snapshot_cron`标志的示例。使用以下命令启动 Dragonfly 实例：

```bash
$> ./dragonfly --logtostderr --dir my-snapshot-dir --snapshot_cron "*/5 * * * *"
```
Dragonfly 实例将每 5 分钟自动创建备份快照。

```bash
$> ls my-snapshot-dir
dump-2023-08-10T00:00:00-0000.dfs
dump-2023-08-10T00:00:00-summary.dfs
dump-2023-08-10T00:05:00-0000.dfs
dump-2023-08-10T00:05:00-summary.dfs
dump-2023-08-10T00:10:00-0000.dfs
dump-2023-08-10T00:10:00-summary.dfs
dump-2023-08-10T00:15:00-0000.dfs
dump-2023-08-10T00:15:00-summary.dfs
dump-2023-08-10T00:20:00-0000.dfs
dump-2023-08-10T00:20:00-summary.dfs
dump-2023-08-10T00:25:00-0000.dfs
dump-2023-08-10T00:25:00-summary.dfs
# ... more
# ... output
# ... omitted
```
## 云[存储](https://www.dragonflydb.io/docs/managing-dragonfly/backups#cloud-storage "直接链接到云存储")
[Dragonfly 支持在AWS S3](https://aws.amazon.com/s3/)而不是本地磁盘上保存和恢复备份。

要使用 S3 存储或 S3 兼容服务（例如[MinIO ）](https://github.com/minio/minio)，请在使用您的 S3 存储桶 URL 配置上`--dir`标志。

例如使用 Bucket `acme-backups`，配置 Dragonfly 为：

```bash
$> ./dragonfly --logtostderr --dir s3://acme-backups
```
您还可以包含可选的路径前缀，例如将所有备份存储在 path 下`dragonfly/staging/`，配置：

```bash
$> ./dragonfly --logtostderr --dir s3://acme-backups/dragonfly/staging
```
与磁盘备份一样，Dragonfly 将在启动时自动从 S3 存储桶加载最新快照，并在关闭时存储快照。

⚠️ Dragonfly S3 云存储是预览功能，尚不支持 HTTPS。如果您发现任何问题或需要 HTTPS 支持，请打开[GitHub问题。](https://github.com/dragonflydb/dragonfly)

### 云存储参数
除了上述备份标志之外，云存储还包括以下标志：

* s3\_endpoint-- 当使用 S3 兼容服务覆盖 S3 端点时。
* s3\_ec2\_metadata-- 是否从 EC2 元数据加载 AWS 凭证。仅当在 EC2 中运行时启用此标志。默认禁用。
* s3\_sign\_payload-- 将快照上传到 S3 时是否对请求负载进行签名。默认情况下启用，但签署有效负载确实会产生性能开销，因此可以根据需要禁用。



### [认证](https://www.dragonflydb.io/docs/managing-dragonfly/backups#authentication "Direct link to Authentication")
[Dragonfly 将使用标准 AWS 凭证提供程序](https://docs.aws.amazon.com/sdkref/latest/guide/standardized-credentials.html)从环境中推断您的 AWS 凭证 。

Dragonfly 支持提供商：

* 环境变量：如`AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、 等`AWS_SESSION_TOKEN`。请参阅[AWS 访问密钥](https://docs.aws.amazon.com/sdkref/latest/guide/feature-static-credentials.html)。
* 共享凭据文件：从本地`~/.aws/credentials`文件加载凭据。使用 覆盖凭证路径并`AWS_SHARED_CREDENTIALS_FILE`使用 覆盖默认 AWS 配置文件`AWS_PROFILE`。请参阅[配置和凭证文件设置](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)。
* EC2 元数据：在 EC2 中运行时，从 EC2 实例元数据加载凭证。必须启用`--s3_ec2_metadata`。

您还必须使用环境变量、共享配置文件或 EC2 元数据配置存储桶的 AWS 区域`AWS_REGION`。

### S3 兼容[服务](https://www.dragonflydb.io/docs/managing-dragonfly/backups#s3-compatible-services "直接链接到 S3 兼容服务")
要使用 S3 兼容服务（例如[MinIO ）](https://github.com/minio/minio)，请按上述方式配置凭据，然后将`--s3_endpoint` 标志设置为您的服务端点。

例如您在本地运行 MinIO 容器`localhost:9000`，请配置：

```bash
$> ./dragonfly --logtostderr --dir s3://acme-backups --s3_endpoint localhost:9000
```
