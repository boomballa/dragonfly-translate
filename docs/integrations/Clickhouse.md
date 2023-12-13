# Clickhouse
# Clickhouse
## [简介](https://www.dragonflydb.io/docs/integrations/clickhouse#introduction "Direct link to Introduction")
ClickHouse® 是一种用于在线分析处理 (OLAP) 的高性能、面向列的 SQL 数据库管理系统 (DBMS)。ClickHouse 提供了多种与外部系统集成的方式，包括表引擎。一旦配置了using`CREATE TABLE`或`ALTER TABLE`带有`ENGINE`条件的语句，表引擎就允许ClickHouse查询外部系统。从用户的角度来看，配置的集成看起来就像一个普通的表，但查询被代理到外部系统。

Redis 是 ClickHouse 支持的外部集成之一。由于 Dragonfly 与 Redis 高度兼容，因此 ClickHouse 可以与 Dragonfly 一起使用，并且在应用程序中零代码更改和最少的配置更改。

## 使用[Dragonfly](https://www.dragonflydb.io/docs/integrations/clickhouse#running-clickhouse-with-dragonfly "直接链接到使用 Dragonfly 运行 ClickHouse")
为了利用 Dragonfly 的多线程功能并为您的应用程序实现卓越的性能，请按照以下步骤使用 Dragonfly 作为表引擎来配置 ClickHouse。

### 1.[安装](https://www.dragonflydb.io/docs/integrations/clickhouse#1-installations "直接链接至 1. 安装")
首先，假设您有本地 Dragonfly 二进制文件，您可以使用以下参数运行 Dragonfly：

```bash
$> ./dragonfly --bind localhost --port 6379
```
接下来，[在本地安装 ClickHouse](https://clickhouse.com/docs/en/install#quick-install)并使用`clickhouse local`客户端应用程序，如下所示：

```bash
$> ./clickhouse local
```
请注意，上述安装步骤可能是在本地开始使用 Dragonfly 和 ClickHouse 以演示集成的最简单方法。对于生产部署，我们建议阅读Dragonfly 的[管理 Dragonfly 部分](https://www.dragonflydb.io/docs/managing-dragonfly)和ClickHouse 的[生产部署部分。](https://clickhouse.com/docs/en/install#available-installation-options)

### 2\. 表引擎[配置](https://www.dragonflydb.io/docs/integrations/clickhouse#2-table-engine-configuration "直接链接到2.表引擎配置")
尽管本地安装很简单，但 Dragonfly 与 ClickHouse 的集成是无缝的，这要归功于 ClickHouse 支持各种表引擎，并且 Dragonfly 与 Redis 高度兼容。

`Redis`使用表引擎和`localhost:6379`Dragonfly 服务器的地址在 ClickHouse 中创建一个表：

```sql
-- Within the ClickHouse CLI:
CREATE TABLE dragonfly_table
(
    `key` String,
    `v1` UInt32,
    `v2` String,
    `v3` Float32
)
ENGINE = Redis('localhost:6379') PRIMARY KEY(key);
-- 'localhost:6379' is the Dragonfly server address initialized above.
```
### 3.[使用](https://www.dragonflydb.io/docs/integrations/clickhouse#3-usage "直接链接到 3. 使用")
向表中插入数据：

```sql
INSERT INTO dragonfly_table Values('1', 1, '1', 1.0), ('2', 2, '2', 2.0);
```
查询表：

```sql
SELECT * FROM dragonfly_table WHERE key='1';
SELECT COUNT(*) FROM dragonfly_table;
```
就是这样！就这么简单，您已经成功将 ClickHouse 与 Dragonfly 集成了。接下来，我们将探讨使用 Dragonfly 和 ClickHouse 的优势。

## Dragonfly[优势](https://www.dragonflydb.io/docs/integrations/clickhouse#dragonfly-advantages%20%22%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E8%9C%BB%E8%9C%93%E4%BC%98%E5%8A%BF%22)
### 1\. 高[吞吐量](https://www.dragonflydb.io/docs/integrations/clickhouse#1-high-throughput "直接链接到 1. 高吞吐量")
将 Dragonfly 与 ClickHouse 集成的主要优势之一是提高吞吐量的潜力。在单个 AWS EC2`c6gn.16xlarge`实例上，Dragonfly 能够实现 4M 操作/秒的吞吐量`GET`和`SET`命令。虽然 Dragonfly/ClickHouse 集成的精确基准测试数据仍在研究中，但 Dragonfly 的底层多线程架构和设计旨在确保更高的吞吐量。一旦基准测试数据可用，我们将用更多详细信息更新本节。

### 2\. 大[数据集](https://www.dragonflydb.io/docs/integrations/clickhouse#2-large-datasets "直接链接至 2. 大数据集")
正如ClickHouse建议的，当使用表引擎进行键值模型时，建议使用点查询而不是范围查询。但是，如果需要范围查询，ClickHouse仍然可以使用该`SCAN`命令来完成它们。

* 点查询示例——`WHERE key = xx`或者`WHERE key IN (xx, yy)`
* 范围查询示例--`WHERE key > xx`

Dragonfly 能够在单个实例上支持高达 1TB 的容量。另一方面，Redis 通常只能在单个实例上处理数十 GB 的数据。此外，使用 Redis 集群（这似乎是一种解决方案）是不可能的，因为它不支持`SCAN`集群模式下的命令。因此，对于需要扩展数据集和范围查询的应用程序，与 ClickHouse 配合使用时，Dragonfly 是键值表引擎最有效、最可靠的选择。我们确实知道 1TB 可能不是现代术语中的“大数据”，但在单个实例上支持 1TB 仍然是一个优势，并且对于许多临时分析用例来说应该足够了。

## 有用的[资源](https://www.dragonflydb.io/docs/integrations/clickhouse#useful-resources "直接链接到有用资源")
* ClickHouse[主页](https://clickhouse.com/)、[GitHub](https://github.com/ClickHouse/ClickHouse)和[文档](https://clickhouse.com/docs/en/intro)。
* 阅读更多有关 ClickHouse 表引擎与 Redis 集成[的](https://clickhouse.com/docs/en/engines/table-engines/integrations/redis)信息，可以通过更改子句`host:port`中的服务器地址来替换为 Dragonfly `ENGINE`。
* [在此处](https://clickhouse.com/docs/en/sql-reference/dictionaries)阅读有关 ClickHouse 如何与字典配合使用的更多信息。

