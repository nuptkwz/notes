> 本文主要介绍了mongoDB分片集群概念，以及分片集群搭建过程，方便下次参考。
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
# 分片集群搭建
## 第一套副本集
本次相关的配置文件、数据、日志都放在sharded_cluster相应的子目录下面
### 新建副本集数据和日志的目录
myshardrs01
```
mkdir -p /mongodb/sharded_cluster/myshardrs01_27018/log \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27018/data/db \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27118/log \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27118/data/db \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27218/log \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27218/data/db
```
### 新建或修改配置文件
myshardrs01_27018
```
vim /mongodb/sharded_cluster/myshardrs01_27018/mongod.conf
```
```
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/sharded_cluster/myshardrs01_27018/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
  dbPath: "/mongodb/sharded_cluster/myshardrs01_27018/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/sharded_cluster/myshardrs01_27018/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.30.129
  #bindIp
  #绑定的端口
  port: 27018
replication:
  #副本集的名称
  replSetName: myshardrs01
sharding:
  #分片角色
  clusterRole: shardsvr
```











