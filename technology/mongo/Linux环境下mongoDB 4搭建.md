# 前言
> 本篇文章主要描述了Linux环境下mongoDB 4搭建的搭建笔记，方便以后快速查阅。

# 环境
- Ubuntu 5.4.0-6ubuntu1~16.04.12
- mongodb-linux-x86_64-ubuntu1604-4.2.8.tgz

#步骤
1. 下载
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1604-4.2.8.tgz
2.  解压
tar -zxvf mongodb-linux-x86_64-ubuntu1604-4.2.8.tgz
3. 移动解压后的文件夹到指定的目录中
mv mongodb-linux-x86_64-ubuntu1604-4.2.8 /usr/local/mongodb
4. 建两个目录用来存储数据和日志
mkdir -p /mongodb/single/data/db  //数据存储目录
mkdir -p /mongodb/single/data/log //日志存储目录
5. 新建并修改配置文件
vi /mongodb/single/mongod.conf
内容如下：
```
systemLog:
    # MongoDB发送所有日志输出的目标指定为文件
    destination: file
    # mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/single/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
    ##The directory where the mongod instance stores its data.Default Value is "/data/db".
    dbPath: "/mongodb/single/data/db"
    journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
        enabled: true
processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
net:
    #服务实例绑定的IP，默认是localhost
    bindIp: localhost,192.168.30.129
    #bindIp #绑定的端口，默认是27017
    port: 27017
```
6. 启动MongoDB服务
 /usr/local/mongodb/bin/mongod -f /mongodb/single/mongod.conf
显示 started successfully则启动成功
7. 查看Mongo进程启动情况：ps -ef | grep mongo
```
root       7845      1  0 22:35 ?        00:00:15 /usr/local/mongodb/bin/mongod -f /mongodb/single/mongod.conf
root       8467   6829  0 23:49 pts/9    00:00:00 grep --color=auto mongo
```
8. 连接Mongo
./mongo
显示：
```
MongoDB shell version v4.2.8
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("f57259fc-9e08-4537-9f46-0d91d7e11538") }
MongoDB server version: 4.2.8
Server has startup warnings:
2020-09-16T22:35:04.845+0800 I  STORAGE  [initandlisten]
2020-09-16T22:35:04.845+0800 I  STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2020-09-16T22:35:04.845+0800 I  STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2020-09-16T22:35:05.549+0800 I  CONTROL  [initandlisten]
2020-09-16T22:35:05.549+0800 I  CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2020-09-16T22:35:05.549+0800 I  CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2020-09-16T22:35:05.549+0800 I  CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2020-09-16T22:35:05.549+0800 I  CONTROL  [initandlisten]
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
>
```

