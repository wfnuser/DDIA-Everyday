### 批处理与流处理

批处理和流处理的输出是衍生数据集，例如搜索索引、物化视图等。

批处理和流处理有许多共同的原则，主要的区别在于流处理器在无限数据集上运行，而批处理输入是已知的优先大小。Spark在批处理引擎上执行流处理，将流分解为微批次，而Flink则在流处理引擎上执行批处理。



#### 维护衍生状态

批处理和流处理的风格是鼓励无副作用的函数，有利于容错。

衍生数据系统可以同步的维护，但异步可以避免更多数据一致性的错误。

#### 应用演化后重新处理数据

#### 铁路上的模式迁移

衍生视图允许 渐进演化，如果你想重新构建数据集，不需要执行突然切换式的迁移，而是在相同基础数据上维护两个独立的衍生视图，然后进行灰度测试，最终删除旧视图。



#### Lambda架构

Lambda架构的核心思想是将不可变事件附加到不断增长的数据集来记录传入数据，为了从这些事件中衍生出读取优化的视图，lambda架构建议并行运行两个不同的系统：批处理系统和独立的流处理系统。

在 Lambda 方法中，流处理器消耗事件并快速生成对视图的近似更新；批处理器稍后将使用同一组事件并生成衍生视图的更正版本。这个设计背后的原因是批处理更简单，因此不易出错，而流处理器被认为是不太可靠和难以容错的。而且，流处理可以使用快速近似算法，而批处理使用较慢的精确算法。



#### 统一批处理和流处理

在一个系统中统一批处理和流处理需要以下功能，这些功能也正在越来越广泛地被提供：

1. 通过处理最近事件流的相同处理引擎来重播历史事件的能力。例如，基于日志的消息代理可以重播消息，某些流处理器可以从 HDFS 等分布式文件系统读取输入。
2. 对于流处理器来说，恰好一次语义 —— 即确保输出与未发生故障的输出相同，即使事实上发生故障
3. 按事件时间进行窗口化的工具，而不是按处理时间进行窗口化，因为处理历史事件时，处理时间毫无意义





## 分拆数据库





Unix 和关系数据库以非常不同的哲学来处理信息管理问题。Unix 认为它的目的是为程序员提供一种相当低层次的硬件的逻辑抽象，而关系数据库则希望为应用程序员提供一种高层次的抽象，以隐藏磁盘上数据结构的复杂性、并发性、崩溃恢复等等。Unix 发展出的管道和文件只是字节序列，而数据库则发展出了 SQL 和事务。

Unix 是 「简单的」，因为它是对硬件资源相当薄的包装；关系数据库是 「更简单」 的，因为一个简短的声明性查询可以利用很多强大的基础设施（查询优化、索引、连接方法、并发控制、复制等），而不需要查询的作者理解其实现细节

在这一部分我将试图调和这两个哲学，希望我们能各取其美。



### 组合使用数据存储技术



#### 创建索引

当在关系型数据库创建一个新的索引会发生什么：

数据库必须扫描表的一致性快照，挑选出索引中的字段值，对它们进行排序，然后写出索引。然后它必须处理从一致快照依赖所做的写入操作。一旦完成，只要事务写入表中，数据库就必须继续保持索引更新。

这个过程也非常类似于设置新的从库副本。

#### 

### 围绕数据流设计应用



使用应用代码组合专用存储与处理系统来分拆数据库的方法，也被称为 “**数据库由内而外（database inside-out）**” 方法【26】。

在本节中，我将详细介绍这些想法，并探讨一些围绕分拆数据库和数据流的想法构建应用的方法。

#### 应用代码作为衍生函数



当一个数据集衍生自另一个数据集时，它会经历某种转换函数。例如：

- 次级索引是由一种直白的转换函数生成的衍生数据集：
- 全文搜索索引是通过应用各种自然语言处理函数而创建的；



用于次级索引的衍生函数是如此常用的需求，以致于它作为核心功能被内建至许多数据库中

当创建衍生数据集的函数不是像创建次级索引那样的标准搬砖函数时，需要自定义代码来处理特定于应用的东西。

#### 应用代码和状态的分离

系统的某些部分专门用于持久数据存储，其他部分专门运行应用程序代码，两者可以保持独立的同时互动。

现在大多数 Web 应用程序都是作为无状态服务部署的，其中任何用户请求都可以路由到任何应用程序服务器，并且服务器在发送响应后会忘记所有请求。这种部署方式很方便，因为可以随意添加或删除服务器，但状态必须到某个地方：通常是数据库。

状态和应用解耦：不将应用程序逻辑放入数据库中，也不将持久状态置于应用程序中。

