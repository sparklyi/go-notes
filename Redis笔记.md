## Redis笔记



### 基础配置

#### 1.启动redis服务器

```
redis-server
```

####  

#### 2.启动redis客户端

```
redis-cli [-h host] [-p port] [-a password] --raw
```

参数：

> -h 连接的主机地址
>
> -p 连接的主机的端口号
>
> -a 连接的主机的密码
>
> --raw  存储内容以原样显示，不加此参数中文会以十六进制显示



#### 3.选择数据库

```
select index //自动创建 默认是0号数据库
```



### 键命令

**Redis中所有值都是以字符串存储的**

#### 1. 设置键

```
set key value
```



#### 2.获取键值

```
get key
```



#### 3.移除键

```
del key
```



#### 4.判断键是否存在

```
exists key
```

返回(integer)1 表示存在，(integer)0 表示不存在



#### 5.设置键过期时间

```
expire key seconds		 //秒
pexpire key milliseconds //毫秒
```



#### 6.查看键的生存时间

```
ttl key	 //以秒查看生存时间
pttl key //以毫秒查看生存时间
```

**-1**为持久有效，**-2**为已失效，非负数表示剩余时间



#### 7.查看键所存储的值的数据类型

```
type key
```



#### 8.设置键为持久有效

```
persist key
```



#### 9.清除库中所有键

```
flushdb
```



#### 10.正则匹配键名

```
keys patterm //key demo*   不加*则是查找demo
```



#### 11.移动键

```
move key db //移动key到db数据库
```



#### 12.重命名

```
rename key newkey //不检查是否存在，都是直接替换数据
renamenx key newkey //存在时不允许替换，推荐使用
```



#### 13.序列化键值

```
dump key
```



#### 14.恢复序列化的键值到newkey

```
restore newkey ttl val [replace]
```

参数

> ttl:newkey的生存时间 0为持久连接 单位为毫米 
>
> val : 序列化值
>
> replace参数 加入此参数可以替换成某个存在的键，不加此参数newkey不能使用存在的键





### 数据类型-string



#### 1.设置和获取与键一致

```
127.0.0.1:6379> set demo 1234
OK
127.0.0.1:6379> get demo
"1234"
```



#### 2.设置key，返回旧值

```
getset key value 

127.0.0.1:6379> getset demo2 dddd
(nil)
127.0.0.1:6379> getset demo2 aaa
"dddd"
```

以前不存在此key返回nil



#### 3.获取区间值

```
getrange key start end

127.0.0.1:6379> set demo 1234
OK
127.0.0.1:6379> getrange demo 0 2
"123"
```

区间为闭区间 start从0开始



#### 3.获取长度

```
strlen key

127.0.0.1:6379> strlen demo
(integer) 4
```



#### 4.获取多个键值

```
mget key1 [keys].

127.0.0.1:6379> mget demo demo2
1) "1234"
2) "aaa"
```



#### 5.设置键的值

```
1.set key val
2.setex key seconds val
3.setnx key val

127.0.0.1:6379> set demo 111
OK
127.0.0.1:6379> setex demo 100 222
OK
127.0.0.1:6379> persist demo
(integer) 1
127.0.0.1:6379> setnx demo 333
(integer) 0

设置多个键值
mset key1 val1 key2 val2 [keys values]
msetnx key1 val1 [keys values]
```

区别

> set key value 无视key的类型，直接覆盖
>
> setex key seconds val 是set key value + expire key seconds结合操作
>
> setnx key val 只有当key不存在时可以成功，不可覆盖key



#### 6.键中的数字+1 -1

```
incr key
decr key
127.0.0.1:6379> incr demo
(integer) 223
127.0.0.1:6379> decr demo
(integer) 222
127.0.0.1:6379> set demo aaa
OK
127.0.0.1:6379> incr demo
(error) ERR value is not an integer or out of range
```

值非数字会报错



#### 7.追加字符串

```
append key val

127.0.0.1:6379> append demo bbb
(integer) 6
127.0.0.1:6379> get demo
"aaabbb"
```



### 数据类型-hash

#### 1.创建

```
hmset key field value [field value ...]

127.0.0.1:6379> hmset obj id 1 name A age 3
OK
```

#### 2.查看字段和值

**hget key field** 

查看key的具体某一个字段的值

```
127.0.0.1:6379> hget obj id
"1"
```

**hgetall key** 

查看key的所有字段及值

```
127.0.0.1:6379> hgetall obj 
1) "id"
2) "1"
3) "name"
4) "A"
5) "age"
6) "3"
```

