> 本文主要介绍了mongoDB常用的一些命令，方便下次查阅。
# 数据库操作
* 选择和创建数据库(如果没有则进行自动创建)
```
use 数据库名称
```
* 查看所有有权限的数据库
```
show dbs 或者 show databases
```
* 数据库的删除
```
db.dropDatabase()
```
# 集合（对应关系型数据库的表）操作
* 显式创建集合
```
db.createCollection(name)
```
* 查看当前数据库中所有的表
```
show collections 或者 show tables
```









