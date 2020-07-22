# 前言
> 本文主要记录了阿里云Linux环境下Redis哨兵模式的搭建过程，记录搭建的一些过程，下次可以少走弯路，快速搭建。本文搭建的是一主两从三哨兵的模式，是通过一台阿里云服务器的不同的端口号来模拟不同机器上的搭建。

# 环境

 - Linux version 4.18.0-80.11.2.el8_0.x86_64（Red Hat 8.2.1-3）
 - redis-5.0.7.tar.gz
# 准备工作
 关于redis的单机下载、安装，请参考之前的博客：[redis基础以及ubuntu16.04环境下搭建](https://blog.csdn.net/sinat_22797429/article/details/87384173)，比较简单，这里就不重复描述了。

# 步骤
## 搭建从节点
 1. 我们将原来的redis-5.0.7又复制了两份，作为它的从节点，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716211948566.png)
 2. 修改端口号：我们分别进入redis-R1，redis-R2目录下修改它的redis.conf文件，将其端口号分别修改为6389、6399
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
requirepass "123456"
```
4. 主库密码与当前库密码同步，保证从库能够提升为主库
```config
masterauth "123456"
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
修改redis-sentinel配置文件，步骤如下：