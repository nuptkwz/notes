# 0. 前言
>本文主要介绍了es聚合操作中的Metrics aggregations,metrics聚合又称为指标聚合，对桶内的文档进行某种
 聚合分析的操作，比如说求平均值，求最大值，求最小值。类似于MySQL中的COUNT()、SUM()、MAX()等。

# 1. metric聚合类型(AGG_TYPE)
metric聚合类型主要有count，avg，max，min，sum等 
- avg：求平均值
- max：求一个bucket内指定field值最大的那个数据
- min：求一个bucket内指定field值最小的那个数据
- sum： 求一个bucket内指定field值的总和

# 2. Metrics聚合例子
按照color去分bucket，可以得到每种color bucket的数量，这个仅仅只是一个bucket操作，doc_count其实只是es的bucket操作默认执行的一个内置metric。
metric聚合就是除了对bucket操作进行聚合分组外，对每个bucket进行聚合统计等。下面介绍一下利用Metrics Bucket进行聚合的例子。

1 统计哪种颜色的电视的平均销售价格
```
GET /television/_search
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": { 
            "avg_price": { 
               "avg": {
                  "field": "price" 
               }
            }
         }
      }
   }
}
```
结果如下：
![metric聚合计算每种颜色电视平均值.png](https://upload-images.jianshu.io/upload_images/9905084-b85757628592f332.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中
请求参数说明：
在一个aggs执行的bucket操作（terms），平级的json结构下，增加一个aggs，这个第二个aggs内部，同样取个名字，执行一个metric操作：
avg（一个AGG_TYPE），对之前的每个bucket中的数据的指定的field，price field，求一个平均值。这样就是就是一个metric，就是一个对一个bucket分组操作之后，
对每个bucket都要执行的一个metric操作。

返回结果说明：
- buckets：聚合结果的数组，key和doc_count，bucket aggs已介绍
- avg_price：我们自己取的metric aggs的名字
- value：我们的metric计算的结果，即每个bucket中的数据的price字段求平均值后的结果

上面这串脚本相当于MySQL的如下语法：
```
select avg(price) from television group by color
```

2 按照多个维度嵌套聚合
从颜色到品牌进行嵌套聚合，计算每种颜色的平均价格，以及找到每种颜色每个品牌的平均价格。例如：已经对各种
家电按照颜色进行分组了，还要对这个组里面的数据进行再分组，例如在一个颜色内按照品牌再分组，然后再对每个
小组里面的数据再进行聚合分析操作
```
GET /television/_search
{
  "size": 0,
  "aggs": {
    "group_by_color": {
      "terms": {
        "field": "color"
      },
      "aggs": {
        "color_avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "group_by_brand":{
          "terms": {
            "field": "brand"
          },
          "aggs": {
            "brand_avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}
```
结果如下：
![es按照颜色的品牌嵌套聚合.png](https://upload-images.jianshu.io/upload_images/9905084-b85757628592f332.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3 按照多种类型聚合
按照多种类型聚合，分别计算出按照color分组后每组数据价格的平均值、最小值、最大值、总数
```
GET /television/_search
{
  "size": 0,
  "aggs": {
    "group_by_color": {
      "terms": {
        "field": "color"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "min_price": {
          "min": {
            "field": "price"
          }
        },
        "max_price": {
          "max": {
            "field": "price"
          }
        },
        "sum_price": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}
```
结果如下：
![按照多种类型聚合.png](https://upload-images.jianshu.io/upload_images/9905084-7a11645d94d9c9e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4 柱状图功能-Histogram
直方图histogram本质上是就是为柱状图功能设计的,它类似于terms，也是进行bucket分组操作，接收一个field，
按照这个field的值的各个范围区间，进行bucket分组操作。histogram本质上属于bucket的一种划分方法，前面
是将color相同的放到一个bucket中。
例如：按照电视价格2000的区间对电视进行分组统计
```
GET /television/_search
{
  "size": 0,
  "aggs": {
    "price_histogram": {
      "histogram": {
        "field": "price",
        "interval": 2000
      },
      "aggs": {
        "price_sum": {
          "sum": {
            "field": "price"
          }
        },
        "price_avg": {
          "avg": {
            "field": "price"
          }
        },
        "price_min": {
          "min": {
            "field": "price"
          }
        },
        "price_max": {
          "max": {
            "field": "price"
          }
        }
      }
    }
  }
}
```
其中
- interval：2000，指的是划分范围，0~2000，2000~4000，4000~6000，6000~8000，8000~10000，这些区间数据
  会放到不同的bucket中

结果如下：
![histogram聚合.png](https://upload-images.jianshu.io/upload_images/9905084-fb14af20c50ee1d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5 按照日期间隔划分-date histogram
date histogram也是划分不同bucket的一种方法，它是按照我们指定的某个date类型的日期field，以及日期
interval，去划分bucket
- calendar_interval
  calendar_interval，对应的值为1y,1M,1d等。
  如果calendar_interval：1M，则2019-01-01 -- 2019-01-31，就是一个bucket，2019-02-01 -- 2019-02-28，就是一个
  bucket
- min_doc_count：0
  即使某个interval如2019-01-01~2017-01-31中一条数据都没有，那么这个区间也是要返回的，不然默认是会过滤掉这个区间的
- extended_bounds，min，max
  划分bucket的时候，会限定在这个起始日期，和截止日期内
  
```
GET /television/_search?size=0
{
  "aggs": {
    "sales": {
      "date_histogram": {
        "field": "sold_date",
        "calendar_interval": "1M",
        "format": "yyyy-MM-dd",
        "min_doc_count": 0,
        "extended_bounds": {
          "min": "2019-01-01",
          "max": "2021-01-01"
        }
      }
    }
  }
}
```
结果如下：
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
      "value" : 8,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "sales" : {
      "buckets" : [
        {
          "key_as_string" : "2019-01-01",
          "key" : 1546300800000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-02-01",
          "key" : 1548979200000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-03-01",
          "key" : 1551398400000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-04-01",
          "key" : 1554076800000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-05-01",
          "key" : 1556668800000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-06-01",
          "key" : 1559347200000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-07-01",
          "key" : 1561939200000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-08-01",
          "key" : 1564617600000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-09-01",
          "key" : 1567296000000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-10-01",
          "key" : 1569888000000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-11-01",
          "key" : 1572566400000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2019-12-01",
          "key" : 1575158400000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2020-01-01",
          "key" : 1577836800000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2020-02-01",
          "key" : 1580515200000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2020-03-01",
          "key" : 1583020800000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2020-04-01",
          "key" : 1585699200000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2020-05-01",
          "key" : 1588291200000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2020-06-01",
          "key" : 1590969600000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2020-07-01",
          "key" : 1593561600000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2020-08-01",
          "key" : 1596240000000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2020-09-01",
          "key" : 1598918400000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2020-10-01",
          "key" : 1601510400000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2020-11-01",
          "key" : 1604188800000,
          "doc_count" : 2
        },
        {
          "key_as_string" : "2020-12-01",
          "key" : 1606780800000,
          "doc_count" : 0
        },
        {
          "key_as_string" : "2021-01-01",
          "key" : 1609459200000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2021-02-01",
          "key" : 1612137600000,
          "doc_count" : 1
        }
      ]
    }
  }
}

```

6 基于搜索结果的聚合
搜索小米电视机，并且按照颜色进行聚合
```
GET /television/_search
{
  "size": 0,
  "query": {
    "term": {
      "brand": {
        "value": "小米"
      }
    }
  },
  "aggs": {
    "group_by_color": {
      "terms": {
        "field": "color"
      }
    }
  }
}
```
结果如下：
![基于搜索结果的聚合.png](https://upload-images.jianshu.io/upload_images/9905084-553bebf67b0bd0e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上脚本类似于MySql语法
```
select count(*) from television where brand = "小米" group by price
```

7 基于搜索聚合和全部聚合
```
GET /television/_search
{
  "size": 0,
  "query": {
    "term": {
      "brand": {
        "value": "小米"
      }
    }
  },
  "aggs": {
    "single_brand_avg_price": {
      "avg": {
        "field": "price"
      }
    },
    "all": {
      "global": {},
      "aggs": {
        "all_brand_avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```
结果如下：
![基于搜索聚合和全部聚合.png](https://upload-images.jianshu.io/upload_images/9905084-aabef121847e75f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中
请求参数说明：
- global：就是global bucket，将所有数据纳入聚合的scope，而不管之前的query条件
- single_brand_avg_price：针对query搜索结果执行，拿到的就是小米电视品牌的平均价格
- all.all_brand_avg_price：忽略query条件，拿到是所有电视品牌的平均价格

8 基于过滤结果的聚合
过滤出价格大于1200但是小于2000的电视机，并算出他们的平均值
```
GET /television/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "price": {
            "gte": 1200,
            "lte": 2000
          }
        }
      }
    }
  },
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

结果如下：
![基于过滤结果的聚合.png](https://upload-images.jianshu.io/upload_images/9905084-00f78337d45d2df6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

9 对不同的bucket下的聚合进行过滤
统计近20个月和18个月电视销售的平均价格
```
# 对不同bucket下的agg进行聚合
GET /television/_search
{
  "size": 0,
  "query": {
    "term": {
      "brand": {
        "value": "长虹"
      }
    }
  },
  "aggs": {
    "recent_600d": {
      "filter": {
        "range": {
          "sold_date": {
            "gte": "now-600d"
          }
        }
      },
      "aggs": {
        "recent_600d_avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    },
    "recent_530d":{
      "filter": {
        "range": {
          "sold_date": {
            "gte": "now-530d"
          }
        }
      },
      "aggs": {
          "recent_530d_avg_price": {
            "avg": {
              "field": "price"
            }
          }
      }
    }
  }
}
```

结果如下：
![对不同的bucket下的聚合进行过滤.png](https://upload-images.jianshu.io/upload_images/9905084-88787e28b93c9f2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


10 指定聚合字段进行排序
```
# 指定聚合字段进行排序
GET /television/_search
{
  "size": 0,
  "aggs": {
    "group_by_color": {
      "terms": {
        "field": "color",
        "order": {
          "avg_price": "asc"
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

结果如下：
![指定聚合字段进行排序.png](https://upload-images.jianshu.io/upload_images/9905084-a95bbfbf51101d8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

11 颜色+品牌下钻分析按最深层的metric进行排序
```
GET /television/_search
{
  "size": 0,
  "aggs": {
    "group_by_color": {
      "terms": {
        "field": "color"
      },
      "aggs": {
        "group_by_brand": {
          "terms": {
            "field": "brand",
            "order": {
              "avg_price": "asc"
            }
          },
          "aggs": {
            "avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}
```

结果如下：
![颜色+品牌下钻分析按最深层的metric进行排序.png](https://upload-images.jianshu.io/upload_images/9905084-0489f2fb7c262788.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)















