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
* 集合的删除
```
//删除article集合
db.article.drop()
```
# 文档（document）的基本操作
## 插入
* 单文档插入
使用insert()或者save()向集合中插入文档，区别在于insert用于首次插入，重复会报错，而save是全量更新。语法如下：
```
db.collection.insert(
<document or array of documents>,
  {
    writeConcern: <document>,
    //插入的顺序，2.6版本默认为true
    ordered: <boolean>
  }
)
```
* 多文档插入（批量插入）
```
db.collection.insertMany(
    [ <document 1> , <document 2>, ... ],
      {  
        writeConcern: <document>,
        ordered: <boolean>
      }
)
```
## 查询
* 根据参数查询
```
db.collection.find(<query>, [projection])
```
例子：
1. 查询comment集合中所有
```
db.comment.find()或者
db.getCollection("comment").find()或者
db.comment.find({})
```
2.按一定的条件查询
如按articleId和nickname查询
```
db.comment.find({"articleId":"100003","nickName":"Rose"})
```
3.返回符合条件的第一条记录
```
db.comment.findOne({"articleId":"100003"})
```
4.投影查询
投影查询指的是查询结果只返回部分字段，不返回所有字段，如查询userId=1001结果只显示_id、userId、nickName：
```
db.comment.find({userId:"1001"},{userId:1,nickName:1})
```









