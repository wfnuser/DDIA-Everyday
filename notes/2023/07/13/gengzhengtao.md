P316-P321
###线性化的代价

CAP理论，
不要求线性化的应用更能容忍网络故障，这种思路通常被称为CAP定理，
很多分布式数据库他们选择不支持线性化是为了提高性能，而不是为了保住容错特性。无论是否发生了网络故障，
线性化对性能的影响都是巨大的，

### 顺序保证，

顺序与因果关系
因果顺序并非全序