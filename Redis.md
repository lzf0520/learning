# Redis

## 一、定义

什么是Redis？

- redis是一个开源的支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
- 其中包含有 5 种基础数据结构，String、Hash、List、Set、SortedSet。
  - 这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作
  - 这些操作都是原子性的。

## 二、数据结构

所有的数据结构是指redis中value中存储的值类型。key是数据查询和存储的依据，需要自定义。

### 2.1 String（字符串）

value 是具体的值，value其实不仅是字符串， 也可以是数字（整数或浮点数），value 最多可以容纳的数据长度是 `512M`。

常用指令操作：

```bash
# 设置键值对
set name longzhifeng

# 读取值
get name

#是否存在key
exists name

#在value后面追加字符串
append name ",nihao"

#获取字符串的长度
strlen name

#设置键值对过期时间.单位是秒
expire name 10

#清除当前数据库的内容
flushdb

#批量插入
mset age 1 detail nihao

#批量查询
mget age detail

#自减1，只能用于integer类型
decr age

#自增1
incr age

#自增或者自减任意值
incrby/decrby age 10

#获取字符串的部分字段[0,3]
getrange name 0 3

#查看全部的字符串
getrange name 0 -1

#替换指定位置开始的字符串
setrange name 1 XX

#设置过期时间 set with expire
setex name 30 nihao

#设置键值对，如果不能存在键的话 set if not exist（常用于分布式锁中）
#msetnx表示进行批量操作，但是具有原子性，要么都成功要么都失败
setnx mykey redis

#先get再set，如果存在则返回原值，再将新值存进去
getset db redis
```

### 2.2 list(列表)

在redis中，可以将list当成栈、队列或者双端队列。

所有的list命令都是用l开头的

常用指令操作：

```bash
#往list这个key中插入list从左边添加
LPUSH list one two three

#往右边插入
RPUSH list four

#查询list中的所有值
LRANGE list 0 -1

#从左边移除一个值
LPOP list

#从右边移除一个值
RPOP list

#获取对应下标的值
LINDEX list 0

#查询list 的长度
LLEN list

#移除对应个数的某个值
LREM list 1 one

#修剪list中从某个下标到另一个下标的值
LTRIM list 0 1 #只保留了前两个，后面的就会全部删除

#将list中最右边的值取出，并从左边存储在另外一个list中
RPOPLPUSH list otherlist

#更新替换对应下标的值，如果下标对应的值不存在则会报错
LSET list 0 item

#在某个存在的值前面或者后面插入指定的值
LINSERT list before three two
LINSERT list after four five
```

### 2.3 set（集合）

set中的值是不能重复的

常用指令操作：

```bash
#添加key和value，其中value是set类型，这个例子中在set中添加了hello和world两个值
sadd myset hello world

#查看set中的所有元素值 
SMEMBERS myset

#查看元素是否存在set中
SISMEMBER myset hello

#获取当前元素的个数
SCARD myset

#移除set中的指定元素
SREM myset hello

#随机抽选出一个元素
SRANDMEMBER myset
#随机抽取指定个数的元素
SRANDMEMBER myset 2

#随机删除set中的元素
SPOP myset

#将一个set中指定的值移动到另外一个set中，将myset中的longzhifeng这个元素移动到myset2中
SMOVE myset myset2 longzhifeng

#计算两个set中的差集，以key1为参考，与key2不同的元素
SDIFF key1 key2

#计算两个set的交集，以key1为参考，与key2中相同的元素
SINTER key1 key2

#计算两个set的并集
SUNION key1 key2
```



### 2.4 hash（哈希）

key对应的value中存储的是一个map（键值对）

**使用场景**：1.存储变更的数据

常用指令操作：

```bash
#往myhash这个key中插入一个键值对<field1，longzhifeng>
hset myhash field1 longzhifeng

#获取hash中的值
hget myhash field1

#批量插入
hmset myhash field1 hello field2 world

#批量获取
hmget myhash field1 field2

#查询hash中的所有键值对，返回的是map中的key和value
hgetall myhash
1) "field1"
2) "hello"
3) "field2"
4) "world"

#删除hash中指定的key，hash中的不是redis中的
hdel myhash field1

#查看hash中的键值对数量
hlen myhash

#判断hash中的某个key是否存在
hexists myhash field1

#只获取所有的key值
hkeys myhash

#只获取所有的value值
hvals myhash

#hash中某个key自增或者自减
HINCRBY myhash field3 5
```



### 2.5 Zset（有序集合）

和set类似，但是需要在set的基础上新增加一个值，例如：set k1 v1；Zset k1 score  v1

**使用场景**：1. 成绩表排序 2.工资表排序 3. 排行榜的应用

常用指令操作：

