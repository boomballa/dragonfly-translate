od# 使用 Dragonfly 构建电子商务应用程序
在这篇博文中，我们将了解 Dragonfly 如何通过高效缓存、个性化内容和管理高流量事件来推动电商平台的发展。了解 Dragonfly 确保高峰时段平稳运行的功能。

[周乔](https://www.dragonflydb.io/blog/authors/joe-zhou)   2023 年 11 月 16 日

![image](/images/0N67QaKNACX5-RfPQRN-7Ld_7v2wFwbRa7wQO5ElzsM.png)

## 介绍
在电商平台（e-commerce platform）的高强度（high-octane）世界中，响应速度和数据准确性都至关重要。 客户希望无缝访问搜索的商品、过去的订单、最近查看的产品和个性化推荐。 与此同时，这些应用程序的流量经常波动，尤其是在圣诞节或黑色星期五等高峰时段。 高流量事件还带来了重大挑战，需要快速响应和精确的数据处理。 解决这些变化通常需要可扩展且强大的内存数据存储解决方案。

Dragonfly 是一种超高性能内存数据存储，采用多线程、无共享架构，将硬件推向极限， 在单个实例上支持高达 400 万次操作/秒和 1TB 内存。 这可以大大降低操作复杂性，同时为电子商务应用程序提供高性能解决方案。 对于要求更高的场景，除了令人惊叹的单节点性能之外，Dragonfly 还提供集群模式。 这种适应性使 Dragonfly 成为应对不可预测和变化的流量模式的电商平台的理想选择。

在本博客中，我们将探讨如何以各种方式使用 Dragonfly 来提升电商平台的性能和用户体验，特别是在以下领域：

* **缓存** - **字符串**、**哈希** 和 **JSON** 数据类型非常适合缓存。
* **个性化** - **Sorted-Set**非常适合跟踪用户偏好。
* **高流量闪购** - **原子操作**和**分布式锁**可用于在要求极高的情况下，例如管理库存验证和扣除。



## 缓存
缓存是 Web 技术中的一项强大策略，对于电子商务应用程序尤其重要。 它通过将数据库查询或 API 请求的结果存储在缓存中来实现更快的数据检索。 当发出相同的请求时，可以从缓存（在本例中为 Dragonfly）快速提供数据，从而避免与主数据库进行耗时的交互。

选择适当的缓存数据类型对于简化电商平台的实施和优化性能至关重要。 最容易访问的数据类型是 `String`，非常适合缓存 blob 数据。 它用途广泛且安全，可以存储各种格式，无论是文本还是二进制，例如 JSON 字符串、MessagePack 或 Protocol Buffer。 例如，用户最近的订单摘要可以缓存为 JSON 字符串。 但是，使用 `String` 数据类型的缺点是难以操作缓存数据中的各个字段。

```Plain Text
# Using the 'String' data type for caching.

# To cache an order:
dragonfly$> SET order_string:<order_id> '{"id": "<order_id>", "items": [{"id": "001", "name": "Laptop", "quantity": 1}], "total": 1799.99}'

# To retrieve the entire cached order:
dragonfly$> GET order_string:<order_id>
```
或者，`Hash` 数据类型是单级字符串到字符串（single-level string-to-string）的平面哈希映射，适合存储 field/value 对。 这可用于缓存用户订单的特定属性，例如商品 ID 和数量，以便更快地访问和更新。

```Plain Text
# Using the 'Hash' data type for caching.

# To cache quantities for different items and the total price in the order:
dragonfly$> HSET order_hash:<order_id> item_001 5 item_002 6 item_003 7 total 2799.99

# To retrieve the quantity of a specific item:
dragonfly$> HGET order_hash:<order_id> item_003

# To retrieve the entire cached order:
dragonfly$> HGETALL order_hash:<order_id>
```
最后，对于更复杂的数据结构，Dragonfly 中的`JSON` 数据类型原生且完全支持[JSON](https://www.json.org/json-en.html) 规格 和 [JSONPath](https://github.com/json-path/JsonPath) 语法，可以轻松操作各个字段。 这对于详细的订单信息特别有用，其中订单的各个方面（例如产品详细信息、定价和运输信息） 可以单独访问和修改，提供数据处理的灵活性和效率。

```Plain Text
# Using the 'JSON' data type for caching.

# To cache an order as native JSON:
dragonfly$> JSON.SET order_json:<order_id> $ '{"id": "<order_id>", "items": [{"id": "001", "name": "Laptop", "quantity": 1}], "total": 1799.99}'

# To update the quantity of a specific item:
dragonfly$> JSON.SET order_json:<order_id> $.items[0].quantity 2

# To retrieve the total price of the order:
dragonfly$> JSON.GET order_json:<order_id> $.total

# To retrieve the entire cached order:
dragonfly$> JSON.GET order_json:<order_id>
```
有关更多与缓存相关的技术，请查看我们之前的博客文章：

* [使用 Dragonfly 进行开发：Cache-Aside](https://www.dragonflydb.io/blog/developing-with-dragonfly-part-01-cache-aside) 跟随分步教程了解如何使用 Dragonfly 实现 Cache-Aside 模式。
* [使用 Dragonfly 进行开发：解决缓存问题](https://www.dragonflydb.io/blog/developing-with-dragonfly-part-02-solve-caching-problems) 了解如何使用 Dragonfly 解决 3 个常见的缓存问题（穿透、崩溃和雪崩）。
* [Dragonfly Cache Design](https://www.dragonflydb.io/blog/dragonfly-cache-design)了解有关 Dragonfly 内部驱逐算法的更多信息。

---
## 个性化
在电子商务应用中，个性化用户体验是关键。 实现这一目标的一种有效方法是展示优先项目，例如特定用户查看次数最多的产品类别或当天在应用程序上全局查看最多的项目。 `Sorted-Set` 是 Dragonfly 中提供的一种数据结构，非常适合此任务。 它是唯一元素的集合，每个元素都与一个分数相关联，分数决定了元素的顺序。

例如，要跟踪用户查看次数最多的产品类别，我们可以使用`Sorted-Set`，其中每个类别都是一个成员，以及用户查看该类别的次数类别就是 score。 每次用户查看某个类别时，score 都会增加，确保该类别始终反映用户当前的偏好。

```Plain Text
# Using the 'Sorted-Set' data type to track user preferences.

# To increment the view count for a category for a user:
dragonfly$> ZINCRBY viewed_product_categories_by_user_id:<user_id> 1 "electronics"

# To retrieve the top 5 viewed categories for a user:
dragonfly$> ZREVRANGE viewed_product_categories_by_user_id:<user_id> 0 4 WITHSCORES
```
类似地，对于全局查看，我们可以为整个应用程序维护一个`Sorted-Set`，其中任何用户对产品类别的每次查看都会增加该类别的 score。 这种方法可以根据受欢迎程度对类别或项目进行动态、实时排名，为用户和平台提供有价值的展示。

`Sorted-Set` 是许多应用程序中的重要数据结构，特别是对于上述场景。 Redis 长期以来因其对这种数据结构的稳健实现、促进高效的数据排序和检索而闻名。 然而，从 v1.11 开始，Dragonfly 引入了基于 [B+ Tree](https://en.wikipedia.org/wiki/B%2B_tree) 的实现。 这种新的实现不仅增强了性能，而且还提高了大小方面的内存效率，使其成为处理大规模数据排序和排名任务的绝佳选择。 我们计划在未来的博客文章中更深入地探讨这个主题。

---
## 高流量闪购
在我们之前的讨论中，我们强调了电商平台面临流量波动的挑战，特别是在黑色星期五闪购等备受瞩目的活动期间。 在这些高峰期，Dragonfly 可以发挥关键作用，特别是在管理库存验证和扣除方面。 虽然这些任务可以使用传统的 SQL 数据库来执行，但大量用户同时尝试购买有限库存商品可能会很快压垮主数据库。 这就是 Dragonfly 作为超高性能内存数据存储的能力变得无价的地方。

Dragonfly 可以通过两种机制应对这一挑战：**原子操作**和**分布式锁**。 每种机制又可以通过不同的方式实现，我们将在下面详细探讨。

### 1\. 使用 `INCR` 或 `DECR` 进行原子操作
Dragonfly 的原子性确保每个命令（例如递增或递减值）完全独立地执行，而不会受到其他操作的干扰。 考虑这样一个场景，我们的闪购产品库存有限，我们可以通过设置数量来初始化 Dragonfly 中的库存计数：

```Plain Text
# Assuming the flash sale has 100 units for a particular item.
dragonfly$> SET item_on_sale:<item_id> 100
```
为了简单起见，我们假设每个请求都是针对该商品的单个操作单元。 当发出购买请求时，我们使用`DECR`命令自动扣除库存：

```Plain Text
dragonfly$> DECR item_on_sale:<item_id>
```
`DECR` 命令的返回值至关重要。 如果大于零，则表明该产品仍然可用，可以继续购买。 相反，如果返回值为零或更少，则表明该产品已售完，应拒绝进一步购买。 这种方法很容易实现，对于库存直接更新就足够了并且可以稍后管理更复杂的流程（例如订单取消），对于这种简单场景特别有效。

### 2\. Lua 脚本的原子操作
对于更复杂的场景，可以使用 Lua 脚本来实现原子库存验证和扣除。 值得注意的是，Dragonfly 允许使用 `disable-atomicity` 脚本参数在 Lua 脚本中进行非原子操作，如本篇 [博客文章](https://www.dragonflydb.io/blog/leveraging-power-of-lua-scripting)。 因此，我们需要确保脚本参数不配置于我们的电子商务库存扣除场景。 假设我们使用 `Hash` 数据类型存储商品的库存：

```Plain Text
dragonfly$> HSET item_on_sale:<item_id> inventory 100 purchased 0
```
为了验证和扣除库存，我们可以使用以下Lua脚本：

```Plain Text
-- atomic_inventory_deduction.lua

local key = KEYS[1];
local num_to_purchase = tonumber(ARGV[1]);

if num_to_purchase <= 0 then
   return nil;
end

local item = redis.call("HMGET", key, "inventory", "purchased");
local inventory = tonumber(item[1]);
local purchased = tonumber(item[2]);

if purchased + num_to_purchase <= inventory then
   redis.call("HINCRBY", key, "purchased", num_to_purchase);
   return num_to_purchase;
end

return nil;
```
在上面的脚本中，我们首先解析传递给脚本的 key 和参数。 该脚本对单个 key 进行操作，该 key 是代表所售商品的。 同样，该脚本需要一个参数，即要购买的单位数量。 然后，我们使用 `HMGET` 命令检索当前库存和该商品购买的单位数量。 如果购买的单位总数加上要购买的单位数量小于或等于库存，我们会增加购买的单位数量并返回购买的单位数量。 否则，我们将返回 `nil` 以表明购买无法继续。 上面的脚本可以使用`EVAL`命令执行：

```Plain Text
# General syntax of the 'EVAL' command:
#   EVAL script num_of_keys [key [key ...]] [arg [arg ...]]

# Try to purchase 5 units of the item:
dragonfly$> EVAL "<script>" 1 item_on_sale:<item_id> 5
```
或者，我们可以加载脚本并使用`EVALSHA`命令，该命令效率更高，因为它将脚本存储在 Dragonfly 中：

```Plain Text
# Load the script into Dragonfly and get the SHA1 digest of the script.
dragonfly$> SCRIPT LOAD "<script>"

# Try to purchase 5 units of the item using the SHA1 digest of the script:
dragonfly$> EVALSHA "<script_sha>" 1 item_on_sale:<item_id> 5
```
与`DECR`命令相比，Lua脚本选项允许更复杂的库存验证和扣除逻辑，并涵盖更多边缘场景。 例如，在上面的脚本中，我们进行了健全性检查，以确保要购买的单位数量大于零。 与此同时，我们允许购买多于一件的商品，并且它可以处理库存不足以很好地满足请求数量的边缘场景。

### 3.带条件 `SET` 的分布式锁
Dragonfly 的另一个强大功能是它能够充当分布式锁，在处理流量激增方面发挥着关键作用。 在高流量期间（例如限时抢购），每个传入请求都会尝试从 Dragonfly 获取 lock。 只有成功获得 lock 的请求才能获得对该特定待售商品进行进一步操作的独占权利。 这些操作可能包括数据库操作、支付流程或需要与电商平台内的主数据库或第三方服务直接交互的任何其他操作。

无法获取 lock 的请求将被拒绝进一步处理，并且可以重定向到等待页面或带有倒计时器的重试页面，具体取决于实现。 这可确保**每个特价商品一次只允许处理一个请求**，从而防止主数据库因过多的并发请求而不堪重负。< /span>

使用分布式锁的常见逻辑可能类似于以下伪代码：

```Plain Text
// purchase_item_pseudocode.js

const itemId = getItemId();
const userId = getUserId();
const expiration = getLockExpirationTime();

const lockKey = `item_lock:${itemId}`;
const lockVal = userId;

const lockAcquired = acquireLock(lockKey, lockVal, expiration);

if (!lockAcquired) {
    sendResponse("Another user got this item, please try again later.");
}

try {
    purchaseItem(itemId, userId);
} catch (purchaseError) {
    throw purchaseError;
} finally {
    releaseLock(lockKey, lockVal);
}

sendResponse("You have successfully purchased the item!");
```
值得注意的是，`acquireLock` 和 `releaseLock` 都考虑了商品 ID 和用户 ID。 我们希望确保在高并发情况下获取的 lock 不会被其他用户意外释放，`releaseLock`的实现应该符合这个要求。 另外， lock 过期时间的选择也很重要。 它应该足够长，以允许用户完成购买过程，但不能太长，以免服务进程在未释放锁的情况下意外终止时防止其他用户获取锁。

`acquireLock` 功能可以在 Dragonfly 中使用 `SET` 命令以及 `NX` 和 < 来实现a i=4> 选项：`EX`

```Plain Text
# Using the 'SET' command with the 'NX' option to acquire a lock.
# The 'NX' option ensures that the lock is only acquired if the key does not exist.
# Also, we set the expiration time for the lock to prevent the lock from being held indefinitely.
dragonfly$> SET item_lock:<item_id> <user_id> NX EX <expiration>
```
另一方面，`releaseLock`函数需要在Lua脚本中实现，以确保只有当用户ID与获取锁的用户ID匹配时才释放锁。 这可以在 Dragonfly 中使用 `EVAL` 命令和以下 Lua 脚本来实现：

```Plain Text
-- release_lock.lua

local key = KEYS[1];
local val = ARGV[1];

local lock_val = redis.call("GET", key);

if lock_val == val then
   redis.call("DEL", key);
end

return nil;
```
### 4.RedLock 分布式锁
在 Dragonfly 中使用条件`SET` 命令和 Lua 脚本进行分布式锁是管理高度并发操作的一种简单而有效的方法。 然而，在需要更高级别的可靠性和容错能力的环境中，特别是跨分布式系统，**RedLock**分布式锁算法增加了额外的安全性。< /span>

RedLock 旨在跨 Redis 的多个主实例扩展锁定机制。 由于 Dragonfly 与 Redis 高度兼容，因此 RedLock 也可以与 Dragonfly 实例一起使用。 RedLock 确保在所有这些实例中正确且一致地获取和释放 lock，从而增强分布式锁定过程的可靠性和完整性。

使用 RedLock 时，`acquireLock` 和 `releaseLock` 函数通常已由客户端库提供。 这通常涉及尝试同时获取多个实例上的锁并确保大多数实例在继续之前授予锁。 同样，释放过程涉及尝试在所有实例之间释放锁以保持一致性。

有关 RedLock 的更多信息，请阅读文档[此处](https://redis.io/docs/manual/patterns/distributed-locks/)。

---
## 结论
在本博客中，我们探讨了如何以各种方式使用 Dragonfly 来提升电商平台的性能和用户体验。 我们讨论了 Dragonfly 如何用于缓存、个性化和高流量限时抢购。

总体而言，Dragonfly 是一款用于构建和维护电商平台的多功能且强大的工具，能够熟练处理从日常用户交互到最苛刻的销售活动的各个方面。 尽管本博客中没有直接展示，但 Dragonfly 的性能是惊人的，正如我们之前的 [博客文章](https://www.dragonflydb.io/blog) 中详细讨论的那样。 我们鼓励您[亲自尝试 Dragonfly](https://www.dragonflydb.io/docs/getting-started) 并亲身体验其功能。 另外，请考虑订阅下面的时事通讯，以了解最新的 Dragonfly 新闻和更新！

![image](/images/Nq2DN8w2lWawiAMBG1vLHtL2s7Ufbj7t_5CYM__ZBZw.png)





























