@[toc]
# 0.通用命令
## 一些简单通用命令
主要包括keys、dbsize、exists key、del key[key ...] 、expire key seconds、type key 
 -  **keys(redis数据库中所有的键)**
遍历出所有的key
keys一般不建议在生产环境中使用，生产环境的key值比较多，一方面这么多的key值找起来很麻烦，另一方面这个是一个o(n)的操作
keys hel*可以显示出来所有的hel相关的
 - **dbsize(可以算出数据库的大小)**
可以算出key的总数，redis内置的计数器，实时的更新redis中key的总数。时间复杂度是o(1)
 - **exists key**
可以判断一个key值是否存在，key可以是任意数据类型
如 exist a来判断a这个键是否存在
 - **del key[key ...]**   
删除指定的key，value，这个命令可以删除多个的
如删除a：del a，成功删除就是1，错误删除就是0，删除多个会返回删除的数量

![演示](https://img-blog.csdnimg.cn/20191229205640401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)
 - **expire key seconds(为Key设置过期时间)**
key在seconds秒后过期如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229225746685.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)
 
 **ttl key** 
 
 查看当前key剩余的过期时间
 
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229230133926.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)
 
 **persist key**
 去掉key的过期时间
 
 - **type key** 
 返回key的类型，主要类型有string、hash、list、set、zSet、none
 
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229230904522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)

### 时间复杂度比较
命令 | 时间复杂度 |   
-|-
keys | o(n)
dbsize | o(1)  
del| o(1) |
exists| o(1) |
expire| o(1) |
type| o(1) |
## 数据结构和内部编码
**编码方式（encoding）**: string、hash、list、set、sorted set

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191230215054400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)

## 单线程架构 
redis使用了单线程的架构，那为什么使用了单线程速度还是那么快呢？主要原因有一下几点：
 - 纯内存（我们知道数据放入内存中是非常快的）
 - 非阻塞IO
 - 避免线程切换和竞态消耗
 
 主要还是第1条，第2、3条是起到辅助的作用
 **redis使用中需要注意的地方：**
 1.一次只运行一条命令
 2.拒绝长（慢）命令
   keys,flushall,flushdb,slow lua script,mutil/exec, operate big value(collection)
   一般这些命令，比如你用了一个keys，而存在很多key，这样后面的命令就会一直在等待。
3.其实redis也不是绝对的单线程，例如执行以下命令的时候，就会新开一个线程去执行
fysnc file descriptor
close file descriptor
# 1.字符串类型
## 基本结构和命令
对于redis，它的所有键都是字符串，它的值可以多种类型，值可以为字符串、数字、bit形、JSON串等
字符串类型的value是不能大于512MB的，一般建议key、value在100M以内。
### 字符串类型的使用场景
 - 缓存
 - 计数器（如视频网站的计数器，比如每次播放视频）
 - 分布式锁等
### 命令
 - **get(o(1))**
get key获取key指定的value
 - **set(o(1))**
设置key-value
 - **del(o(1))**
del key,删除指定的key
 - **incr(o(1))** 
incr key表示自增1，如果key不存在，自增后get(key)=1
 - **decr(o(1))**
decr key 表示自减1，如果key不存在，自增后get(key)=-1
 - **incrby(o(1))**
incrby key k 表示自增k，如果key不存在，自增后get(key)=k
 - **decrby(o(1))**
decrby key k 表示自减k，如果key不存在，自增后get(key)=-k
## 实战
 - **用redis写个小程序来记录网站每个用户个人主页的访问量？**
incr userId: pageview(单线程：无竞争)，每个userId页面访问量进行了区分，自增，redis是天然适合作为计数器的，因为它是**单线程：无竞争**。
 - **缓存视频的基本信息（数据源在MySQL中）**
 先从缓存中根据Id取数据，如果没有则从数据库中取，并存入缓存中
 - **分布式id生成器**
incr id(原子操作)
 - **set key value(o(1))**
不管key是否存在，	都设置
 -  **setnx key vaue(o(1))**
 key不存在，才设置
 - **set key value xx(o(1))**
