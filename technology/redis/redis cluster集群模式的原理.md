# 学习redis cluster集群模式之前需要思考的问题?
- redis集群模式的工作原理是什么？
- 在集群模式下，redis的key是如何寻址的？
- 分布式寻址都有哪些算法？
- 了解一致性hash算法吗？
# redis cluster集群架构的引入
## 单机小数据量的Redis架构
如下为redis一主一从一哨兵架构，这种架构的特点是：
 1. master节点的数据和slave节点的数据一模一样
 2. master最大能容纳多大的数据量，那么slave也就只能容纳多大的数据量
![redis一主一从一哨兵架构.png](https://upload-images.jianshu.io/upload_images/9905084-48e298f877e62143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


