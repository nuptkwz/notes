# 0. 前言
>本文主要介绍了es聚合操作中的pipeline aggregations,pipeline aggregations又称为管道聚合，
 其作用是产生的输出结果而不是文档集，用于向输出树添加信息。
# pipeline聚合的例子
1. 通过cardinality进行去重
es中的cardinality metric是对每个bucket中的指定的field进行去重，得到去重后的count，类似于MySQL
中的count(distinct)
按照sold_date按照每年分组，然后去重计算品牌
```
GET /television/_search
{
  "size": 0,
  "aggs": {
    "months": {
      "date_histogram": {
        "field": "sold_date",
        "calendar_interval": "1y"
      },
      "aggs": {
        "distinct_brand": {
          "cardinality": {
            "field": "brand"
          }
        }
      }
    }
  }
}
```

结果如下：
![通过cardinality进行去重.png](https://upload-images.jianshu.io/upload_images/9905084-b689654adb7c3c43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. cardinality的参数优化
cardinality类似于count(distinct)，不是完全准确的，可能还有5%的错误率，性能在100ms左右
- precision_threshold优化准确率和内存开销
  precision_threshold表示在多少个unique value以内，cardinality，几乎保证100%准确。例如：
利用brand进行去重，如果brand的unique value，在100个以内，小米，长虹等，它可以保证几乎保证100%准确。
cardinality算法，会占用precision_threshold * 8 byte 内存消耗，如precision_threshold=100，则占用字节数为：100 * 8 = 800个字节
它占用内存很小。而且unique value如果的确在值以内，那么可以确保100%准确。precision_threshold=100，也表示数百万的unique value，错误率在5%以内。
precision_threshold，值设置的越大，占用内存越大，1000 * 8 = 8000 / 1000 = 8KB，可以确保更多unique value的场景下，100%的准确。在使用
precision_threshold参数的时候结合使用场景、内存考虑
脚本示例：
```
#cartinality
GET /television/_search
{
  "size": 0,
  "aggs": {
    "distinct_brand": {
      "cardinality": {
        "field": "brand",
        "precision_threshold": 100
      }
    }
  }
}
```

- HyperLogLog++ (HLL)算法性能优化
cardinality的底层算法使用了HLL，如果要去优化cardinality需要优化HLL算法的性能
底层会对所有的unique value取hash值，通过hash值近似去求distinct count，所以还有一些误差。如果要优化它的性能，默认情况下，发送一个
cardinality的时候，它会动态的对所有的field value取hash值。如果要优化性能，可以将取hash值的操作前移到建立索引的时候，例如：
索引映射脚本示例：
```
PUT /television/
{
  "mappings": {
    "sales": {
      "properties": {
        "brand": {
          "type": "text",
          "fields": {
            "hash": {
              "type": "murmur3" 
            }
          }
        }
      }
    }
  }
}
```

优化后的distinct使用聚合示例
```
GET /television/_search
{
    "size" : 0,
    "aggs" : {
        "distinct_brand" : {
            "cardinality" : {
              "field" : "brand.hash",
              "precision_threshold" : 100 
            }
        }
    }
}
```

3. percentiles百分比算法以及网站访问延时统计
记录网站每次请求的访问的耗时，需要统计tp50，tp90，tp99
tp50，tp90，tp99代表的意义分别如下：
```
tp50：50%的请求的耗时最长在多长时间
tp90：90%的请求的耗时最长在多长时间
tp99：99%的请求的耗时最长在多长时间
```
例子如下：
- tp50，tp90，tp99数据
批量插入一批数据，如下：
```
POST /website/_bulk
{"index":{}}
{"latency":105,"province":"江苏","timestamp":"2020-10-28"}
{"index":{}}
{"latency":83,"province":"江苏","timestamp":"2020-10-29"}
{"index":{}}
{"latency":92,"province":"江苏","timestamp":"2020-10-29"}
{"index":{}}
{"latency":112,"province":"江苏","timestamp":"2020-10-28"}
{"index":{}}
{"latency":68,"province":"江苏","timestamp":"2020-10-28"}
{"index":{}}
{"latency":76,"province":"江苏","timestamp":"2020-10-29"}
{"index":{}}
{"latency":101,"province":"新疆","timestamp":"2020-10-28"}
{"index":{}}
{"latency":275,"province":"新疆","timestamp":"2020-10-29"}
{"index":{}}
{"latency":166,"province":"新疆","timestamp":"2020-10-29"}
{"index":{}}
{"latency":654,"province":"新疆","timestamp":"2020-10-28"}
{"index":{}}
{"latency":389,"province":"新疆","timestamp":"2020-10-28"}
{"index":{}}
{"latency":302,"province":"新疆","timestamp":"2020-10-29"}
```
查看tp50,tp90,tp99的数据：
```
GET /website/_search
{
  "size": 0,
  "aggs": {
    "lantency_percentiles": {
      "percentiles": {
        "field": "latency",
        "percents": [
          50,
          90,
          99
        ]
      }
    },
    "lantency_avg": {
      "avg": {
        "field": "latency"
      }
    }
  }
}
```
结果数据如下：
```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 12,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "lantency_percentiles" : {
      "values" : {
        "50.0" : 108.5,
        "90.0" : 468.5000000000002,
        "99.0" : 654.0
      }
    }
  }
}
```
- tp50，tp90，tp99数据，
