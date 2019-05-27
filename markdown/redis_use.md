---
title: "Redis_use"
date: 2019-05-21T21:56:26+08:00
tags:
- redis
---



​		redis入门，这遍博客主要是记录redis的基础使用。介绍了安装过程以及五种数据结构的基础使用。

<!--more-->

### Redis安装

我是在阿里云的centos7上面进行部署的。

````
yum update
yum install redis
````

​		redis服务器默认会使用6379端口，可以通过--port参数来自定义端口。通过redis-server来启动服务器，不过可以最好是配置redis-server默认是后台运行。这样就可以直接通过redis-cli来直接使用了。关闭是redis-cli SHUTDOWN。

### Redis简介

​		Redis是Remote Dictionary Server（远程**字典**服务器）的缩写。允许TCP协议读写字典中的内容。到目前支持的**键值**类型为：

> 1. 字符串类型
> 2. 散列类型
> 3. 列表类型
> 4. 集合类型
> 5. 有序集合类型

​		数据全部存储在**内存**中，由于内存的读写远快于硬盘，因此在性能上具体有明显的优势，不过这也存在程序退出的时候内存数据会丢失的问题。不过Redis也提供了持久化的支持，既可以将内存中的数据异步写入磁盘

## 基础操作

​		就目前，我对redis理解来说。redis的操作主要是分为两个部分，一个是独立于数据类型的操作，一个是对通用的操作。虽然在命令上，并不是一直的，但是在逻辑上是一致的。虽然不知道为什么要这样设计。可能是为了避免数据的误操作吧。

#### 脱离具体类型的操作

- KEYS pattern

> 作用是获取符合规则的键名列表。pattern是支持glob风格通配符格式。
>
> | 符号 | 含义                                                         |
> | ---- | ------------------------------------------------------------ |
> | ？   | 匹配一个字符                                                 |
> | *    | 匹配任何一个（包括0个）字符                                  |
> | []   | 匹配括号间的任一个字符，可以使用“-”符号表示一个范围，如a[b-d]可以匹配“ab”、“ac”和“ad”; |
> | \x   | 匹配字符x，比如？就需要\?                                    |

- TYPE key

> TYPE 命令主要是获取键指的数据类型，可能是string | hash | list | set | zset。

#### 抽象操作

这里总结的可能有问题

| 数据类型 | 判断存在 | 添加键 | 删除键 | 获取键值 | 递增整数 | 减少整数 |
| -------- | -------- | ------ | ------ | -------- | -------- | -------- |
| 字符串   | EXISTS   | SET    | DEL    | GET      | INCR     | DECR     |
| 散列     | HEXISTS  | HSET   | HDEL   | HGET     |          |          |



### 字符串类型

​		字符串是Redis中最基础的数据类型，能存储任何形式的字符串。字符串也是其他4中数据类型的基础，其他的数据类型和字符串类型的差别从一定角度来说只是组织的形式不同。

##### 赋值与取值

+ SET KEY value
+ GET KEY

##### 同时获取／设置多个键指

- MGET key [key …]
- MSET key value [key value …]

##### 递增与减少数字

+ INCR key
+ INCRBY key increment
+ DECR key
+ DECRBY key decrement
+ INCRBYFLOAT key increment

##### 获取字符串长度

+ STRLEN key

+ 

##### 位操作

+ SETBIT key offset 0/1

> 当offer大于长度的时候，中间会被填充为0

+ GETBIT key offset
+ BITCOUNT foo

> 获取字符串类型键中是1的二进制个数。

+ BITPOS

### 散列类型

​		散列类型（hash）的键值也是一种字典结构，其存储了**字段（field）和字段值的映射**，但**字段值只能是字符串**，不支持其他数据类型，换句话说，散列类型不能嵌套其他的数据类型。

![example](/images/redis_hash.png)

##### 赋值与取值

+ HSET key field value
+ HSETNX key field value

> 与HSET类似，但是只有字段不存在的时候赋值。

+ HGET key field
+ HGETALL
+ HKEYS   key

> 只获取字段名

+ HVALS   key

> 只获取字段值

+ HLEN key

> 获取字段（field）数量

##### 同时获取／设置多个键指

+ HMSET key field value [field value …]

+ HMGET key field [field …]

##### 判断**字段**存在

+ HEXISTS key field

> EXISTS，是用来判断键是不是存在的，而HEXISTS是用来判断HASH类型里面field是不是存在的。

##### 递增与减少数字

+ HINCRBY key field increment

##### 删除字段

+ HDEL key field [field …]

### 列表类型

​		列表类型（list）可以存储一个有序的字符串列表，常用的操作是向列表的两端添加元素，或者是获得列表的某一个片段。内部实现的使用双向链表(double linked list)实现的。借助这个特点，使得Redis可以作为队列使用。

##### 向列表两端增加元素

+ LPUSH key value [value …]
+ RPUSH key value [value …]

##### 从列表两端弹出元素

+ LPOP key

+ RPOP key

##### 获取列表中元素的个数

+ LLEN key

> LLEN( list len )的时间复杂度为O(1)，使用时Redis会直接读取现成的值

##### 获得列表片段

+ LRANGE key start stop

> LRANGE( list range)命令也支持负索引，表示从右边开始计算序数，如"−1"表示最右边第一个元素，"-2"表示最右边第二个元素
>
> 如果start的索引位置比stop的位置靠后，则会放回空列表。
>
> 如果stop大于实际的索引范围，则会放回最右边的元素。

##### 删除列表中指定的值

