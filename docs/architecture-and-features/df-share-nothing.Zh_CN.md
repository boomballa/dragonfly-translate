# Dragonfly架构

Dragonfly 是 Redis 和 Memcached 等内存存储的现代替代品。它在单个实例上垂直扩展以支持每秒数百万个请求。它的内存效率更高，设计时考虑到了可靠性，也包括更好的缓存设计。

## 线程模式

Dragonfly 采用单进程多线程架构。每个 Dragonfly 线程都通过纤程间接分配多个职责。

其中一项职责是处理传入连接。 一旦 socket 监听器接受客户端请求，该连接的整个生命周期就会绑定到纤程内的单个线程。Dragonfly 被编写为 100% 非阻塞（non-blocking）; 它使用纤程在每个线程中提供异步性。异步性的基本属性之一是，只要线程有待处理的 CPU 任务，就不会被阻塞。Dragonfly 通过将每个执行上下文单元包装在纤程中来保留此属性；我们包装可能在 I/O 上被阻塞的执行单元。例如， 在纤程内一个连接环路了; 写入快照的函数在纤程内运行，等等。

旁注 - 异步性和并行性是不同的术语。例如， Nodejs提供异步执行，但是是单线程的（single-threaded）。同样， 每个 Dragonfly 线程本身也是异步的；因此，几十载处理长时间运行的命令（例如保存快照到磁盘或者运行Lua脚本）。Dragonfly 也能响应传入事件。 

### DF中的线程参与者

DF 内存数据库被分为 `N` 部分，其中 `N` 小于等于系统中的线程数， 每个数据库分片都由单个线程拥有和访问。同一线程可以处理 TCP 连接并同时托管数据库分片。参见下图。

<br>
<img src="http://static.dragonflydb.io/repo-assets/thread-per-core.svg" border="0"/>

在这里，我们的 DF 进程生成 4 个线程，其中线程 1 到 3 处理 I/O（即管理客户端连接），线程 2 到 4 管理数据库分片。例如，线程 2 将其 CPU 时间划分为处理传入请求和处理其拥有的分片上的数据库操作。

因此，当我们说线程 1 是 I/O 线程时，我们的意思是 Dragonfly 可以将管理客户端连接的纤程固定到线程 1。一般来说，任何线程都可以承担许多需要 CPU 时间的职责；数据库管理和连接处理只是其中的两个职责。


## 纤程

这里建议阅读 Romange（CTO OF DF）关于 `Boost.Fibers`的文章 [介绍文章](https://www.romange.com/2018/12/15/introduction-to-fibers-in-c-/) 去学习更多关于纤程的信息。

顺便说一句，要称赞一下 `Boost.Fibers` 这个库–它的设计非常好：
它不干扰、轻量级且高效。此外，它的默认调度程序可以被覆盖。 在为 Dragonfly 提供支持的 `helio` I/O库中，我们重写了 `Boost.Fibers` 调度程序以支持无共享架构并将其与 I/O 轮询循环集成。

重要的是，光纤需要应用层自下而上的支持以保持其异步性。例如，在下面的代码片段中， 阻塞写入 `fd` 不会神奇的允许一个纤程抢占并切换到另一个纤程。不，整个线程将被阻塞住。

```cpp
...
write(fd, buf, 1000000);

...
pthread_mutex_lock(...);

```

同样，通过调用 `pthread_mutex_lock` ，整个线程可能会被阻塞，从而浪费宝贵的 CPU 时间。 因此，Dragonfly 代码使用 *fiber-friendly(友好的纤程)* 来进行 I/O、通信和协调。这些原语是有 `helio` 和 `Boost.Fibers` 库提供的。

## 命令请求的生命周期

本节介绍 Dragonfly 如何在无共享架构的上下文中处理命令。在当今使用的大多数架构中，多线程服务器使用互斥锁来保护其数据结构，但 Dragonfly 没有。为什么是这样？

Dragonfly 中的线程间交互仅通过在线程之间传递消息来发生。例如，考虑以下处理 SET 请求的序列图：

```uml
@startuml

actor       User       as A1
boundary    connection  as B1
entity      "Shard K"   as E1
A1 ->  B1 : SET KEY VAL
B1 -> E1 : SET KEY VAL / k = HASH(KEY) % N
E1 -> B1 : OK
B1 -> A1 : Response

@enduml
```

<img src="https://www.plantuml.com/plantuml/svg/NOn12m8X48Nl_eh7Gb272Az1WGl2Wb6G5NGqLsW9PaBjqBzlL-lId6Q-zxvnFdD4dNCAlzKbA2bk_ABUnJS0U2OAFWzC9Msb29I7N3AWiNSNUvYckbeA9R7SOknX3QjFCFgAYzg9jd3zXx720njqodRp4IqmmrxegLe_7CnNLDDr3Ed9bC87"/>

这里，一个连接纤程驻留在线程中，与一个处理 `KEY` 请求的实体线程是不同的。 我们使用哈希来决定哪个分片拥有哪个 key。

考虑此流程的另一种方式是连接纤程充当向其他线程发出事务命令的协调器（coordinator）。 在这个简单的示例中， 外部 "SET" 命令需要从协调器（coordinator）传递到目标分片线程的单个消息。 当我们在单个命令请求的上下文中考虑 Dragonfly 模型时，我更喜欢使用下图而不是 [上面的图](#thread-actors-in-df)。

<br>
<img src="http://static.dragonflydb.io/repo-assets/coordinator.svg" border="0"/>

一个协调器（coordinator） 或者是一个连接纤程甚至可能驻留在同时拥有其中一个分片的线程之一上。然而，更容易将其视为一个从不直接访问任何分片数据的单独实体。

协调器服务可以被看作一个虚拟化层，隐藏与所有分片通信的所有复杂性。它采用最先进的算法为 "MSET， MGET 和 BLPOP" 等多键命令提供原子性 (和严格的可序列化性) 语义。它还为 Lua 脚本和多命令事务提供严格的可串行化。 

隐藏这种复杂性对于最终客户来说很有价值，但它会带来一些 CPU 和延迟成本。我们相信，考虑到 Dragonfly 提供的价值，这种权衡是值得的。

如果您想深入研究 Dragonfly 架构，而又不想要复杂的事务代码， 那么值得查看一下 [Midi Redis](https://github.com/romange/midi-redis/),
它实现了一个支持 `PING`, `SET`, and `GET` [命令](https://github.com/romange/midi-redis/blob/main/server/main_service.cc#L239)的玩具后端。

事实上， Dragonfly 就是从这个项目发展而来的，他们有共同的提交历史。

顺便说一句，为了学习如何构建比 `midi-redis` 更简单的 TCP 后端， `helio` 库提供了如下示例后端： [echo_server](https://github.com/romange/helio/blob/master/examples/echo_server.cc) 和 [ping_iouring_server.cc](https://github.com/romange/helio/blob/master/examples/pingserver/ping_iouring_server.cc)。这些后端在多核服务器上达到数百万 QPS，就像 Dragonfly 和 midi-redis 一样。