**hkeys key** 

查看key的所有字段

```
127.0.0.1:6379> hkeys obj
1) "id"
2) "name"
3) "age"
```

**hvals key**

获取key中所有字段的值

```
127.0.0.1:6379> hvals obj
1) "1"
2) "A"
3) "3"
```



#### 3.查看key中某一字段是否存在

```
hexists key field

127.0.0.1:6379> hexists obj id
(integer) 1
```



#### 4.删除一个或多个字段

```
hdel key field1 [field2]

127.0.0.1:6379> hdel obj id
(integer) 1
```



#### 5.获取哈希表中字段的数量

```
hlen key

127.0.0.1:6379> hlen obj
(integer) 2
```



#### 6.设置字段值

**hset key field value**

设置key某一个字段的值，无视字段是否存在

```
127.0.0.1:6379> hset obj id 2
(integer) 1
```

**hmset key field1 value1 [fields values ]**

设置一个或多个字段的值，无视字段是否存在

```
127.0.0.1:6379> hmset obj id 1 name C
OK
```

**hsetnx key field value**

设置某一个字段的值，只有字段不存在时可以设置 

```
127.0.0.1:6379> hsetnx obj id 4
(integer) 0  设置失败
```



#### 7.增加指定字段的值(数字)

**hincrby key field num**

为字段值增加num(**整数**)

```

127.0.0.1:6379> hincrby obj id 300
(integer) 304
```

**hincrbyfloat key field num**

为字段值增加num(**浮点数**)

```
127.0.0.1:6379> hincrbyfloat obj id 222.222
"748.22199999999997999"
```



### 数据类型-list  双向链表

**允许值重复**

#### 1.创建

**lpush key value [values]** 

 头部插入一个或多个值，key不存在自动创建

```
127.0.0.1:6379> lpush list 1
1
```

**rpush key value [values]**

尾部插入值一个或多个值，key不存在自动创建

```
127.0.0.1:6379> rpush list 1
2
```

**lpushx key value**

向已存在的key的头部插入一个值

```
127.0.0.1:6379> lpushx list 22
4
```

**rpushx key value**

向已存在的key的尾部插入一个值

```
127.0.0.1:6379> rpushx list 22
5
```

#### 2.移除

**lpop key**

移除并返回头部元素

```
127.0.0.1:6379> lpop list
22
```

**rpop key**

移除并返回尾部元素

**blpop key1 [keys] timeout**

移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止

```
127.0.0.1:6379> blpop list 100
list
2
```

**brpop key1 [keys] timeout**

移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止

```
127.0.0.1:6379> brpop list 1
list
1
```

**rpoplpush source destination **

将source的尾部元素移除并添加到destination的头部

```
127.0.0.1:6379> rpoplpush list list1
1
127.0.0.1:6379> lrange list1 0 10
1
1
```

不存在时不会自动创建

**brpoplpush source destination timeout** 

效果与rpoplpush一致，无值时阻塞



**ltrim  key start stop**

移除除开[start, stop]以外的所有元素

```

127.0.0.1:6379> ltrim list1 0 1
OK
127.0.0.1:6379> lrange list1 0 3
1
2
```



**lrem key count val**

参数

> count :
>
> 移除数量，count > 0 从头移除值==val的count个
>
> 移除数量，count < 0 从尾部移除值==val的-count个
>
> 移除数量, count == 0 移除所有值==val 

```
127.0.0.1:6379> lrange list1 0 3
1
1
2
127.0.0.1:6379> lrem list1 2 1
2
127.0.0.1:6379> lrange list1 0 3
2
```



#### 3.获取值

**lrange key start stop **

获取闭区间[start,stop]的值

```
127.0.0.1:6379> lrange list1 0 1
1
1
```

**lindex keys index**

通过索引获取

```
127.0.0.1:6379> lindex list1 1
1
```



#### 4.获取list长度

```
llen key
127.0.0.1:6379> LLEN list1
2
```



#### 5.插入

**linsert key before|after pivot val**

在key的值pivot前或后插入val

```
127.0.0.1:6379> linsert list1 after 1 2
3
127.0.0.1:6379> lrange list1 0 3
1
2
1
```

**存在相同值时以第一个piovt为准**



#### 6.设置值

**lset key index val**

设置已存在的key的index下标的值为val，不存在或越界时报错

```
127.0.0.1:6379> lset list1 0 222
OK
127.0.0.1:6379> lrange list1 0 3
222
```



