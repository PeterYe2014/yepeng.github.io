# Redis 学习笔记

## 五种常见的数据类型

### String(SDS 简单字符串)
字面量数据，最简单的一种数据类型，可以存储字符串、整数、浮点数数据
```
# 获取值
get <key>
# 设置值
set <key> <value>
# 删除值
del <key>
```
redis还提供了自增命令：
```
# key 对应值加1
incr <key> 
# key 对应值减1
decr <key>
# key 对应值加amount
incrby <key> <amount>
# key 对应值减amount
decrby <key> <amount>
# key对应值加上浮点数
incrbyfloat <key> <amount>
```
此外还有对于串和二进制位的操作
```
# 添加值到末尾
append <key> <value>
# 获取子串 [start，end]
getrange <key> <start> <end>
# 获取二进制串偏移位
getbit <key> <offset>
# 设置二进制串某个偏移位
setbit <key> <offset> <value>
# 统计二进制串值为1的位的数量
bitcount <key> [start end]
# 二进制串进行逻辑操作后存储到dest-key
# operation：add、or、xor、not
bitop <operation> <dest-key> key1 [key2…]
```
### List（链表）
列表元素，可以通过key存储一个列表
```
# 添加item到key对应list的末尾
rpush <key> <item>
# 添加item到key对应list的开始
lpush <key> <item>
# 移除list末尾元素/出栈
rpop <key> <item>
# 移除list第一个元素/出队
lpop <key> <item>
# 获取list范围的值，闭区间
lrange <key> <start> <end>
# 获取list索引值
lindex <key> <index>
```
观察发现操作list的所有基础命令都是**l**开头的，并且分为三类：pop、push、index，分别对应移除，添加和索引
犹豫list可以被用作队列和栈，那么也应该能提供阻塞式访问的需求：
```
# 阻塞式弹出
blpop/brpop <key> [key…] <timeout>
# 特殊操作：弹出key1右边元素加入key2左边，并返回
rpoplpush <key1> <key2> <timeout>
# 同理上面的也有阻塞版
brpoplpush <key1> <key2> <timeout>
```


### SET
存储一个集合，每个集合的元素不相同，基础操作主要包括：添加、删除、返回所有元素和判断是否包含某个元素
```
# 添加元素到集合
sadd <key> <item>
# 从集合中删除元素
srem <key> <item>
# 返回所有的元素
smembers <key>
# 判断元素是否在集合中
sismember <key> <item>
# 集合数量
scard <key>
# 元素在集合间移动
smove <key1> <key2> <item>
# 随机返回集合中几个元素
srangemember <key> <count>
```
此外还有集合特有的操作：**集合运算**，求两个集合的交集、并集、差集
```
# 返回差集
sdiff <key1> [key2…]
# 存储差集
sdiffstore <dest-key> <key1> [key2…]
# 此外还有：sinter、sunion命令

```
### HASH（哈希表）
散列，可以存储多个键值对的映射，除了resdis数据的一个建键值，还需指定子键。
```
# 添加映射到散列
hset <key> <sub-key> <value>
# 散列中移除某个子键
hdel <key> <sub-key>
# 获取子键对应的值
hget <key> <sub-key>
# 获取所有的键值对
hgetall <key>
# 获取数量
hlen <key>
# 获取多个
hmget <key> <sub-keys>
# 设置多个
hmset <key> <key-values>
```
其它高级操作
```
# 存在子键
hexists <key> <sub-key>
# 获取所有键
hkeys <key>
# 获取所有值
hvalues <key>
# 对应键增加一个int或者float
hincrby/hincrbyfloat <key> <amount>

```
### ZSET(跳跃表)
有序集合，集合是按照每个元素的分数来进行排序的。
```
# 添加带score元素到有序集合
zadd <key> <score> <item>
# 获取一定范围类的 元素
# 其中withoutscores 表示是否返回分数
zrange <key> <start> <end>[withscores]
# 根据key删除元素，返回移除的数量：0 or 1
zrem <key> <item>
# 成员数量
zcard <key>
# 获取分值
zscore <key> <item>
# 获取排名
zrank <key> <item>
# 增加分值
zincrby <key> <amount> <item>
# 统计区间分值数量
zcount <key> <min> <max>
```
高级有序集合的操作

