> 本文主要介绍了ES文档的基本操作

# 插入
```
PUT /wangzhe/user/1
{
  "name":"夏侯惇",
  "age":26,
  "role":"上单",
  "tags":["战士","肉"]
}
```
依次添加鲁班、王昭君，结果如下：
![image.png](https://upload-images.jianshu.io/upload_images/9905084-9ec49fbd9c1df673.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 查询数据
```
GET  /wangzhe/user/1
```
如下：
![查询数据.png](https://upload-images.jianshu.io/upload_images/9905084-827837df26b81e1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 更新数据
## PUT操作全量修改
```
PUT /wangzhe/user/1
{
  "name":"孙策",
  "age":26,
  "role":"上单",
  "tags":["战士","船夫"]
}
```
![更新数据.png](https://upload-images.jianshu.io/upload_images/9905084-b75c465df638b1c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## POST+_update局部更新修改
```
POST /wangzhe/user/1/_update
{
  "doc": {
    "tags": [
      "战士",
      "船夫",
      "肉"
    ]
  }
}
```
![查询数据.png](https://upload-images.jianshu.io/upload_images/9905084-04b59cca406905b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#搜索
## 简单搜索
```
GET  /wangzhe/user/_search?q=name:鲁
//q表示query
//字段是name
//匹配的值是鲁
```
![根据关键字简单查询.png](https://upload-images.jianshu.io/upload_images/9905084-85b8cda702fc61ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 复杂搜索
ES比较复杂的是查询操作，包括排序、分页、高亮、模糊查询、精准查询等
- 查询名称包含鲁班的数据
语法
```
GET /wangzhe/user/_search
{
  "query": {
    "match": {
      "name": "鲁班"
    }
  }
}
```
![模糊匹配结果.png](https://upload-images.jianshu.io/upload_images/9905084-e2943d50613d0f4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



