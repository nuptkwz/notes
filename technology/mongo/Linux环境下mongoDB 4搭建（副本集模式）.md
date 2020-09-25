> 本文主要介绍了Linux环境下mongoDB副本级方式搭建
# 环境
# 步骤
## 创建主节点
1. 建立存放数据和日志的目录
```yml
# 主节点
mkdir -p /mongodb/replica_sets/rs_27017/log
mkdir -p /mongodb/replica_sets/rs_27017/data/db
```
2. 或修改配置文件
```yml
vim /mongodb/replica_sets/rs_27017/mongod.conf
```
```yml
systemLog:
    #MongoDB发送所有日志输出的目标指定为文件
    destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/replica_sets/rs_27017/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
    dbPath: "/mongodb/replica_sets/rs_27017/data/db"
    journal:
         #启用或禁用持久性日志以确保数据文件保持有效和可恢复。    
        enabled: true
processManagement:  
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true  
    #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID  
    pidFilePath: "/mongodb/replica_sets/rs_27017/log/mongod.pid"
net:  
    #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip  
    #bindIpAll: true  
    #服务实例绑定的IP  
    bindIp: localhost,192.168.30.129 
    #bindIp  
    #绑定的端口  
    port: 27017
replication:  
    #副本集的名称  
    replSetName: kwz_rs
```
3. 启动节点服务：
 /usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_27017/mongod.conf
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924235408111.png#pic_center)
此处有一个错误：[Error parsing YAML config file: yaml-cpp: error at line 2, column 13: illegal map value](https://blog.csdn.net/lezeqe/article/details/90518179)，涉及到yml格式空格问题，参考这篇文章解决
## 创建副本级节点
1. 建立存放数据和日志的目录
```yml
# 主节点
mkdir -p /mongodb/replica_sets/rs_27018/log
mkdir -p /mongodb/replica_sets/rs_27018/data/db
```
2. 或修改配置文件
```yml
vim /mongodb/replica_sets/rs_27018/mongod.conf
```
```yml
systemLog:
    #MongoDB发送所有日志输出的目标指定为文件
    destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/replica_sets/rs_27018/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
    dbPath: "/mongodb/replica_sets/rs_27018/data/db"
    journal:
         #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
        enabled: true
processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
    #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
    pidFilePath: "/mongodb/replica_sets/rs_27018/log/mongod.pid"
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
    replSetName: kwz_rs
```
3. 启动节点服务：
 /usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_27018/mongod.conf
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924235905866.png#pic_center)
## 创建仲裁节点
1. 建立存放数据和日志的目录
```yml
# 主节点
mkdir -p /mongodb/replica_sets/rs_27019/log
mkdir -p /mongodb/replica_sets/rs_27019/data/db
```
2. 或修改配置文件
```config
vim /mongodb/replica_sets/rs_27019/mongod.conf
```
```yml
systemLog:
    #MongoDB发送所有日志输出的目标指定为文件
    destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/replica_sets/rs_27018/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
    dbPath: "/mongodb/replica_sets/rs_27018/data/db"
    journal:
         #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
        enabled: true
processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
    #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
    pidFilePath: "/mongodb/replica_sets/rs_27018/log/mongod.pid"
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
    replSetName: kwz_rs
```
3. 启动节点服务：
 /usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/rs_27018/mongod.conf
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200925000438114.png#pic_center)
## 初始化配置副本集和主节点
用客户端连接主节点（27017）
/usr/local/mongodb/bin/mongo --host=localhost --port=27017
连接成功之后，许多命令不能用，需要初始化副本集才行，使用默认的配置来初始化副本集：
**rs.initiate()**
```yml
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "localhost:27017",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1601049270, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1601049270, 1)
}
kwz_rs:SECONDARY>
kwz_rs:SECONDARY>
kwz_rs:PRIMARY>
```
1. “ok”的值为1则说明创建成功 
2. 命令行提示符发生变化，变成了一个从节点角色，此时默认不能读写。稍等片刻，回车，变成主节 点，如上图显示的
这时候执行show dbs就有数据了：
```yml
kwz_rs:PRIMARY> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
```
## 查看副本集的配置
在27017上执行副本集中当前节点的默认节点配置
```yml
kwz_rs:PRIMARY> rs.config()
{
		# 副本集配置数据存储的主键值，默认就是副本集的名字
        "_id" : "kwz_rs",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        # 副本集成员数组,此时只有27017一个
        "members" : [
                {
                        "_id" : 0,
                        "host" : "localhost:27017",
                        # 该成员不 是仲裁节点
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        # 优先级（权重值）
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        # 副本集的参数配置
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
                "replicaSetId" : ObjectId("5f6e12b681a01e73db9b210a")
        }
}
```
值得注意的是 副本集rs.conf()的查看命令，本质查询的是 system.replset 的表中的数据，它是在local库下面
## 查看副本集状态 
检查副本集状态，从其他成员发送的心跳包中获得的数据反映副本集的当前状态，通过rs.status()命令
在27017上查看副本集状态：
```yml
kwz_rs:PRIMARY> rs.status()
{
        "set" : "kwz_rs",
        "date" : ISODate("2020-09-25T16:38:25.293Z"),
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
                        "ts" : Timestamp(1601051900, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2020-09-25T16:38:20.800Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1601051900, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityWallTime" : ISODate("2020-09-25T16:38:20.800Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1601051900, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1601051900, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2020-09-25T16:38:20.800Z"),
                "lastDurableWallTime" : ISODate("2020-09-25T16:38:20.800Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1601051850, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1601051850, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-09-25T15:54:30.537Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1601049270, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2020-09-25T15:54:30.567Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-09-25T15:54:30.580Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 89235,
                        "optime" : {
                                "ts" : Timestamp(1601051900, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-09-25T16:38:20Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1601049270, 2),
                        "electionDate" : ISODate("2020-09-25T15:54:30Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1601051900, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1601051900, 1)
}
```
