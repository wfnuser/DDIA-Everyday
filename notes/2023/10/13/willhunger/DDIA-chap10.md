##### Broadcast hash joins
Broadcast hash joins 适用于执行 Map 端连接的最简单场景是大数据集与小数据集连接的情况。要点在于小数据集需要足够小，以便可以将其全部加载到每个 Mapper 的内存中。

**广播散列连接（broadcast hash join）**：**广播** 一词反映了这样一个事实，每个连接较大输入端分区的 Mapper 都会将较小输入端数据集整个读入内存中（所以较小输入实际上 “广播” 到较大数据的所有分区上），**散列** 一词反映了它使用一个散列表。 Pig（名为 “**复制链接（replicated join）**”），Hive（“**MapJoin**”），Cascading 和 Crunch 支持这种连接。它也被诸如 Impala 的数据仓库查询引擎使用。

除了将较小的连接输入加载到内存散列表中，另一种方法是将较小输入存储在本地磁盘上的只读索引中。索引中经常使用的部分将保留在操作系统的页面缓存中，因而这种方法可以提供与内存散列表几乎一样快的随机查找性能，但实际上并不需要数据集能放入内存中。