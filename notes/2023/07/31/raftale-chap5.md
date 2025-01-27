## Replication

### 复制是什么

复制(Replication) 是分布式架构中多个节点上需要保留相同数据的副本。


### 为什么需要复制

为什么要通过复制保留副本？

1. 即使系统的部份节点出现故障，系统也能继续工作，增加可用性
    1. 还有异地灾备，多个副本可以最大可能保证数据不会丢失
2. 节点能够异地部署时，使数据离用户更近，减少延迟
3. 横向扩展提高节点数量，增加系统的读数据吞吐



### 复制有什么难点

#### 复制策略的差异

最简单的复制方式是这样的，每次写入数据时，就同步到所有的从库(slave)，确认同步完成后再返回给用户确认写入成功。这种复制方式称为全同步复制，但这样会有个问题：

1. 从库性能差处理慢，或者正在从故障中恢复，或者网络故障，就会导致响应特别长，因为写数据和复制是同时在生产上进行的，所以就会阻塞客户端的写入操作；

当然他的优点是一定能保证从库与主库有一致的数据备份。

与之相对的是「全异步复制」，每次写入，异步复制所有的从库，客户端不必等待从库响应，直接返回给用户确认写入成功。这样的优势是客户端的性能高，但也有问题：

1. 写入的数据不能保证同时在2个节点上，如果此时主库崩溃数据丢失了，数据就永远丢失了；

「全同步复制」和「全异步复制」的一个折中是「半同步复制」，只同步复制一半的节点，提高响应性能，至少保证一半的节点拥有最新的数据副本。


总结下复制的难点：

1. 复制方式影响系统对客户端的响应性能；
2. 复制方式影响客户端保存数据的安全性，全异步复制时，数据丢失可能性增加。
    1. 副本的数据不能与原始数据产生冲突，如新增的数据没有复制，更新的数据没有复制。


#### 新的节点如何加入从库

不停服加入：

1. 在某个时刻获取主库的一致性快照
2. 将快照复制到新的从库节点
3. 从库连接到主库，拉取快照之后的所有数据变更；
4. 继续处理主库产生的数据变化。



#### 节点宕机了怎么办

##### 从库失效，追赶恢复

如果是从库宕机，从库可以从日志中知道，在发生故障之前的最后一个事务，因此从库可以连接到主库，并请求在从库断开期间发生的所有数据变更。

##### 主库失效，故障切换

主库是负责写入的，主库挂了意味着写入的阻塞，因此需要有可用的从库升级为新的主库来保证客户端的写入不被中断。

故障切换可以手动进行（人工管理员手动处理），但更酷的应该是系统自动的故障切换。

一个完整的自动故障切换应该包含：

1. 确认当前主库真的失效了：一般经验化的选择一个超时时间，超过这个时间没有响应就认为它挂了
2. 选择一个新的主库：通过共识算法选择新的主库
3. 重新配置系统失效旧的主库，启用新的主库：修改配置，启用新主库，使接下来的写入请求打到新的主库中



故障切换不是一个简单的过程，很多地方都可能出错，比如：

1. 如果是异步复制，则新主库很可能没有收到宕机前的写入操作，当旧的主库上线成为从库时，两个节点数据产生了不一致。
2. 业务上的数据混乱：一个过时的MySQL从库被升级为主库，自增主键重用使得MySQL和Reids的数据不一致，导致用户数据混乱。 [github一场事故](https://github.blog/2012-09-14-github-availability-this-week/)
3. 脑裂
4. 超时时间的选择困难

