原子写操作

-   避免应用层完成“读-修改-写回”操作
-   通常使用对读取对象加独占锁来实现（又称游标稳定性）
-   或者强制所有的原子操作都在单线程中执行



显示加锁

-   有应用层显示锁定待更新的对象



自动检测跟新丢失

-   原子操作和锁都是强制串行化执行来避免
-   先并发执行，如果事务管理器检测到丢失风险，则中止，并强制回退到“读-修改-写回”方式
-   可以借助快照级别隔离来检查



冲突解决与复制

-   多节点上的副本，不同节点可能会并发修改数据
-   原子操作和锁的前提是只有一个最新的副本
-   如果操作可交换，则原子操作可以
-   如果一个值被不同的客户端同时更新，Riak自动将更新合并在一，避免发生丢失



写倾斜和幻读

-   举例，A，B在医院值班，医院必须保证至少有一名医生值班。此时数据库正在使用快照级别隔离，两个检查都返回两个医生；A更新自己为离开，B更新自己为离开，事务提交成功，
-   写倾斜；两笔事务更新的是两个不同的对象
    -   两个事务读取相同的一组对象，再更新其中一部分
    -   不同的事务更新不同的对象，则可能发生写倾斜
    -   不同的事务更新同一个对象，则可能发生脏写或更新丢失
-   幻读：一个事务的写入改变了另一个事务查询的结果
    -   快照级别隔离可以避免只读查询时的幻读
-   实体化冲突：少数情况下使用



Pp. 230-237