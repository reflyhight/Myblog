title: Redis 入门基础篇
tags: 
	- Redis
	- 入门

categories:
	- Redis

description: 天下事有难易乎？为之，则难者亦易矣；不为，则易者亦难矣。
-------------------

# 简介
redis 是基于键值对的数据库，是一个非关系型数据库（nosql）。redis是基于内存的数据库，常用于网站缓存。目前支持 strings, hashes, lists, sets, sorted sets, bitmaps, hyperloglogs , geospatial indexes数据类型的存储。


**官方介绍摘要**

> Redis is an open source (BSD licensed), in-memory data structure store, used as database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

# 安装运行

## 安装

	# wget http://download.redis.io/releases/redis-3.0.7.tar.gz
	# tar xzf redis-3.0.7.tar.gz
	# cd redis-3.0.7
	# make
	# make install

- 第一步 下载redis压缩包  在Linux中用wget命令。前提是当前linux能联网。或者手动去官网下载压缩包再上传到Linux环境。
- 接下来：解压压缩包，再进入（cd）到解压后的目录 redis-3.0.7  再编译构建（make命令）。最后安装(make install命令)


## 运行
	
	# redis-server 
	

> 执行 redis-server命令后，如下面所示，则证明安装成功。

	10714:C 03 May 22:24:47.216 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
	10714:M 03 May 22:24:47.216 * Increased maximum number of open files to 10032 (it was originally set to 1024).
	                _._                                                  
	           _.-``__ ''-._                                             
	      _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
	  .-`` .-```.  ```\/    _.,_ ''-._                                   
	 (    '      ,       .-`  | `,    )     Running in standalone mode
	 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
	 |    `-._   `._    /     _.-'    |     PID: 10714
	  `-._    `-._  `-./  _.-'    _.-'                                   
	 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
	 |    `-._`-._        _.-'_.-'    |           http://redis.io        
	  `-._    `-._`-.__.-'_.-'    _.-'                                   
	 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
	 |    `-._`-._        _.-'_.-'    |                                  
	  `-._    `-._`-.__.-'_.-'    _.-'        


<!-- more -->

> 执行type命令查看 redis-server。

	# type redis-server 
	redis-server is hashed (/usr/local/bin/redis-server) 



> make install命令会将redis相关的命令放到/usr/local/bin，/usr/local/bin在Linux的环境变量中
> 【注意】当前Linux的编译相关的依赖如果没有全部安装，可能会构建(make)失败

## 后台运行

> redis提供了一个redis.conf的文件，可以作为redis-server命令的参数,redis.conf默认放在解压后的目录中（redis-3.0.7）。
	
	# mv redis.conf  /etc/  			//将配置文件移动到/etc便于统一管理
	# redis-server  /etc/redis.conf 	//redis.conf 中配置启动redis服务
	
> 修改：redis.conf中的 daemonize no 为  daemonize yes  
	
	# redis-server  /etc/redis.conf    //此时再次运行时，为后台运行了。

> 测试后台运行是否成功

	# ps -ef|grep redis
	root     14206     1  0 23:25 ?        00:00:00 redis-server *:6379         
	root     14218  6826  0 23:26 pts/1    00:00:00 grep redis

## 连接到redis
>用redis-cli命令连接到redis服务器

> 格式 【redis-cli -h ip -p port(端口号)】 ip为具体ip,port为具体端口号。不指定-h，和-p，则默认连接到本地6379端口



	# redis-cli
	127.0.0.1:6379>  //表明连接到了 127.0.0.1的redis服务器，端口号为6379

	# redis-cli -h 127.0.0.1 -p 6379
	127.0.0.1:6379> 


# redis数据库的特点


> 每个数据库对外都是以一个从0开始的递增数字命名，不支持自定义的	
redis默认支持16个数据库，可以通过修改databases参数来修改这个默认值
redis默认选择的是0号数据库
SELECT 数字： 可以切换数据库
多个数据库之间并不是完全隔离的,flushall
flushall：清空redis实例下的所有数据
flushdb：清空当前数据库中的所有数据


# redis 基础命令
## exists 命令
> 格式： EXISTS key [keyName ...]
> 检查数据库中是否存在建为 keyName 的数据，返回存在键的个数

	# redis-server /etc/redis.conf 			//启动数据库服务
	# redis-cli								//命令行连接数据库
	127.0.0.1:6379> set user1name "haima"	//存入数据 user1name
	OK
	127.0.0.1:6379> EXISTS user1		//存在user1，返回1
	(integer) 1
	127.0.0.1:6379> EXISTS user2		//存在user2，返回0
	(integer) 0
	127.0.0.1:6379> EXISTS user2 user1	//只存在user1，还是返回1
	(integer) 1

## keys命令
> 格式： KEYS pattern
> 查看数据库中匹配到的键

	127.0.0.1:6379> set user1 "haima"
	OK
	127.0.0.1:6379> set user2 "vicky"
	OK
	> keys *     //查看数据库中所有键
	1) "user1"
	2) "user2"

	> keys *1    //查看数据库中所有以1结尾的键
	1) "user1"

## del命令
>格式 DEL key [key ...]
>删除一条或多条数据，根据键来删除该条数据

	127.0.0.1:6379> del user1 user2
	(integer) 2


## type命令
> 格式：TYPE key
获得键值的数据类型

	127.0.0.1:6379> set user1 "haima"
	OK
	127.0.0.1:6379> type user1
	string

## help命令
格式：HELP commond  
查看某个命令的使用方法，commond表示具体的命令名称

	127.0.0.1:6379> help type

	  TYPE key
	  summary: Determine the type stored at key
	  since: 1.0.0
	  group: generic


## 数据类型
>数据类型均是指Key-value中的value，Redis中的键均为String型。而value有string, hashe, list, set, sorted set 等 数据类型。其中string型是最基本的类型。比如list中存放的每个元素都是string型。

# string型数据
string型数据是redis中最基础的数据，也作为一种独立的类型存在。

## set命令
> set 存储一条String 型的数据
> 格式 SET key value [EX seconds] [PX milliseconds] [NX|XX]

1. EX seconds -- Set the specified expire time, in seconds.
1. PX milliseconds -- Set the specified expire time, in milliseconds.
1. NX -- Only set the key if it does not already exist.
1. XX -- Only set the key if it already exist.

- EX seconds 设置失效时间，seconds为具体的时间，单位为秒
- PX milliseconds 设置失效时间，milliseconds为具体的时间，单位为毫秒
- NX 如果没有此键，才添加该条数据。如果数据库中有此键，添加失败。（只添加，不覆盖）
- XX 如果有此键，才添加。没有此键，则不操作。（相当于只做修改操作）

eg | 1、

	127.0.0.1:6379> set user1 "haima" ex 5	//设置5秒后，数据失效，自动销毁。
	OK
	127.0.0.1:6379> get user1
	"haimda"

	//等五秒后再执行下一条命令

	127.0.0.1:6379> get user1
	(nil) //已经自动销毁

eg |2、

	127.0.0.1:6379> set user1 "haima" PX 5000 //设置5秒(5000毫秒)后，数据失效，自动销毁。
	OK
	127.0.0.1:6379> get user1
	"haimda"

	//等五秒后再执行下一条命令

	127.0.0.1:6379> get user1
	(nil) //已经自动销毁


## get 命令
> 显示string类型的数据，或者说取出string类型的值，也就是使用set存如的值。
> 格式 GET key

	127.0.0.1:6379> set user1 "haima"
	OK
	127.0.0.1:6379> get user1
	"haima"
	127.0.0.1:6379> del user1
	(integer) 1
	127.0.0.1:6379> get user1	//没有该数据就返回nil
	(nil)

## getset 命令
> 设置某个键的值，同时返回该键的原有值
> 格式 GETSET key value

	127.0.0.1:6379> get user
	(nil)
	127.0.0.1:6379> getset user "haima"
	(nil)
	127.0.0.1:6379> getset user "vicky"
	"haima"


## mset，mget，msetnx 命令
> 一次性存取多组键值对。
> 格式： MSET key value [key value ...]，MGET key [key ...]， MSETNX key value [key value ...]
> msetnx 命令设置多组Key-value，仅当key不存在时，才设置，key存在，则不覆盖。

	127.0.0.1:6379> mset user1 "haima" user2 "vicky"
	OK
	127.0.0.1:6379> mget user1 user2
	1) "haima"
	2) "vicky"

## setnx, setex, pserex
> SETNX命令，格式【SETNX key value】，相当于 set key value NX  
> SETEX命令，格式【SETEX key seconds value】，相当与 set key value EX seconds 
> PSETEX命令，格式【PSETEX key milliseconds value】，相当与 set key value PX milliseconds

## expire expireat
> expire 【 EXPIRE key seconds】 设置过期时间，用秒来设置
> expireat 【EXPIREAT key timestamp】 设置过期时间，用时间戳

## incr，decr命令

> incr：自增加1，在值为整数的前提下，否则操作失败。
> decr：自减减1，在值为整数的前提下，否则操作失败。
> 格式 ：INCR key，DECR key

	127.0.0.1:6379> set int1 1.1
	OK
	127.0.0.1:6379> incr int1	//整数型才能自增加1
	(error) ERR value is not an integer or out of range

	127.0.0.1:6379> set int1 1
	OK
	127.0.0.1:6379> incr int1
	(integer) 2
	127.0.0.1:6379> get int1
	"2"

	127.0.0.1:6379> decr int1
	(integer) 1
	127.0.0.1:6379> get int1
	"1"

## incrby，decrby命令
> 同incr,和 decr相似，还可以指定一个自增或自减的数
> 格式 INCRBY key increment， DECRBY key decrement 。 decrement是具体的数字

	127.0.0.1:6379> set int1 1
	OK
	127.0.0.1:6379> incrby int1 3
	(integer) 4
	127.0.0.1:6379> get int1
	"4"

	127.0.0.1:6379> decrby int1 3
	(integer) 1
	127.0.0.1:6379> get int1
	"1"

## incrbyfloat命令
> 在原基础上加一个float型的数值，原数值也可以是float型的
> 格式 INCRBYFLOAT key increment

	127.0.0.1:6379> set fl1 1.5
	OK
	127.0.0.1:6379> INCRBYFLOAT fl1 4.5
	"6"
	127.0.0.1:6379> INCRBYFLOAT fl1 4.1
	"10.1"

## append 命令
> 追加字符串到原来的值中，如果值是数字，也会类型自动转换
> 格式 APPEND key value

	127.0.0.1:6379> get int2
	"1"
	127.0.0.1:6379> append int2 "haima"
	(integer) 6
	127.0.0.1:6379> get int2
	"1haima"

## strlen 命令
> 获取一个string型字符串的长度，不存的键会返回0
> 格式  STRLEN key

	127.0.0.1:6379> set user1 "haima1"
	OK
	127.0.0.1:6379> strlen user1
	(integer) 6

## getrange 命令
> 获取一个子字符串
> 格式  GETRANGE key start end

	127.0.0.1:6379> get user1
	"haima1"
	127.0.0.1:6379> getrange user1 2 5
	"ima1"


## setbit 命令
> 设置某个位置上的二进位0,1数
> 格式  SETBIT key offset value 

	127.0.0.1:6379> get user1
	"haima1"
	127.0.0.1:6379> setbit user1 4 0
	(integer) 1
	127.0.0.1:6379> get user1
	"`aima1"

## getbit 命令
> 获取某个位置上的二进位0,1数
> 格式  GETBIT key offset

	127.0.0.1:6379> get user1
	"haima1"
	127.0.0.1:6379> setbit user1 4 0
	(integer) 1
	127.0.0.1:6379> get user1
	"`aima1"

## bitcount 命令
> 二进制数中1的个数
> 格式 BITCOUNT key [start end]


## bitop 命令
> 对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上 
> 格式 BITOP operation destkey key [key ...] 
> 其中位运算operation包括  AND （位与）、 OR（位或） 、 NOT（位非） 、 XOR（位异或）

# hash数据类型
> hash数据类型 指数据key-value中的value是一个hash表。而hash表又由多组key-value组成。
> hash表中的Key- value key 和value都是string型（string型为基本数据类型）

## hset 命令
> 设置创建一个hash类型的数据
> 格式 HSET key field value

	127.0.0.1:6379> hset userhs name "haima"
	(integer) 1
	127.0.0.1:6379> type userhs
	hash
## hsetnx 命令
> 设置创建一个hash类型的数据，如果这个值存在，则不会生效
> 格式 HSETNX key field value

	127.0.0.1:6379> hsetnx userhs name "vicky"
	(integer) 0
	127.0.0.1:6379> hget userhs name
	"haima"	//设置vicky时，并未生效

## hget 命令
> 获取hash中的某个键对应的值
> 格式 HGET key field

	127.0.0.1:6379> hset userhs age 27
	(integer) 1
	127.0.0.1:6379> hget userhs age
	"27"

## hmset，hmget
> hmset 设置hash中多组键值对，hmset 获取多组键的值
> 格式HMSET key field value [field value ...]， HMGET key field [field ...]

	127.0.0.1:6379> hmset userhs job "java" place "beijing"
	OK
	127.0.0.1:6379> hmget userhs job place
	1) "java"
	2) "beijing"

