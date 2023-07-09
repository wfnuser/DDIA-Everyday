### 单对象和多对象操作

之前的ACID主要讨论的是多对象操作，也就是一个事务会同时对多个数据库对象进行操作，ACID保证了多个事务对多个对象进行写入操作时的原子性、隔离性、持久性。

如何确保多个操作数据同一事务：

1. 事务的开启和结束需要BEGIN和COMMIT语句来标示，我们可以假定这两个关键字之间的所有内容属于一个事务。但是有时因为网络故障，会导致事务明明执行完了却没法通知用户。
2. 为事务分配一个ID

### 单对象写入

虽然单对象写入的粒度很小，但是仍然无法和编程语言中的原子操作比拟，一次数据库的写操作几乎必然存在着2次磁盘读写，因此这对于CPU和网络来说是一个很长时间的操作，出现意外在所难免。

比如：写入一个20KB的文档，但是写入到一半网络中断，我们根据原子性，应该认为数据库会将已经写入的一半舍弃。

> 严格地说，**原子自增（atomic increment）** 这个术语在多线程编程的意义上使用了原子这个词。 在 ACID 的情况下，它实际上应该被称为 **隔离的（isolated）** 的或 **可串行的（serializable）** 的增量。 

根据我的理解，其实原子自增，就算在编程语言中，也是和隔离性挂钩的，毕竟我们的需求是其他线程看不到该数据更改的中间状态。

### 多对象事务的需求

很多NoSQL都舍弃了事务的支持是，但是我们真的能够将所有的操作都用KV模型和单对象操作完成吗？

1. 外键更新
2. 更新非规范化数据（关系表）
3. 更新多个索引（二级索引）

可以发现，上述的情景还是更加适合事务模型来处理，尤其是原子性和隔离性。