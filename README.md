<p align="center">
  <a href="https://dragonflydb.io">
    <img  src="https://raw.githubusercontent.com/dragonflydb/dragonfly/main/.github/images/logo-full.svg"
      width="284" border="0" alt="Dragonfly">
  </a>
</p>

## 目录

* [说明文档](/docs/documentation/)
  * [FAQ](/docs/documentation/Frequently-Asked-Questions.md)
  * [License](/docs/documentation/License.md)
* [架构及特性](/docs/architecture-and-features/)
  * [Dragonfly架构](/docs/architecture-and-features/df-share-nothing.Zh_CN.md)
  * [Dragonfly中的Dashtable](/docs/architecture-and-features/dashtable.Zh_CN.md)
  * [Dragonfly的事务的生命周期](/docs/architecture-and-features/transaction.Zh_CN.md)
  * [Dragonfly RDB快照设计](/docs/architecture-and-features/rdbsave.Zh_CN.md)
  * [Dragonfly与Redis的区别](/docs/architecture-and-features/differences.Zh_CN.md)
* [入门及部署](/docs/getting-start/)
  * [使用 Docker 安装 DragonflyDB](/docs/getting-start/install-with-docker.md)
  * [Docker Compose部署](/contrib/docker/)
  * [使用 Helm Chart进行 Kubernetes 部署](/contrib/charts/dragonfly/)
  * [通过源代码构建 Dragonfly](/docs/getting-start/build-from-source.Zh_CN.md)
* [管理 Dragonfly](/docs/managing-dragonfly/)
  * [Dragonfly 时间点快照设计](/docs/managing-dragonfly/Dragonfly-Point-in-Time-Snapshotting-Design.md)
  * [复制](/docs/managing-dragonfly/Replication.md)
  * [备份保存](/docs/managing-dragonfly/Saving-Backups.md)
  * [集群模式](/docs/managing-dragonfly/Cluster-Mode.md)
  * [高可用性质](/docs/managing-dragonfly/High-Availability.md)
  * [Memchached模式](/docs/managing-dragonfly/Memcached-Mode.md)
  * [监控](/docs/managing-dragonfly/Monitoring.md)
  * [Dragonfly operator](/docs/managing-dragonfly/Dragonfly-Operator/)
    * [安装 Dragonfly Kubernetes Operator](/docs/managing-dragonfly/Dragonfly-Operator/Install-Dragonfly-Kubernetes-Operator.md)
    * [Dragonfly配置](/docs/managing-dragonfly/Dragonfly-Operator/Dragonfly-Configuration.md)
    * [Dragonfly实例认证](/docs/managing-dragonfly/Dragonfly-Operator/Dragonfly-Instance-Authentication.md)
    * [通过PVC进行快照](/docs/managing-dragonfly/Dragonfly-Operator/Snapshots-through-PVC.md)
    * [具有服务器 TLS 的 Dragonfly](/docs/managing-dragonfly/Dragonfly-Operator/Dragonfly-With-Server-TLS.md)
    * [将 Prometheus 与 Dragonfly Operator 集成](/docs/managing-dragonfly/Dragonfly-Operator/Integrate-Prometheus-with-the-Dragonfly-Operator.md)
    * [将 Grafana 与 Dragonfly operator集成](/docs/managing-dragonfly/Dragonfly-Operator/Integrate-Grafana-with-Dragonfly-operator.md)
    * [使用 Dragonfly Operator 配置 S3 快照](/docs/managing-dragonfly/Dragonfly-Operator/Configure-Snapshots-to-S3-with-the-Dragonfly-Operator.md)
  * [使用 Lua 编写脚本](/docs/managing-dragonfly/Scripting-with-Lua.md)
  * [将 Dragonfly 与 TLS 结合使用](/docs/managing-dragonfly/Using-Dragonfly-With-TLS.md)
  * [已知限制](/docs/managing-dragonfly/Known-Limitations.md)
  * [访问控制列表（ACL）](/docs/managing-dragonfly/Access-Control-Lists-(ACL).md)
  * [命令行参数](/docs/managing-dragonfly/Command-line-arguments-(flags).md)
  * [HTTP协议](/docs/managing-dragonfly/HTTP.md)
