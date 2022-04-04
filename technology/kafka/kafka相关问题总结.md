# 前言
> 本文主要介绍了学习kafka遇到的一些问题总结

# 安装部署过程中
## bin/kafka-run-class.sh: line 342: exec: java: not found ”问题
参考：
1. [解决：“/****/kafka_2.13-3.0.0/bin/kafka-run-class.sh: line 342: exec: java: not found ”问题](https://blog.csdn.net/qq_27706119/article/details/123411490)
2. [kafka踩坑——java找不到kafka-run-class.sh: line 309: exec: java: not found](https://blog.csdn.net/qq_41094332/article/details/104366315)

## 创建topic出现zookeeper is not a recognized option
参考：
1.[创建topic出现zookeeper is not a recognized option](https://cloud.tencent.com/developer/article/1892086)
2.[Exception in thread "main" joptsimple.UnrecognizedOptionException: zookeeper is not a recognized option](https://stackoverflow.com/questions/69297020/exception-in-thread-main-joptsimple-unrecognizedoptionexception-zookeeper-is)