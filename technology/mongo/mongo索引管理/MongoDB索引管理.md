> 本文主要介绍了mongoDB的索引管理
# 前言
索引支持在MongoDB中高效地执行查询。如果没有索引，MongoDB必须执行全集合扫描，即扫描集合中的每个文档，以选择与查询语句匹配的文档。
这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

如果查询存在适当的索引，MongoDB可以使用该索引限制必须检查的文档数。索引是特殊的数据结构，它以易于遍历的形式存储集合数据集的一小部分。
索引存储特定字段或一组字段的值，按字段值排序。索引项的排序支持有效的相等匹配和基于范围的查询操作。此外，MongoDB还可以使用索引中的排序
返回排序结果。MongoDB索引使用B树数据结构(MySQL用的是B+Tree)。

# 索引的类型
## 单字段索引
MongoDB支持在文档的单个字段上创建用户定义的升序/降序索引，称为单字段索引（Single Field Index）。对于单个字段索引和排序操作，索引键
的排序顺序（即升序或降序）并不重要，因为MongoDB可以在任何方向上遍历索引。
下图说明了使用索引选择和排序匹配文档的查询：
![image.png](https://upload-images.jianshu.io/upload_images/9905084-407957fb2677c896.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

它的排列顺序呢就是按照18，30，45等顺序排列，会存储这样的索引数据结构。比如我们去查询score=30的时候先去查索引数据，找到之后再去collection
中找它真实的文档。单字段索引创建，无论是升序或者降序，那么它的效率没多大区别。

## 复合索引
MongoDB还支持多个字段的索引，即复合索引（Compound index）。复合索引中列出的字段的顺序具有重要的意义。
例如如果复合索引由{userId:1,score:-1}组成，则索引首先通过ask的顺序按照userId正序排序，然后在每个
userId内，再按照core倒序排序(比如图中的ca2三个一样，然后在内部再做一个userId的降序排序)。

## 其他索引
MongoDB除了默认的id索引，单字段索引，复合索引，还有其他的一些索引。如地理空间索引(Geospatial Index)、
文本索引（Text Indexes）、哈希索引（Hashed Indexes）。

- 地理空间索引(Geospatial Index)
为了支持对地理空间坐标的有效查询，MongoDB提供了两种特殊的索引：返回结果时使用平面几何的二维索引和返回
结果时使用球面几何的二维球面索引。

- 文本索引（Text Indexes）
MongoDB提供了一种文本索引类型，支持在集合中索引字符串的内容。这些文本索引不存储特定语言的停止词（例如
“the”、“a”、“or”），而将集合中的词作为词干，只存储词根。文本索引有点类似于Lucene，性能不高，如果
我们需要做一些文本索引去搜索或者查询的话，那么还是建议使用Lucene或者es这些成熟的搜索引擎。

- 哈希索引（Hashed Indexes）
为了支持基于hash的分片，MongoDB提供了hash索引类型，它对字段值的hash进行索引。这些索引在其范围内的值
分布更加随机，但值支持相等匹配，不支持基于范围的查询。



#索引的基本操作
## 索引的查看
返回是一个数组，这个主键是默认会创建的一个索引，而且这个索引是唯一索引，和咱们的MySQL是很像。
v:表示索引引擎的版本号（2表示咱们MongoDB索引的引擎是第二个版本，这个不用管MongoDB底层的东西，后面还会
更新）
key：表示你在哪个字段上加的索引，1表示你这个索引创建的是升序的方式
name：索引的名称，默认是xx_1(或者xx_-1)
ns:namespace,是哪个数据库下面的哪个个集合

## 创建索引
```javascript
db.collection.createIndex(keys,options)
```
参数的可以选项options：

- 单字段索引创建


- 复合索引创建



## 删除索引
不用的时候需要把索引移除，因为索引也是一个小的集合，会占用一定的存储空间，如果索引中建了许多不必要的索引,
也会拖累你的插入效率的，因为在插入的时候也需要维护索引数据。
语法：
```javascript
db.collection.dropIndex(index)
```
支持通过名称删除，或者之前建立的条件删除。

- 删除指定的索引


- 删除所有的索引


# 索引的使用
## 索引的执行计划
通常使用执行计划来分析查询性能，如查询耗时的时间，是否基于索引查询等。那么通常我们想知道建立的索引是否
有效，效果如何，都需要通过执行计划来查看。
通常我们最关心的问题：
- 查询是否使用了索引？
- 索引是否减少了扫描记录的数量？
- 是否存在低效的内存排序？

MongoDB提供了explain命令，它可以帮助我们指定查询模型（querymodel）的执行计划，根据实际情况进行调整，
然后提高查询效率。比如你扫描了大量的集合，没有用到索引等，这些都是可以去优化的。
explain方法的形式如下：
```javascript
db.collection.find().explain(<berbose>)
```
berbose可选模式有3种。
- queryPlanner
![queryPlanner模式](./images/index_explain_queryPlanner.png)


- executionStats
executionStats模式的返回信息中包括了queryPlanner模式的所有字段，并且包含了最佳执行计划的执行情况。
![executionStats模式](./images/index_explain_executionStats.png)

- allPlansExecution
allPlansExecution返回的信息包含executionStats模式的内容，且包含allPlansExecution:[]块


语法：
```javascript
db.collection.find(query,options).explain(options)
```
执行计划中的stage类型
![stage类型](./images/index_explain_stage.png)
- COLLSCAN : 集合扫描，说明没有用到索引走的是全表扫描
- FETCH : 抓取，根据索引去检索

执行计划的返回结果中尽量不要出现以下的stage：
- COLLSCAN(全表扫描)，数据小无所谓，数据大了可能就会导致慢查询了
- SORT(使用sort但是无index)：尽量使用索引的排序，特别是复合索引
- 不合理的SKIP
- SUBPLA(未用到index的$or)
- COUNTSCAN(不使用index进行count)



## 涵盖的查询（Covered Queries）
当查询条件和查询的投影仅包含索引的字段时，MongoDB直接从索引返回结果，而不扫描任何文档或将文档带入到内存。
这些覆盖的查询效率是非常高的。

