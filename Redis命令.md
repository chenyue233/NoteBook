## Redis命令
### 基础命令

#### 通配符

| 符号 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| ?    | 匹配一个字符                                                 |
| *    | 匹配任意个字符                                               |
| []   | 匹配括号间的任一字符，可以使用 `“-”` 符号表示一个范围，例如 `[a-d]` 等价于 `[abcd]` |
| \x   | 转义符号 `x`，例如要匹配 `“?”`，则需要使用 `\?`              |

​	例:   
- 查看所有键  
  `KEYS *`

- 查看名字包含name的键  

  `KEYS *name*`

- 查看名字以a开头的且长度为3的键

  `KEYS a??`

#### 检查键是否存在

```redis
EXISTS Key1
结果:(interger)1 存在
EXISTS nosuchkey 
结果:(interger)0 不存在
```

#### 删除键

`DEL key1 key2`

#### 查看键的数据类型

`TYPE key1`

结果：成功返回数据类型，不成功返回none

#### 重命名键

`RENAME oldname newname`

结果:ok ，如果重命名之前，newname已存在，那么newname会被覆盖

防止被覆盖:

`RENAMENX oldname newname`

结果:ok成功，0代表newname已存在

#### 随机获取一个键

`RANDOMKEY` 

结果:从当前数据库随机返回一个键，如果数据库是空则返回nil



### 键过期

#### 置顶键的存活时间

`EXPIRE key seconds`,表示key在seconds秒后过期

例: `EXPIRE foo 3`

毫秒:PEXPIRE

#### 查看键的存活时间

`TTL KEY`

返回值:

- 大于等于0的整数:键存活时间，单位秒
- -1：键没设过期时间
- -2：键不存在

例：TTL foo

`PTTL key`返回的单位是毫秒



#### 键在指定的时间戳过期