```
# 获取分值区间的元素，不排序
zrangebyscore <key> <min> <max> [withscores]
#获取分值区间元素，按分数排序返回
# zrevrangebyscore …
# 移除区间排名成员
zremrangebyrank <key> <start> <stop>
# 移除区间分数成员
zremrangebyscore <key> <min> <max>
```


#### 底层存储结构

**SDS**

```c
struct sdshdr{
   //记录buf数组中已使用字节的数量
   //等于 SDS 保存字符串的长度
   int len;
   //记录 buf 数组中未使用字节的数量
   int free;
   //字节数组，用于保存字符串
   char buf[];
}
```

为什么要这样设置？

**双向链表**

**HASH表**

**跳跃表**

1. 多层结构
2. 每层都是有序链表
3. 最底层包含所有的元素
4. 一个节点有两个指针，一个指向同一层下一个节点，另一个指向下一层同个节点
5. 查找平均时间复杂度为log(n)

**压缩列表**

相对于定长的数组优化得到的一种特殊列表结构，采用连续存储且不浪费存储空间，节省内存；

一个列表保存字节数、列表节点数、列表尾部节点的位置，压缩列表结束符；

每个节点格式：

```
[previous_entry_length][encoding][content]
# previous_entry_length 上一个节点的字节数
# encoding 当前节点编码 字节数组和正数
# content 节点数据内容
```

## 一些高级的命令

除了基础数据操作命令外，redis还提供了其他的高级命令，主要包含下面的类别：消息通知、排序、事物、过期设置

### 消息通知
redis支持发布（publish）和订阅（subscribe）的消息通知模式。订阅者订阅了某个channel后就能够通过监听（listen）获取发布者发布的消息内容。

```
# 订阅一个或者多个channel
subscribe <channel> [channel…]
# 发布消息到一个channel
publish <channel> <msg>
# 退订
unsubscribe <channel> [channel…]
# 此外还有支持channel的模式订阅的 psubscribe命令
```

### 排序
redis提供sort命令对5种基本数据类型里的数据进行排序：
```
# alpha是根据字符熟悉来排序
sort <key> [by pattern] [limit offset count] [get pattern [get pattern …]] [asc|desc] [alpha] [store dest-key]
```
其中by key指定外部的键值来进行排序。比如现在有学生键：student，同时里面对应每个学生的id：0，1，2；同时student_0 键存有id为0学生对应的name，同理student_1，student_2，在某些情况下，我们需要让学生id根据学生name来排序我们就可以使用sort命令：
```
sort student by student_*
```
使用通配符*表示匹配学生id
### 事务
对于一个数据存储系统来说，最关键莫非对事务的支持了。支持事务的存储系统可以保证多个数据操作的操作要不一定支持，要不全部不执行，从而保证数据的一致性。Redis提供支持事务的命令有：multi、exec、watch、unwatch。

在multi和exec之间的指令会通过打包一起发送给redis服务器，然后再通过流水线（pipeline）一条一条执行（事务性流水线），除了这种还有非事务性流水线

#### watch 和unwatch
redis为了保证数据的一致性，提供了watch和unwatch函数，可以通过watch监视一个键如果在exec命令之前键的值发生了变化，那么就会抛出错误；然后redis通过重试机制继续执行操作（乐观锁）

## 过期时间

在一些项目需求中，会设置数据保存的时间，或者进行一些统计性的操作，我们都可以通过设置键值的过期时间来实现。同时定期清楚无效的数据，可以节省有限的内存空间。
```
# 设置key在second秒后过期
expire <key> <seconds>
# 设置key在指定事件过期
expireat <key> <timestamp>
# 设置键不过期
persist <key>
# 查看key剩余生存时间
ttl <key>
```

## 数据安全和性能

redis数据都是存在内存里面的，不过计算机在某些情况总有被关闭的时候，所以我们需要对数据进行持久化，即存储数据到硬盘里面，redis提供两种持久化的方法：快照和追加修改；前者会一次性保存所以缓存数据到文件，而后者只会增量添加数据到文件。

### 快照
设置快照可以通过两个命令save 和bgsave来手动执行持久化。bgsave命令在linux上会创建一个子进程来执行保存任务，而父进程负责继续接受客服户端的请求命令；save命令则是阻塞的 所以大多数情况下会用bgsave命令，不过当缓存文件有几十个gb时，bgsave会特别慢，此时可以采用save命令。

