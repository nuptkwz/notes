> 本文主要介绍了ES集群相关的内容

# 集群健康
## 集群状态
1.查看集群状态
```
GET /_cluster/health
```

2.群中status字段的状态
- green
所有的主分片和副本分片都正常运行
- yellow
所有的主分片都正常运行，但不是所有的副本分片都正常运行
- red
有主分片没能正常运行

3.快速查看集群中有哪些索引
```
GET /_cat/indices?v
```

## es的节点
启动了一个es进程，相当于就只有一个node。现在es中有一个index，就是kibana自己内置建立的index。
由于默认的配置是给每个index分配5个primary shard和5个replica shard，而且primary shard和
replica shard不能在同一台机器上（为了容错）。现在kibana自己建立的index是1个primary shard
和1个replica shard。当前就一个node，所以只有1个primary shard被分配了和启动了，但是一个
replica shard没有第二台机器去启动。

### es集群状态实验
做一个小实验：此时只要启动第二个es进程，就会在es集群中有2个node，然后那1个replica shard就会自动分配过去，
然后cluster status就会变成green状态。

# 添加索引
一个分片是一个 Lucene 的实例，它本身就是一个完整的搜索引擎。一个分片可以是主分片或者副本分片。索引内任意一个文档都归属于一个主分片，
所以主分片的数目决定着索引能够保存的最大数据量。一个分片可以是主分片或者副本分片。 索引内任意一个文档都归属于一个主分片，所以主分片
的数目决定着索引能够保存的最大数据量。在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。

# es扩容
## 水平扩容
Node 1 和 Node 2 上各有一个分片被迁移到了新的 Node 3 节点，现在每个节点上都拥有2个分片，而不是之前的3个。 这表示每个节点的硬件资源
（CPU, RAM, I/O）将被更少的分片所共享，每个分片的性能将会得到提升。
![拥有三个节点的集群——为了分散负载而对分片进行重新分配](https://upload-images.jianshu.io/upload_images/9905084-4bd1f99cc061673d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)







