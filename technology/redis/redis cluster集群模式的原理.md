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

redis cluster架构下，每个redis要放开两个端口号，一个6379和另外一个就是加10000的端口号，16379。16379端口号是用来进行节点间的通信的，也就是cluster bus的东西，集群总线。cluster bus用来进行故障检查，配置更新，故障转移授权。cluster bus用了另外一种二进制协议，主要用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。
## hash算法
在缓存这块很少使用hash算法，一般用于数据库分库分表。
![hash算法（最简单的数据分布算法）.png](https://upload-images.jianshu.io/upload_images/9905084-a38af88cba24cb8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
hash算法的问题：
- 某个master宕机了，所有的请求过来会基于最新的2个master去取模，尝试去取数据，导致所有的缓存都失效了
- 对于高并发场景来说，是不可以接受的。高并发场景下，1/3的数据不能走缓存全部走数据库，那么数据库就会被压垮，它最大的问题就是只要任意一个master宕机，那么大量的数据就需要重新计算写入缓存，风险很大。

## 一致性hash算法
有个key过来之后，计算它的hash值，然后用hash值在圆环对应的各个点上（每个点都有一个hash值）去比对，看hash值应该落在圆环的哪个部位，key落到圆环上之后，就会顺时针旋转去寻找距离自己最近的一个节点。
一致性hash算法相对于hash算法，只有1/n的缓存失效，需要重新查询数据库。
![一致性hash算法.png](https://upload-images.jianshu.io/upload_images/9905084-6a55f37f61a337f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一致性hash可能导致的问题：
- 缓存热点问题，可能集中在某个hash区间内的值特别多，会导致大量的数据都涌入同一个master内，造成该master热点问题，造成性能瓶颈

## 一致性hash算法（自动缓存迁移）+虚拟节点（自动负载均衡）
为了解决一致性hash导致的这种问题，引入了一致性hash算法（自动缓存迁移）+虚拟节点（自动负载均衡），给每个master都做了均匀分布的虚拟节点，这样在每个区间内，大量的数据，都会均匀地分布到不同的节点内，而不是按照顺时针的顺序去走，全部涌入同一个master内，造成缓存的热点问题。
![一致性hash算法（自动缓存迁移）+虚拟节点（自动负载均衡）.png](https://upload-images.jianshu.io/upload_images/9905084-53db6f9e90108b55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## redis cluster的hash slot算法
redis cluster有固定的16384个hash slot，对每个key计算CRC16值，然后对16384取模，可以获取key对应的hash slot，redis cluster中每个master都会持有部分slot，比如3个master，那么可能每个master持有5000多个hash slot，hash slot让node增加和移除很简单，增加一个master，就将其他master的hash slot移动部分过去，减少一个master，就将它的hash slot移动到其他master上去，移动hash slot的成本是非常低的。

# redis cluster VS replication + sentinal
replication: 一个master，多个slave，要几个slave跟你的要求的读吞吐量来决定，然后自己搭建一个sentinal集群，去保证redis主从架构的高可用性。
redis cluster：针对海量数据+高并发+高可用的场景，海量数据，如果你的数据量很大，那么建议用redis cluster。