key存在才设置，相当于更新操作
 - **mget （o(n)）**
批量获取key,是一个原子操作，如：mget key1 key2 key3...
***n次get=n次网络时间+n次命令时间***，像mget这种操作就可以省去大量的网络时间，***1次mget=1次网络时间+n次命令时间***。使用的时候也要节制去使用。
 - **mset（o(n)）**
 批量设置key-value，mset key1 value1 key2 value2...
## 查缺补漏 
 - **getset key newvalue o(1)**
set key newvalue并返回旧的value 
 - **append key value o(1)**
 将value追加到旧的value
 - **strlen key o(1)**
返回字符串的长度（注意中文）
 - **incrbyfloat o(1)**
增加key对应的值3.5
 - **getrange key start end o(1)**
 获取字符串指定下标所有的值
 - **setrange key index value o(1)**
 设置指定下标所有对应的值
 **字符串命令总结：**
 
 命令| 含义 |复杂度|   
-|-|-
set key value|设置key-value |o(1)
get key | 获取key-value |o(1)
del key| 删除key-value |o(1) 
setnx setxx| 根据key是否存在设置key-value |o(1) 
Incr desc| 计数 |o(1) 
mget mset| 批量操作key-value |o(n) 

# 2.哈希类型
## 重要API
所有Hash的命令都是以h开头的，最简单的三个命令**hget、hset、hdel**

 - **hget key field (o(1))**
获取hash key 对应的field的value
 - **hset key field value（o(1)）**
设置hash key 对应field的value
 - **hdel key field(o(1))**
删除hash key 对应field的value
 - **hexists key field(o(1))**
判断hash key是否有field
 - **hlen key(o(1))**
获取hash key field的数量
 - **hmget key field1 field2 ... fieldN(o(n))**
 批量获取hash key 的一批field对应的值
 - **hmset key field1 value1 field2 value2...fieldN valueN(o(n))**
 批量设置hash key的一批field value
## hash实战
 - 记录网站每个用户个人主页的访问量？
使用的命令：**hincrby user:1:info pageview count** 
## hash vs string
 string| hash|
-|-
get|hget
set setnx| hset hsetnx
incr incrby decr decrby| hincrby
del| hdel
mset| hmset
mget| hmget
## 查缺补漏
 - **hgetall key(o(n))**
返回hash key对应所有的field和value
 - **hvals key(o(n))**
返回hash key对应所有field的value
 - **hkeys key(o(n))**
 返回hash key对应所有field
 
 - **hsetnx（o(1)）**
hsetnx key field value：设置hash key对应field的value（如field已经存在，则失败）
 - **hincrby（o(1)）**
hincrby key field intCounter： hash key对应的field的value自增intCounter
 - **hincrbyfloat（o(1)）**
hincrbyfloat key field floatCounter
hincrby浮点数版本

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200103223615507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)

## hash总结
 命令| 复杂度|
-|-
hget、hset、hdel|o(1)
hexists| o(1)
hincrby| o(1)
hgetall、hvals、hkeys| o(n)
hmget、hmset|o(n)
# 3.列表类型
## 特点
有序、可以重复、左右两边插入弹出
## 重要API

 - **rpush（o(1~n)根据实际插入的元素个数来计）**
rpush key value1 value2 ...valueN
从列表右端插入值（1-N个）

 -  **lpush（o(1~n)根据实际插入的元素个数来计）**
从列表左端插入值

 - **linsert(o(n))**
linsert key before | after value newValue
在list指定的值前|后插入newValue，它的时间复杂度是o(n)因为它要遍历这个列表
用法，比如：a->b->c->d
执行linsert listkey before b java之后
a->java->b->c->d

 - **lpop key(o(1))**
从列表左边弹出一个item
 - **rpop key(o(1))**
 从列表右边弹出一个item
 - **lrem(o(n))**
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105133706554.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)

 - **ltrim	key start end(o(n))**
按照索引范围修剪列表

 - **lrange（o(n)）**
