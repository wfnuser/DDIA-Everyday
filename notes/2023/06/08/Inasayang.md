拓扑结构

-   环形结构
    -   MySQL支持
    -   每个节点接收来自前序节点的写入，并把接收的加上自己的写入转发到后序节点
-   星型结构
    -   一个指定的根节点将写入转发到所有其他节点
        -   推广到树状结构
-   全链接结构（常见）
    -   每个主节点将写入同步到其他所有节点
-   环形和星型结构中，写请求需要经过多个节点，每个节点需要有唯一的标识符，并写到写请求中，否则会造成无限循环。
-   环形和星型结构中，一个节点故障，会影响转发。需要手动重新配置拓扑结构。所以全链接拓扑结构，避免了单点故障，容错性高。
-   全链接结构的问题，某些链路比其他链路更快，可能会造成先果后因（前缀一致读问题）,需要使用版本向量技术。







无主节点复制

-   放弃主节点，允许任何副本直接接受来自客户端的写请求
    -   方案A：客户端将写请求发送到多副本
    -   方案B：由一个协调者代表客户端写入，不维护写入顺序



节点失效时写入数据库

-   未全部写入成功，且未同步，可能读取到过期数据
-   并行发送读请求到多个副本（可以使用版本号判断新旧指）



读修复和反熵

-   Dynamo使用的两种机制
    -   读修复
        -   客户端并行读取多副本，检测过期值，并把新值写入旧副本
        -   适合频繁读取场景
        -   只有在读取时才能执行修复
    -   反熵过程
        -   后台进程不断查找并复制副本之间的差异
        -   不保证以特定的顺序写入
        -   明显的同步滞后



读写`quorum`（保证数据冗余和最终一致性的投票算法）

-   n个副本，需要w个节点写确认，至少查询r个节点，保证`w+r>n`，即可读取到最新值
-   通常，读取和写入总是并行发送到n个副本，w和r只是决定等待的节点数。





Pp. 166-171