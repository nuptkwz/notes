# 学习redis cluster集群模式之前需要思考的问题?
- redis集群模式的工作原理是什么？
- 在集群模式下，redis的key是如何寻址的？
- 分布式寻址都有哪些算法？
- 了解一致性hash算法吗？
# redis cluster集群架构的引入
## 单机Redis在海量数据面前的瓶颈
如下为redis一主一从一哨兵架构，这种架构的特点是：
 1. master节点的数据和slave节点的数据一模一样
 2. master最大能容纳多大的数据量，那么slave也就只能容纳多大的数据量
![redis一主一从一哨兵架构.png](https://upload-images.jianshu.io/upload_images/9905084-48e298f877e62143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如上图所示redis一主一从一哨兵架构，这种架构数据量少还行，如果数据量大了，则redis存不下了，比如Master最大内存8G，超过8G则使用LRU算法进行清除（将旧的很少使用的数据给清除出内存，然后保证内存中，就只有固定大小的内存，不可能超过master内存的物理内存）。如果要存放128G的物理内存该怎么办呢？
## 突破单机瓶颈，让redis支撑海量数据
将redis横向扩容用于支撑更大数据量的缓存，那就横向扩容更多的master节点，每个master节点存放更多的数据，单台机器是8G，16台机器就可以达到128G。
## redis的集群架构--redis cluster
redis cluster支撑n个redis master node，每个master node都可以挂载多个slave node,读写分离的架构，对于每个master来说，写就写到master，读就从该master对应的slave去读。
高可用，每个master都有一个slave节点，如果master挂掉，redis cluster这套机制就会自动将某个slave切换成master。
redis cluster（多master+读写分离+高可用）
# redis cluster的介绍
redis cluster的特点：
- 自动将数据进行分片，每个master上放一部分数据
- 提供内置的高可用支持，部分master不可用时，还可以继续工作
# redis cluster VS replication + sentinal
replication: 一个master，多个slave，要几个slave跟你的要求的读吞吐量来决定，然后自己搭建一个sentinal集群，去保证redis主从架构的高可用性。
redis cluster：针对海量数据+高并发+高可用的场景，海量数据，如果你的数据量很大，那么建议用redis cluster。


