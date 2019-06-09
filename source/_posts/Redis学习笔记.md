---
title: Redis学习笔记
date: 2019-04-18 21:24:22
tags: 学习笔记
categories: Redis
---
# <center>入门</center>
**注意点**
1. Redis的数据类型不支持类型嵌套，比如集合类型的每个元素都只能是字符串，不能是另外一个集合或者散列类型等。

## 1、获得符合规则的键名列表

> KEYS pattern

pattern支持glob通配符格式，见下表

| 符号 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| ?    | 匹配一个字符                                                 |
| \*    | 匹配任意个(包括0个字符)                                      |
| []   | 匹配括号中间的任一字符，可以使用“-”符号表示一个范围，如a[b-d]匹配“ab”,“ac”,“ad” |
| \x   | 匹配字符x，用于转义符号，如果要匹配“?”就需要使用\?           |

<!--more-->

## 2、redis数据类型

### 2.1 字符串类型

- SET key：设置键

- Get key：获取键

- INCR key : 递增数字

- incrby key increment：增加指定的整数

- decr key|decrby key decrement ： 减少指定的整数

- Incrbyfloat key 2.7 :增加指定的浮点数

- append key value：向尾部追加值

- strlen key:获取字符串长度

- mget key [key…]：同时获得多个键值

**位操作：**

- getbit key offset：获取该key的第N位二进制运算符，如果超出实际长度则默认为0

- Setbit key offset value：替换第N位的二进制运算符，如果超出实际长度默认会自动将中间的二进制位设置为0，同理设置一个不存在的键的制定二进制位的值会自动将其前面的位赋值位0

- Bitcount 获取字符串类型键中值是1的二进制位的个数

bitop or res key1 key2 ： 对多个字符串类型键进行位运算，并将结果存储在destkey参数指定的键中，bitop支持的运算操作有and，or，xor和not。

### 2.2 散列类型
**散列类型不能嵌套其他的数据类型**
- hset key field value : 设置散列类型某个列的值
- hget key field value : 获取散列类型某个列的值
- hgetall key 获取散列类型所有的键和值
- hexists key field : 判断字段是否存在
- hsetnx key field ：当字段不存在时赋值 ，**原子操作**
- hincrby key field increment：增加数字
- hdel key field [field...]：删除一个或多个字段
- hkeys key ：获取所有字段
- hvals key ：获取所有值
- hlen key ： 获取字段数量

