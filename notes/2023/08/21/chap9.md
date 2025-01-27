### 线性一致化在什么时候有用？



#### 锁定与领导选举

Apache Zookeeper 和 etcd之类的协调服务通常用于实现分布式锁和领导者选举，它们使用一致性算法，以容错的方式实现线性一致性的操作；

#### 约束和唯一性保证

用户名或电子邮件必须标识唯一用户；

文件存储服务中，不能有两个具有相同路径和文件名的文件；

如果要在写入数据时强行执行此约束，则需要线性一致性。

#### 跨信道的时序依赖

如果没有线性一致性，外部信道可能会造成很多问题。但线性一致性不是解决这些问题的唯一方法，如果可以控制额外信道，可以使用读已之写。



### 实现线性一致性的系统

需要共识算法，zookeeper和etcd就利用了这一点。



### 线性一致性的代价

网络中断迫使在线性一致性和可用性之间做出选择。

#### CAP定理

已经没有任何帮助的定理

#### 线性一致性和网络延迟

如果你想要线性一致性，读写请求的响应时间至少与网络延迟的不确定性成正比。

更快的线性一致性算法不存在，但更弱的一致性模型可以快得多，权衡很重要。



## 顺序保证

线性一致性的行为就像是只有单个数据副本一样，且每个操作似乎都是在某个时间点以原子性的方式生效的。

### 顺序与因果关系

顺序有助于保持因果关系

1. 一致前缀读：修复复制延迟引起的违反因果性
2. 因为复制延迟更新不存在的记录
3. happens before 就是因果关系的另一种表述
4. 快照隔离中的上下文中，事务时从一致性快照读取的，一致性的意思就是与因果关系保持一致：

1. 1. 如果快照包含答案，它也必须包含被回答的问题；
   2. 因果上在该时间点之前发生的所有操作，其影响都是可见的，但因果上在该时间点之后发生的操作，其影响对观察者不可见；
   3. 不可重复读就是违反了因果关系

1. 写偏差（幻读），可串行化快照隔离跟踪事务之间的因果依赖来检测写偏差；
2. 跨信道的时序可能会造成违反因果性



因果关系对事件施加了一种顺序。



#### 因果顺序不是全序的

全序是指集合中任意两个元素进行比较，都是可以比较顺序的。

偏序(partially order)是指：集合中的元素有时可以比较的，有时是无法比较的。

全序和偏序的差异体现在不同的数据库一致性模型中：

1. 线性一致性：根据线性一致性的定义，它必然是全序的；
2. 因果一致性：如果两个事件有happens before 关系，则它们是有因果关系的；如果两个事件是并发的，则它们的顺序无法比较；所以因果一致性是偏序的。







#### 线性一致性强于因果一致性

线性一致性隐含着因果关系，但线性一致性并不是保持因果关系的唯一途径。许多情况下，看上去需要线性一致性的系统，实际上只是需要因果一致性，因果一致性可以更高效的实现。



#### 捕获因果关系

实现因果性的一个关键的思想，就是要捕获因果关系。

需要跟踪整个数据库的因果依赖，向量版本一种解决方法；

为了确定因果关系，数据库需要知道应用读取了哪个版本的数据，这也是可串行化快照隔离中，事务提交时要检测它所读取的数据版本是否是最新的。





### 序列号顺序

实际上跟踪所有的因果关系是不切实际的，在许多应用中，客户端在写入内容之前会先读取大量数据，我们无法弄清写入因果依赖于先前全部的读取内容，还是仅包括其中一部分。显式跟踪所有已读数据意味着巨大的额外开销。

但我们可以用序列号 和 时间戳来排序事件，时间戳可以是一个逻辑时钟，比如自增的计数器。

序列号和时间戳提供了一个全序的关系，确定因果则用happens before关系，**并发则随机选一个关系。**

单主复制中，复制日志定义了与因果关系一致的写操作，主库可以简单的为每个操作自增一个计数器，从而为复制日志中的每个操作分配一个单调递增的序列号。如果一个从库按照它们在复制日志中出现的顺序来应用写操作，那么从库的状态始终是因果一致的。
