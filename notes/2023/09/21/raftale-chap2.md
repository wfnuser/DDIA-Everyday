## x关系模型与文档模型



数据被组织成关系（表），其中每个关系是元祖（行）的无序集合。

### NoSQL

Not only SQL

### 对象关系不匹配

ORM框架可以减少模型转化层的代码。



JSON有时候更方便，可以直接将其保存到SQL中。

### 多对一和多对多的关系

存储文本字符串面临着 原始数据被修改导致多个副本都需要被修改的开销和数据不一致的风险，如果用ID连接就没有这一问题。



文档模型对多对多和多对一的关系支持非常弱，但关系型数据库的连接就支持。

### 文档数据库

其实最开始是IBM的信息管理系统IMS发明的层次模型，能支持一对多的关系，但很难应对多对多的关系，所以后来发明了关系模型和网状模型。