```bash
#添加值
zadd myset 1 one

#添加多个值
zadd myset 2 two 3 three

#查看所有的元素
zrange myset 0 -1

#从小到大排序，也可以指定区间，但是区间要求从小到大
ZRANGEBYSCORE salary -inf +inf
#从小到大排序，并且显示score
ZRANGEBYSCORE salary -inf +inf withscores
#从大到小排序，但是不包含区间
ZREVRANGE salary 0 -1
#从大到小排序，并且显示score
ZREVRANGE salary 0 -1 withscores

#移除一个元素
zrem salary longzhifeng

#获取集合中的个数
zcard salary

#获取区间的成员数量，区间是指score的值，并且可以用（100 10000表示（100，10000]开区间
zcount salary 100 10000
```

## 三、事务

redis事务：一次性、顺序性、排他性的执行一系列的命令

redis的单挑命令是保证原子性的，但是**事务是不保证原子性的**

redis事务**没有隔离级别的概念**

redis事务：

- 开启事务（multi）
- 命令入队
- 执行事务（exec）
- 放弃事务（discard），放弃之后入队的命令都不会被执行

```bash
127.0.0.1:6379> MULTI				#开启事务
OK
#命令入队
127.0.0.1:6379> set v1 k1
QUEUED
127.0.0.1:6379> set v2 k2
QUEUED
127.0.0.1:6379> get v1
QUEUED
127.0.0.1:6379> set v3 k3
QUEUED
127.0.0.1:6379> exec				#执行事务
1) OK
2) OK
3) "k1"
4) OK
```

#### 异常

> 命令有错误（类似于java中的编译型异常），那么事务中的所有命令都不会被执行

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 				#命令错误
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec  				#执行事务报错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1 				#所有的命令都没有执行
(nil)
```



> 命令中存在语法性错误（类似于java中的运行时异常），那么事务执行的时候其他语句会执行成功，而错误命令抛出异常！

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range 			#虽然第一条命令报错了，但是整个事务和后续的命令依旧执行成功了
2) OK
3) OK
127.0.0.1:6379> get k3
"v3"
```

## 四、持久化

### RDB

#### **原理**

RDB持久化就是记录某一个瞬间的所有内存**数据**，将其保存在dump.rdb文件中。恢复数据的时候就是直接将dump.rdb文件中的数据读取到内存中即可。

#### **执行流程**

![image-20230831112142099](C:\Users\lzf\AppData\Roaming\Typora\typora-user-images\image-20230831112142099.png)

#### 执行快照时，父进程中能否修改数据？

当然可以。关键技术：**写时复制技术（Copy-On-Write）**

执行时会通过`fork()`创建子进程，此时子进程和父进程是共享同一片内存数据的，因为创建子进程的时候会复制父进程的页表，并且页表指向的物理地址是一样的。

当父线程要修改共享数据中的某块数据时（比如说键值对`A`），就会发生写时复制，于是这块数据的物理内存就会被复制一份（键值对 `A'`），然后主线程在这个数据副本（键值对 `A'`）进行修改操作。与此同时，子进程可以继续把原来的数据（键值对 `A`）写入到 RDB 文件。

#### 优缺点

**优点**

- 适合大规模的数据恢复！
- 对数据的完整性要求不高！

**缺点**

- 需要进行一定的时间间隔进行操作！如果redis意外的宕机了，那么最后的修改数据就会丢掉！！！
- fork进程时，会占用一定的内存空间！！！



### AOF

#### 原理

AOF持久化是通过记录每次执行的操作命令，从而实现持久化。操作的命令是保存在当前目录的`appendonly.aof`文件中，恢复时就将命令执行一遍即可。

> 注意！！
>
> 文件中存储的命令只包含有写操作，读操作是不进行记录的，因为没有意义！！！

#### 执行流程

![image-20230831150116056](C:\Users\lzf\AppData\Roaming\Typora\typora-user-images\image-20230831150116056.png)

#### 写回策略

redis.config配置文件中`appendsync`配置项包含以下几种参数：

- **Always**，表示每有一条写指令执行完成都会同步到AOF文件中
- **Everysec**，表示每次写操作命令执行完成之后，首先将命令写入到AOF文件的内核缓存区，然后每隔一秒钟将缓冲区的内容写进磁盘；**默认使用的是这种配置**
- **NO**，表示不由redis控制写回磁盘的时机，转交给操作系统控制写回的时机，也就是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，再由操作系统决定何时将缓冲区内容写回硬盘。



## 五、主从复制

> 复制原理

slave启动成功链接到master后会发送一个sunc同步命令

master接到命令，启动后台的存盘进度，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步

**全量复制**：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中

**增量复制**：master继续将新的所有收集到的修改命令依次传给slave，完成同步

但是只要是重新连接master，一次完全同步（全量复制）将被自动执行！我们的数据一定可以在从机中看到！