`EXPIREAT key timestamp`,表示键在秒级别时间戳timestamp后过期。例如将键foo设置在2018-05-10 23:30:00(秒级时间戳为 1525966200）后过期：

例：

`EXPIREAT foo 1525966200`

毫秒：PEXPIREAT

####  注意事项

- 键不存在

  如果`EXPIRE`命令指定的键不存在，返回结果为0:

- 指定负值时间

  如果过期时间为负值，键会被立即删除，就像使用del命令一样

  为EXPIREAT命令指定一个过去的时间也是如此  

#### 清楚过期时间

PERSIST 命令可已清除过期时间:

例:

`PERSIST foo`

#### SET命令

<font color=#FF0000 >对于字符串类型的键，执行SET命令会清除键的过期时间</font>，这个问题在开发过程中很容易被忽视

#### 二级数据结构

Redis 不支持二级数据结构（例如哈希、列表）内部元素的过期功能，例如不能对列表的一个元素做过期时间操作

#### SETEX命令

`SETEX key seconds value`

例: SETEX cache_user_id 60 10086

SETEX命令作为SET+EXPIRE的组合，不但是原子执行，同时也减少了一个网络开销



### 字符串

字符串类型是 Redis 最基础的数据结构。字符串类型的值可以存储任何形式的字符串：JSON、XML、数字、图片等。一个字符串类型键允许存储的数据的最大容量是<font color=#FF0000 > 512 MB</font>



#### 常用字符串命令

#### 赋值

` SET key value [EX seconds] [PX milliseconds] [NX|XX]`

例:SET foo "hello"

Set命令有几个选择：

-EX seconds：为键设置秒级过期时间。
-PX milliseconds：为键设置毫秒级过期时间
-NX：键必须不存在，才可以设置成功，用于添加键。
-XX：与 NX 相反，键必须存在，才可以设置成功，用于更新键。

除了这些选项，Redis 还提供了 SETEX、PSETEX 和 SETNX 等命令。

#### 为键赋值同时设置秒级过期时间

`SETEX key seconds value`

在为键赋值的同时为键设置秒级过期时间

PSETEX 命令与 SETEX 命令类似，但它为键设置的是毫秒级过期时间

#### 键不存在才能赋值成功 

`SETNX key value`

键必须不存在，才能成功赋值。当赋值成功时结果返回 1，否则结果返回 0



#### 取值

`GET key`

如果键不存在，那么结果返回 nil。如果键存在但它的数据类型不是字符串，则会返回一个错误。

#### 批量赋值

`MUST key value [key value ...]`

例： MSET a "1" b "2" c "3"

#### 批量赋值，要求键都不存在

`MSETNX key value [key value ....]`

要求所有的键都不存在，只要有一个键已存在，则所有的键都不会被赋值

MSETNX 命令是原子的，所以 在 Redis 客户端不会看到部分键先被赋值而部分键还未被赋值的情况

#### 批量取值

`MGET key [key ...]`

例:MGET c b a

### 计数

INCR命令

`INCR key`

INCR 命令用于对键值做自增操作,返回结果分为三种情况

-键值不是整数，返回错误
-键值是整数，返回自增后的结果
-键不存在，按照值为 0 自增，返回结果是 1

#### 其他计数命令

除了INCR命令，Redis还提供了几个计算操作的命令:

-DECR 命令用于自减操作，它的用法是`DECR key`
-INCRBY命令可以自增指定数值，它的用法是`INCRBY key increment`
-DECRBY 命令可以自减指定数值，它的用法是`DECRBY key decrement`
-INCRBYFLOAT 命令可以自增浮点数，它的用法是`INCRBYFLOAT key increment`

#### 不常用字符串命令

#### 追加值

APPEND 命令可以向字符串尾部追加值，它的用法是：APPEND key value

如果指定的键不存在，则将该键设置为空字符串再执行追加操作，这样就与set命令类似

#### 取字符串长度

`STRLEN`命令可以获取键值的长度，它的用法是`STRLEN key`

UTF8编码的中文字符占三个字节

#### 复制并返回原值

GETSET命令与SET命令一样可以为键赋值，但同时返回键原来的值。它的用法是`GETSET key value`

#### 覆盖指定位置的字符

SETRANGE命令从指定的偏移位置开始覆盖字符串值，它的用法是`SETRANGE key offset value`

#### 获取字符

GETRANGE命令可以获取字符串的长度，它的用法是`GETRANGE key start end`。start和end分别是开始和结束的偏移量，偏移量是从0开始计算，当偏移量是负数时，表示从右至左数起。



### 哈希

#### 常用的命令

#### 为哈希字段赋值

`HSET key field value`  当指定键或者字段不存在时，则会自动创建该键或字段

例:HSET user:1 name huey

#### 字段不存在时才能赋值成功

`HSETNX key field value` 要求只有该字段不存在时，才能赋值成功。

例 :HSETNX user:1 name "huey"

#### 取值

`HGET key field`

例：HGET user:1 name

#### 批量赋值

`HMSET key field value [field value]`

例：HSMET user:1 name "hury" age 18

#### 批量取值

`HMGET key field [field..]`  结果按字段顺序返回，不存在返回nil

例: HMGET user:1 name age gender

#### 获取键的所有键值对

`HGETALL key` 支持一次性返回哈希键上所有的键值对

例：HGETALL user:1

ds

<font color=#FF0000 >注:</font> 使用HGETALL命令时，如果哈希元素个数比较多，会存在堵塞redis的可能。如果只需要获取部分字段，使用HMGET命令。如果一定要获取全部的键值对，可以使用`HSCAN`命令，该命令可以渐进式遍历哈希类型

#### 判断字段是否存在

`HEXISTS key field` 如果键包含指点的字段，那么结果返回1，否则返回0

#### 删除字段

`HDEL key field [field...]` 用户删除哈希键上的一个或多个字段，返回成功删除字段的数量

例: HDEL user:1 age gender

#### 获取字段的数量

`HLEN key`  返回指定的哈希键有多少个字段

例：HLEN user:1



#### 不常用的命令

#### 获取所有的字段名

`HKEYS key` 返回哈希键上的所有字段

 #### 获取所有的字段值

`HVALS key` 返回指定哈希键上所有字段值

#### 获取字段的长度

`HSTRLEN key field` 返回指定字段的长度

例:HSTRLEN user:1 name

#### 字段值计数

`HINCRBY key field increment` 支持哈希字段自增指定整数值

`HINCRBYFLOAT key field increment` 命令支持哈希字段自增指定浮点数





### 列表

列表类型是用来存储多个有序字符串，列表中的每个字符串成为元素，一个列表最大可以存储 2的32次方-1个元素。列表类型有两个特点

1、列表中的元素是有序的

2、列表中的元素可以是重复的

#### 常用的命令

#### 从列表右端插入元素

`RPUSH key value [value...]`往指定的列表右端插入元素，如果指定了多个元素，这些元素将按指定的顺序依次插入列表的右端

`LRANGE key 0 -1`命令能够从左至右获取列表的所有元素

#### 从列表左端插入元素

`LPUSH key value [value...]` 与RPUSH类似，只不过往列表左端插入元素

#### 在某个元素的前面或后面插入元素

`LINSERT key BEFORE|AFTER pivot value`  它会在列表中找到与pivot相等的元素，在其前面或后面插入元素value，并返回当前列表当长度作为结果。如果在列表中找不到与 pivot 相等的元素，则返回 -1。

#### 查找元素

`LINDEX key index` 用户获取指定索引下标的元素。<font color=#FF0000 >redis列表的索引下标从左往右分别是0到N-1，从右往左分别是-1到-N</font>

#### 获取指定范围内的元素列表

`LRANGE key start stop` stop 参数包含了自身，这和许多编程语言不包含 stop 不太相同

#### 删除元素

`LPOP key` 移除列表的第一个元素，并将该元素作为结果返回

#### 从列表右端弹出

`RPOP key` 移除列表的最后一个元素

#### 删除指定的元素

`LREM key count value` 删除列表中与value的元素，根据count的不同分为三种情况

- count >0:从左至右，最多删除count个与value相等的元素

- count<0:从右至左，最多删除count个与value相等的元素

- count=0：删除列表中所有与value相等的元素

  



#### 保留指定范围内的元素

`LTRIM key start stop` 仅保留列表指定范围内的元素，不在范围内的元素将会被删除

#### 修改元素

`LSET key index value`修改指定索引下表的元素

#### 获取列表的长度

`LLEN key` 返回指定列表的长度

#### 不常用的命令

`RPOPLPUSH source destination` 现制定RPOP操作再置顶LPUSH操作：从source列表的右端弹出一个元素，再把这个元素插入destination列表的左端，最后返回这个元素作为结果，整个过程是原子的

<font color=#FF0000 >注：</font> 可以将source与destination都指定成同一个列表，那么这个列表就相当于一个列表循环

#### RPUSHX 

`RPUSHX key value` 与RPUSH命令类似，只要它要求指定的 列表必须存在，才能成功往列表插入元素

#### LPUSHX

与 RPUSHX 命令类似，只是当满足条件时，它往列表的左端插入元素







### 集合

集合类型也是用来存储多个有序的字符串元素，它和列表不一样的是，集合不允许有重复元素，而且集合中的元素是无序的，不能通过索引下标获取元素

#### 常用的命令

#### 添加命令

`SADD key member [member...]` 用于向集合添加元素，如果指定的元素集合中已经存在，那么就会忽略该元素，结果返回的是成功添加到集合的元素个数



#### 删除元素

`SREM key member[]member...` 接受一个或多个元素作为参数，从指定的集合将这些元素移除，如果某个元素不是集合的成员，那么忽略此元素，最后返回成功移除元素的个数



#### 获取集合的所有元素

`SMEMBERS key` 获取指定集合的所有元素

#### 计算集合元素的个数

`SCARD key`计算指定集合的基数

#### 判断元素是否在集合中

`SISMEMBER key member` 判断指定的元素是否是集合的成员，如果是返回1，否则返回0

### 集合运算

#### 交集

`SINTER key [key ...]` 计算多个集合的交集

例: SINTER set1 set2

#### 并集

 `SUNION key [key...]` 用于计算多个集合的并集

例:SUNION set1 set2

#### 差集

`SDIFF key [key...]`

例:SDIFF set1 set2

#### 保存集合运算的结果

集合运算在元素较多的情况下会比较耗时，如果能将运算结果保存，下次需要做相同的运算将极大地提升效率

SINTER、SUNION、SDIFF 这三个命令分别对应一个命令将运算后的结果保存在一个指定的集合：

- SINTERSTORE 命令的用法是`SINTERSTORE destination key [key ...]` 它对多个集合做交集运算并将结果保存在destination

- SUNIONSTORE 命令的用法是`SUNIONSTORE destination key [key ...]`,它多多个集合做交集运算并将结果保存在destination中

- SDIFFSTORE命令的用法是` SDIFFSTORE destination key [key ...]`，它对多个集合做差集运算并将结果保存在destionation中

  

#### 不常用的命令

#### 删除集合第一个元素并作为结果返回

`SPOP key [count]` 将返回元素但元素从集合中弹出



#### 将元素从一个集合移动至另一个集合

`SMOVE source destination member` 如果member不是source的成员或者source不存在，那么忽略此操作并返回0，否则member被移动至destination并返回1.如果destination已经存在member，那么仅仅是将member从source移除



### 有序集合

有序集合类型保留了集合不能有重复元素的特点，但不同的是，有序集合中的元素可以排序。有序集合类型为集合中的每个元素都关联了一个分数，这个分数便是排序的依据。

#### 常用的命令

#### 添加元素

`ZADD key [NX|XX] [CH] []INCR source member [source member ...]` 用户为指定的有序集合添加成员，NX、XX、CH和INCR四个选项从3.02版本才开始支持:

- NX:member必须不存在，才可以设置成功，用于添加

- XX：member必须存在，才可以设计成功，用于更新

- CH：返回此次操作后，有序集合元素和分数发生变化的个数

- INCR:对score做自增操作

  

  ![1578495730035](C:\Users\chenyue\AppData\Roaming\Typora\typora-user-images\1578495730035.png)



#### 删除元素

`ZREM key member [member...]` 删除有序集合的成员

#### 计算有序集合元素的个数

`ZCARD key` 计算指定有序集合的基数

#### 获取元素的分数

`ZSCORE key member` 获取有序集合中指定成员的分数，如果成员不存在，返回nil

#### 计算成员排名

`ZRANK key member`获取指定成员在有序集合中的排名，排名以成员分数为依据，分数值越低，排名越高

`ZREVERANK key member`分数越高，排名越高

#### 获取指定排名范围内的元素

`ZRANGE key start stop [WITHSCORES]` 以成员分数升序的方式返回排名在区间的成员,指定WITHSCORES,结果会返回分数

`ZREVRANGE key start stop [WITHSCORES]` 以分数降序的方式返回排名在区间的成员

#### 获取指定分数范围内的元素

`ZRANGEBYSCORE key min max [WITHSCORE][LIMIT offset count]` 用户获取分数值在区间[mix,max]的成员，按分数从小到大排序。如果置顶WITHSCORES选项，返回结果会包含成员分数。[LIIMIT offset count]选项可以限制输出的起始位置和个数

`ZREVRANGEBYSCORE key min max [WITHSCORE][LIMIT offset count]`ZRANGEBYSCORE 命令类似，但它是按分数从大到小排序



### 获取在指定分数范围内的元素个数

`ZCOUNT key min max` 获取分数值在指定范围内的成员个数

### 集合运算

#### 并集

`ZUNIONSTORE destination numkeys key [key...] [WEIGHTS weight...] [AGGREGATE SUM|MIN|MAX]` 

- destination:保存并集运算的结果

- numkeys:需要做并集运算的键的个数

- key[key...]:需要做并集运算的键

- [WEIGHTS weight[weight]]:每个键的权重，在做并集运算时，每个键中的每个成员会将自己的分数乘以这个权重，每个键的权重默认是1

- [AGGREGATE SUM|MIN|MAX] :计算成员并集和，分数可以按照SUM、min、max汇聚,默认是sum

  

![1578496755238](C:\Users\chenyue\AppData\Roaming\Typora\typora-user-images\1578496755238.png)



#### 交集

ZINTERSTORE 命令的用法与 ZUNIONSTORE 命令类似，只不过它是做交集运算，此处不再赘述

#### 不常用的命令

#### 删除在指定片名范围内的元素

`ZREMRANGEBYRANK key start stop`  用于删除指定排位范围内的成员

#### 删除在指定分数范围内的元素

`ZREMRANGEBUSCORE key min max` 命令用于删除指定分数范围内的成员

#### 增加元素的分数

`ZINCRBY key increment member` 指定的元素增加分数





 ### 遍历键

SCAN 命令是从 2.8.0 版本才开始支持的，它和 KEYS 命令有所不同，SCAN 命令是采用渐进的方式遍历键。每次执行 SCAN 命令的时间复杂度是 O(1)，但要真正实现 KEYS 命令的功能，需要执行多次 SCAN 命令

`SCAN cursor [match pattern] [COUNT count]`

- cursor：这是必选项，实际上cursor是一个游标，第一次遍历从0开始，每次执行SCAN命令都会返回当前游标的值，直到游标值为0，表示遍历结束
- MATCH pattern ：这是可选项，它的作用是做模式的匹配，这个KEYS命令的模式类型
- COUNT count：可选项，它的作用是指定每次要遍历的键的个数，默认是10



假设数据库中有26键，现在要使用SCAN遍历所有的键，第一次执行`SCAN 0`：

```
127.0.0.1:6379> SCAN 0
1) "22"
2)  1) "o"
    2) "f"
    3) "y"
    4) "r"
    5) "i"
    6) "d"
    7) "j"
    8) "t"
    9) "g"   10) "l
```

返回结果包括两部分：第一部分是游标,22就表示下次需要指定的是SCAN 22;第二部分是10个键

```
127.0.0.1:6379> SCAN 22
1) "21"
2)  1) "x"
    2) "a"
    3) "p"
    4) "u"
    5) "e"
    6) "v"
    7) "q"
    8) "m"
    9) "k"
   10) "z"
```

这次得到的cursor是21，继续执行SCAN命令

```
127.0.0.1:6379> SCAN 21
1) "0"
2) 1) "c"
   2) "b"
   3) "s"
   4) "n"
   5) "w"
   6) "h"
```

现在得到cusor是0，即表示已经遍历所有的键



除了SCAN命令以为，redis还提供了面向哈希类型、集合类型、有序集合类型的扫描遍历命令，解决诸如HGETTALL、SMEMBERS、ZRANGE可能产生的阻塞问题，对应的命令分别是HSCAN、SSCAN、ZSCAN，它们的用法和SCAN基本类似，已sscan为例:

`SSCAN keu corsor [MATCH pattern][COUNT count]`

假设现在有个集合，集合包含 26 个元素（26 个英文字母），现在使用 SSCAN 命令遍历集合的所有元素

```
127.0.0.1:6379> SADD myset a b c d e f g h i j k l m n o p q r s t u v w x y z
(integer) 26
127.0.0.1:6379> SSCAN myset 0
1) "14"
2)  1) "o"
    2) "f"
    3) "y"
    4) "r"
    5) "i"
    6) "d"
    7) "j"
    8) "t"
    9) "l"
   10) "g"
   11) "x"
   12) "a"
   13) "p"
127.0.0.1:6379> SSCAN myset 14
1) "7"
2)  1) "e"
    2) "u"
    3) "v"
    4) "q"
    5) "m"
    6) "k"
    7) "z"
    8) "c"
    9) "b"
   10) "s"
127.0.0.1:6379> SSCAN myset 7
1) "0"
2) 1) "n"
   2) "h"
   3) "w"
```





### 数据库管理

#### 切换数据库

edis 则是用数字区分不同的数据库。Redis 默认配置是有 16 个数据库，用 0...15 标识。不同数据库之间的数据没有任何联系，甚至可以存在相同的键。客户端连接 Redis 时，默认使用 0 号数据库。

SELECT 命令用于切换当前的数据库，它的用法是 `SELECT index`。下面的操作将当前的数据库切换至 15 号数据库：

```
127.0.0.1:6379> SELECT 15
OK
```



实际开发过程中，不建议使用多数据库，主要是因为:

- Redis 是单线程的。如果使用多个数据库，那么这些数据库仍然是使用一个 CPU，彼此之间还是会受到影响。
- 多数据库的使用方式，会让调试和运维不同业务的数据库变得困难。假如有一个慢查询存在，依然会影响其他数据库，这样会使得别的业务方定位问题非常的困难
- 部分 Redis 客户端不支持多数据库
- 如果需要使用多数据库的功能，完全可以在一台机器上部署多个 Redis 实例，用不同的端口来区分。这样既能保证业务之间不会受到影响，又合理地使用了 CPU 资源



#### 清空数据库数据

#### 清空当前数据库数据

`PLSUSHDB [ASYNC]` FLUSHDB 命令用于清空当前数据库的所有数据，如果指定了 ASYNC 选项，那么 Redis 会以异步的方式清空数据。

#### 清空所有数据库数据

`FLUSHALL [ASYNC]` 会清空所有数据库的数据

#### 查看当前数据库键的数量

`DBSIZE`  返回当前数据库键的数量



### 键迁移

迁移键功能非常重要，开发和运维过程中经常需要将虚数据从一个库迁移到另外一个库中，Redis 发展历程中提供了 MOVE、DUMP + RESTORE、MIGRATE 三组迁移键的方法，它们的实现方式以及使用场景不太相同。

#### MOVE

`MOVE key db`

move命令用于在redis内部进行数据迁移，它的作用是将当前数据库的键key迁移到指定的数据库db

下面的操作将键 foo 迁移至 15 号数据库

```
127.0.0.1:6379> SET foo "hello"
OK
127.0.0.1:6379> MOVE foo 15
(integer) 1
127.0.0.1:6379> GET foo
(nil)
127.0.0.1:6379> SELECT 15
OK
127.0.0.1:6379[15]> GET foo
"hello"
```



#### DUMP + RESTORE

DUMP的用法是`DUMP key` ,将以RDB格式序列化指定的键，并返回序列化后的键，这个键可以通过RESTORE命令还原

RESTORE命令的用法是`RESTORE key ttl serialized-value [REPLACE]` 它的作用是将序列化的值 serialized-value还原并赋值给键key，ttl用户置顶键的过期时间，ttl=0表示不设置过期时间。如果不指定REPLACE选项，当key原本已经存在，则返回错误信息。从 3.0 以及之后的版本支持 REPLACE 选项，强制覆盖原有的键值。



通过DUMP+RESTRORE的方式实现在不同redis实例之间进行数据迁移的功能，分为三个步骤：

1、在源Redis实例上，通过DUMP命令将键序列化

2、在目标redis实例上，使用RESTORE命令将序列化的值复原

3、使用DEL命令删除源Redis实例上的键。(不删除则实现了键赋值的功能)



下面通过一个例子来演示完整的过程:

1、在源Redis实例(6379上指定序列化键值

```
127.0.0.1:6379> SET foo "hello"
OK
127.0.0.1:6379> DUMP foo
"\x00\x05hello\a\x00\x9c@\n\x85m\xfe\xf5\x10"
```

2、在目标redis实例(6380)上执行RESTORE命令复原键值

```
127.0.0.1:6380> GET foo
(nil)
127.0.0.1:6380> RESTORE foo 0 "\x00\x05hello\a\x00\x9c@\n\x85m\xfe\xf5\x10"
OK
127.0.0.1:6380> GET foo
"hello"
```

3、删除源Redis实例(6379)上的键：

```
127.0.0.1:6379> DEL foo
(integer) 1
```

使用DUMP + RESTORE方式进行数据迁移，有集合点需要注意

- 为了方便演示，上面的例子使用同一台机器上的两个 Redis 实例，不同机器的 Redis 实例也是一样可以实现的
- DUMP + RESTORE + DEL 的过程不是原子的，而是通过客户端分步完成的。
- 迁移的过程是开启了两个客户端连接，所以这并不是一个数据传输的过程



#### MIGRATE

MIGRATE命令也是用于在redis实例之间进行数据库迁移，实际上，MIGRATE命令是将DUMP+ RESTORE + DEL三个命令进行组合，从而简化了操作流程。MIGRATE命令具有原子性，而且从 3.0.6 版本开始支持同时迁移多个键的功能，提高了迁移效率。

`MIGRATE host port key|"" destination-db timeout [COPY] [REPLACE] [KEYS key [key ...]]`

- host : 目标redis的ip地址
- port：目标redis的端口号
- key|""：在redis3.0.6版本之前，MIGRATE命令只会支持迁移一个键，所以此处是要迁移的键。但是redis3.0.3版本开始支持迁移多个键，如果需要迁移多个键，此处用空字符""
- destination-db:目标redis的数据库索引
- timeout：迁移的超时时间，单位是毫秒
- COPY:如果指定此选项，迁移后不删除源键
- REPLACE：如果指定此选项，不管目标redis的数据中心存在该键，则会强制覆盖键值
- [ KEYS key[key...]]:迁移多个键



下面的操作演示将一个键值复制到目标 Redis 实例：

```
127.0.0.1:6379> SET foo "hello"
OK
127.0.0.1:6379> MIGRATE 127.0.0.1 6380 foo 0 1000 COPY REPLACE
OK
```

下面的操作演示将多个键值迁移到目标 Redis 实例：

```
127.0.0.1:6379> MIGRATE 127.0.0.1 6380 "" 0 1000 REPLACE KEYS foo bar baz
OK
```

三方式的异同点:

下表总结了 MOVE、DUMP + RESTORE、MIGRATE 这三种数据迁移方式的异同点

| 命令         | 作用域        | 原子性 | 支持多个键 |
| ------------ | ------------- | ------ | ---------- |
| MOVE         | redis实例内部 | 是     | 否         |
| DUMP+RESTORE | redis实例之间 | 否     | 否         |
| MIGRATE      | redis实例之间 | 是     | 是         |



### 内部编码

#### 字符串

字符串类型的内部编码有三种

- int：8 个字节的长整型
- embstr：小于等于 44 个字节的字符串。(注：不同版本的 Redis 可能取不同的值，有的版本是 39 字节，这个取值定义在源码 object.c 文件中的 OBJ_ENCODING_EMBSTR_SIZE_LIMIT)
- raw：大于44个字节的字符串

Redis 会根据字符串的键值决定使用的内部编码实现

#### 哈希

哈希类型的内部编码有两种

- ziplist（压缩列表）：当哈希类型的元素个数小于 hash-max-ziplist-entries 配置（默认 512 个），同时所有值都小于 hash-max-ziplist-value 配置（默认 64 字节）时，Redis 会使用 ziplist 作为哈希的内部实现。ziplist 使用更加紧凑的结构实现多个元素的连续存储，在节省内存方面比 hashtable 更为优秀
- hashtable（哈希表）：当哈希类型的键值无法满足 ziplist 的条件时，Redis 会使用 hashtable 作为哈希的内部实现。hashtable 的读写时间复杂度为 O(1)，读写效率比 ziplist 更优



#### 列表

列表类型的内部编码有两种：

- intset：当集合中的元素都是整数且元素个数小于 set-max-intset-entries 配置（默认 512 个）时，Redis 会选用 intset
- hashtable（哈希表）：当集合类型的键值无法满足 intset 的条件时，Redis 会使用 hashtable 作为列表的内部实现



#### 有序集合

有序集合类型的内部编码有两种

- ziplist（压缩列表）：当有序集合的元素个数小于 zset-max-ziplist-entries 配置（默认 128 个），同时每个元素的值都小于 zset-max-ziplist-value 配置（默认 64 字节）时，Redis 会使用 ziplist 来作为有序集合的内部实现
- skiplist（跳跃表）：当有序集合类型的键值无法满足 ziplist 的条件时，Redis 会使用 skiplist 作为有序集合的内部实现



### Redis慢查询分析

许多存储系统（例如 MySQL）提供慢查询日志帮助开发和运维人员定位系统存在的慢操作。所谓的慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阈值，就想这条命了的相关信息（如发生时间、耗时、命令详情等）记录下来，Redis 也提供了类似的功能

![1578500355349](C:\Users\chenyue\AppData\Roaming\Typora\typora-user-images\1578500355349.png)



如上图所示，Redis客户端执行了一条命令分为如下4个部分:

1、发送命令

2、命令排队

3、命令执行

4、返回结果

需要注意的是，慢查询只统计命令执行这一步骤的时间，所以没有慢查询并不代表客户端没有超时问题



#### 参数配置

Redis 提供了两个与慢查询相关的配置：slowlog-log-slower-than 和 slowlog-max-len。slowlog-log-slower-than 用于配置预设阈值，它的单位是微秒，默认值是 10000

```
127.0.0.1:6379> CONFIG GET slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "10000"
```

如果一条命令的执行时间超过了预设阈值，那么它将被记录在慢查询日志中。如果 slowlog-log-slower-than = 0，则会记录所有的命令；如果 slowlog-log-slower-than < 0，则不会记录任何命令。

Redis 使用了一个列表来存储慢查询日志，slowlog-max-len 用于配置这个列表的最大长度。 一个命令满足慢查询条件时，就会被追加到这个列表中，当慢查询日志列表已处于其最大长度时，再有新的慢查询命令时，列表头部的第一条数据就会出列，新的数据就会入列。

在 Redis 中有两种修改配置的方法：一种是修改配置文件，另一种是使用 `CONFIG SET` 命令动态修改。例如，下面的命令将 slowlog-log-slower-than 设置成 20000 微秒，slowlog-max-len 设置成 1000:

```
127.0.0.1:6379> CONFIG SET slowlog-log-slower-than 20000
OK
127.0.0.1:6379> CONFIG SET slowlog-max-len 1000
OK
127.0.0.1:6379> CONFIG REWRITE
OK
```

``CONFIG REWRITE`` 命令能够将修改的配置持久化到本地配置文件

#### 常用的命令

虽然慢查询是存放在 Redis 内存列表中的，但是 Rediscover 并没有暴露这个列表的键，而是通过一组命令来实现对慢查询日志的访问和管理。

获取慢查询日志

`SLOWLOG GET [n]` 命令可以查询当前 Redis 的慢查询日志列表，参数 n 可以指定条数

```
127.0.0.1:6379> SLOWLOG GET
1) 1) (integer) 1
   2) (integer) 1533872710
   3) (integer) 5
   4) 1) "KEYS"
      2) "*"
2) 1) (integer) 0
   2) (integer) 1533872692
   3) (integer) 4
   4) 1) "CONFIG"
      2) "SET"
      3) "slowlog-log-slower-than"
      4) "0"
```

可以看到每个慢查询日志由 4 个属性组成，分别是慢查询日志的标识ID、命令执行时的时间戳、命令耗时和命令内容。

#### 获取慢查询日志列表的长度

`SLOWLOG LEN` 查询慢查询列表当前的长度

#### 重置慢查询列表

`SLOWLOG RESET` 可以重置慢查询日志列表，即对列表做情况操作



### 最佳实践

慢查询功能可以有效地帮助开发和运维人员找到 Redis 可能存在的瓶颈，但在实际使用过程中要注意以下几点

- slowlog-max-len 配置建议：线上建议调大慢查询列表，这样可以减少慢查询被剔除的可能，例如线上可以设置为 1000 以上
- slowlog-log-slower-than 配置建议：默认超过 10 毫秒判定为慢查询，需要根据 Redis 的并发量调整该值。由于 Redis 采用单线程响应命令，对于高流量的场景，如果命令执行时间在 1 毫秒以上，那么 Redis 最多可支撑 OPS（Operation Per Second）不到 1000。因此，对于高 OPS 场景的 Redis，建议设置为 1 毫秒。
- 慢查询只记录命令执行时间，并不包括命令排队和网络传输时间。因此，客户端执行命令的时间会大雨命令实际执行时间。因为命令执行排队机制，慢查询会导致其他命令级联阻塞，因此当客户端出现请求超时，需要检查该时间点是否有对应的慢查询，从而分析是否为慢查询导致的命令级联阻塞。
- 由于慢查询日志是一个先进先出的队列，当慢查询比较多的情况下，可能会丢失部分慢查询命令。为了防止这种情况发生，可以定期执行 SLOW GET 命令将慢查询日志持久化到其他存储中（如 MySQL），然后可以制作可视化界面进行查询。



### Pipeline

Redis客户端执行一条命令分为如下四个过程:

1、发送命令

2、命令排队

3、命令执行

4、返回结果

其中，步骤1和4称为Round Trip Time(往返时间)

在执行多个命令时，每条命令都需要等待上一条命令执行完（即收到 Redis 的返回结果）才能执行，即使命令不需要上一条命令的结果。Redis 提供的批量操作命令（如 MGET、MSET 等），有效地节约 RTT。但大部分的命令是不支持批量操作的。例如要批量删除若干个键，并没有 MDEL 命令操作，分 n 次执行 DEL 命令，需要消耗 n 次 RTT。受限于网络性能，多个命令的 RTT 累加起来对性能还是有一定的影响的。

Redis 的 Pipeline 机制能够改善上面这类问题，它能将一组 Redis 命令进行组装，通过一次 RTT 传输给 Redis，再将这组 Redis 命令的执行结果按顺序返回给客户端。

Pipeline 虽然好用，但是每次 Pipeline 组装的命令个数不能没有节制，否则一次组装 Pipeline 数据量过大，一方面会增加客户端的等待时间，另一方面也会造成一定的网络阻塞，可以将一次包含大量命令的 Pipeline 拆分成多次较小的 Pipeline 来完成。

redis-cli 的 --pipe 选项可以使用 Pipeline。但通常情况下，我们并不会在 redis-cli 在使用 Pipeline，而是倾向于使用高级语言客户端的 Pipeline。