## hgetall 命令
> 获取所有键值对 （注意，其实返回的是hash表本身，在jedis中会返回一个Map，详见后文jedis）
> 格式 HGETALL key

	127.0.0.1:6379> hgetall userhs
	1) "name"
	2) "haima"
	3) "age"
	4) "27"
	5) "job"
	6) "java"
	7) "place"
	8) "beijing"


## hexists 命令  
> 判断hash表中某个字段是否存在  （判断整个hash表直接用基础命令 exists ）
> 格式  HEXISTS key field

	127.0.0.1:6379> HEXISTS userhs name
	(integer) 1


## hkeys，hvals	
> 获取所有键，获取所有值
> 格式 HKEYS key，HVALS key
	
	127.0.0.1:6379> hkeys userhs
	1) "name"
	2) "age"
	3) "job"
	4) "place"

	127.0.0.1:6379> hvals userhs
	1) "haima"
	2) "27"
	3) "java"
	4) "beijing"

## hlen 命令
> 查看表中键值对的对数 
> 格式 HLEN key
	127.0.0.1:6379> HLEN userhs
	(integer) 4

## hdel 命令
> 删除一个或多个键值对 （删除整个hash表，用基础命令del）
> 格式 HDEL key field [field ...]

	127.0.0.1:6379> hdel userhs place
	(integer) 1
	127.0.0.1:6379> HKEYS userhs
	1) "name"
	2) "age"
	3) "job"



