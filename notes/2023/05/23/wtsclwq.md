# 第一章

### 常见的数据系统

- 数据库：mysql、pg
- 高速缓存：redis
- 索引：ES
- 流处理系统：Flink
- 批处理系统：Hadoop
- 消息队列：RocketMQ、Kafka

#### 为什么统称为数据系统？

1. 界限逐渐模糊，一个系统有时能兼具多种功能
2. 经常被作为组件用来构建大型应用

### 可靠性

#### 什么样的系统是可靠的？

1. 能够提供正常的功能服务
2. 能容忍用户的错误操作
3. 在给定硬件和数据量下，能够满足承诺的性能指标
4. 防止未授权的操作

#### 区分Fault和Failure

1. Fault可以认为是错误或者故障，通常是某些组件出现异常（对应fault-tolerant），很多厂商强调的**弹性**应该就是能高效处理这种问题
2. Failure则是系统完全失效，无法为用户提供服务

> 我们不太可能将故障概率降低到零, 因此通常设计容错机制来避免从故障引发系统失效。 

#### 常见的Fault类型

1. 硬件故障

   强调随机性，无法预测某个硬件是否会崩溃。

   最重要的是不要损坏数据，单机可以用RAID做磁盘冗余，多机用多副本

2. 软件错误

   强调相关性，某个进程或者组件的问题可能会引发级联故障

   > 软件系统问题有时没有快速解决办法, 而只能仔细考虑很多细节

3. 人为错误

   强调预防，在设计层面尽可能消除人对系统影响

#### 可靠性的重要性

事关用户数据安全，事关企业声誉，企业存活和做大的基石。要不然为什么这么喜欢问Raft？

### 可扩展性

> 可伸缩性，即系统应对负载增长的能力。它很重要，但在实践中又很难做好，因为存在一个基本矛盾：**只有能存活下来的产品才有资格谈伸缩，而过早为伸缩设计往往活不下去**。

#### 如何衡量系统负载

> 负载可以用称为负载参数的若干数字来描述。 参数的最佳选择取决于系统的体系结构, 它可能是Web服务器的每秒请求处理次数, 数据库中写入的比例, 聊天室的同时活动用户数量, 缓存命中率等。 有时平均值很重要, 有时系统瓶颈来自于少数峰值。

在DB中就是TPS、QPS、Cache命中率、磁盘IO吧

#### 如何描述系统性能

> 注意和系统负载区分，系统负载是从用户视角来审视系统，是一种**客观指标**。而系统性能则是描述的系统的一种**实际能力**。

1. **吞吐量（throughput）**：每秒可以处理的单位数据量，通常记为 QPS。

2. **响应时间（response time）**：从用户侧观察到的发出请求到收到回复的时间.

    通常以百分位点来衡量，比如 p95，p99 和 p999，它们意味着 95％，99％或 99.9％ 的请求都能在该阈值内完成。比如`P99=1.5s`就代表100个请求中有95个请求的响应时间小于1.5s，剩余的5个则大于1.5s。

3. **延迟（latency）**：日常中，延迟经常和响应时间混用指代响应时间；但严格来说， 只有我们把请求真正处理耗时认为是瞬时，延迟才能等同于响应时间。

    ```tex
    一个请求从t=0时刻开始
    花了1秒时间到达服务端（t=1）
    服务端花了2秒时间进行处理（t=3）
    最后花了1秒时间到达客户端（t=4）
    ```

    延时（Latency）就是2秒。

    响应时间为4秒。

#### 负载增加时，如何保持系统的良好性能

也就是如何保证系统的可拓展性

2023.5.23 未完待续...