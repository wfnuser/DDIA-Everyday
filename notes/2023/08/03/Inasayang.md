数据系统的未来

数据集成

采用派生数据来组合工具

Lambda架构

-   进来的数据以不可变事件形式追加写到不断增长的数据集
-   基于这些总事件，可以派生出读优化的视图
-   建议运行两个不同的系统
    -   一个批处理系统
        -   使用较慢的精确算法
        -   简单，不易出错
        -   处理同一组事件并产生派生视图的校正版本
    -   一个单独的流处理系统
        -   使用快速的近似算法
        -   难以实现容错
        -   快速产生对视图的近似更新



分拆数据库

有状态，可离线客户端

状态更改推送至客户端

审计数据系统的工具

-   密码审计和完整性检查通常依赖于Merkle树，哈希散列树，有效的证明某个数据集和其他一些数据集中出现的记录
-   证书透明度也是依赖Merkle树来检查TLS证书的



Pp. 460-