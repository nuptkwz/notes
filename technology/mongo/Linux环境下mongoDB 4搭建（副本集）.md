> 本文主要介绍了Linux环境下mongoDB副本级方式搭建
# 环境
# 步骤
## 创建主节点
1. 建立存放数据和日志的目录
```config
# 主节点
mkdir -p /mongodb/replica_sets/rs_27017/log
mkdir -p /mongodb/replica_sets/rs_27017/data/db
```
2. 或修改配置文件
```config
vim /mongodb/replica_sets/rs_27017/mongod.conf
```
```config
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
```config
# 主节点
mkdir -p /mongodb/replica_sets/rs_27018/log
mkdir -p /mongodb/replica_sets/rs_27018/data/db
```
2. 或修改配置文件
```config
vim /mongodb/replica_sets/rs_27018/mongod.conf
```
```config
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
```config
# 主节点
mkdir -p /mongodb/replica_sets/rs_27019/log
mkdir -p /mongodb/replica_sets/rs_27019/data/db
```
2. 或修改配置文件
```config
vim /mongodb/replica_sets/rs_27019/mongod.conf
```
```config
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
## 初始化配置副本集合主节点
