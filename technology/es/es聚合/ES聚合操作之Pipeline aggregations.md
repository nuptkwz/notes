# 0. 前言
>本文主要介绍了es聚合操作中的pipeline aggregations,pipeline aggregations又称为管道聚合，
 其作用是产生的输出结果而不是文档集，用于向输出树添加信息。
# pipeline聚合的例子
1. 通过cardinality进行去重
es中的cardinality metric是对每个bucket中的指定的field进行去重，得到去重后的count，类似于MySQL
中的count(distinct)