### 2.3 列表类型
- LPUSH ： 向列表左边插入元素
- RPUSH ：向列表右边插入元素
- LPOP ： 从列表左边弹出元素
- RPOP ： 从列表右边弹出元素
- LRANGE ： 查看列表的元素 lrange key start end
- LINSERT : 在某个元素前后插入数据 linsert key before|after pivot value 
- LTRIM ： ltrim key srart end
- LINDEX ： 获取列表下表元素的值 lindex key index
- LSET ： 将索引为index的元素赋值为value lset key index value
- llen ： 获取元素列表长度
- LREM ： 删除列表中前count个值为value的元素 lrem key count value 
1. 当count > 0时会从列表左侧开始删除前count个值为value的元素
2. 当count < 0时会从列表右侧开始删除前count个值为value的元素
3. 当count = 0时会删除所有值为value的元素
### 2.4 集合类型
- SADD ： 向集合插入元素
- SREM ： 删除集合中的元素
- SMEMBERS ： 获取集合中的所有元素
- SISMEMBER ： 集合中是否存在某元素
- SDIFF ： 获取集合中不同的元素
- SINTER ： 获取集合中相同的元素
- SUNION ： 获取集合的并集 
- CARD ：获取元素中的个数
- SDIFFSTORE destination key ： 与SDIFF命令功能一样，区别是前者不会直接返回运算值，而是直接将结果存储在destination键中，SINTERSTORE 和 SUNIONSTORE 命令与之类似
- SRANDMEMBER ： 随机获取集合中的一个元素
### 2.5 有序集合类型
- zadd key score member :增加元素
- zscore key member ：获取元素分数
- zrange key start stop [withscores]：按元素的分数从小到大返回索引从start到stop之间的所有元素（包含两端的元素），如果需要分数的话同时在尾部加上WITHSCORES,**zrevrange** 索引从大到小排序 
- zrangebyscore key min max [withscores]  [limit offset count]:获取分数之间从小到大的顺序
eg:zrevrangebyscore scoreboard 100 -inf withscores limit 0 3，从大到小
- zincrby key increment member:增加一个元素的分数
- zcard key ：获取元素中的数量
- zcount key min max : 获取指定分数范围内的元素个数
- zrem key member ： 删除一个或多个元素
- zremrangebyrank key start stop : 按照排名范围删除元素
- zremrangebyscore key min max ： 按照分数范围删除元素
- zrank key member ：获得元素的排名 zrevrank从大到小
- zinterstore destination numkeys key [key...]  [weight weight[weight...]  [aggergate SUM|MIN|MAX]
1. 当aggergate 时SUM时，destination键中元素的分数时每个参与计算的集合中该元素的分数的和；为MIN时，destination键中元素的分数时每个参与计算的集合中该元素的分数的最小值;为MAX时，destination键中元素的分数时每个参与计算的集合中该元素的分数的最大值
2. weight表示每个集合在参与计算时元素的分数会被乘上该集合的权重
- zunionstore与上相同
# <center>进阶</center>

## 1、redis事务

### 1.1 命令

``` 
MULIT // 事务开始
SADD "user:1" 2
SADD "user:2" 1
EXEC  // 事务结束
```

MULIT命令告诉Redis，我下面发给你的命令属于一个事务，你先不要执行，而是把它们暂存起来，当使用EXEC时，Redis会将等待执行的事务队列中的命令按发送顺序执行，返回值顺序和命令顺序相同。

### 1.2 错误处理

1. 语法错误：Redis2.6.5之前的版本会忽略语法错误的命令，然后执行事务中的其他命令；之后的版本只要有一个命令有语法错误，执行EXEC命令后Redis会直接返回错误，连语法正确的命令也不会执行。
2. 运行错误：假如用散列类型的命令操作集合类型的键，这种错误在实际执行前Redis是无法发现的，所以事务里的这种命令是会被Redis接受执行的。如果事务里的一条命令出现了运行错误，那么其他包括出错命令之后的命令依然会被执行。

### 1.3 WATCH 命令

watch命令可以监控一个或多个键，一旦其中有任意一个键被删除，之后的事务就不会被执行。监控一直持续到EXEC命令。

**注意：事务中的命令是exec之后才执行的，所以在MULTI命令之后是可以修改WATCH监控的值的，但是如果事务之前修改过WATCH监控的键值的话，之后的MULTI命令则不会修改。**

## 2、过期时间

expire key second 设置key的存活时间，单位是秒；

TTL可以查看一个键还有多长时间被删除，当键不存在返回-2，没有设置过期时间返回-1；

>   在2.6版本中，无论键不存在还是没有设置过期时间都是返回-1，知道2.8版本之后在区分两种情况。
PERSIST：取消键的过期时间，或者用SET或GETSET命令为键复制也会清除键的过期时间；
## 3、 缓存

修改配置文件中的maxmemory参数，限制Redis最大可用内存大小（单位是字节），当超出了这个限制时，Redis会根据maxmemory-policy参数制定的策略来删除不需要的键知道Redis占用的内存小于指定内存

maxmemory-policy支持的规则如下，其中的LRU（Least Recently Used）算法即“最近最少使用”

| 规则            | 说明                                            |
| :-------------- | :---------------------------------------------- |
| volatile-lru    | 使用LRU算法删除一个键（只对设置了过期时间的键） |
| allkeys-lru     | 使用LRU算法删除一个键                           |
| volatile-random | 随机删除 一个键（只对设置了过期时间的键）       |
| allkeys-random  | 随机删除一个键                                  |
| volatile-ttl    | 删除过期时间最近的一个键                        |
| noeviction      | 不删除键，只返回错误                            |

如当maxmemory-policy设置为allkeys-lru时，一旦Redis占用的内存超过了限制值，Redis会不断的删除数据库中最近最少使用的键，知道占用的内存小于限制值。

>   事实上Redis 并不会准确的删除整个数据库中最久未被使用的键，而是每次从数据库中随机取3个键并删除这3个键中最久未被使用的键。3这个数字可以通过Redis配置文件中的maxmemory-samples参数设置。

## 4、排序
### 4.1 有序集合的排序
```
multi
zinterstore tempkey ...
zrange tempkey
del tempkey
exce
```

### 4.2 SORT命令

sort key DESC limit 1 3

### 4.3 BY参数
sort key by key:*->field desc

**注意***

1. 参考类型虽然支持散列类型，但是*只能在->符号的前面（即键名部分才有用），在->后字段部分会被当成字段名本身而不会作为占位符被元素的值替换，即常量名。但实际运行会发现一个有趣的效果：

   ```
   sort sortbylist by somekey->somefield:*
   "1"
   "2"
   "3"
   "4"
   "5"
   ```

   上面仅对元素本身进行了排序，原因是Redis判断参考键名是不是常量键名的方式是判断参考键中是否包含“\*”，而somekey->somefield:\*中包含“\*”，所以不是常量名名，所以在排序的时候会对每个元素都读取somekey->somefield:\*值，\*不会被替换掉，无论是否能获取值，每个元素的参考键值是相同的，所以redis会按照元素本身的大小排序。

2. 如果要对散列类型的键值通过其键值的value进行排序，需先将散列类型的键放到一个集合中，再排序。



### 4.4 GET参数
sort key by key->field desc get key->field [get key-field...]
get # 会返回元素本身的值
### 4.5 store参数
sort key by key->field desc get key->field [get key-field...]
get # store result
将排序后的结果保存在result键中，保存后的键为列表类型，如果键已经存在会覆盖，加上store参数后sort命令的返回值为结果的个数

----

未完待续


    


