数据库充当一种可以通过网络同步访问的可变共享变量。应用程序可以读取和更新变量，而数据库负责维持它的持久性，提供一些诸如并发控制和容错的功能

但是，大多数编程语言中，你无法订阅可变变量的变更，你只能定期读取它。

但使用观察者模式则能实现

数据库继承了这种可变数据的被动方法：如果你想知道数据库的内容是否发生了变化，通常你唯一的选择就是轮询（即定期重复你的查询）。 订阅变更只是刚刚开始出现的功能。



#### 数据流：应用代码与状态变化的交互

从数据流的角度思考应用程序，意味着我们不再将数据库视作被应用代码操作的被动变量，数据中的数据可以看作状态，我们需要考虑的是状态变更和处理它们的代码之间的相互作用和协同关系，应用代码通过在另一个地方触发状态变更来响应状态变更。



数据库的变更日志就能视为一种我们可以订阅的事件流，维护衍生数据时：

1. 在维护衍生数据时，状态变更的顺序通常很重要（投递时有顺序，处理时也要按顺序）：

1. 1. 消息代理要支持顺序消费
2. 双写会有很多问题，谨慎使用

1. 容错是衍生数据的关键：消息丢失会丢失数据，因此系统必须可靠，且出错时可以重试恢复。稳定的消息排序和容错消息处理是相当严格的要求，但与分布式事务相比，它们开销更小，运行更稳定。



#### 流处理器和服务

当今流行的应用开发风格涉及将功能分解为一组通过同步网络请求（如 REST API）进行通信的 **服务**（service）。这种**面向服务的架构**优于单体架构的优势主要在于：通过松散耦合来提供组织上的可伸缩性：不同的团队可以专职于不同的服务上，从而减少团队之间的协调工作（因为服务可以独立部署和更新）。

数据流系统还能实现更好的性能。例如，假设客户正在购买以一种货币定价，但以另一种货币支付的商品。为了执行货币换算，你需要知道当前的汇率。这个操作可以通过两种方式实现：

1. 在微服务方法中，处理购买的代码可能会查询汇率服务或数据库，以获取特定货币的当前汇率。
2. 在数据流方法中，处理订单的代码会提前订阅汇率变更流，并在汇率发生变动时将当前汇率存储在本地数据库中。处理订单时只需查询本地数据库即可。

第二种方法能将对另一服务的同步网络请求替换为对本地数据库的查询（可能在同一台机器甚至同一个进程中）。数据流方法不仅更快，而且当其他服务失效时也更稳健（但要保证订阅变更流能及时被消费）。最快且最可靠的网络请求就是压根没有网络请求。



### 观察衍生数据状态

衍生数据的写路径（write path)和读路径（read path）



#### 物化视图和缓存



全文搜索索引就是一个很好的例子：写路径更新索引，读路径在索引中搜索关键字。读写都需要做一些工作。写入需要更新文档中出现的所有关键词的索引条目。读取需要搜索查询中的每个单词，并应用布尔逻辑来查找包含查询中所有单词（AND 运算符）的文档，或者每个单词（OR 运算符）的任何同义词。



如果没有索引，搜索查询将不得不扫描所有文档，如果有着大量文档，这样做的开销巨大。没有索引意味着写入路径上的工作量较少（没有要更新的索引），但是在读取路径上需要更多工作。



另一方面，可以想象为所有可能的查询预先计算搜索结果。在这种情况下，读路径上的工作量会减少：不需要布尔逻辑，只需查找查询结果并返回即可。但写路径会更加昂贵：可能的搜索查询集合是无限大的，因此预先计算所有可能的搜索结果将需要无限的时间和存储空间。那肯定没戏 。



另一种选择是预先计算一组固定的最常见查询的搜索结果，以便可以快速提供它们而无需转到索引。不常见的查询仍然可以通过索引来提供服务。这通常被称为常见查询的 **缓存（cache）**，尽管我们也可以称之为 **物化视图（materialized view）**，因为当新文档出现，且需要被包含在这些常见查询的搜索结果之中时，这些索引就需要更新。

但写还是需要更新缓存或者物化视图



#### 有状态、可离线的客户端

web是在线的应用，状态存储在服务器，但能离线使用的客户端或者移动端，状态是可以存储在本地的，在网络可用时与远程服务器再做同步。



#### 将状态变更推送给客户端

除了轮询，离线设备还可以订阅服务器的状态流。



#### 端到端的事件流

新的编程模型：有状态的客户端订阅状态状态变化，如react的redux。

从请求 / 响应交互转向发布 / 订阅数据流是一种对常规观念的挑战。



#### 读也是事件

读取也可以作为事件持久存储，好处是可以追踪整个系统中的因果关系与数据来源，但会产生额外的存储与IO成本。

#### 多分区数据处理

合并处理多个分区的数据。