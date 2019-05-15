---
title: Redis学习笔记
date: 2019-04-18 21:24:22
tags: 学习笔记
categories: 学习笔记
---
# <center>入门</center>

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


### 2.2 散列类型


### 2.3 列表类型

- LPUSH ： 向列表左边插入元素
- RPUSH ：向列表右边插入元素
- LPOP ： 从列表左边弹出元素
- RPOP ： 从列表右边弹出元素
- LRANGE ： 查看列表的键 lrange key start end
- LINSERT : 在某个元素前后插入数据 linsert key before|after pivot value 
- LTRIM ： ltrim key srart end
- LINDEX ： 获取列表下表元素的值 lindex key index
- LSET ： 将索引为index的元素赋值为value lset key index value
- LREM ： 从右边删除第一个值为value的元素 lrem key -1 value 

### 2.4 集合类型
- SADD ： 向集合插入元素
- SREM ： 删除集合中的元素
- SMEMBERS ： 获取集合中的所有元素
- SISMEMBER ： 集合中是否存在某元素
- SDIFF ： 获取集合中不同的元素
- SINTER ： 获取集合中相同的元素
- SUNION ： 获取集合的并集 
- CARD ：获取元素中的个数
- SDIFFERSTRE destination key ： 与SDIFF命令功能一样，区别是前者不会直接返回运算值，而是直接将结果存储在destination键中
- SINTERSTORE 和 SUNIONSTORE 命令与之类似
- SRANGMEMBER ： 随机获取集合中的一个元素
\- 
### 2.5 有序集合类型

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















