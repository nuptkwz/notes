@[toc]
# 前言
> 本文主要记录了阿里云Linux环境下Redis哨兵模式的搭建过程，记录搭建的一些过程，下次可以少走弯路，快速搭建。本文搭建的是一主两从三哨兵的模式，是通过一台阿里云服务器的不同的端口号来模拟不同机器上的搭建。

# 介绍
## sentinel（哨兵模式）
sentinel，中文名是哨兵。哨兵是 Redis 集群架构中非常重要的一个组件，主要有以下功能：
- 集群监控：负责监控 redis master 和 slave 进程是否正常工作
- 消息通知：如果某个 redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员
- 故障转移：如果 master node 挂掉了，通过选举从slave node上产生新的master node
- 配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址

哨兵用于实现 redis 集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作

## 核心概念
- 哨兵至少需要 3 个实例，来保证自己的健壮性
- redis主从+哨兵的模式，不保证数据零丢失的，只能保证redis 集群的高可用性

# 环境
 - Linux version 4.18.0-80.11.2.el8_0.x86_64（Red Hat 8.2.1-3）
 - redis-5.0.7.tar.gz

# 架构图
一主两从三哨兵架构图
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://img-blog.csdnimg.cn/2020081519522430.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">一主两从三哨兵架构图</div>
</center>

# 步骤
## 环境准备
1.  关于redis的单机下载、安装，请参考之前的博客：[redis基础以及ubuntu16.04环境下搭建](https://blog.csdn.net/sinat_22797429/article/details/87384173)，比较简单，这里就不重复描述了。
 
2. 我们将原来的redis-5.0.7又复制了两份，作为它的从节点，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716211948566.png)
 3. 修改端口号：我们分别进入redis-R1，redis-R2目录下修改它的redis.conf文件，将其端口号分别修改为6389、6399
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716212437855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)
 3. 启动redis：src/redis-server redis.conf，这时候我们就可以看见启动了3个redis，这相当于在三台机器上启动redis。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716212716878.png)
## 主节点配置
修改redis.conf配置文件，步骤如下：
 1. 编辑redis.conf修改，绑定ip为服务器内网ip地址，做绑定，本文ip都是一样的，端口号不一样。
```config
bind 172.19.210.221
```
2. 保护模式修改为否，允许远程连接
```config
protected-mode no
```
3. 设置密码
```config
requirepass "keweizhou"
```
4. 主库密码与当前库密码同步，保证从库能够提升为主库
```config
masterauth "keweizhou"
```
5. 打开AOF持久化
```config
appendonly yes
```
## 两个从节点配置
两个从节点和主节点的配置基本相同，额外需要添加主库的同步配置
```config
slaveof 172.19.210.221 6379
```
## 哨兵配置
修改3个节点的redis-sentinel配置文件、分别为：usr/tool/redis/sentinel.conf、usr/tool/redis-R1/sentinel.conf、usr/tool/redis-R2/sentinel.conf，步骤如下：
```java
# 1. 保护模式修改为否，允许远程连接
	 protected-mode no
# 2. 设定监控地址，为对应的主redis库的内网地址，mymaster为集群名字，可以根据情况自己修改
	 sentinel monitor mymaster 172.19.210.221 6379 2
# 3. 表示如果master重新选出来后，其它slave节点能同时并行从新master同步缓存的台数有多少个，显然该值    		  越大，所有slave节点完成同步切换的整体速度越快，但如果此时正好有人在访问这些slave，可能造成读取失败，影响面会更广。最保定的设置为1，只同一时间，只能有一台干这件事，这样其它slave还能继续服务，但是所有slave全部完成缓存更新同步的进程将变慢。
	 sentinel parallel-syncs mymaster 2
# 4. 主数据库密码,需要将配置放在sentinel monitor master 127.0.0.1 6379 1下面
 	 sentinel auth-pass mymaster keweizhou
```
sentinel.conf里面还有一些其他的参数，可以根据项目情况自我设置，如：sentinel failover-timeout、sentinel down-after-milliseconds等
## 启动
启动顺序需要按照Master->Slave->Sentinel进行启动，分别把redis的config里面的daemonize参数设置为yes，这样就可以后台启动进程了，不用开启多个窗口了，启动步骤如下：
```java
# 启动redis-server的master和slave(根据配置文件启动)
redis-server /usr/tool/redis/redis.conf
redis-server /usr/tool/redis-R1/redis.conf
redis-server /usr/tool/redis-R1/redis.conf

# 启动3个redis-sentinel(根据配置文件启动)
redis-sentinel /usr/tool/redis/sentinel.conf
redis-sentinel /usr/tool/redis-R1/sentinel.conf
redis-sentinel /usr/tool/redis-R2/sentinel.conf
```
redis-sentinel启动如下：
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://img-blog.csdnimg.cn/20200815192122612.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">redis-sentinel 启动</div>
</center>

