# 前言
> 本文主要介绍了MongoDB的聚合操作

# 聚合操作
聚合操作可以处理记录数并返回计算结果，如统计总数、平均值、求和等（类似于MySQL里面的Sum、Count、Group By等）。
聚合操作组值来自于多个文档，可以对分组数据执行各种操作，返回单个结果。MongoDB中的聚合操作主要分为3类：
- 单一作用聚合：
从单个集合聚合文档，提供了常见聚合过程的简单访问

- 管道聚合：
是一个数据聚合的框架，模型是基于数据处理流水线的概念。文档进入多级管道，转换成最后的聚合结果。

- MapReduce：
它具有两个阶段，Map阶段：处理每个文档并向每个输入文档发射一个或者多个对象，reduce阶段：组合map操作的输出阶段

## 单一作用聚合(Single Purpose Aggregation)
单一目的聚合方法聚合来自单个集合的文档。这些方法很简单，但缺乏聚合管道的功能。
主要方法有：
- db.collection.estimatedDocumentCount()
- db.collection.count()
- db.collection.distinct()

## 聚合管道(Aggregation Pipeline)




# 参考
1. https://www.mongodb.com/docs/v5.0/aggregation/
2. https://itcn.blog/p/56326157.html