+ LREM key count value

> LREM(list remove)命令会删除列表中前count个值为value的元素，返回值是实际删除的元素个数。根据count值的不同，LREM命令的执行方式会略有差异。
>
> （1）当 count > 0时 LREM 命令会从列表左边开始删除前 count 个值为 value的元素。
>
> （2）当 count < 0时 LREM 命令会从列表右边开始删除前|count|个值为 value 的元素。
>
> （3）当 count = 0是 LREM命令会删除所有值为

##### 获得/设置指定索引的元素值

+ LINDEX key index

+ LSET key index value

##### 只保留列表指定片段

+ LTRIM key start end

> LTRIM 命令可以删除指定索引范围之外的所有元素

##### 向列表中插入元素

+ LINSERT key BEFORE|AFTER pivot value

> LINSERT 命令首先会在列表中从左到右查找值为 pivot 的元素，然后根据第二个参数是BEFORE还是AFTER来决定将value插入到该元素的前面还是后面。

lzk_baggk

> 在STL里面一般区间是[ ，)的，最后一个元素是一个end元素，所以只需要插入在元素的前面就可以了。但是redis的区间是[，]的。这样的话，就需要指定前后，不然第一个或者是最后一个就无法插入。

##### 将元素从一个列表转到另一个列表

+ RPOPLPUSH source destination

> 先执行RPOP命令再执行LPUSH命令。RPOPLPUSH命令会先从source列表类型键的右边弹出一个元素，然后将其加入到destination列表类型键的左边，并返回这个元素的值，整个过程是原子的

### 集合类型

##### 增加/删除元素

+ SADD key member [member …]

+ SREM key member [member …]

##### 获得集合中的所有元素

+ SMEMBERS key

+ SISMEMBERS key member

> 判断一个元素是否在集合中是一个时间复杂度为O(1)的操作，无论集合中有多少个元素

##### 集合间运算

+ SDIFF key [key „]
+ SDIFFSTORE destination key [key …]

> 差集

+ SINTER key [key „]
+ SINTERSTORE destination key [key …]

> 交集

+ SUNION key [key „]
+ SUNIONSTORE destination key [key …]

> 并集

##### 获取集合中元素的个数

+ SCARD key

> 1）当count为正数时，SRANDMEMBER会随机从集合里获得count个不重复的元素。如果count的值大于集合中的元素个数，则SRANDMEMBER会返回集合中的全部元素。
>
> （2）当count为负数时，SRANDMEMBER会随机从集合里获得|count|个的元素，这些元素有可能相同。

##### 随机获得集合中的元素

+ SRANDMEMBER key [count]

##### 从集合中弹出一个元素

+ SPOP key

### 有序（sorted set）集合

> 在实现上，使用的是散列表和跳表实现的，所以即使读取位于中间部分的数据速度也比较快。

##### 增加元素

+ ZADD key score member [score member …]

##### 获得元素的分数

+ ZSCORE key member

##### 获得排名在某个范围的元素列表

+ ZRANGE key start stop [WITHSCORES]

+ ZREVRANGE key start stop [WITHSCORES]

> ZRANGE命令的时间复杂度为O(log n+m)（其中n为有序集合的基数，m为返回的元素个数）如果两个元素的分数相同，Redis会按照字典顺序（即"0"<"9"<"A"<"Z"<"a"<"z"这样的顺序）

##### 获得指定分数范围的元素

+ ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

##### 获得集合中元素的数量

+ ZCARD key

##### 获得指定分数范围内的元素个数

+ ZCOUNT key min max

##### 删除一个或多个元素

+ ZREM key member [member …]

+ ZREMRANGEBYRANK key start stop

> 按照排名范围删除元素

+ ZREMRANGEBYSCORE key min max

> 按照分数范围删除元素

##### 获得元素的排名

+ ZRANK key member

+ ZREVRANK key member

##### 计算有序集合的交集

ZINTERSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATESUM|MIN|MAX]

## 事务

​		Redis保证一个事务中的所有命令要么都执行，要么都不执行。如果在发送EXEC命令前客户端断线了，则 Redis 会清空事务队列，事务中的所有命令都不会执行。而一旦客户端发送了EXEC命令，所有的命令就都会被执行，即使此后客户端断线也没关系，因为Redis中已经记录了所有要执行的命令。

> redis> MULTI
>
> OK
>
> redis> SADD "user:1:following" 2
>
> QUEUED
>
> redis> SADD "user:2:followers" 1
>
> QUEUED
>
> redis> EXEC
>
> 1) (integer) 1
>
> 2) (integer) 1

##### WATCH

WATCH 命令可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行。监控一直持续到EXEC命令。

> redis> SET key 1
>
> OK
>
> redis> WATCH key
>
> OK
>
> redis> SET key 2
>
> OK
>
> redis> MULTI
>
> OK
>
> redis> SET key 3
>
> QUEUED
>
> redis> EXEC
>
> (nil)
>
> redis> GET key
>
> "2"

​		由于WATCH命令的作用只是当被监控的键值被修改后阻止之后一个事务的执行，而不能保证其他客户端不修改这一键值，所以我们需要在EXEC执行失败后重新执行整个函数。

执行 EXEC 命令后会取消对所有键的监控，如果不想执行事务中的命令也可以使用UNWATCH命令来取消监控。

## 过期

​		而在Redis中可以使用 EXPIRE命令设置一个键的过期时间，到时间后Redis会自动删除它。

+ EXPIRE key seconds

+ PERSIST key
+ TTL