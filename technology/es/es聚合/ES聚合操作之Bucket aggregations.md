# 0 前言
>本文主要介绍了es聚合操作中的Bucket操作，bucket聚合又称为桶聚合，表示满足某些特定条件的文档的集合，被分到一个组内，类似与MySQL的分组（Group）

# 1 Bucket聚合例子
以家电卖场中的电视销售数据为背景，来对不同品牌不同颜色的电视的销量和销售额，进行不同角度的分析
## 1.1 数据准备
1 创建索引指定映射
```
PUT /television
{
  "mappings": {
    "properties": {
      "price": {
        "type": "long"
      },
      "color": {
        "type": "keyword"
      },
      "brand": {
        "type": "keyword"
      },
      "sold_date": {
        "type": "date"
      }
    }
  }
}
```
2 插入数据
```
POST /television/_bulk
{"index":{}}
{"price":1000,"color":"红色","brand":"长虹","sold_date":"2020-10-28"}
{"index":{}}
{"price":2000,"color":"红色","brand":"长虹","sold_date":"2020-11-05"}
{"index":{}}
{"price":3000,"color":"绿色","brand":"小米","sold_date":"2020-05-18"}
{"index":{}}
{"price":1500,"color":"蓝色","brand":"TCL","sold_date":"2020-07-02"}
{"index":{}}
{"price":1200,"color":"绿色","brand":"TCL","sold_date":"2020-08-19"}
{"index":{}}
{"price":2000,"color":"红色","brand":"长虹","sold_date":"2020-11-05"}
{"index":{}}
{"price":8000,"color":"红色","brand":"三星","sold_date":"2021-01-01"}
{"index":{}}
{"price":2500,"color":"蓝色","brand":"小米","sold_date":"2021-02-12"}
```
## 1.2 聚合
1 统计哪种颜色的电视销量最好
按照color字段进行分组,定义单个桶的类型terms
```
GET /television/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color"
            }
        }
    }
}
```
结果如下：
![bucket聚合按照color进行分组.png](https://upload-images.jianshu.io/upload_images/9905084-bd5f2f8395fd365b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中
请求参数说明：
- size：0表示只获取聚合结果，而不要执行聚合的原始数据，如果想获取原始数据可以指定size具体的数量
- aggs：固定语法，要对一份数据执行分组聚合操作，是aggregations的缩写
- popular_colors：就是对每个aggs，你自己起一个别名
- terms：根据字段的值进行分组
- field：根据指定的字段的值进行分组

返回结果说明：
- hits.hits：指定了size是0，所以hits.hits就是空的，否则会把执行聚合的那些原始数据给你返回回来
- aggregations：聚合的返回结果
- popular_color：指定的聚合名称
- buckets：根据我们指定的field划分出的buckets
- key：每个bucket对应的那个值
- doc_count：这个bucket分组内，有多少个数据量，对应就是聚合出来颜色的销量

每种颜色对应的bucket中的数据的默认的排序规则：按照doc_count降序排序