lrange key start end(包含end)，获取列表指定索引范围所有item

 - **lindex(o(n))**
 lindex key index
 获取列表指定索引的item

 - **llen（o(1)）**
llen key:算出列表的长度

 - **lset(o(n))**
lset key index newValue:设置列表指定索引值为newValue
## 应用
比如微博中，你关注的人更新的微博动态，lpush
## 查缺补漏
 - **blpop（o(1)）**
blpop key timeout
lpop阻塞版本，timeout是阻塞超时时间，timeout=0为永远不阻塞	
 - **brpop (o(1))**
brpop key timeout
rpop 阻塞版本，timeout是阻塞超时时间，timeout=0为永远不阻塞
## Tips	
 - LPUSH + LPOP = Stack
 - LPUSH + RPOP = Queue
 - LPUSH + LTRIM = Capped Collection
 - LPUSH + BRPOP = Message Queue
# 4.集合类型
## 特点
无序、无重复元素、集合间操作
所有的API都是以s开头
## 集合内API和实战
 - **sadd（o(1)）**
sadd key element
向集合key添加element（如果element已经存在，添加失败）

 - **srem key element(o(1))**
将集合key中的element移除掉

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105150711684.png)

 - **scard user:1:follow**
计算集合大小

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105150412448.png)

 - **sismember user:1:follow it (存在)**
判断it是否在集合中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105150331831.png)

 - **srandmember user:1:follow count**  
从集合中随机挑count个元素

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020010514560954.png)

 - **spop user:1:follow** 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105145336769.png)

从集合中随机弹出一个元素
 - **smembers user:1:follow**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105145221240.png)

获取集合所有元素，如果集合中有非常多的数据要注意，**因为redis是单线程的，这样会阻塞。**
**注意：**
spop从集合中弹出元素，srandmember不会破坏集合
## 应用
抽奖系统、like、赞、踩、标签（tag）
## 集合间API和实战
比如说微博中共同关注的好友呀，共同关注的兴趣呀，就会用到set数据结构
 - sdiff:差集
 - sinter：交集
 - sunion：并集

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105152953476.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)

## Tips
SADD = Tagging
SPOP/SRANDMEMBER = Random item
SADD + SINTER = Social Graph
# 5.有序集合类型
## 特点
按照scrore分值来组成有序的集合、有序集合都是以z开头的命令
## 重要API
 - **zadd(o(logN))**
zadd key core element(可以是多对)，添加score和element
 - **zrem(o(1))**
 zrem key element(可以是多个)
 删除元素
 - **zscore（o(1)）**
zscore key element
返回元素的分数
 - **zincrby(o(1))**
zincrby key increScore element
增加或减少元素的分数
 - **zcard（o(1)）**
zcard key
返回元素的总个数
 - **zrank**
 获取某个元素的排名
 - **zrange （o(log(n) + m)）**
zrange key start end [WITHSCORES]
返回指定索引范围内的升序元素[分值]
**WITHSCORES**：是否将元素的分值进行打印

 - **zrangebyscore o(log(n) + m)**
zrangebyscore key minScore maxScore[WITHSCORES]
返回指定分数范围内的升序元素[分值]

 - **zcount (o(log(n) + m))**
zcount key minScore maxScore
返回有序集合内在在指定分数范围内的个数
 - **zremrangebyrank(o(log (n) + m))** 
 zremrangebyrank key start end
 删除指定排名内的升序元素
 - zremrangebyscore(o(log(n) + m))
zremrangebyscore key minScore maxScore

**操作：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200105160734441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)

## 实战
排行榜（销售榜、音乐排行榜、图书排行榜）
## 查缺补漏
 - zrevrank 
 - zrevrange
 -  zrevrangebyscore
 - zinterstore
 - zunionstore
## 有序集合总结
 操作类型| 命令|
-|-
基本操作|zadd、zrem、zcard、zincrby、zscore
范围操作| zrange、zrangebyscore、zcount、zremrangebyrank
集合操作| zunionstore、zinterstore
