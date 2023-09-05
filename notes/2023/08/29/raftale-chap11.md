从高层次上看，存储和处理数据的系统可以分为两大类：

- 记录系统：持有数据的权威版本，当新的数据进入时，用户首先会记录在这里。
- 衍生数据系统：通常是另一个系统的现有数据以某种方式进行转换或处理的结果。如果丢失了衍生数据，可以从原始来源进行重建。典型的例子就是缓存系统。



衍生数据是冗余的。



三种不同类型的系统：

1. 服务（在线系统）：响应时间、可用性非常重要
2. 批处理系统（离线系统）：主要性能衡量标准为吞吐量
3. 流处理系统（准实时系统）：介于在线和离线之间，准在线系统，事件发生后不久就会对事件进行操作。



流处理背后的思想是 批处理时间太长，为了减少延迟，我们可以更频繁地运行处理，比如在每秒钟的末尾、甚至事件发生后就立刻进行处理。



本章中，我们将把事件流（event stream）视为一种数据管理机制：无界限、增量处理。



## 传递事件流

一个事件由生产者生成一次，然后可能由多个消费者进行处理。

1. 生产者：producer，publisher、sender
2. 消费者：consumer、subscribers

相关的事件通常被聚合为一个主题或流，



### 消息传递系统

向消费者通知新事件的常用方式是使用消息传递系统：生产者发送包含事件的消息，然后将消息推送给消费者。



1. 生产者发送消息的速度比消费者能够处理的速度快会发生什么？一般有三种处理方式

1. 1. 系统丢掉消息；
   2. 将消息放入缓冲队列：
   3. 背压backpressure，流量控制，阻塞生产者以免发送更多消息：Unix管道和TCP就使用了背压

1. 如果节点崩溃或暂时脱机，会发生什么情况？是否会有消息丢失？

1. 1. 持久性可能需要写入磁盘和/或复制的某种组合，但这是有代价的，如果你能接受有时消息会丢失，则可能在同一硬件上能获得更高的吞吐量和更低的延迟。

是否能接受消息却决于应用。



### 直接从生产者传递给消费者

许多消息传递系统使用生产者和消费者之间的直接网络通信，而不是通过中间节点。

- UDP组播广泛用于金融行业，例如股票市场，其中低时延非常重要；虽然UDP本身是不可靠的，但应用层的协议可以恢复丢包的数据包（生产者必须记住它发送的数据包，以便能按需重新发送数据包）
- 无代理的消息库
- 如果消费者在网络上公开了服务，生产者可以直接发送HTTP或RPC请求，将消息推送给使用者。这就是webhooks背后的想法，一种服务回调的url被注册到另一个服务中，并且每当事件发生时就会向该URL发出请求。

- - webhooks：微信后端推送消息给客户端、git push时构建应用



这种方式在消费者处于脱机状态，可能会丢失其不可达时发送的消息。