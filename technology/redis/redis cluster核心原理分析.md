> 本文主要分析了redis cluster的核心原理，gossip通信，jedis smart定位，主备切换。
# 节点间的通信机制
## 基础通信原理
1.redis cluster节点间采用gossip协议进行通信
  根集中式不同，不是将集群元数据（节点信息，故障等等）集中存储在某个节点上，而是互相之间不断通信，保持个集群所有节点的数据是完整的
  ![集中式的元数据集群存储和维护.png](https://upload-images.jianshu.io/upload_images/9905084-f96e7bcee01ef1ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![gossip协议维护集群元数据.png](https://upload-images.jianshu.io/upload_images/9905084-fbb19615a10e65e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

集中式和gossip协议维护集群元数据对比：
**集中式**：
- 好处在于元数据的更新和读取，时效性非常好，一旦元数据出现了变更，立即就更新到集中式的存储中，其他节点读取的时候立即就可以感知到
- 不好在于，所有元数据的更新压力全部集中在一个地方，可能会导致元数据的存储有压力
**gossip**:
2.10000端口号
  每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，比如1端口号，节点间通信的端口号就是10001。

3.交换的信息
  节点间交换了故障信息，节点的增加和移除，hash slot信息，等等

## gossip协议
  gossip协议包含多种消息，包括ping，pong，meet，fail等等。
  meet: 某个节点发送meet给新加入的节点，让新节点加入集群中，然后新节点就会开始与其他节点进行通信
  ping: 每个节点都会频繁给其他节点发送ping，其中包含自己的状态还有自己维护的集群元数据，互相通过ping
        交换元数据
  fail: 某个节点判断另一个节点fail之后，就发送fail给其他节点，通知其他节点，指定的节点宕机了

## ping消息探入
    ping很频繁，而且要携带一些元数据，所以可能会加重网络负担
    每个节点每秒会执行10次ping，每次会选择5个最久没有通信的其他节点，当然如果发现某个节点通信延迟达到了cluster_node_timeout/2，那么
    立即发送ping，避免数据交换延时过长，所以cluster_node_timeout可以调节，如果调节比较大，那么会降低发送的频率
##