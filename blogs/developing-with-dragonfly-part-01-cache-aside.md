# 使用 Dragonfly 进行开发：Cache-Aside
在使用 Dragonfly 进行开发系列博客文章中，我们将探索使用 Dragonfly 开发应用程序的不同技术和最佳实践。我们从最常见的用法之一开始：Cache-Aside。

[Joe Zhou](https://www.dragonflydb.io/blog/authors/joe-zhou)  2023 年 8 月 30 日

![image](/images/AKXBNRMZXIrzmhbl76CMKkMxd_iKO86xzpWAVCf6hfE.png)

## 介绍
在 Web 应用程序开发（或涉及后端服务器的任何其他应用程序）领域，对最佳性能的追求通常与有效管理数据检索的挑战相交叉。 想象一个场景，Web 应用程序的任务是从其主数据库中获取大量数据：用户配置文件、博客文章、产品详细信息、交易历史记录等。 这些对主数据库的查询将不可避免地引起显着的延迟，从而导致糟糕的用户体验，并可能将数据库推向它的性能极限。

常见的解决方案是添加缓存，以减轻主数据库的大部分负载。 此缓存机制作为战略中介运行，位于构成应用程序的复杂服务网络和主数据库本身之间。 其根本目的是存储最频繁请求的数据或预计即将访问的数据的副本。

## 缓存该何去何从？
虽然单个服务内的缓存似乎是一种方便的解决方案，但它带来了挑战。 这种方法会导致服务变得有状态，从而可能导致同一服务应用程序的多个实例之间的数据不一致。

Redis 传统上扮演着集中式缓存服务的角色，提供无缝的数据存储和检索。 然而，由于 Redis 往往受到单线程数据操作和复杂集群管理的限制，Dragonfly 的出现引起了人们的关注。 Dragonfly 可以作为 Redis 的直接替代品，保留兼容性，同时在多线程、无共享架构之上利用新颖的算法和数据结构，将应用程序的缓存管理提升到前所未有的速度和可扩展性水平。

在这篇博文中，我们将看到在服务中实现缓存层的代码示例，并讨论不同的缓存逐出策略。

---
## 基本的缓存旁路实现
服务代码示例将使用 [Fiber](https://github.com/gofiber/fiber) Web 框架而不是标准库在 Go 中实现。 Fiber 的灵感来自[Express](https://github.com/expressjs/express)，它是在本博文中演示缓存层的更简单的选择。 但一般思想和技术也可以应用于其他编程语言和框架。 如果您想继续操作，可以在 [这个 GitHub repo](https://github.com/dragonflydb/dragonfly-examples/tree/main/cache-aside-with-expiration) 中找到示例应用程序代码。

### 1\. Cache-Aside 模式
本博文中提供的代码示例使下面描述的 Cache-Aside 模式变得生动起来。 Cache-Aside（或延迟加载  Lazy-Loading）是最常见的可用缓存模式。 基本的数据检索逻辑可以概括为：

* 当应用程序中的服务需要特定数据时，它会首先查询缓存。
* 理想情况下，所需数据在缓存中随时可用；这通常称为 “**缓存命中 cache hit”**。通过缓存命中，服务可以直接响应查询。 这种快速通道避免了使用数据库的需要，从而大大减少了响应时间。
* 如果缓存中缺少所需数据（恰当地称为 “**缓存未命中 ** **cache miss**”），服务会将查询重定向到主数据库。
* 从数据库检索数据后，缓存也会被延迟的查询填充，服务最终可以响应查询。

当我们深入研究代码和概念的复杂性时，维护 Cache-Aside 模式的可视化是很有价值的。 在不确定或思维有点混乱的时候，此图可以作为坚定的参考点。

![image](/images/KRFnREKP4L6A8LQ9JiDC3YN7nWtPdFZeDzxqIU33mt0.png)

### 2\. 连接 Dragonfly
首先，让我们创建一个与 Dragonfly 对话的客户端：

```Plain Text
import (
    "context"
    "log"
    "time"

    "github.com/gofiber/fiber/v2"
    "github.com/redis/go-redis/v9"
)

// Use local Dragonfly instance address & credentials.
func createDragonflyClient() *redis.Client {
    client := redis.NewClient(&redis.Options{Addr: "localhost:6379"})

    err := client.Ping(context.Background()).Err()
    if err != nil {
        log.Fatal("failed to connect with dragonfly", err)
    }

    return client
}
```
在提供的代码片段中，值得注意的是 [go-redis](https://github.com/redis/go-redis) 库的使用。 需要强调的是，由于 Dragonfly 与 RESP 有线协议具有令人印象深刻的兼容性，只要提供正确的网络地址和认证凭据，相同的代码就可以无缝连接到 Dragonfly。 这种固有的适应性最大限度地减少了考虑切换到 Dragonfly 时的冲突磨合，展现了它可以作为在你想优化管理你的 Web 应用程序中缓存时，可以成为一个轻松升级的途径和方式。

### 3\. 创建缓存中间件
接下来，让我们创建一个缓存中间件：

```Plain Text
import (
    "context"
    "log"
    "time"

    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/cache"
    "github.com/redis/go-redis/v9"
)

func createServiceApp() *fiber.App {
    client := createDragonflyClient()

    // Create cache middleware connecting to the local Dragonfly instance.
    cacheMiddleware := cache.New(cache.Config{
        Storage:              &CacheStorage{client: client},
        Expiration:           time.Second * 30,
        Methods:              []string{fiber.MethodGet},
        MaxBytes:             0, // 0 means no limit
        StoreResponseHeaders: false,
    })

    // ...
}
```
在 Fiber 框架中，中间件功能在 HTTP 请求周期期间充当操作链中的链接。 它可以访问上下文，封装请求的范围，以执行特定的任务。 在本教程的上下文中，中间件是使用即用型缓存中间件模板构建的。 观察应用于缓存中间件的配置参数至关重要：

* `Storage` 指定缓存的数据存储库。 这里，使用了内存存储 `client` 上的包装层`CacheStorage`，它连接并向 Dragonfly 发出命令。
* `Expiration`规定每个缓存条目的生存时间。
* `Methods` 控制哪些 HTTP 方法进行缓存。 在上面的代码片段中，重点在于仅缓存 `GET` 端点。
* `MaxBytes` 定义存储的响应正文字节的上限。 将其设置为 `0` 表示没有限制，放弃对 Dragonfly 的 `maxmemory` 和 `cache_mode` 服务器端缓存驱逐标志的控制管理。 稍后将详细介绍这一点。

### 4\. 存储接口
在上面的代码片段中，我们传入了一个包装 Dragonfly 客户端的结构体指针。

```Plain Text
Storage: &CacheStorage{client: client}
```
这一选择背后的基本原理源于这样一个事实：Fiber 的创建者在中间件设计过程中预见到了不同的场景。 因此，直接传递 Dragonfly 客户端是不可行的，因为 Dragonfly 客户端配备了用于各种命令的独特类型方法。 相反，巧妙的解决方案涉及引入 `CacheStorage` 包装器。 该包装器遵循 Fiber 中定义的通用接口，即 `Storage`，有效地表示表示特定行为的方法签名的固定集合。

随后的代码片段深入研究了`CacheStorage`的机制：

```Plain Text
// CacheStorage implements the fiber.Storage interface.
type CacheStorage struct {
    client *redis.Client
}

func (m *CacheStorage) Get(key string) ([]byte, error) {
    // Use the actual GET command of Dragonfly.
    return m.client.Get(context.Background(), key).Bytes()
}

func (m *CacheStorage) Set(key string, val []byte, exp time.Duration) error {
    // Use the actual SET command of Dragonfly.
    return m.client.Set(context.Background(), key, val, exp).Err()
}

func (m *CacheStorage) Delete(key string) error {
    // Use the actual DEL command of Dragonfly.
    return m.client.Del(context.Background(), key).Err()
}

func (m *CacheStorage) Close() error {
    return m.client.Close()
}

func (m *CacheStorage) Reset() error {
    return nil
}

// In https://github.com/gofiber/fiber/blob/master/app.go
// Below is the 'Storage' interface defined by the Fiber framework.
// The 'CacheStorage' type needs to conform all the methods defined in this interface.
type Storage interface {
    Get(key string) ([]byte, error)
    Set(key string, val []byte, exp time.Duration) error
    Delete(key string) error
    Reset() error
    Close() error
}
```
在此构造中，`CacheStorage` 精心地将自身与 Fiber 框架的要求保持一致。 这种对齐是通过诸如 `Get`、`Set` 和 `Delete` 之类的方法实现来促进的，这些方法实现旨在与指定键处的字节数据进行交互。 虽然上面的实现利用了 Dragonfly 固有的 `GET`、`SET` 和 `DEL` 命令，但这里的基本原理是实现这些命令Fiber 接口规定的方法。

### 5\. 注册中间件&处理程序
现在缓存中间件就位，利用 Dragonfly 作为底层内存存储，下一步是注册缓存中间件和处理程序以读取用户和博客信息。 需要强调的是，实现的优雅通过 `app.Use(cacheMiddleware)` 指令体现出来，该指令将缓存中间件的效果传播到所有路由的全局范围。 但是，我们将缓存中间件配置为仅缓存使用 HTTP GET 方法（即 `Methods: []string{fiber.MethodGet}`）的路由上的响应。 使用任何其他 HTTP 方法（例如 HEAD、POST、PUT 等）的路由仍然不受缓存影响。 请注意，为简单起见，服务应用程序中只有 GET 路由。

```Plain Text
func createServiceApp() *fiber.App {
    client := createDragonflyClient()

    // Create cache middleware connecting to the local Dragonfly instance.
    cacheMiddleware := cache.New(cache.Config{
        Storage:              &CacheStorage{client: client},
        Expiration:           time.Second * 30,
        Methods:              []string{fiber.MethodGet},
        MaxBytes:             0, // 0 means no limit
        StoreResponseHeaders: false,
    })

    // Create Fiber application.
    app := fiber.New()

    // Register the cache middleware globally.
    // However, the cache middleware itself will only cache GET requests.
    app.Use(cacheMiddleware)

    // Register handlers.
    app.Get("/users/:id", getUserHandler)
    app.Get("/blogs/:id", getBlogHandler)

    return app
}
```
### 6\. 启动Dragonfly & 服务申请
Go 代码的最后部分需要创建服务应用程序实例，这是前面几节中精心建立的构造。 随后，该实例在 `main` 函数中被利用，如下所示：

```Plain Text
func main() {
    app := createServiceApp()
    if err := app.Listen(":8080"); err != nil {
        log.Fatal("failed to start service application", err)
    }
}
```
但是，在运行 `main` 函数之前，我们不要忘记另一个重要步骤：在本地启动 Dragonfly 实例。 有多种选项可用于 [快速启动并运行 Dragonfly](/docs/getting-start/README.md)。 在本教程中，使用下面的 `docker-compose.yml` 文件以及 `docker compose up` 命令，我们将在本地运行一个 Dragonfly 实例。

```Plain Text
version: '3'
services:
  dragonfly:
    container_name: "dragonfly"
    image: 'docker.dragonflydb.io/dragonflydb/dragonfly'
    ulimits:
      memlock: -1
    ports:
      - "6379:6379"
    command:
      - "--maxmemory=2GB"
      - "--cache_mode=true"
```
如前所述，在 Dragonfly 实例初始化期间指定 `maxmemory` 和 `cache_mode` 标志参数具有至关重要的意义。 最后，我们可以运行 `main` 函数，使用 Dragonfly 作为缓存层来初始化服务应用程序。

### 7\. 与服务交互
服务应用程序现在正在运行，下一步是向服务发起 HTTP GET 请求，从而与缓存机制进行交互：

```Plain Text
curl --request GET \
  --url http://localhost:8080/blogs/1000
```
HTTP 调用应该有成功的响应。 同时，响应正文应该由我们的缓存中间件自动保存在 Dragonfly 中：

```Plain Text
dragonfly$> GET /blogs/1000_GET_body
"{\"id\":\"1000\",\"content\":\"This is a micro-blog limited to 140 characters.\"}"
```
### 8\. 回顾 - 缓存读取和缓存读取写入路径
现在，让我们追溯到 [Cache-Aside 模式说明](/blogs/developing-with-dragonfly-part-01-cache-aside.md#1-cache-aside-模式)，重点关注读取路径：< /span>

* 发起 HTTP GET 请求。
* 熟练的缓存中间件会介入，在处理程序执行之前将自身定位，并尝试从 Dragonfly 读取数据。
* 发生缓存未命中，因为这是第一次检索数据。
* 缓存中间件委托给处理程序，处理程序依次从主数据库读取数据。
* 一旦缓存中间件从处理程序获取响应，它就会在返回之前将响应保存在 Dragonfly 中。
* 缓存可以轻松响应未来对相同数据的请求，从而避开主数据库。

就写入路径而言，为了简单起见，示例中的服务应用程序没有合并 HTTP POST/PUT 路由。 因此，缓存失效或替换策略仍然是本博文的未知领域。 服务应用程序依赖于缓存中间件中的过期配置。 这种安排会在某些缓存项目逐渐失去相关性时将其逐出。

但是，尽管存在自动过期功能，但重要的是要承认 Dragonfly 实例仍然会发现自己正在接近其定义的 `maxmemory`，在示例中为 2GB配置如上。 为了可视化这种潜在场景，请考虑我们的服务应用程序努力回答用户和博客请求，跨越数千万个 ID 的不同范围。 在这种情况下，接近`maxmemory`的可能性是显而易见的。

幸运的是，还指定了`cache_mode=true` 参数。 通过这样做，当接近 `maxmemory` 限制时，Dragonfly 将驱逐将来最不可能再次请求的项目。 我们将在下面进一步讨论`cache_mode=true`论证的力量。

## Dragonfly 作为缓存存储的优势
在这篇博文中，我们创建了一个带有 Dragonfly 支持的缓存层的服务应用程序。 正如我们所看到的，Fiber 作者为中间件设计了一个通用存储接口。 这意味着许多内存数据存储可以调整并用作后备存储。 这一发现自然引发了一个问题：既然有多种可用选项，为什么 Dragonfly 是现代缓存的最佳选择？

### 内存效率和内存效率超高吞吐量
传统的哈希表是建立在动态链表数组之上的，[Dragonfly 的 Dashtable](/docs/architecture-and-features/dashtable.Zh_CN.md) 是一个动态数组大小恒定的平面哈希表。 这种设计可实现更高的内存效率，从而使稳态内存使用量减少多达 30%，如 [Redis 与 Dragonfly 可扩展性和性能博客文章中详细介绍的](/blogs/scaling-performance-redis-vs-dragonfly.md)。

同时，得益于 Dragonfly 的先进架构，单个 AWS EC2 `c6gn.16xlarge` 实例的吞吐量可飙升至惊人的 400 万次操作/秒。

### LFRU 驱逐政策的高命中率
当在初始化期间传递 `cache_mode=true` 时，Dragonfly 会使用通用的 **最近最少使用 (LFRU)** 缓存策略在接近 `maxmemory` 限制时逐出项目。 与 Redis 相比LRU缓存策略，LFRU能够抵抗流量波动，不需要随机采样，每项内存开销为零，运行时开销非常小。 [Dragonfly Cache Design 博文](https://www.dragonflydb.io/blog/dragonfly-cache-design) 中全面描述了 LFRU 背后的算法，即 2Q，同时考虑了频率和时间敏感性：

> 2Q 通过承认仅仅因为新 item 被添加到缓存中并不意味着它有用而改进了 LRU。 2Q 要求项目之前至少被访问过一次才能被视为高质量 item。

通过采用 2Q 算法，Dragonfly 的 LFRU 驱逐策略可以带来更高的缓存命中率。 命中率越高，通过缓存层传播并到达主数据库的请求就越少，从而带来更稳定的延迟和更好的服务质量。

充分利用 Dragonfly 的优势：内存效率、高吞吐量和高命中率，它成为现代云驱动环境中缓存存储的一个引人注目的选择。

## 结论
在这篇博文中，我们演示了如何使用 Fiber Web 框架中间件构建缓存层。 我们的探索主要深入研究具有固定缓存过期时间的 Cache-Aside 模式。 此外，我们还讨论了 Dragonfly 的通用 LFRU 逐出策略如何提供更高的缓存命中率，并通过减少主数据库的负载而带来巨大益处。

尽管我们没有深入讨论缓存替换策略，特别是在写操作的上下文中， **使用 Dragonfly 进行开发** 系列即将发布的博客文章将探索更高级的缓存模式来解决这个问题。

请继续关注我们即将发布的帖子，了解更多见解。 一如既往，[几分钟内即可开始使用 Dragonfly](/docs/getting-start/)，祝您构建愉快，直到我们再次见面！

---
## 附录
术语**中间件** 在本教程中多次提及。 在不同的上下文中，该术语可能具有截然不同的含义。 在本博客的讨论中，我们特别引用了 Web 框架领域内的中间件概念。 此解释与 [Express 文档](https://expressjs.com/en/guide/using-middleware.html) 中提供的定义一致，其中中间件函数描述为：

> 中间件函数是可以访问请求对象（req）、响应对象（res）以及应用程序请求-响应周期中的下一个中间件函数的函数。

此外，中间件功能可以执行以下任务：

* 执行任意代码。
* 更改请求和响应对象。
* 结束请求-响应周期。
* 调用堆栈中的下一个中间件函数。

许多 Web 框架的概念与可能适合实现缓存的中间件相似（即使不相同）。 以下是使用相同概念的流行框架的简短但不完整的列表：

* [Actix](https://github.com/actix/actix-web) & [Axum](https://github.com/tokio-rs/axum) (Rust) 都有中间件。
* [Django](https://github.com/django/django) (Python) 有中间件。
* [Express](https://github.com/expressjs/expressjs.com) (JavaScript) 有中间件。
* [Fiber](https://github.com/gofiber/fiber) (Go) 有中间件。
* [Nest](https://github.com/nestjs/nest) (TypeScript/JavaScript) 具有拦截器。
* 请注意，Nest 有中间件、拦截器和异常过滤器。
* 尽管名称不同，Nest 拦截器可能最接近上面的中间件定义。\* [Spring Boot](https://github.com/spring-projects/spring-boot) (Java) 有拦截器。