* [迁移 Dragonfly](/docs/migratin-to-dragonfly/)
  * [快照和恢复](/docs/migratin-to-dragonfly/from-redis-instance/snapshot-and-restore.md)
  * [研究复制](/docs/migratin-to-dragonfly/from-redis-instance/replication.md)
  * [哨兵升级](/docs/migratin-to-dragonfly/from-redis-instance/sentinel-promotion.md)
* [命令参考](/docs/command-reference/)
* [开发](/docs/development/)
  * [软件开发工具包](/docs/development/SDKs.md)
  * [命令行界面](/docs/development/CLI.md)
* [集成](/docs/integrations/)
  * [BullMQ](/docs/integrations/BullMQ.md)
  * [Clickhouse](/docs/integrations/Clickhouse.md)
  * [Feast](/docs/integrations/Feast.md)
* [博客精选](/blogs/)
  * [监控内存数据库](/blogs//monitoring-in-memory-datastores.md)
  * [平衡与不平衡](/blogs/balanced-vs-unbalanced.md)
  * [使用Dragonfly无所畏惧地处理BigKey](/blogs/fearlessly-handling-bigkeys-with-dragonfly.md)
  * [宣布Dragonfly Kubernetes Operator全面可用](/blogs/announcing-kubernetes-operator-general-availability.md)
  * [Dragonfly内存数据库现在支持复制以实现高可用性](/blogs/replication-for-high-availability.md)
  * [Redis与Dragonfly的可扩展性和性能](/blogs/scaling-performance-redis-vs-dragonfly.md)


## 翻译说明

### 翻译规范
* 不译的英文名词复数改为单数，因为中文没有复数的概念；
* 专有名词保持大写：HTML, HAML, SASS, REST... 等等；
* 英文和数字与中文之间要留空格。中文标点符号和中文之间不需要留空格；
* 使用中文的标点符号。句号是 `。` 不是 `.`，破折号是 `——` 不是 `-`；
* 英文有斜体、中文没有，可用强调取代 `**强调**`；
* 代码缩进使用两个空格，禁止使用 Tab；
* 按照语义把英文逗号改为中文顿号；
* 译完保证语句通顺

### 保留不译的名词
有些为数据库常用的专有名词，在不造成读者理解困难的情况下，尽量保持原状。

英文  | 说明
----- | ------
key | 键
value | 值
entry | 条目
item | 项目
segment | 段
bucket | 桶
slot | 槽
hook | 钩子
hop | 消息跳跃
dispatch | 发起
dispatcher | 分发器
replica | 从库，副本
non-blocking | 非阻塞
coordinator | 协调器
fiber | 纤程
mutable | 可变
immutable | 不可变
flag | 参数

### 说明并翻译的名词
英文  | 说明
----- | ------
shard | 分片
queue | 队列
transaction | 事务
socket | 网络嵌套字
SnapshotShard | 快照分片



### 小贴士

* [Bing 词典](http://cn.bing.com/dict/) 可以查询相关的翻译
* [谷歌翻译](https://translate.google.com/) 对整句处理的较好


### 建议与反馈

有翻译不准确的地方欢迎指正，也欢迎提出任何建议！直接开一个 github issues 就好。

### 持续更新中...
目前把相对重要的文档翻译完毕，后续会持续补充其他文档以及近期优秀 Blog ...

### 官方页面
* [Dragonfly 官方 github 主页](https://github.com/dragonflydb/dragonfly)
* [Dragonfly 官方文档](https://www.dragonflydb.io/docs/)
* [Dragonfly 官方博客地址](https://www.dragonflydb.io/blog)
* [Dragonfly 社区版本主页](https://www.dragonflydb.io/features)

### 有趣的扩展阅读资料
* [单独链接法（Separate Chaining）](https://www.baeldung.com/cs/hashing-separate-chaining)
* [开放地址法（Open Addressing）](https://alrightchiu.github.io/SecondRound/hash-tableopen-addressing.html)
* [近因效应（recency effect）](https://wiki.mbalib.com/zh-tw/%E8%BF%91%E5%9B%A0%E6%95%88%E5%BA%94)
* [可扩展哈希（Extendible hashing）](https://en.wikipedia.org/wiki/Extendible_hashing)