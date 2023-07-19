## 分布式事务和共识协议

前面很多内容都是为了这里做铺垫，包括复制、线性一致性等。

### 共识的重要性

1. Leader选举避免脑裂，从而导致不一致
2. 原子提交，避免某些事务在部分节点提交，部分节点回滚

### 原子提交

单机系统用了WAL（undo）保证写入要么成功要么失败（会滚），但是在分布式情景下，我们不能保证所有节点的本地提交都成功或者全都回滚

不能做这种假设：

1. 向所有节点发送一个事务，让他们在本地完成并提交
2. 检查所有节点的事务完成情况，如果有节点没能提交成功，就再发送一个回滚指令

这种做法是不负责任的，我们应当认为在本地提交之后，事务的影响就是永久性的，即使能够完全回滚该操作也会违反一致性。

也许确实不会造成什么影响，但是这违反数据库隔离性的基本理念。

> **事务提交后是不可撤销的**——在事务提交后，你不能再改变主意说，我要重新中止这个事务。这是因为，一旦事务提交了，就会对其他事务可见，从而可能让其他事务依赖于该事务的结果做出一些新的决策

### 2PC

看图就够了

![Untitled](https://wtsclwq.oss-cn-beijing.aliyuncs.com/ch09-fig09.png)

关键概念：协调者（coordinator/transcation manager）

两阶段：

1. PreWrite：协调者向所有参与者发送事务的内容，参与者在本地执行，但不提交，协调者接受参与者的返回信息（成功与否）做下一步决策
2. Commit：PreWrite所有参与者都成功-> 向所有参与者发送Commit指令，有任何一个失败->向所有参与者发送Rollback指令