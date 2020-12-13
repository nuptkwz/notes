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
本次搭建一主一副本一仲裁，相关的配置文件、数据、日志都放在sharded_cluster相应的子目录下面，具体步骤如下：
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
#### myshardrs01_27018
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
设置sharding.clusterRole需要mongod实例运行复制。 要将实例部署为副本集成员，请使用
replSetName设置并指定副本集的名称。
#### myshardrs01_27118
```
vim /mongodb/sharded_cluster/myshardrs01_27118/mongod.conf
```
```
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/sharded_cluster/myshardrs01_27118/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
  dbPath: "/mongodb/sharded_cluster/myshardrs01_27118/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/sharded_cluster/myshardrs01_27118/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.30.129
  #bindIp
  #绑定的端口
  port: 27118
replication:
  #副本集的名称
  replSetName: myshardrs01
sharding:
  #分片角色
  clusterRole: shardsvr
```
#### myshardrs01_27218
```
vim /mongodb/sharded_cluster/myshardrs01_27218/mongod.conf
```
```
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/sharded_cluster/myshardrs01_27218/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
  dbPath: "/mongodb/sharded_cluster/myshardrs01_27218/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/sharded_cluster/myshardrs01_27218/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.30.129
  #bindIp
  #绑定的端口
  port: 27218
replication:
  #副本集的名称
  replSetName: myshardrs01
sharding:
  #分片角色
  clusterRole: shardsvr
```
### 依次启动三个mongod服务
```
root@keweizhou-virtual-machine:~# /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_270
 18/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 5773
child process started successfully, parent exiting
root@keweizhou-virtual-machine:~# /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_271
 18/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 5810
child process started successfully, parent exiting
root@keweizhou-virtual-machine:~# /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_272
 18/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 5849
child process started successfully, parent exiting
```
### 查看服务是否启动
```
root@keweizhou-virtual-machine:~# ps -ef | grep mongo
root       5773      1  2 00:37 ?        00:00:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27018/mongod.conf
root       5810      1  2 00:37 ?        00:00:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27118/mongod.conf
root       5849      1  2 00:38 ?        00:00:03 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27218/mongod.conf
root       5903   5729  0 00:40 pts/19   00:00:00 grep --color=auto mongo
```
### 初始化副本集
#### 初始化副本集和创建主节点
使用客户端命令连接主节点，这里最好连接主节点
```
/usr/local/mongodb/bin/mongo --host 192.168.30.129 --port 27018
```
执行初始化副本集命令：
```
> rs.initiate()
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "192.168.30.129:27018",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607273363, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607273363, 1)
}
```
查看副本集情况：
```
myshardrs01:PRIMARY> rs.status()
{
        "set" : "myshardrs01",
        "date" : ISODate("2020-12-06T16:51:22.235Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 1,
        "writeMajorityCount" : 1,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1607273474, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2020-12-06T16:51:14.010Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1607273474, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityWallTime" : ISODate("2020-12-06T16:51:14.010Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1607273474, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1607273474, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2020-12-06T16:51:14.010Z"),
                "lastDurableWallTime" : ISODate("2020-12-06T16:51:14.010Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1607273413, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1607273413, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-12-06T16:49:23.955Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1607273363, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2020-12-06T16:49:23.989Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-12-06T16:49:24.076Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "192.168.30.129:27018",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 846,
                        "optime" : {
                                "ts" : Timestamp(1607273474, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T16:51:14Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1607273363, 2),
                        "electionDate" : ISODate("2020-12-06T16:49:23Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607273474, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607273474, 1)
}
```
#### 主节点配置查看
```
myshardrs01:PRIMARY> rs.conf()
{
        "_id" : "myshardrs01",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "192.168.30.129:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5fcd0b93ae50cdd5f8dd65a8")
        }
}
```
#### 添加副本节点
```
myshardrs01:PRIMARY> rs.add("192.168.30.129:27118")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607273787, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607273787, 1)
}
```
#### 添加仲裁节点
```
myshardrs01:PRIMARY> rs.addArb("192.168.30.129:27218")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607273887, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607273887, 1)
}
```
#### 查看副本集的配置情况
```
myshardrs01:PRIMARY> rs.conf()
{
        "_id" : "myshardrs01",
        "version" : 3,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "192.168.30.129:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "192.168.30.129:27118",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "192.168.30.129:27218",
                        "arbiterOnly" : true,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 0,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5fcd0b93ae50cdd5f8dd65a8")
        }
}
```
## 第二套副本集
同样搭建一主一副本一仲裁，相关的配置文件、数据、日志都放在sharded_cluster相应的子目录下面，
具体步骤如下：
### 新建副本集数据和日志的目录
myshardrs02
```
mkdir -p /mongodb/sharded_cluster/myshardrs01_27318/log \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27318/data/db \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27418/log \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27418/data/db \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27518/log \ &
mkdir -p /mongodb/sharded_cluster/myshardrs01_27518/data/db
```
### 新建或修改配置文件
myshardrs01_27318
```
vim /mongodb/sharded_cluster/myshardrs02_27318/mongod.conf
```
```
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/sharded_cluster/myshardrs02_27318/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
  dbPath: "/mongodb/sharded_cluster/myshardrs02_27318/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/sharded_cluster/myshardrs02_27318/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.30.129
  #bindIp
  #绑定的端口
  port: 27318
replication:
  #副本集的名称
  replSetName: myshardrs02
sharding:
  #分片角色
  clusterRole: shardsv2
```
设置sharding.clusterRole需要mongod实例运行复制。 要将实例部署为副本集成员，请使用
replSetName设置并指定副本集的名称