## hincrby，hincrbyfloat
> 增加某个键的值
> 格式 HINCRBY key field increment， HINCRBYFLOAT key field increment

# List 类型
list是一个有序的字符串列表，列表内部实现是使用双向链表(linked list)实现的
list 的元素从左到右，脚标从0开始，依次增大

## 常见命令
>**lpush** 格式：    LPUSH key value [value ...] //从左边压入队列
**rpush** 格式：	 RPUSH key value [value ...] //从右边压入队列
**lpop** 格式：		LPOP key //从左边弹出第一个元素（元素会被删除）
**rpop** 格式：		RPOP key //从右边弹出第一个元素(元素会被删除)
**llen** 格式：		  LLEN key //获取list的长度，即list元素的个数	
**lrange** 格式：	  LRANGE key start stop //显示 list中某个范围的元素，可以用-1表示list中最后一个元素脚标
**lindex** 格式：		LINDEX key	//查询指定角标数据
**lset** 格式：		LSET key index value //修改指定角标的值。
**ltrim** 格式： LTRIM key start stop	//保留指定位置的元素
**rpoplpush** 格式：RPOPLPUSH source destination //将元素从一个列表转到另一个列表

eg| lpush,rpush

	127.0.0.1:6379> LPUSH list1 "haima" "vicky"	//从左边压入队列
	(integer) 2
	127.0.0.1:6379> LRANGE list1 0 -1
	1) "vicky"
	2) "haima"
	127.0.0.1:6379> RPUSH list1 "hanmei"	//从右边压入队列
	(integer) 3
	127.0.0.1:6379> LRANGE list1 0 -1
	1) "vicky"
	2) "haima"
	3) "hanmei"

