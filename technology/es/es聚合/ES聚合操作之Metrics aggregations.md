# 0. 前言
>本文主要介绍了es聚合操作中的Metrics aggregations,metrics聚合又称为指标聚合，对桶内的文档进行某种
 聚合分析的操作，比如说求平均值，求最大值，求最小值。类似于MySQL中的COUNT()、SUM()、MAX()等。

# 1. Metrics Bucket聚合例子
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