myshardrs01_27418
```
vim /mongodb/sharded_cluster/myshardrs02_27418/mongod.conf
```
```
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/sharded_cluster/myshardrs02_27418/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
  dbPath: "/mongodb/sharded_cluster/myshardrs02_27418/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/sharded_cluster/myshardrs02_27418/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.30.129
  #bindIp
  #绑定的端口
  port: 27418
replication:
  #副本集的名称
  replSetName: myshardrs02
sharding:
  #分片角色
  clusterRole: shardsv2
```

myshardrs01_27518
```
vim /mongodb/sharded_cluster/myshardrs02_27518/mongod.conf
```
```
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/sharded_cluster/myshardrs02_27518/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
  dbPath: "/mongodb/sharded_cluster/myshardrs02_27518/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/sharded_cluster/myshardrs02_27518/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.30.129
  #bindIp
  #绑定的端口
  port: 27518
replication:
  #副本集的名称
  replSetName: myshardrs02
sharding:
  #分片角色
  clusterRole: shardsv2
```
启动第二套副本集：一主一副本一仲裁

依次启动三个mongod服务：
```
root@keweizhou-virtual-machine:/# /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27318/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 8814
child process started successfully, parent exiting
root@keweizhou-virtual-machine:/# /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27418/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 8853
child process started successfully, parent exiting
root@keweizhou-virtual-machine:/# /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27518/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 8891
child process started successfully, parent exiting
```
查看服务是否启动：
```
root@keweizhou-virtual-machine:/# ps -ef | grep mongod
root       5773      1  1 00:37 ?        00:02:41 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27018/mongod.conf
root       5810      1  1 00:37 ?        00:02:36 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27118/mongod.conf
root       5849      1  1 00:38 ?        00:01:58 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27218/mongod.conf
root       8814      1  3 03:09 ?        00:00:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27318/mongod.conf
root       8853      1  3 03:10 ?        00:00:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27418/mongod.conf
root       8891      1  3 03:10 ?        00:00:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27518/mongod.conf
root       8933   8753  0 03:12 pts/11   00:00:00 grep --color=auto mongod
```
### 初始化副本集
#### 初始化副本集和创建主节点
* 使用客户端命令连接主节点
```
/usr/local/mongodb/bin/mongo --host localhost --port 27318
```
* 执行初始化副本集命令,查看副本集情况：
```
 > rs.initiate()
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "192.168.30.129:27318",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607283102, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607283102, 1)
}
myshardrs02:SECONDARY> rs.status()
{
        "set" : "myshardrs02",
        "date" : ISODate("2020-12-06T19:32:26.089Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 1,
        "writeMajorityCount" : 1,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1607283142, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2020-12-06T19:32:22.128Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1607283142, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityWallTime" : ISODate("2020-12-06T19:32:22.128Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1607283142, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1607283142, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2020-12-06T19:32:22.128Z"),
                "lastDurableWallTime" : ISODate("2020-12-06T19:32:22.128Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1607283102, 4),
        "lastStableCheckpointTimestamp" : Timestamp(1607283102, 4),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-12-06T19:31:42.101Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1607283102, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2020-12-06T19:31:42.124Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-12-06T19:31:42.127Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "192.168.30.129:27318",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1366,
                        "optime" : {
                                "ts" : Timestamp(1607283142, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T19:32:22Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1607283102, 2),
                        "electionDate" : ISODate("2020-12-06T19:31:42Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607283142, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607283142, 1)
}
```
* 主节点配置查看
```
myshardrs02:PRIMARY> rs.conf()
{
        "_id" : "myshardrs02",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "192.168.30.129:27318",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5fcd319d2790e96839e1982f")
        }
}
```
* 添加副本节点
```
myshardrs02:PRIMARY> rs.add("192.168.30.129:27418")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607283321, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607283321, 1)
}
```
* 添加仲裁节点
```
myshardrs02:PRIMARY> rs.addArb("192.168.30.129:27518")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607283403, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607283403, 1)
}
```
* 查看副本集的配置情况
```
myshardrs02:PRIMARY> rs.conf()
{
        "_id" : "myshardrs02",
        "version" : 3,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "192.168.30.129:27318",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "192.168.30.129:27418",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "192.168.30.129:27518",
                        "arbiterOnly" : true,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 0,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5fcd319d2790e96839e1982f")
        }
}
myshardrs02:PRIMARY> rs.status()
{
        "set" : "myshardrs02",
        "date" : ISODate("2020-12-06T19:38:11.227Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1607283482, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2020-12-06T19:38:02.401Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1607283482, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityWallTime" : ISODate("2020-12-06T19:38:02.401Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1607283482, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1607283482, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2020-12-06T19:38:02.401Z"),
                "lastDurableWallTime" : ISODate("2020-12-06T19:38:02.401Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1607283452, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1607283452, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-12-06T19:31:42.101Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1607283102, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2020-12-06T19:31:42.124Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-12-06T19:31:42.127Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "192.168.30.129:27318",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1711,
                        "optime" : {
                                "ts" : Timestamp(1607283482, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T19:38:02Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1607283102, 2),
                        "electionDate" : ISODate("2020-12-06T19:31:42Z"),
                        "configVersion" : 3,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "192.168.30.129:27418",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 169,
                        "optime" : {
                                "ts" : Timestamp(1607283482, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1607283482, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T19:38:02Z"),
                        "optimeDurableDate" : ISODate("2020-12-06T19:38:02Z"),
                        "lastHeartbeat" : ISODate("2020-12-06T19:38:09.974Z"),
                        "lastHeartbeatRecv" : ISODate("2020-12-06T19:38:11.133Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "192.168.30.129:27318",
                        "syncSourceHost" : "192.168.30.129:27318",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 3
                },
                {
                        "_id" : 2,
                        "name" : "192.168.30.129:27518",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 87,
                        "lastHeartbeat" : ISODate("2020-12-06T19:38:09.980Z"),
                        "lastHeartbeatRecv" : ISODate("2020-12-06T19:38:10.134Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : 3
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607283482, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607283482, 1)
}
```
## 配置节点副本集的创建
### 准备存放数据data文件和日志的目录
```
mkdir -p /mongodb/sharded_cluster/myconfigrs_27019/log \
mkdir -p /mongodb/sharded_cluster/myconfigrs_27019/data/db \

mkdir -p /mongodb/sharded_cluster/myconfigrs_27119/log \
mkdir -p /mongodb/sharded_cluster/myconfigrs_27119/data/db \

mkdir -p /mongodb/sharded_cluster/myconfigrs_27219/log \
mkdir -p /mongodb/sharded_cluster/myconfigrs_27219/data/db
```
新建或修改配置文件：
myconfigrs_27019：
```
vim /mongodb/sharded_cluster/myconfigrs_27019/mongod.conf
```
```
systemLog:
    #MongoDB发送所有日志输出的目标指定为文件
     destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/sharded_cluster/myconfigrs_27019/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有
   日志文件的末尾。
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
    dbPath: "/mongodb/sharded_cluster/myconfigrs_27019/data/db"
    journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
    #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或
   mongod将写入其PID
    pidFilePath: "/mongodb/sharded_cluster/myconfigrs_27019/log/mongod.pid"
net:
    #服务实例绑定所有IP
    #bindIpAll: true
   #服务实例绑定的IP
    bindIp: localhost,192.168. 30.129
    #绑定的端口
    port: 27019
replication:
    replSetName: myconfigrs
sharding:
    clusterRole: configsvr
```
新建或修改配置文件：
myconfigrs_27119
```
vim /mongodb/sharded_cluster/myconfigrs_27219/mongod.conf
```
```
systemLog:
    #MongoDB发送所有日志输出的目标指定为文件
     destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/sharded_cluster/myconfigrs_27119/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有
   日志文件的末尾。
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
    dbPath: "/mongodb/sharded_cluster/myconfigrs_27119/data/db"
    journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
    #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或
   mongod将写入其PID
    pidFilePath: "/mongodb/sharded_cluster/myconfigrs_27119/log/mongod.pid"
net:
    #服务实例绑定所有IP
    #bindIpAll: true
   #服务实例绑定的IP
    bindIp: localhost,192.168. 30.129
    #绑定的端口
    port: 27119
replication:
    replSetName: myconfigrs
sharding:
    clusterRole: configsvr
```
新建或修改配置文件：
myconfigrs_27219
```
vim /mongodb/sharded_cluster/myconfigrs_27219/mongod.conf
```
```
systemLog:
    #MongoDB发送所有日志输出的目标指定为文件
     destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/sharded_cluster/myconfigrs_27019/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有
   日志文件的末尾。
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
    dbPath: "/mongodb/sharded_cluster/myconfigrs_27019/data/db"
    journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
    #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或
   mongod将写入其PID
    pidFilePath: "/mongodb/sharded_cluster/myconfigrs_27019/log/mongod.pid"
net:
    #服务实例绑定所有IP
    #bindIpAll: true
   #服务实例绑定的IP
    bindIp: localhost,192.168. 30.129
    #绑定的端口
    port: 27019
replication:
    replSetName: myconfigrs
sharding:
    clusterRole: configsvr
```
依次启动配置的mongod副本集：一主两副本
```
/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfigrs_27019/mongod.conf
/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfigrs_27119/mongod.conf
/usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfigrs_27219/mongod.conf
```
查看服务是否启动：
```
root@keweizhou-virtual-machine:~# ps -ef | grep mongo
root       5773      1  1 00:37 ?        00:03:33 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27018/mongod.conf
root       5810      1  1 00:37 ?        00:03:34 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27118/mongod.conf
root       5849      1  1 00:38 ?        00:02:39 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs01_27218/mongod.conf
root       8814      1  1 03:09 ?        00:01:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27318/mongod.conf
root       8853      1  1 03:10 ?        00:00:56 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27418/mongod.conf
root       8891      1  1 03:10 ?        00:00:45 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myshardrs02_27518/mongod.conf
root      10111  10014  0 04:07 pts/20   00:00:00 vim /mongodb/sharded_cluster/myshardrs01_27018/mongod.conf
root      10156      1  2 04:11 ?        00:00:07 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfigrs_27019/mongod.conf
root      10216      1  2 04:14 ?        00:00:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfigrs_27119/mongod.conf
root      10264      1  3 04:15 ?        00:00:04 /usr/local/mongodb/bin/mongod -f /mongodb/sharded_cluster/myconfigrs_27219/mongod.conf
root      10317   9513  0 04:16 pts/19   00:00:00 grep --color=auto mongo
```
## 初始化副本集和创建主节点
* 使用客户端命令连接主节点
```
/usr/local/mongodb/bin/mongo --host localhost --port 27019
```
* 执行初始化副本集命令：
```
 rs.initiate()
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "192.168.30.129:27019",
        "ok" : 1,
        "$gleStats" : {
                "lastOpTime" : Timestamp(1607286823, 1),
                "electionId" : ObjectId("7fffffff0000000000000001")
        },
        "lastCommittedOpTime" : Timestamp(0, 0),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607286823, 2),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607286823, 1)
}
```
* 查看副本集情况
```
myconfigrs:PRIMARY> rs.status()
{
        "set" : "myconfigrs",
        "date" : ISODate("2020-12-06T20:34:05.938Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "configsvr" : true,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 1,
        "writeMajorityCount" : 1,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1607286841, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2020-12-06T20:34:01.804Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1607286841, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityWallTime" : ISODate("2020-12-06T20:34:01.804Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1607286841, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1607286841, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2020-12-06T20:34:01.804Z"),
                "lastDurableWallTime" : ISODate("2020-12-06T20:34:01.804Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1607286823, 32),
        "lastStableCheckpointTimestamp" : Timestamp(1607286823, 32),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-12-06T20:33:43.769Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1607286823, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2020-12-06T20:33:43.781Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-12-06T20:33:43.892Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "192.168.30.129:27019",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1330,
                        "optime" : {
                                "ts" : Timestamp(1607286841, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T20:34:01Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1607286823, 2),
                        "electionDate" : ISODate("2020-12-06T20:33:43Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "$gleStats" : {
                "lastOpTime" : Timestamp(1607286823, 1),
                "electionId" : ObjectId("7fffffff0000000000000001")
        },
        "lastCommittedOpTime" : Timestamp(1607286841, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607286841, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607286841, 1)
}
```
* 主节点配置查看
```
myconfigrs:PRIMARY> rs.config()
{
        "_id" : "myconfigrs",
        "version" : 1,
        "configsvr" : true,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "192.168.30.129:27019",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5fcd40279fe937f6ca2c75a2")
        }
}
```
* 添加两个副本节点
```
rs.add("192.168.30.129:27119")
rs.add("192.168.30.129:27219")
```
* 查看副本集的配置配置情况
```
myconfigrs:PRIMARY> rs.conf()
{
        "_id" : "myconfigrs",
        "version" : 3,
        "configsvr" : true,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "192.168.30.129:27019",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "192.168.30.129:27119",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "192.168.30.129:27219",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5fcd40279fe937f6ca2c75a2")
        }
}
myconfigrs:PRIMARY> rs.status()
{
        "set" : "myconfigrs",
        "date" : ISODate("2020-12-06T20:43:19.377Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "configsvr" : true,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1607287393, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2020-12-06T20:43:13.944Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1607287393, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityWallTime" : ISODate("2020-12-06T20:43:13.944Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1607287393, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1607287393, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2020-12-06T20:43:13.944Z"),
                "lastDurableWallTime" : ISODate("2020-12-06T20:43:13.944Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1607287363, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1607287363, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-12-06T20:33:43.769Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1607286823, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2020-12-06T20:33:43.781Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-12-06T20:33:43.892Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "192.168.30.129:27019",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1884,
                        "optime" : {
                                "ts" : Timestamp(1607287393, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T20:43:13Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1607286823, 2),
                        "electionDate" : ISODate("2020-12-06T20:33:43Z"),
                        "configVersion" : 3,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "192.168.30.129:27119",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 152,
                        "optime" : {
                                "ts" : Timestamp(1607287393, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1607287393, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T20:43:13Z"),
                        "optimeDurableDate" : ISODate("2020-12-06T20:43:13Z"),
                        "lastHeartbeat" : ISODate("2020-12-06T20:43:18.273Z"),
                        "lastHeartbeatRecv" : ISODate("2020-12-06T20:43:19.304Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "192.168.30.129:27219",
                        "syncSourceHost" : "192.168.30.129:27219",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 3
                },
                {
                        "_id" : 2,
                        "name" : "192.168.30.129:27219",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 135,
                        "optime" : {
                                "ts" : Timestamp(1607287393, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1607287393, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-12-06T20:43:13Z"),
                        "optimeDurableDate" : ISODate("2020-12-06T20:43:13Z"),
                        "lastHeartbeat" : ISODate("2020-12-06T20:43:18.273Z"),
                        "lastHeartbeatRecv" : ISODate("2020-12-06T20:43:18.504Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "192.168.30.129:27019",
                        "syncSourceHost" : "192.168.30.129:27019",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 3
                }
        ],
        "ok" : 1,
        "$gleStats" : {
                "lastOpTime" : {
                        "ts" : Timestamp(1607287264, 1),
                        "t" : NumberLong(1)
                },
                "electionId" : ObjectId("7fffffff0000000000000001")
        },
        "lastCommittedOpTime" : Timestamp(1607287393, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1607287393, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1607287393, 1)
}
```
# 路由

