eg| ltrim

	127.0.0.1:6379> LPUSH list1 "xuandong"
	(integer) 4
	127.0.0.1:6379> LPUSH list1 "aobama"
	(integer) 5
	127.0.0.1:6379> LRANGE list1 0 -1	//显示 list中某个范围的元素
	1) "aobama"
	2) "xuandong"
	3) "vicky"
	4) "haima"
	5) "hanmei"

	127.0.0.1:6379> LTRIM list1 1 -1	//从第一个截取到最后
	OK

	127.0.0.1:6379> LRANGE list1 0 -1
	1) "xuandong"
	2) "vicky"
	3) "haima"
	4) "hanmei"

eg| lset 

	127.0.0.1:6379> LSET list1 3 "haima"	//设置第三个为 haima
	OK
	127.0.0.1:6379> LRANGE list1 0 -1
	1) "xuandong"
	2) "vicky"
	3) "haima"
	4) "haima"

eg| rpoplpush

	127.0.0.1:6379> RPOPLPUSH list1 list2	//list1的右边弹出，list2的左边压入
	"haima"
	127.0.0.1:6379> LRANGE list1 0 -1
	1) "xuandong"
	2) "vicky"
	3) "haima"
	127.0.0.1:6379> LRANGE list2 0 -1
	1) "haima"

# Set 类型
set集合中的元素都是不重复的，无序的
set集合中的元素也是string型

## 常见命令