## 验证
### 同步状态查看
进入master节点通过info replication查看
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815195949461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70#pic_center)
### 连接Sentinel状态查看
```java
[root@iZuf6b24c7i2yeaa3eldgfZ redis]# redis-cli -h 172.19.210.221 -p 26379 INFO Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.19.210.221:6379,slaves=2,sentinels=3
[root@iZuf6b24c7i2yeaa3eldgfZ redis]# redis-cli -h 172.19.210.221 -p 26389 INFO Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.19.210.221:6379,slaves=2,sentinels=3
[root@iZuf6b24c7i2yeaa3eldgfZ redis]# redis-cli -h 172.19.210.221 -p 26399 INFO Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.19.210.221:6379,slaves=2,sentinels=3
[root@iZuf6b24c7i2yeaa3eldgfZ redis]#

```
### 主从复制验证
主节点负责写，从节点负责读，只有主节点可以删除数据、从节点只能同步主节点的数据，如下:
在master节点添加一条数据
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://img-blog.csdnimg.cn/20200815194031906.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">master节点增加数据</div>
</center>

从节点数据6389同步数据：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://img-blog.csdnimg.cn/20200815194245323.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">从节点6389同步数据</div>
</center>

从节点数据6399同步数据：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://img-blog.csdnimg.cn/20200815195553109.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">从节点6399同步数据</div>
</center>

### 哨兵高可用验证
分别连接对应的redis服务端，手动停止哨兵、停止主redis服务，查看主是否切换成功
- 把6379master节点停了
```java
# redis实例挂掉两台，剩下一台能够成为主，自动切换
# 1.利用kill -9 pid直接暴力杀掉当前role:master的对应的redis服务,重新检查状态看是否切换
  kill -9 7400
# 2.查看连接：redis-cli -h 172.19.210.221 -p 26379 INFO Sentinel
  # Sentinel
  sentinel_masters:1
  sentinel_tilt:0
  sentinel_running_scripts:0
  sentinel_scripts_queue_length:0
  sentinel_simulate_failure_flags:0
  master0:name=mymaster,status=ok,address=172.19.210.221:6389,slaves=2,sentinels=3
```
可以看到此时master节点已经切换到6389上了（原来6379）

- 再把6389master节点停了，查看连接情况
```java
# redis-cli -h 172.19.210.221 -p 26379 INFO Sentinel
 sentinel_masters:1
 sentinel_tilt:0
 sentinel_running_scripts:0
 sentinel_scripts_queue_length:0
 sentinel_simulate_failure_flags:0
 master0:name=mymaster,status=ok,address=172.19.210.221:6399,slaves=2,sentinels=3
```
可以看见master节点已经切换到6399了，这里的切换并不是一下子完成的，需要稍微等待几秒

# 搭建过程遇到的问题
- The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128. 参考：[https://www.jianshu.com/p/a86e0248af58](https://www.jianshu.com/p/a86e0248af58)
```java
在/etc/sysctl.conf中添加: net.core.somaxconn = 2048
然后在终端中执行
sysctl -p
```
- 启动redis-client的时候出现：Error: Connection reset by peer
重新启动redis服务端，根据配置文件启动


