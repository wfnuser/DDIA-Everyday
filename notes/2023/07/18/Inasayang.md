批处理输出键值

-   批处理的另一个常见用途：构建机器学习系统
    -   分类器：垃圾邮件过滤器，异常检测，图像识别
    -   推荐系统
-   批量作业的输出通常是某种数据库（数据库用于保存批处理结果）



对比`Hadoop`与分布式数据库

-   MPP数据库专注于在一个机器集群上并行执行SQL查询分析
-   MapReduce和分布式文件系统的结合是一个可以运行任意程序的通用操作系统



存储多样性

-   数据库要求特定的模型
    -   MPP数据库的谨慎设计模式减慢了集中式数据的收集速度
-   分布式文件系统的文件只是字节序列
    -   文本，图像，视频，传感器数据，稀疏矩阵，特征向量，基因组序列等等
    -   仅仅以原始形式收集数据，之后再考虑模式设计，从而加快收集速度（也称为数据湖，企业数据中心）
    -   以原始形式简单的转储数据可以进行多次转换，寿司原则
    -   Hadoop经常被用于实现ETL过程
        -   来自事务处理系统的数据以某种原始形式转储到分布式文件系统
        -   编写MapReduce作业进行数据清理
        -   转换为关系表单
        -   导入MPP数据仓库进行分析



Pp. 388-392