> **sadd**	【 SADD key member [member ...]】　添加元素
**smembers**　【SMEMBERS key】	　获取所有的元素
**srem**	【SREM key member [member ...]】  删除一个元素，或多个元素
**sismember**	【 SISMEMBER key member】 判断元素是不是该set中的元素
**sdiff**	【SDIFF key [key ...]】  前一个集合减去后一个集合，返回差集
**sinter**	【SINTER key [key ...]】 返回集合的交集
**sunion**	【 SUNION key [key ...]】 返回集合的并集
**sdiffstore**	【SDIFFSTORE destination key [key ...]】 集合求差集，然后再保存到 destination集合中
**sinterstore**	【 SINTERSTORE destination key [key ...]】集合求交集，然后保存到destination集合中
**sunionstore**	【SUNIONSTORE destination key [key ...]】集合求并集，然后保存到destination集合中
**scard**	【SCARD key】 集合中元素的个数
**spop**	【SPOP key】	随机从集合中取出并删除一个元素
**srandmember**	【 SRANDMEMBER key [count]】  随机返回 集合的子集。注意当count大于零时，返回的元素最多为全集，count小于0时，返回的值中可以重复出现集合中的元素。
**smove** 【SMOVE source destination member】 将 member 元素从 source 集合移动到 destination 集合。

# Sorted Set 类型
有序集合，在集合类型的基础上为集合中的每个元素都关联了一个分数，这样可以很方便的获得分数最高的N个元素(topN)
sorted set 中的每个元素也是string型

## 常见命令
**zadd** 【ZADD key score member [[score member] [score member] ...]】 
**zscore**	【ZSCORE key member】 返回分值
**zrange**	【ZRANGE key start stop [WITHSCORES]】返回有序集 key 中，指定区间内的成员，按分值正序，WITHSCORES参数表示连同score一起返回
**zrevrange** 【zrevrange key start stop [WITHSCORES]】返回有序集 key 中，指定区间内的成员，按分值逆序，WITHSCORES参数表示连同score一起返回
**zrangebyscore** 【ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]】
可选的 LIMIT 参数指定返回结果的数量及区间(就像SQL中的 SELECT LIMIT offset, count )，注意当 offset 很大时，定位 offset 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。
**zincrby** 【ZINCRBY key increment member】 为有序集 key 的成员 member 的 score 值加上增量 increment
**zcard**  【ZCARD key】 返回元素个数
zcount	【ZCOUNT key min max】 返回分值在区间内的 元素个数
**zrem** 【ZREM key member [member ...]】 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略

## Sorted Set 和List类型比较

- 有序集合类型和列表类型的差异
- 相同点
- （1）二者都是有序的
- （2）二者都可以获得某一范围的元素
- 不同点
- （1）列表类型是通过双向链表实现的，获取靠近两端的数据速度极快，当列表中元素增多后，访问中间的数据速度会很慢，所以它比较适合很少访问中间元素的应用
- （2）有序集合类型是使用散列表和跳跃表（skip list）实现的，所以即使读取位于中间部分的数据速度也很快
- （3）列表中不能简单的调整某个元素的位置，但是有序集合可以（通过更改这个元素的分值）
- （4）有序集合要比列表类型更耗费内存


# HyperLogLog 类型

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

## PFADD 命令
> 格式【 PFADD key element [element ...]】 
> 添加元素，重复元素不会被添加进去

## PFCOUNT 命令
> 格式【PFADD key element [element ...]】
> 获取元素个数（会用HyperLogLog数据结构去重）

	127.0.0.1:6379> pfadd pf "haima" "xuandong"
	(integer) 1
	127.0.0.1:6379> PFCOUNT pf 
	(integer) 2
	27.0.0.1:6379> PFADD pf1 "hh" "haima"
	(integer) 1
	127.0.0.1:6379> PFCOUNT pf pf1
	(integer) 3	//pf和pf1中共有 hh,haima,xuandong三个元素

## PFMERGE命令
格式 【PFMERGE destkey sourcekey [sourcekey ...]】
将多个 HyperLogLog 合并（merge）为一个 HyperLogLog

	127.0.0.1:6379> PFADD  nosql  "Redis"  "MongoDB"  "Memcached"
	(integer) 1
	127.0.0.1:6379> PFADD  rdbms  "MySQL" "MSSQL" "PostgreSQL" "Redis"
	(integer) 1
	127.0.0.1:6379> PFMERGE all  nosql rdbms
	OK
	127.0.0.1:6379> PFCOUNT all 
	(integer) 6