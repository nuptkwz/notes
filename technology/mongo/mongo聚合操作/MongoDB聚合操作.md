> 本文的介绍的聚合操作包括2种，Aggregation Pipeline、Map-Reduce，基于mongo 4.4版本。

# Aggregation Pipeline
Aggregation Pipeline又称之为管道聚合，聚合管道由一个或多个处理文档的阶段(stages)组成，MongoDB提供了db.collection.aggregate() 方法和运行聚合管道的聚合命令。
- 每个阶段对输入文档执行一个操作。例如，一个阶段可以过滤文档、分组文档和计算值
- 从一个阶段输出的文档将输入到下一个阶段
- 聚合管道可以返回文档组的结果，例如:返回总值、平均值、最大值和最小值

## Aggregation Pipeline示例
创建以下包含产品订单的集合：
```javascript
db.orders.insertMany( [
   { _id: 0, productName: "Steel beam", status: "new", quantity: 10 },
   { _id: 1, productName: "Steel beam", status: "urgent", quantity: 20 },
   { _id: 2, productName: "Steel beam", status: "urgent", quantity: 30 },
   { _id: 3, productName: "Iron rod", status: "new", quantity: 15 },
   { _id: 4, productName: "Iron rod", status: "urgent", quantity: 50 },
   { _id: 5, productName: "Iron rod", status: "urgent", quantity: 10 }
] )
```
1. 一个stage(match)，相当于find加条件的过滤
```javascript
db.orders.aggregate([
{$match:{status:"urgent"}}
])
```
$match stage:
- 将文档过滤到状态为"urgent"的文档
- 将过滤后的文档输出到 $group 阶段

2. 增加第2个stage(group)，在match的基础上对productName进行group，每个group统计quantity的总数
```javascript
db.orders.aggregate([
{$match:{status:"urgent"}},
{$group: { _id: "$productName", sumQuantity: { $sum: "$quantity" } } }
])
```
$group stage:
- 承接match的stage,按产品名称对输入文档进行分组
- 使用 $sum 计算每个productName组的总数量，该总数量存储在聚合管道返回的sumQuantity字段中。

## 聚合管道阶段
聚合管道由一个或多个处理文档的阶段组成：
- 每个阶段都会在文档通过管道时对其进行转换。
- 同一阶段可以在管道中多次出现，但以下阶段例外：$out、$merge 和 $geoNear。

### db.collection.aggregate()阶段
语法:
```
db.collection.aggregate( [ { <stage> }, ... ] )
```
除了 $out、$merge 和 $geoNear 阶段之外的所有阶段都可以在管道中出现多次。

#### $count
$count常用的包括以下几个:
- db.collection.countDocuments()
- $collStats
- db.collection.estimatedDocumentCount()
- count
- db.collection.count()
例如：匹配copies小于等于2的文档，并且统计这些文档的数量
```javascript
db.books.aggregate([
{
  $match:{
       copies:{$lte:2}
      }    
},
{    
  $count:"lte_2_copies_count"
}
])
```
#### $match
过滤出条件匹配的文档传递到下一个管道阶段，语法：
```
{ $match: { <query> } }
```
$match条件尽可能早地将放在聚合管道中，因为 $match 限制了聚合管道中的文档总数，较早的 $match 操作将管道的处理量降至最低。如果在管道的最开始
放置 $match，则查询可以像任何其他 db.collection.find() 或 db.collection.findOne() 一样利用索引。
例如：
```javascript
//_id:null match管道所有的当做一组
db.books.aggregate( [
    { $match: { $or: [ { copies: { $gt: 1, $lt: 5 } }, { title: "abc123" } ] } },
    { $group: {_id:null,count:{$sum:1}}}
] );
```

#### $sort
对所有输入文档进行排序并按排序顺序将它们返回到管道，$sort 接受一个文档，该文档指定要排序的字段和相应的排序顺序。语法：
```
{ $sort: { <field1>: <sort order>, <field2>: <sort order> ... } }
```
排序的field，最多支持对32个键进行排序。
具有相同排序值的文档，在多次执行中可能执行结果排序不一致。
例子：
```javascript
db.books.aggregate( [
  { $match: { $or: [ { copies: { $gt: 1, $lt: 5 } }, { title: "abc123" } ] } },
  { $sort: {title:1,_id:-1}}
] );
```