除了手动保存外，我们还可以通过设置配置文件，设置自动保存：
```
# 300s之内至少有一次写入操作，就自动保存
save 300 1
```
为了避免数据丢失，redis默认在下面的情况下会执行bgsave命令进行自动持久化：
1. 执行shutdown命令后
2. 有子服务器通过sync连接到该服务器时，该服务器会执行bgsave先保存数据

### 追加 AOF

相比于快照，追加方式的持久化更更快捷，同时数据犹豫故障的损失也比较小，可以通过设置配置文件来配置自动追加修改到文件：
```
#开启 AOFA
appendonly yes
# 每一秒添加修改到文件
appendfsync everysec|always|no
```
虽然AOF有快速保存的优点，不过由于不断写入文件操作命令，会让持久化文件越来越大，所以我们需要通过bgrewriteaof 指令来压缩文件，此外也可以设置配置文件自动重写AOF文件：
```
# aof文件增加一倍且，文件大小至少增加
# 64mb，启动bgrewriteaof命令
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```
### 数据文件检查和修复
```
redis-check-aof --fix
redis-check-dump --fix
```

### 复制
为了扩展redis的读写能力和分布式的保存文件，防止数据丢失；redis提供了复制机智，使得redis可以实现主从复制，主服务器(master)把数据副本发送给从服务器(slave)。

redis提供了两种方法来设置从服务器：1）配置文件
```
slaveof host port
```
2）命令
```
slaveof host port
# 取消复制
slaveof no none
```
同时从服务器也能有从服务器，形成**主从链**

### master-slave建立过程

1）从服务器发送sync建立与服务器的连接
2）主服务器收到连接请求，执行bgsave命令（子进程保存数据，父进程缓冲新的指令）
3）主服务器发送快照给从服务器
4）从服务器收到快照，**清除自己的数据**，根据快照恢复数据
5）主服务器发送缓冲的写命令给slave
6）从服务器接受缓冲的命令，并且执行
7）master没收到一条新命令就转发给slave（数据同步）

## 降低内存使用

### 压缩列表(ziplist)

在redis中默认的list、hash、zset结构为了提高操作的性能，都有很复杂的存储结构，会使用很多额外的内存使用。以list为例子，list在redis中是通过双向链表来存储的，同时还加一个额外的指针指向存储数据的结构，该结构里面还有一些额外描述信息，这些额外的存储信息都是为了在单个结构数据存储量特别大的时候，优化计算复杂度的设计。但是当单个结构长度较小时，这种优化效果就有限，同事会浪费内存资源。故此redis提供了压缩列表，可以通过配置文件配置当这些结构长度小于某个值，以及单个元素长度小于一个值的时候采用压缩列表结构，从而优化内存的使用。

### 整数集合(intset)
对与set元素，如果set元素键以及所有的值都可以看成整数等价的值，那么redis也提供了类似压缩列表的压缩结构（整数集合）

```c
typedef struct intset{
	//编码方式
   uint32_t encoding;
   //集合包含的元素数量
   uint32_t length;
   //保存元素的数组,存储格式由encoding决定
   int8_t contents[];
  }intset;
```

### 分片

整数集合和压缩列表都有一个缺陷就是当数据不断增加时，单个结构存储的数据越来越多，会造成读写性能的下降。所以为了能享受压缩结构带来的内存优化的优点，我们可以**将大量的数据拆分为多个小数据**，这就是分片的主要思想。对于不用的应用场景和不用的数据结构，我们都可以设计恰当的方法来对数据进行分片，主要的设计步骤如下：

1. 确定每个分片的大小，SHARD_SIZE，最好单个结构大小不要超过使用压缩结构的临界值）
2. 根据数据原始的key，计算shard_id
3. 存储数据到key:shard_id，这个键

### 定长数据的存储优化
在系统中有很多定长的常量数据，比如使用的地理位置信息，这些常量并且经常重复使用的数据，如果按照一般都方法来进行存储，会耗费大量的内存资源浪费。这里可以设置结构表的方法来压缩存储，例如存储一个用户的城市和地区两维信息，就可以设置两个字符串：city和area，存储所有的地区和area信息；这样就能使用两个字节来代表任何一个用户的地理信息，再使用另一个字符串location来存储每个用户的地理信息（每两个字节存储一个用户）。如果用户数量很多，造成单字符串特别大，还可以采用分片技术，讲一个字符串分为多个小字符串来进行存储。


