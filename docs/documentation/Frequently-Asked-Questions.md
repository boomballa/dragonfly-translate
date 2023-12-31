## 经常被问到的问题

- [Dragonfly的许可模式是什么？它是开源的吗？](#dragonfly的许可模式是什么？它是开源的吗？)
- [我可以在生产中使用 Dragonfly 吗？](#我可以在生产中使用-Dragonfly-吗？)
- [Dragonfly 的垂直扩展与 Redis 集群相比如何？](#Dragonfly-的垂直扩展与-Redis-集群相比如何？)
- [什么时候支持 X 命令？](#什么时候支持-X-命令？)


### Dragonfly的许可模式是什么？它是开源的吗？
Dragonfly 是根据 [BSL 1.1](../LICENSE.md) (商业源代码许可证)。 我们认为 BSL 许可证比类似 AGPL 的许可证更宽松。一般来说，这意味着只要您不销售与 Dragonfly 或内存数据存储直接相关的服务，Dragonfly 的代码就可以免费使用和自由更改。
我们跟随 Elastic、Redis、MongoDB、Cockroach Labs、Redpanda Data 等其他科技公司的趋势，保护我们为我们正在构建的软件提供服务和支持的权利。


### 我可以在生产中使用 Dragonfly 吗？
就许可证而言，只要您不将 Dragonfly 作为托管服务提供，您就可以在生产中自由使用 Dragonfly。从代码成熟度的角度来看，Dragonfly的代码都覆盖了单元测试。然而，与任何新软件一样，有些用例很难测试和预测。我们建议您在考虑生产使用之前在 Dragonfly 上运行您自己的特定用例几天。如果您需要 DragonflyDB 团队的启动和运行帮助，您可以申请我们的[免费早期访问计划](https://www.dragonflydb.io/early-access)。



### Dragonfly 的垂直扩展与 Redis 集群相比如何？
Dragonfly 以最佳方式利用底层硬件。这意味着它可以在小型 8GB 实例上运行，并可垂直扩展到具有 64 核的大型 768GB 机器。这种多功能性可以大大降低在单个节点上运行集群工作负载的复杂性，从而节省硬件资源和成本。更重要的是，它降低了处理多节点集群的复杂性（总拥有成本）。此外，Redis 集群模式对多键和事务操作施加了一些限制，而 Dragonfly 提供了与单节点 Redis 相同的语义。


### 什么时候支持 X 命令？
您可以查看我们的[命令参考](https://www.dragonflydb.io/docs/category/command-reference)页面，了解哪些命令已受支持。Dragonfly 已经实现了 200 多个 Redis 命令，我们认为这些命令很好地覆盖了绝大多数用例。话虽如此，如果您需要不受支持的命令，请随时提出问题或对现有问题进行投票。我们将尽力根据这些命令的受欢迎程度确定其优先级。