#### $unwind
从输入文档中解构一个数组字段以输出每个元素的文档。语法：
```
{ $unwind: <field path> }

```
以前，如果字段路径指定的字段中的值不是数组，则 db.collection.aggregate() 会产生错误，而3.2版本之后则不会报错了
例子：
1. 例如查询_id为1，sizes数组的展开列表：
数据插入:
```javascript
db.inventory.insertOne({ "_id" : 1, "item" : "ABC1", sizes: [ "S", "M", "L"] })
```
```javascript
db.inventory.aggregate([
{
  $match:{_id:1}    
},
{
  $unwind:"$sizes"    
}
])
```
2. 包括数组索引(includeArrayIndex)
数据插入:
```javascript
db.inventory2.insertMany([
  { "_id" : 1, "item" : "ABC", price: NumberDecimal("80"), "sizes": [ "S", "M", "L"] },
  { "_id" : 2, "item" : "EFG", price: NumberDecimal("120"), "sizes" : [ ] },
  { "_id" : 3, "item" : "IJK", price: NumberDecimal("160"), "sizes": "M" },
  { "_id" : 4, "item" : "LMN" , price: NumberDecimal("10") },
  { "_id" : 5, "item" : "XYZ", price: NumberDecimal("5.75"), "sizes" : null }
])
```
```javascript
db.inventory2.aggregate( [
  {
    $unwind:
      {
        path: "$sizes",
        includeArrayIndex: "arrayIndex"
      }
   }])
```
如果该sizes字段不是一个数组，则为null，如_id=3的sizes=M

3. 保留null和空数组(preserveNullAndEmptyArrays)
以下 $unwind 操作使用 preserveNullAndEmptyArrays 选项来包含 size 字段为 null、缺失或空数组的文档。
```javascript
db.inventory2.aggregate( [
{ $unwind: { path: "$sizes", preserveNullAndEmptyArrays: true } }
] )
```

4. 通过展开的字段进行分组
```javascript
db.sales.aggregate([
// First Stage
{
   $match:{_id:{$in:["1","2"]}} 
},
// Second Stage
{
   $unwind:"$items"
},
// Third Stage
{
    $unwind:"$items.tags"
},
//Four Stage
{
    $group:{
       _id:"$items.tags",    
       totalSalesAmount:{
         $sum:{$multiply:["$items.price","$items.quantity"]}
       }       
    }    
}
])
```

5. 


