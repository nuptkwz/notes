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
hit:包含了索引和文档的信息、查询的结果总数、查询出来的具体的文档、分数（通过分数可以判断哪个更符合）
- 查询结果返回固定字段（类似于mongo中的倒影查询）
```
GET /wangzhe/user/_search
{
  "query": {
    "match": {
      "name": "鲁班"
    }
  },
  "_source": ["name","role"]
}
```
![查询结果返回固定字段.png](https://upload-images.jianshu.io/upload_images/9905084-a75d7da20bf2d2e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 排序
根据关键字符合鲁班，age排序，asc:升序，desc:降序
![根据age进行排序的结果.png](https://upload-images.jianshu.io/upload_images/9905084-634156b4444d95d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 分页查询
from:从第几条数据开始，size:返回多少条数据
![分页查询.png](https://upload-images.jianshu.io/upload_images/9905084-9ad983d02846e2e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 通过bool进行多条件的匹配查询
must(相当于MySQL中的and)，所有条件都要符合
must_not(相当于MySQL中的!=)
```
GET /wangzhe/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "鲁班"
          }
        },
        {
          "match": {
            "age": 5
          }
        }
      ]
    }
  }
}
```
![通过bool进行多条件的匹配查询.png](https://upload-images.jianshu.io/upload_images/9905084-a35729f6b84b836d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

should(相当于MySQL中的or)，所有条件或的查询
![should命令查询.png](https://upload-images.jianshu.io/upload_images/9905084-9b7dd6d00f1509b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 通过filter进行过滤查询
```
GET /wangzhe/user/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "name": "鲁班"
          }
        }
      ],
      "filter": {
        "range": {
          "age": {
            "lte": 8
          }
        }
      }
    }
  }
}
```
![通过filter进行过滤查询.png](https://upload-images.jianshu.io/upload_images/9905084-f42f729af3b12a71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 数组匹配查询
数组里的多个匹配条件通过空格隔开即可，只要满足其中一个条件即可被查出
```
GET /wangzhe/user/_search
{
  "query": {
    "match": {
      "tags": "学生 肉"
    }
  }
}
```
![数组匹配查询.png](https://upload-images.jianshu.io/upload_images/9905084-dce5d2849ea68aff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 精确查询
term查询是直接通过倒排索引指定的词条进程精确的查找！
 **关于分词**
a. term:查询精确的
b. match:会使用分词器解析（先分析文档，再通过分析的文档进行查询）
**两个类型 text keyword**
创建testdb索引并插入两条数据，name为text类型，desc为keyword类型。text类型会被当成分词器普通解析，如果是keyword类型则不会解析。
```
PUT testdb
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "desc":{
        "type": "keyword"
      }
    }
  }
}

PUT testdb/_doc/1
{
  "name":"刻威舟",
  "desc":"刻威舟desc1"
}


PUT testdb/_doc/2
{
  "name":"刻威舟",
  "desc":"刻威舟desc2"
}
```
通过head插件查看索引的映射规则：
![通过head插件查看索引的映射规则.png](https://upload-images.jianshu.io/upload_images/9905084-b181dc2442775b13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**测试text、keyword两种类型**
利用keyword会把它当做一个整体，而利用普通的默认分词器，会把它拆分成一个个字，如下图：
![keyword没有被分析.png](https://upload-images.jianshu.io/upload_images/9905084-ba1bccfc079e095e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![standard可以看见被拆分了.png](https://upload-images.jianshu.io/upload_images/9905084-64caf9acaff87598.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![keyword类型不会被分词器解析.png](https://upload-images.jianshu.io/upload_images/9905084-cb2bb275bb917a05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 多个值匹配精确查询
![多个值匹配精确查询.png](https://upload-images.jianshu.io/upload_images/9905084-ea787939bd4a20b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 高亮查询
1.默认高亮查询
```
GET wangzhe/user/_search
{
  "query": {
    "match": {
      "name": "王昭君"
    }
  },
  "highlight": {
    "fields": {
      "name":{}
    }
  }
}
```
搜索相关的结果会被高亮显示，通过highlight里面的fields进行字段设置
![高亮查询.png](https://upload-images.jianshu.io/upload_images/9905084-983e35810ce60a41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.自定义高亮查询
```
GET wangzhe/user/_search
{
  "query": {
    "match": {
      "name": "王昭君"
    }
  },
  "highlight": {
    "pre_tags": "<p class='key' style='color:red'",
    "post_tags": "</p>",
    "fields": {
      "name":{}
    }
  }
}
```
![自定义高亮查询.png](https://upload-images.jianshu.io/upload_images/9905084-d39e7fe27b46557a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






