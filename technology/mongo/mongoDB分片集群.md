> 本文主要介绍了mongoDB分片集群概念，MongoDB分片集群搭建过程，。
# 概念
分片（sharding）是一种跨多台机器分布数据的方法，MongoDB使用分片来支持具有非常大的数据集和高吞吐量操作的部署。换句话说：分片就是将数据拆分，将其分散存在不同的机器上的过程，将数据分散到不同的机器上，不需要功能强大的大型计算机就可以存储更多的数据，处理更多的负载。
# 分片集群包含的组件
MongoDB分片集群包含以下组件：
- 分片（存储）：每个分片包含分片数据的子集，每个分片都可以部署为副本集
- mongos（路由）：mongos充当查询路由器，在客户端应用程序和分片集群之前提供接口
- config servers（“调度”的配置）：配置服务器存储集群的元数据和配置设置。从mongoDB 3.4开始，必须将配置服务器部署为副本集（CSRS）

下图描述了分片集群中组件的交互：
![mongoDB分片集群](https://upload-images.jianshu.io/upload_images/9905084-5a93f7dd7a2a3f4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 分片集群架构
本文搭建的副本集集群是两个分片节点副本集（3+3）+一个配置节点副本集（3）+两个路由节点（2），共11个服务节点，具体如下图所示：
![mongoDB分片集群架构](https://upload-images.jianshu.io/upload_images/9905084-53db2d9118f06ba2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

