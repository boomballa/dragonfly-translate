# 使用 Dragonfly 进行开发：解决缓存问题
在使用 Dragonfly 进行开发系列博客文章中，我们将探索使用 Dragonfly 开发应用程序的不同技术和最佳实践。在这篇博文中，我们探讨了 3 个常见的缓存问题以及缓解这些问题的最佳实践。

[Joe Zhou](https://www.dragonflydb.io/blog/authors/joe-zhou)  2023 年 9 月 13 日

![image](/images/An5TBXflrAoCXpjZRPLxp0no8S8WlkFAyeGcgjNifUQ.png)

## 介绍
在我们的[上一篇博文](/blogs/developing-with-dragonfly-part-01-cache-aside.md#1-cache-aside-模式)中，**使用 Dragonfly 进行开发** 系列中，我们深入研究了最简单的缓存模式之一，称为 Cache-Aside。 这是一种简单的缓存方法，相对容易实现。 然而，正如我们将在这篇博文中探讨的那样，它还可以发现一些可能对您的服务产生重大影响的常见问题。 我们将重点介绍缓存管理中的 3 个主要问题，并分享避免这些问题的最佳实践。

值得注意的是，这些缓存问题并不是 Dragonfly 所独有的。 即使 Dragonfly 具有卓越的效率和性能，这些挑战在各种缓存解决方案中仍然存在。 Dragonfly 的协议和命令与 Redis 兼容，这意味着它可以轻松集成到您的后端服务中，无需开发成本。 通过解决这些缓存问题并利用 Dragonfly 的优势，我们旨在帮助您顺利过渡到 Dragonfly，并在缓存基础设施方面取得更大的提升。

---
## 1\. 缓存穿透
首先，我们来看看缓存穿透。 当缓存层找不到请求的数据并将请求直接转发到主数据库时，就会发生缓存未命中。 这种情况在缓存系统中很常见。 **但是，当缓存未命中，但主库也缺少所需数据时，就会发生缓存穿透。**

![image](/images/CCI_HfwwR31N08fCkM2Md8wmwPx3bmfeNLwg0dGnkFI.png)

缓存穿透有效地使缓存层失效，主库成为争夺点。 如果黑客向系统发送大量包含随机密钥的请求，所有这些请求都会导致缓存未命中，并随后从主数据库中消失， 这可能会导致主数据库不堪重负，从而可能导致其失败。 现在，让我们深入研究两种广泛使用的技术来缓解缓存穿透问题。

### 缓存空结果
解决缓存穿透问题的一种有效技术是缓存空结果。 当主库不包含请求的数据时，作为缓存层，我们仍然可以将空结果（即`null`、`nil`）存储在Dragonfly。 这种方法实施起来相对简单，但它确实有一个明显的缺点。 尽管空值占用的内存空间非常小，但如果服务遇到大量使用不存在 key 的请求， 它可能会导致宝贵的内存存储的浪费。

为了有效地管理这个问题，从缓存层中驱逐空结果至关重要。 这里有两种方法：

* 在一段通常很短的时间后使空结果过期。 这是一种简单有效的驱逐空结果的方法，而且也很容易实现。
* 采用强大且智能的逐出策略来逐出空结果。 此方法涉及根据各种因素（例如缓存使用模式、访问频率或内存限制）做出驱逐决策。 我们将在下面的 [Dragonfly LFRU 驱逐政策](/blogs/developing-with-dragonfly-part-02-solve-caching-problems.md#dragonfly-lfru-驱逐政策) 部分中更详细地讨论这种方法。

### 布隆过滤器
对抗缓存穿透（cache penetration）的另一个有价值的技术是使用 [布隆过滤器](https://en.wikipedia.org/wiki/Bloom_filter)。 布隆过滤器是一种概率数据结构，用于确定 element 是否属于特定 Set。 它因其空间效率而脱颖而出，通常比传统哈希表需要更少的内存，尽管可能会出现误报。 在缓存穿透的情况下，布隆过滤器可以充当保护机制，实现允许列表或阻止列表，以在请求到达主数据库之前对其进行过滤。

从 Dragonfly 的当前版本 (v1.9.0) 开始，尚未原生支持 Bloom Filter 功能。 然而，Dragonfly 社区非常活跃，不断发展项目以满足用户的需求。 如果像 Bloom Filter 这样的功能对于您的场景至关重要，我们鼓励您与社区互动。 您可以通过 [提出问题](https://github.com/dragonflydb/dragonfly/issues) 或通过 Twitter 或 Discord 与 Dragonfly 社区联系来实现此目的，如本页底部所示。

值得一提的是，Dragonfly 社区在响应用户反馈和增强请求方面有着良好的记录。 例如，他们最近通过从 式样（glob-style）模式备份调度转换为更普遍接受的 cron 式格式来解决可用性问题。 这一变更是由社区成员提出并实施的，展示了 Dragonfly 团队及其卓越社区的协作精神和响应能力。 您可以在我们的 [文档](/docs/managing-dragonfly/Saving-Backups.md#参数flags) 中了解有关基于 cron 的备份计划的更多信息 并见证 Dragonfly 团队成员与热情社区之间的积极协同作用 [此处](https://github.com/dragonflydb/dragonfly/issues/1590)。

---
## 2\. 缓存崩溃
接下来，我们来探讨一下缓存崩溃问题。 **当包含一段热数据的缓存 key 过期或意外从缓存中逐出时，就会发生缓存崩溃。**当这种情况发生时，多个请求同时查询相同的数据，由于高并发而导致数据库不堪重负。

![image](/images/baC_HW0okSEZmsbAXMbbKmIwwmhVyThvsecw8Xh55ZU.png)

Cache-Aside 策略在这种情况下也无效。 由于 Cache-Aside 被动地将数据加载到 Dragonfly 中，因此在活动频繁期间（想象一下明星的博客文章突然从缓存中消失），**多个高度并发的查询 **仍然可以到达并对数据库施加压力**。 **让我们深入研究缓解缓存崩溃问题的技术。

### 提前刷新（Refresh-Ahead）缓存策略
Refresh-Ahead 缓存策略提供了一种有效缓解缓存崩溃问题的实用解决方案。 Refresh-Ahead 是一种采取主动方法的策略，在缓存的 key 过期之前刷新缓存，使数据保持热状态，直到不再需要它。 以下是实施 “Refresh-Ahead” 策略的常见步骤：

* 首先选择 refresh-ahead 因子，该因子确定缓存 key 到期之前应刷新缓存的时间。
* 例如，如果缓存数据的生命周期为 100 秒，并且您选择的刷新提前系数为 50%：
   * 如果第50秒之前查询到数据，则照常返回缓存的数据。
   * 如果50秒后查询数据，仍然会返回缓存的数据，但后台 worker 会触发数据刷新。
* 确保只有一名后台 worker 负责刷新数据，以避免对数据库进行冗余读取，这一点很重要。

这种缓存策略在保持缓存数据鲜活和最小化缓存和主数据库的负载之间取得了平衡。 因此，即使在高流量期间，它也是确保可靠且响应迅速的服务的有效手段。

---
## 3\. 缓存雪崩
最后，我们来看看缓存雪崩问题。 **缓存雪崩是指无法同时在缓存层中定位大量缓存 key 时出现的情况**， 导致查询突然激增，压垮主数据库。

![image](/images/hwuswv14afGSZYSjrgiHbAFTsuCK7G2fMFs4VpIEv_A.png)

缓存雪崩背后有两个主要原因：

* 许多缓存的 key 同时过期。
* 缓存层崩溃或在一段时间内无法访问。

我们还探讨缓解缓存雪崩问题的最佳实践。

### 过期时间浮动
过期时间浮动是用于解决缓存雪崩问题的一项有价值的技术。 当从业务角度来看，严格的缓存过期时间并不重要时，向每个缓存 key 的过期时间引入随机浮动可能是有益的，特别是在处理批量生成的缓存密钥时。 该方法涉及在过期时间上添加随机时间方差，确保缓存的 key 不会同时过期，从而有效缓解缓存雪崩问题。

### 提前刷新（Refresh-Ahead）缓存策略
我们之前在缓解缓存崩溃问题的背景下讨论过的 “Refresh-Ahead” 缓存策略也可以作为解决缓存雪崩问题的一个有价值的工具。 在这种场景中，不再依赖于固定的热数据过期时间，而是应用 Refresh-Ahead 策略来确保数据在过期之前被刷新，只要通过频繁的查询持续高需求即可。

通过在缓解缓存雪崩的上下文中采用 Refresh-Ahead 方法，您可以主动刷新和维护缓存中的热数据，防止其完全过期。 此策略有助于确保经常访问的数据保持随时可用，从而降低缓存雪崩场景的风险，即大量缓存的键突然变得无效并压垮主数据库。

### 高可用性
缓解缓存雪崩问题的最后一项技术是使缓存层高度可用。 当缓存层高度可用时，这意味着即使缓存层的主实例发生崩溃或停机，数据库也不会被意外的查询激增所淹没。 我们将在下面的 [Dragonfly 高可用性](/blogs/developing-with-dragonfly-part-02-solve-caching-problems.md#dragonfly高可用性) 部分中更详细地探讨 Dragonfly 中提供的高可用性选项。

---
## 讨论
在这篇博文中，我们深入研究了一些可能会破坏您的服务的最常见的缓存挑战：

* 缓存穿透——缓存总是被系统中不存在的 key/ID 忽略
* 缓存崩溃——热 key 突然过期或被驱逐，导致查询激增
* 缓存雪崩——许多缓存的键同时变得无效/无法访问

我们还探索了缓解这些问题的技术，例如：

* 缓存空结果（缓存穿透）
* 布隆过滤器（缓存穿透）
* Refresh-Ahead 缓存策略（缓存崩溃和缓存雪崩）
* 过期时间浮动（缓存雪崩）
* 高可用性（缓存雪崩）

在这些技术中，Refresh-Ahead 缓存策略已被证明特别强大。 我们已将其添加到  [Dragonfly 示例存储库](https://github.com/dragonflydb/dragonfly-examples/tree/main/cache-refresh-ahead)  中以说明其实现。 以下是示例代码中最重要片段的简要演练。

收到请求后，服务首先检查缓存中是否有请求的数据。 `GET` 命令和 `TTL` 命令通过管道传输到 Dragonfly 以检索数据及其过期时间。 `cacheKey` 基于 HTTP 路径（即 `/blogs/{blog_uuid}`）。

```Plain Text
// cache_refresh_ahead.go

// Pipeline a GET command and a TTL command.
commands, err := m.client.Pipelined(ctx, func(pipe redis.Pipeliner) error {
    pipe.Get(ctx, cacheKey)
    pipe.TTL(ctx, cacheKey)
    return nil
})

// Extract results & errors from the pipelined commands.
var (
    cacheData, dataErr = commands[0].(*redis.StringCmd).Bytes()
    cacheTTL, ttlErr   = commands[1].(*redis.DurationCmd).Result()
)
```
如果在缓存中没有找到数据，服务将从数据库中读取数据并被动地将其加载到 Dragonfly 中，就像 Cache-Aside 策略一样。 这意味着数据的需求不再很高。否则，其缓存将被不断访问和刷新。

```Plain Text
// cache_refresh_ahead.go

// Cache miss.
if errors.Is(dataErr, redis.Nil) {
    log.Info("cache miss - reading from database")
    return m.refreshCacheAndSendResponse(c, cacheKey)
}
```
如果在缓存中找到数据，服务将检查数据的过期时间。 如果数据即将过期（紧急程度取决于提前刷新因素），服务将刷新缓存并返回数据。 否则，服务将直接返回数据。 需要注意的是，`sendResponseAndRefreshCacheAsync`方法利用了一个新的 goroutine 在后台运行刷新操作 返回当前 HTTP 处理程序后，从而确保响应立即发送到客户端。

```Plain Text
// cache_refresh_ahead.go

// Cache hit.
if cacheTTL < m.refreshAheadDuration {
    log.Info("cache hit - refreshing")
    m.sendResponseAndRefreshCacheAsync(uid, cacheKey)
    return m.sendResponse(c, cacheData)
} else {
    log.Info("cache hit - no refreshing")
    return m.sendResponse(c, cacheData)
}
```
在 `sendResponseAndRefreshCacheAsync` 方法中，中间件首先尝试获取当前 `cacheKey` 的 `refreshLock`。 如果成功获取锁，中间件会通过从数据库读取数据并将其写入 Dragonfly 来刷新缓存。 否则，中间件将简单地返回，而不执行任何其他操作。 因为这意味着当前数据的需求量很大，并且另一个并发请求已经触发了刷新缓存的过程。 锁机制保证了高并发下只有一个 goroutine 进行刷新操作。

虽然 Refresh-Ahead 缓存策略需要在服务代码中实现，如上所示，但其背后的想法很简单： 我们基本上是在尝试 **识别热数据并避免其过早过期**。 类似地，对于缓存空结果技术，即使缓存的值无效（即 `null`、`nil`）， 我们仍在尝试识别热门数据（因为攻击者更频繁地请求它）并避免过早驱逐它。

值得注意的是，这些缓存挑战和策略并不是 Dragonfly 所独有的。它们是许多缓存场景中的常见问题。 虽然这些技术的实现可能需要对服务代码进行一些调整，但 Dragonfly 提供了一些有助于有效解决缓存问题的优势。 下面让我们探讨其中的一些优点。

### Dragonfly LFRU 驱逐政策
在 **使用 Dragonfly 进行开发 **的上一期中系列中，我们介绍了 Dragonfly 创新的 LFRU 驱逐政策。 该策略融合了 LFU（最不常用）和 LRU（最近最少使用）策略，提供了一种有效管理缓存逐出的复杂方法。

要在 Dragonfly 中启用 LFRU 驱逐策略，只需在 Dragonfly 服务器初始化期间传递 `cache_mode=true` 参数即可。 该策略的显着之处在于，每个缓存项的内存开销为零，从而实现了高度的资源效率。 此外，它的设计能够应对流量模式的波动。

与服务层中实施的 Refresh-Ahead 缓存策略结合使用时，LFRU 驱逐策略可充当额外的保护层。 在 Dragonfly 服务器端，LFRU 致力于识别并保留热数据，当缓存接近其最大内存限制时，自动驱逐需求较低的数据。 这种主动和被动管理的结合有助于在动态和高流量环境中保持最佳的缓存性能和响应能力。

### Dragonfly高可用性
Dragonfly 提供了多种强大的方法来确保缓存层的高可用性，这有助于防止缓存雪崩和维护系统稳定性：

* [复制](/docs/managing-dragonfly/Replication.md)： Dragonfly 支持复制，允许您设置从库（secondary）实例。 当主实例发生故障时，从库实例可以通过手动故障转移来接管，从而确保服务的适度高可用性。
* 与Redis哨兵的兼容性： Dragonfly 与 Redis 协议和命令的兼容性意味着它也可以与高可用性解决方案 Redis Sentinel 一起使用。 这种集成使您能够利用 Redis Sentinel 的既定功能来维持基于 Dragonfly 的缓存层的高可用性。
* Kubernetes Operator： Dragonfly 提供了官方的 Kubernetes Operator，简化了 Dragonfly 在 Kubernetes 环境中的部署和管理。 利用此运算符，您可以在 Kubernetes 集群中实现高可用性，从而增强缓存层的弹性和可扩展性。 您可以参阅我们的 [文档](/docs/managing-dragonfly/High-Availability.md) 了解有关此运算符的更多信息。

---
## 结论
这篇博文探讨了常见的缓存挑战，包括缓存穿透、缓存崩溃和缓存雪崩，并提供了缓解这些问题的实用技术。 我们讨论了 Refresh-Ahead 缓存策略、缓存空结果技术和过期时间浮动技术等策略，这些策略有助于优化缓存性能，同时保持数据可用性。 此外，Dragonfly 的通用 LFRU 驱逐策略和高可用性选项已被强调为增强缓存弹性的强大工具。 通过结合这些策略并利用 Dragonfly 的功能，您可以创建强大的缓存解决方案，不仅可以减少常见的陷阱，还可以确保高可用性，保护您的系统免受与缓存相关的中断。 在 **使用 Dragonfly 进行开发系列的未来部分中，我们将总结我们尚未介绍的所有其他缓存策略和技术。**

[通过几分钟来运行 Dragonfly](/docs/getting-start/README.md) 来探索它，并加入我们的社区与我们分享您的反馈和想法。 与 Dragonfly 一起快乐构建，下次再见！