### 数据结构-set

集合成员唯一 ,不能出现重复值，无序



#### 1.创建

**sadd key member1 [members] **

为key添加成员，不存在时自动创建

```
127.0.0.1:6379> sadd set 1 2
2
```



#### 2.获取

**scard key**

获取key的成员数

```
127.0.0.1:6379> scard set
2
```

**smembers key**

返回key的所有成员

```
127.0.0.1:6379> smembers set
1
2
```



#### 3.交并集等

**sdiff  key1 [keys]**

返回key1独有的元素,不是所有集合的差集

```
127.0.0.1:6379> sadd set1 3 4
2
127.0.0.1:6379> sdiff set set1
1
2
```

**sdiffstore destination key1 [keys]**

返回所有集合的差集到destination，存在则覆盖 

```
127.0.0.1:6379> sdiffstore sett set set1
2
```

**sunionstore destination key1 keys **

返回所有集合的并集到destination 

**sinterstore destination key1 keys **

返回所有集合的交集到destination 



**sunion key1 keys**

返回所有给定集合的并集

```
127.0.0.1:6379> sunion set set1
1
2
3
4
```

**sinter key1 keys **

返回所有给定集合的交集

```
127.0.0.1:6379> sunion set set1
1
2
3
4
127.0.0.1:6379> sinter set set1

```



#### 4.判断

**sismember key member**

判断member是否为key中成员 

```
127.0.0.1:6379> sismember set 2
1
```



#### 5.移除

**srem key val1 val2**

移除key中一个或多个元素

```
127.0.0.1:6379> srem set 1
1
```

**spop key**

随机移除一个元素

```
127.0.0.1:6379> spop set
4
```



#### 6.移动

**smove source destination val**

将val从source中删除，添加到destination ,destination中存在则不变，source中不存在不执行

```
127.0.0.1:6379> smove set set2 3
1
127.0.0.1:6379> smembers set2
1
2
3
```



### 数据类型-sorted set

每个成员关联的一个score来实现有序，成员分数可以重复，值依然不可重复，默认从小到大排序

当成员> 64时采用了hash和skiplist两种设计，添加修改时使用skiplist，查询等为hash

小于时使用的ziplist



#### 1.添加

**zadd key score1 member1 [score member]**

添加一个或多个关联分数的成员

```
127.0.0.1:6379> zadd sset 20 1
1
```



#### 2.获取

**zcard key**

返回key的成员数



**zcount key min max**

计算分数在[min,max]区间内的成员数

```
127.0.0.1:6379> zcount sset 10 20
1
```



**zrank key member**

返回指定成员的索引

**zrevrank key member**

...，根据分数从高到低排名

```
127.0.0.1:6379> zrank sset 999
0
```

**zscore key member**

返回指定成员的分数

```
127.0.0.1:6379> zscore sset 999
3
```

**zrangebyscore key min max**

返回分数在[min,max]区间内的元素，从低到高

**zrevrangebyscore key min max**

返回分数在[min,max]区间内的元素，从高到低

```
127.0.0.1:6379> zrangebyscore sset 0 20
999
1
```

集合不存在报错



**zrange key start stop**

返回索引区间内的所有元素



#### 3.移除

**zrem key member1 [members]**

移除一个或多个成员 

**zremrangebyrank key start stop**

移除排名为[start,stop]的成员

**zremrangebyscore key start stop**

移除分数为[start,stop]的成员



### HyperLogLog

**基数统计**，不重复数字的个数

在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

#### 1. 添加

**pfadd key elements**

添加一个或多个

```
127.0.0.1:6379> pfadd cnt 2 1 2
1
```



#### 2. 统计

**pfcount key**

返回基数估算值

```
127.0.0.1:6379> pfcount cnt
2
```



#### 3. 合并

**pfmerge destkey key1 keys**

将多个key合并到destkey中

```
127.0.0.1:6379> pfadd cnt1 3 4 5 67 3 2
1
127.0.0.1:6379> pfmerge cnt cnt cnt1
OK
127.0.0.1:6379> pfcount cnt
6
```



### 事务

redis事务不具有原子性，执行失败不会导致回滚，单个redis命令具有原子性



Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。

- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。

- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。



开始事务-->命令入队-->执行事务

#### 1.事务开始

**multi**

标记一个事务块的开始。



### 2.执行事务

**exec**

执行所有事务块内的命令。



### 3.取消事务

**discard**

取消事务，放弃执行事务块内的所有命令。



