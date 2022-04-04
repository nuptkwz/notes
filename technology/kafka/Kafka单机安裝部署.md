# 0. 前言
> 本文介绍了linux下kafka的安装与部署，其中zookeeper采用kafka自带的,写这篇文章的目的是为了后面可以快速安装与部署kafka。

# 1. 下载
- kafka官网下载地址：https://kafka.apache.org/downloads
- 快速开始：http://kafka.apache.org/quickstart
- 利用下载地址进行下载
   ```
  wget https://dlcdn.apache.org/kafka/3.1.0/kafka_2.13-3.1.0.tgz
   ```
![kafka官网.png](https://upload-images.jianshu.io/upload_images/9905084-f3c58a46367b913d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![下载地址.png](https://upload-images.jianshu.io/upload_images/9905084-65c50ad78f49ad36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2. 解压
```
tar -zxvf kafka_2.13-3.1.0.tgz
```
![kafka解压.png](https://upload-images.jianshu.io/upload_images/9905084-51e898acf1f690a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 3. 修改server.properties文件
- 进入kafka目录
- 进入config目录
- 修改kafka配置文件server.properties
配置文件主要修改了日志的存放地址
```
log.dirs=/kafka/single/kafka_2.13-3.1.0/data/log
port=9092
host.name=localhost
```

# 4. 修改zookeeper.properties文件
```
dataDir=/kafka/single/kafka_2.13-3.1.0/zookeeper/data/dataDir
dataLogDir=/kafka/single/kafka_2.13-3.1.0/zookeeper/data/dataLogDir
maxClientCnxns=100
tickTime=2000
initLimit=10
```

# 5. 启动kafka
## 5.1 启动和停止zookeeper
```
sudo bin/zookeeper-server-start.sh config/zookeeper.properties
sudo bin/zookeeper-server-stop.sh config/zookeeper.properties
```

## 5.2 启动和停止kafka
```
bin/kafka-server-start.sh config/server.properties
bin/kafka-server-stop.sh config/server.properties
```

## 5.3 创建topic
```
./bin/kafka-topics.sh --create --topic kwz_1 --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1
```

## 5.4 查看topic列表
```
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

## 5.5 启动kafka-producer
```
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic kwz_1
```

## 5.6 启动kafka-consumer
```
./bin/kafka-console-consumer.sh  --bootstrap-server localhost:9092 --topic kwz_1 --from-beginning
```

# 6. 参考
- [Kafka 启动命令](https://blog.csdn.net/weixin_38362455/article/details/81671781)
- [4-kafka集群安装部署](https://segmentfault.com/a/1190000023772825)
- [kafka快速开始](https://kafka.apache.org/quickstart)
- [官网](https://kafka.apache.org/documentation/)