#### $project
将带有请求字段的文档传递到管道中的下一个阶段，指定的字段可以是输入文档中的现有字段或新计算的字段。
```
{ $project: { <specification(s)> } }
```
默认情况下，_id 字段包含在输出文档中。要从输出文档中排除 _id 字段，您必须在 $project 中明确指定抑制 _id 字段。
如果您指定排除一个或多个字段，则在输出文档中返回所有其他字段，如：
```
{ $project: { "<field1>": 0, "<field2>": 0, ... } } // Return all but the specified fields
```
例子：
1. 查询status为"urgent"，只返回productName，quantity字段，不返回id字段
```javascript
db.orders.aggregate([
{$match:{status:"urgent"}},
{$project:{productName:1,quantity:1,_id:0}}
])
```
2. 从输出文档中排除字段productName
```javascript
db.orders.aggregate([
{$project:{productName:0}}
])
```
3. 有条件地排除字段
下面的$project阶段使用REMOVE变量来排除author.middle字段，前提是它等于 ""
插入文档:
```
{
  "_id" : 1,
  title: "abc123",
  isbn: "0001122223334",
  author: { last: "zzz", first: "aaa" },
  copies: 5,
  lastModified: "2016-07-28"
}
{
  "_id" : 2,
  title: "Baked Goods",
  isbn: "9999999999999",
  author: { last: "xyz", first: "abc", middle: "" },
  copies: 2,
  lastModified: "2017-07-21"
}
{
  "_id" : 3,
  title: "Ice Cream Cakes",
  isbn: "8888888888888",
  author: { last: "xyz", first: "abc", middle: "mmm" },
  copies: 5,
  lastModified: "2017-07-22"
}
```
```javascript
db.books.aggregate([
    {
        $project:{
           "title":1,
           "author.first": 1,
           "author.last" : 1, 
           "author.middle": {
                $cond:{
                   if:{$eq:["","$author.middle"]},
                   then: "$$REMOVE",
                   else: "$author.middle"
                }
           }
        }
    }
])
```
4. 包括计算字段 
以下 $project 阶段添加了新字段 isbn、lastName 和 copiesSold：
```javascript
db.books.aggregate(
   [
      {
         $project: {
            title: 1,
            isbn: {
               prefix: { $substr: [ "$isbn", 0, 3 ] },
               group: { $substr: [ "$isbn", 3, 2 ] },
               publisher: { $substr: [ "$isbn", 5, 4 ] },
               title: { $substr: [ "$isbn", 9, 3 ] },
               checkDigit: { $substr: [ "$isbn", 12, 1] }
            },
            lastName: "$author.last",
            copiesSold: "$copies"
         }
      }
   ]
)
```
5. 投影新数组字段
将已有的字段，投影到一个新字段上输出结果，如将author.last，author.first两个字段投影到一个新数组上输出
```javascript
db.books.aggregate( [ { $project: { myArray: [ "$author.last", "$author.first" ] } } ] )
```

#### $addFields


#### $lookup
对同一数据库中的未分片集合执行左外连接，以过滤来自“已连接”集合的文档以进行处理。对于每个输入文档，$lookup 阶段添加一个新的数组字段，
其元素是“加入”集合中的匹配文档。$lookup 阶段将这些重新调整的文档传递到下一个阶段。
1. 等价匹配(Equality Match)
```
{
   $lookup:
     {
       //指定同一数据库中的集合以执行连接。 from 集合不能被分片
       from: <collection to join>,
       //指定从文档输入到 $lookup 阶段的字段，
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}
```
用SQL来理解就是:
```sql
SELECT *, <output array field>
FROM collection
WHERE <output array field> IN (SELECT *
                               FROM <collection to join>
                               WHERE <foreignField>= <collection.localField>);
```




更多操作参考：[聚合管道](https://www.mongodb.com/docs/v4.4/reference/operator/aggregation-pipeline/#std-label-aggregation-pipeline-operator-reference)

## 管道聚合行为
mongo中聚合命令对单个集合进行操作，逻辑上将整个集合传递到聚合管道中，要优化操作，请尽可能使用以下策略来避免扫描整个集合。MongoDB的查询计划器
分析聚合管道以确定是否可以使用索引来提高管道性能。例如，以下管道阶段可以利用索引：
- $match: 如果$match 阶段发生在管道的开头，它可以使用索引来过滤文档。
- $sort：$sort 阶段可以使用索引，只要它前面没有 $project、$unwind 或 $group 阶段。
- $group：如果满足以下所有条件，$group 阶段有时可以使用索引来查找每个组中的第一个文档：
1. $group 阶段之前有一个 $sort 阶段，用于对要分组的字段进行排序
2. 分组字段上有一个与排序顺序匹配的索引，并且$group 阶段使用的唯一累加器是 $first。



## 单一目的聚合操作

单一目的的聚合操作包括：
- db.collection.estimatedDocumentCount()
- db.collection.count()
- db.collection.distinct()

单一用途的聚合方法很简单，但缺乏聚合管道的功能。

# map-reduce
聚合管道比 map-reduce 操作提供更好的性能和可用性,Map-reduce 操作可以使用聚合管道操作符重写，例如 $group、$merge 等.对于需要自定义功能
的map-reduce操作，MongoDB 从版本 4.4 开始提供 $accumulator 和 $function 聚合运算符。使用这些运算符在 JavaScript 中定义自定义聚合表达式。



