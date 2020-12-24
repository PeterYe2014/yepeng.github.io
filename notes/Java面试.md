# JAVA 面试知识

### JAVA异常：

```
Throwable 
	Exception
	Error
```

1、Error
2、Runtime Exception 运行时异常
3、Exception
4、throw 用户自定义异常

Error类代表了**编译和系统**的错误，不允许捕获

Exception类代表了**标准Java库方法**所激发的异常，其中的Exception又分为检查性异常和非检查性异常，**检查性异常** 必须在编写代码时，使用try catch捕获（比如：IOException异常）。**非检查性异常** 在代码编写使，可以忽略捕获操作

运行异常类对应于编译错误，它是指Java程序在运行时产生的由解释器引发的各种异常。Java语言中的运行异常不一定被捕获

#### 常见异常

java.lang.NoSuchMethodError

方法不存在错误。当应用试图调用某类的某个方法，而该类的定义中没有该方法的定义时抛出该错误。

java.lang.OutOfMemoryError

内存不足错误。当可用内存不足以让Java虚拟机分配给一个对象时抛出该错误。

java.lang.StackOverflowError

堆栈溢出错误。当一个应用递归调用的层次太深而导致堆栈溢出时抛出该错误



算术异常类：ArithmeticExecption

空指针异常类：NullPointerException

类型强制转换异常：ClassCastException

数组负下标异常：NegativeArrayException

操作数据库异常：SQLException

输入输出异常：IOException

方法未找到异常：NoSuchMethodException



**Q: NoClassDefFoundError 和 ClassNotFoundException 有什么区别**

NoClassDefFoundError发生场景如下：（加载class文件，未知的错误，无法规避的错误）

1、类依赖的class或者jar不存在 （简单说就是maven生成运行包后被篡改）    

2、类文件存在，但是存在不同的域中 （简单说就是引入的类不在对应的包下)  

3、大小写问题，javac编译的时候是无视大小的，很有可能你编译出来的class文件就与想要的不一样！这个没有做验证   

ClassNotFoundException发生场景如下：（加载类元信息，可能预判的，应用程序的错误）

1、调用class的forName方法时，找不到指定的类 

2、ClassLoader 中的 findSystemClass() 方法时，找不到指定的类

**Q:  throw、throws **

throw是存在于方法的代码块中；而throws是存在于方法外围，一般是在方法名后边 throws XXXException;

```

try{
    retrun 3;
}catch{
    e.printStackTrace();
}finally{
    return 4; // 覆盖上面的3
}

//上边情况下，实际返回的是4；

try{
    int x = 3;
    retrun x; // 暂存在返回区域类
}catch{
    e.printStackTrace();
}finally{
    x++;
}

//上边情况下，实际返回的3；
```

## 集合（Collections）

Set、List、Queue

Vector 线程安全，扩容是1倍，ArrayList扩容0.5倍

BlockingQueue 线程安全

vector、statck、hashtable、enumeration线程安全

## 栈

FILO 先进后出的一个结构，可以通过栈来实现递归算法的非递归实现。

Java中可以用Stack或者LinkedList来创建一个栈：

```
Stack<Integer> stack = new Stack<Integer>();
int top = stack.peek(); // 查看栈顶元素，不移除
int num = stack.pop(); // 弹出栈顶元素，并移除
stack.push(100); // 将数据压入栈
int pos = stack.search(100); // 返回数据的位置，从1开始索引
```

使用LinkedList的方式，注意LinkedList中存储栈数据时，第一个元素才是栈顶元素，也就是元素下标为0的元素时栈顶。

```
LinkedList<Integer> stack = new LinkedList<Integer>();
stack.push(100);
stack.pop();
stack.peekFirst(); // 返回栈顶元素
```

## 散列表（Map）

Java 中提供 ```Map```接口来定义散列表的基础操作。常用的实现有```HashMap```,```TreeMap```,```HashTable```,```HashTable```，```ConcurrentHashMap```, ```SynchronizedMap```

底层结构

```java
// HashTable 默认元素11个，不要求2的幂个元素
private transient Entry<?,?>[] table;
// HashMap 默认16个，并且要求容量为2的幂个元素
transient Node<K,V>[] table;
// ConcurrentHashMap 取而代之的是采用Node + CAS + Synchronized来保证并发安全进行实
transient volatile Node<K,V>[] table;
// Node是一个链表节点，并且Node节点继承Map.Entry<k,V>
```

扩容：

```java
// Hashmap
// 当size大于threshold时，就进行扩容，threshold = loadfactor * lastCap
final Node<K,V>[] resize() 
    1. 之前的cap为0，初始化cap为默认大小16
    2. 之前的cap大于0，容量翻倍
// ConcurrentHashMap
    // 超过阈值时扩容，两倍，
    1. 标记头节点的hash为-1（ForwardingNode）表明正在进行扩容，新加入的线程帮忙扩容
    2. 扩容完成后才能够进行更新操作
```

### HashMap的扩容操作是怎么实现的？

①.在jdk1.8中，resize方法是在hashmap中的键值对大于阀值时或者初始化时，就调用resize方法进行扩容；

②.每次扩展的时候，都是扩展2倍；

③.扩展后Node对象的位置要么在原位置，要么移动到原偏移量两倍的位置。

在putVal()中，我们看到在这个函数里面使用到了2次resize()方法，resize()方法表示的在进行第一次初始化时会对其进行扩容，或者当该数组的实际大小大于其临界值值(第一次为12),这个时候在扩容的同时也会伴随的桶上面的元素进行重新分发，这也是JDK1.8版本的一个优化的地方，在1.7中，扩容之后需要重新去计算其Hash值，根据Hash值对其进行分发，**在1.8版本中，则是根据在同一个桶的位置中进行判断(e.hash & oldCap)是否为0，重新进行hash分配后，该元素的位置要么停留在原始位置（为0时），要么移动到原始位置+oldCap大小这个位置上**

```
// 扩容前 cap = 64
0011 1111 & 0100 0001 -> hash = 1 
// 扩容后 cap = 128
0111 1111 & 0100 0001 -> new_hash = hash + 64 = 65
```

loadfactor：

```java
// Hashmap/ConcurrentHashMap//HashTable
 static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

分段锁：

```
1. 如果table[i]位置没有头节点，那么通过cas来添加新的节点
2. 否则，就通过synchronized关键字锁住该链表，然后进行添加（分段锁）
```

hash方法：

```java
// HashTable
hash = key.hashCode();
index = (hash & 0x7FFFFFFF) % tab.length; // 取hash正值，没有重hash
// HashMap
int h;
hash = (key == null) ? 0: (h = key.hashCode()) ^ (h>>>16);
// ConcurrentHashMap
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
h = key.hashCode(); // key 不能为NULL
hash = (h ^ (h >>> 16)) & HASH_BITS;
```

二次hash原因是为了让hash中的位分布更均匀一些，然后如何通过hash来获取元素的位置：

```java
// HashMap和ConcurrentMap 初始大小 1 << 4
// 通过位运算减少复杂度 0 <= (x & hash) <= x  
index = (tab.length - 1) & hash
```

put方法

```
// ConcurrentHashMap
1. 如果table为null，则初始化table
2. 否则计算hash，以及插入元素在table中的位置i
3. 如果table[i]中没有节点，通过casTabAt添加新节点到位置table[i]
4. 否则：如果hash大于等于0，从链表的头节点开始遍历，找到末尾的节点，然后插入新节点
5.		hash小于0，table每个元素就为一颗红黑树，就需要插入节点到红黑树中
6. 如果链表的长度大于等于8时，链表就会变为红黑树
```

## 集合(Set)

```
SortedMap
	NavigableMap
		TreeSet 
```

HashSet通过HashMap实现的，put的value为同一个对象PRESENT

向HashSet 中add ()元素时，判断元素是否存在的依据，不仅要比较hash值，同时还要结合equles 方法比较。
HashSet 中的add ()方法会使用HashMap 的put()方法。

HashMap 的 key 是唯一的，由源码可以看出 HashSet 添加进去的值就是作为HashMap 的key，并且在HashMap中如果K/V相同时，会用新的V覆盖掉旧的V，然后返回旧的V，所以不会重复（ HashMap 比较key是否相等是先比较hashcode 再比较equals ）。

**hashCode（）与equals（）的相关规定**：

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等, 对两个equals方法返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

## 同步容器

**CopyOnwriteArrayList(Set)** 避免ConcurrentModificationException 

**SynchronizedSet(List)**

可以通过Collections.synchronizedSet(Set<E> set) 方法来创建一个同步集合，底层实现是通过继承SynchronizedCollection，重新定义了构造函数实现的，SynchronizedCollection里面包装了具体的集合类，并且设置一个对象锁，通过委托模式，访问每个集合方法时，都会使用synchronized同步块来进行同步

 Collections. unmodifiableCollection(Collection c) 方法来创建一个只读集合

### 线程池

```
Executor(excute 接口)
	ExecutorService (生成Future以及管理异步任务的运行情况)
		AbstractExecutorService
			ThreadPoolExecutor
```

最重要的类 *ThreadPoolExecutor*，构造函数如下：

```java
  public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
/** corePoolSize  核心线程数量，默认来一个任务创建一个线程，也可以选择预先创建
   maximumPoolSize 最大线程数量，队列满并且线程数量大于等于coreSize后，会额外创建新线程
   keepAliveTime 空闲线程保持多久就需要停止
   unit keepAliveTime时间的单位
   workQueue 线程数量不够的时候缓存任务的队列，默认当线程数量大于等于coreSize后任务会添加到队列
   threadFactory 用来生成线程的工厂类
   handler	线程数量达到阈值后，拒绝处理任务的策略, 主要有：
   ThreadPoolExecutor.AbortPolicy;//丢弃任务并抛出RejectedExecutionException异常
   ThreadPoolExecutor.DiscardPolicy;//也是丢弃任务，但是不抛出异常。 
   ThreadPoolExecutor.DiscardOldestPolicy;//丢弃队列最前面的任务，然后重新尝试执行任务
   （重复此过程）
   ThreadPoolExecutor.CallerRunsPolicy;//由调用线程处理该任务 
**/
```

最重要的方法：

```java
// 添加任务待线程池运行
public void execute(Runnable command);
// 等待任务执行完毕后关闭线程池，状态为SHUTDOWN
shutdown();
// 立即关闭线程池，中断任务；状态为STOP
shutdownNow();

// AbstractExecutorService中实现，通过Future异步接收任务的返回
public <T> Future<T> submit(Callable<T> task);
```

重要的辅助类：**Worker**

添加任务时，当没有足够线程时，会通过Worker创建线程并且绑定添加的任务，并且Worker会不断从阻塞队列里面获取任务，如果获取不到任务后，KeepAlive时间过了，就会通过Worker回收这些空闲的线程

线程池创建方法

```java
newFixedThreadPool() // 指定工作线程数量的线程池，不会主动释放空闲线程
newWorkStealingPool()  // forkjoin线程池，不会有空闲线程，在多个线程上面均分任务
newSingleThreadExecutor() // 顺序执行的工作队列
newCachedThreadPool() // 创建数量几乎没有限制，Integer.MAX_VALUE，大量的短作业，空闲线程会自动回收
newScheduledThreadPool() // 用于调度任务执行，通过延迟队列来实现
```



## 一些奇葩的问题

容器能否添加null，map的key或者value能否为null：

```
// 都可以为null
HashMap map = new HashMap();
map.put(null, null);

// 默认不能为null， 可以改写Comparator接口
TreeSet treeSet = new TreeSet();
treeSet.add(null);

ArrayList list = new ArrayList();
list.add(null);

//默认 key 不能为null，可以改写Comparator接口
TreeMap treeMap = new TreeMap();
treeMap.put("a", null);

// key value都不能为null，并发容易出现问题
ConcurrentHashMap conMap = new ConcurrentHashMap();
conMap.put("", "");

// key value 都不能为null
Hashtable hashtable = new Hashtable();
hashtable.put("", "");

Map symap = Collections.synchronizedMap(map);
symap.put(null, null);
System.out.println(symap.get(null));
```

### Redis

string、list、set、hash、zset

过期键的三种删除策略

- 被动删除：当读/写一个已经过期的key时，会触发惰性删除策略，直接删除掉这个过期key
- 主动删除：由于惰性删除策略无法保证冷数据被及时删掉，所以Redis会定期主动淘汰一批已过期的key
- 当前已用内存超过max memory限定时，触发主动清理策略

#### Redis 集群：

- 将数据自动切分（split）到多个节点的能力。
- 当集群中的一部分节点失效或者无法进行通讯时， 仍然可以继续处理命令请求的能力。

#### 数据分片

**哈希槽**

Redis 集群使用数据分片（sharding）而非一致性哈希（consistency hashing）来实现： 一个 Redis 集群包含 16384 个哈希槽（hash slot）， 数据库中的每个键都属于这 16384 个哈希槽的其中一个， 集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。(每个节点预先分配一定数量的哈希槽(hash slot)， 一开始是平均分片槽)

节点添加和删除： 重新分配hash槽，这个过程很快速，不用重新计算hash；并且添加和删除节点时服务一直在线（数据对应hash slot，和节点无关，hash slot在哪里数据就被存在哪里）

**一致性哈希**: 存储缓存到集群中哪一个服务器上

1. 对所有的服务器ip求hash值（mod 2^32），每个服务器都会在hash环上占有一个位置
2. 对需要缓存的key计算hash，这个hash一定落在环上，此时从该位置顺时针开始遇到的第一个服务器就是目标缓存服务器
   - 添加节点需要：重新定位部分数据
   - 删除节点：移动该节点数据到顺时针方向的下一个节点上
   - 问题：数据倾斜，通过虚拟节点均匀分布在分布式环上，从而均衡数据，避免出现缓存热点问题

![白话解析：一致性哈希算法 consistent hashing](http://www.zsythink.net/wp-content/uploads/2017/02/020717_1707_4.png)

Q: 哈希槽与一致性hash相比的优点？

A：一致性hash添加节点会导致新节点到逆时针方向第一台旧节点之间的数据位置发生变化；一致性hash还存在数据倾斜问题（虚拟节点来解决）

**缓冲雪崩、缓存击穿、缓存穿透、如何保证双写一致性的定义和解决方式。**

缓存穿透，是指查询一个一定不存在的数据：解决方法布隆过滤器，通过bitmap拦截这个不存在的key；set key unknow

缓存击穿/并发访问，某个热点key失效时，存在大量访问；解决方法：采用分布式锁，只有拿到锁的第一个线程去请求数据库，然后插入缓存，当然每次拿到锁的时候都要去查询一下缓存有没有；永不失效，就是采用定时任务对快要失效的缓存进行更新缓存和失效时间

缓存雪崩，缓存在某一时刻同时失效，解决方法：缓存失效时间分散开;

- 事前：redis 高可用，主从+哨兵，redis cluster，避免全盘崩溃。
- 事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 被打死。
- 事后：redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。

#### 主从复制

```
slaveof ip:port
```

1. 建立连接，slave客户端创建文件复制线程
2. slave客户端ping检测连接报文，交互认证信息
3. slave发起同步命令psync（全量复制和增量复制）

连接建立阶段、数据同步阶段、命令传播阶段

复制积压缓冲区，复制缓冲区

#### Redis Sentinel

> 哨兵：`Redis Sentinel` 是 `Redis` **高可用** 的实现方案。`Sentinel` 是一个管理多个 `Redis` 实例的工具，它可以实现对 `Redis` 的 **监控**、**通知**、**自动故障转移**、**配置提供者**
>
> ![img](https://user-gold-cdn.xitu.io/2018/8/22/16560ce611d8c4a5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> `Sentinel` 的主要功能包括 **主节点存活检测**、**主从运行情况检测**、**自动故障转移** （`failover`）、**主从切换**
>
> 1. 故障了会通知应用程序
>
> 默认情况下，**每个** `Sentinel` 节点会以 **每秒一次** 的频率对 `Redis` 节点和 **其它** 的 `Sentinel` 节点发送 `PING` 命令，并通过节点的 **回复** 来判断节点是否在线。
>
> - **主观下线**
>
> **主观下线** 适用于所有 **主节点** 和 **从节点**。如果在 `down-after-milliseconds` 毫秒内，`Sentinel` 没有收到 **目标节点** 的有效回复，则会判定 **该节点** 为 **主观下线**。
>
> - **客观下线**
>
> **客观下线** 只适用于 **主节点**。如果 **主节点** 出现故障，`Sentinel` 节点会通过 `sentinel is-master-down-by-addr` 命令，向其它 `Sentinel` 节点询问对该节点的 **状态判断**。如果超过 ``个数的节点判定 **主节点** 不可达，则该 `Sentinel` 节点会判断 **主节点** 为 **客观下线**。
>
> `Sentinel` 无法保证 **强一致性**。

#### 数据库

**MyISAM 与 InnoDB 区别：**

1、InnoDB 支持事务，MyISAM 不支持，这一点是非常之重要。事务是一种高

级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而 MyISAM

就不可以了；

2、MyISAM 适合查询以及插入为主的应用，InnoDB 适合频繁修改以及涉及到

安全性较高的应用；

3、InnoDB 支持外键，MyISAM 不支持；

4、对于自增长的字段，InnoDB 中必须包含只有该字段的索引，但是在 MyISAM

表中可以和其他字段一起建立联合索引；

5、清空整个表时，InnoDB 是一行一行的删除，效率非常慢。MyISAM 则会重

建表；

6、InnoDB 行锁，MyISAM 表锁

7、InnoDB通过主键创建聚簇索引

![1637b08b98619455?w=312&h=305&f=png&s=22430](https://segmentfault.com/img/bVbaVaU?w=312&h=305)

- **脏读（Dirty read）:** 当一个事务正在访问数据并且对数据进行了修改，而这种**修改还没有提交**到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
- **丢失修改（Lost to modify）:** 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。 例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。
- **不可重复读（Unrepeatableread）:** 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于**第二个事务的修改**导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
- **幻读（Phantom read）:** 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）**插入了一些数据**时。在随后的查询中，第一个事务（T1）就会发现**多了一些原本不存在**的记录，就好像发生了幻觉一样，所以称为幻读。

**QL 标准定义了四个隔离级别：**

- **READ-UNCOMMITTED(读取未提交)：** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **READ-COMMITTED(读取已提交):** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **REPEATABLE-READ（可重读）:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **SERIALIZABLE(可串行化):** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。

#### MTYSQL锁：

**共享锁（S锁）/排他锁（X锁）（行锁）**

事务拿到某一行记录的共享S锁，才可以读取这一行，并阻止别的事物对其添加X锁，事务拿到某一行记录的排它X锁，才可以修改或者删除这一行

**意向锁**（IS, IX 表集锁）

预示事务有意向对表中的某些行加共享S锁，预示着事务有意向对表中的某些行加排他X锁

**间隙锁**

封锁记录中的间隔，锁住**开区间**，防止间隔中被其他事务插入。间隙锁主要出现在RR隔离级别，避免出现幻读。

**临键锁（next-key lock）**

临键锁是记录锁和间隙锁的组合，既锁住了记录也锁住了范围。临键锁的主要目的，也是为了避免幻读；

**自增锁**

表级锁，主要用于事务中插入自增字段，也就是我们最常用的自增主键id。

> 对于普通 SELECT 语句，InnoDB 不会加任何锁，不过可以主动添加锁支持：
>
> 共享锁（S）：SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE。
>
> 排他锁（X)：SELECT * FROM table_name WHERE ... FOR UPDATE

#### MVCC

MVCC是为了实现事务的隔离性，通过版本号，避免同一数据在不同事务间的竞争，你可以把它当成基于多版本号的一种乐观锁。读不加锁，读写不冲突（MVCC只工作在REPEATABLE READ和READ COMMITED隔离级别下）

![img](https://img-blog.csdnimg.cn/20190708105553500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlzaHU=,size_16,color_FFFFFF,t_70)

在每开启一个事务时，会生成一个事务的版本号，被操作的数据会生成一条新的数据行（临时），但是在提交前对其他事务是不可见的，对于数据的更新（包括增删改）操作成功，会将这个版本号更新到数据的行中，事务提交成功，将新的版本号更新到此数据行中，这样保证了每个事务操作的数据，都是互不影响的，也不存在锁的问题。

因为在innodb中的操作可以分为当前读(current read)和快照读(snapshot read)：

- 快照读：读取的是快照版本，也就是历史版本（快照读是通过MVVC(多版本控制)和undo log）
- 当前读：读取的是最新版本（record lock(记录锁)，gap lock(间隙锁)）

**undo log、redo log、binlog**

为了满足事务的持久性，防止buffer pool数据丢失，innodb引入了redo log

1. redo log就是保存执行的SQL语句到一个指定的Log文件，当mysql执行数据恢复时，重新执行redo log记录的SQL操作即可

Undo log 是为了实现事务的原子性：（innodb将undo log的内容看作是数据）

1. 为了满足事务的原子性，在操作任何数据之前，首先将数据备份到一个地方（记录历史值）
2. 如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。

binlog 数据库主从复制，可以支持不同的引擎

#### 抽象类和接口的对比

抽象类是用来捕捉子类的通用特性的。接口是抽象方法的集合。

从设计层面来说，抽象类是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

#### 常量池

String和Integer[-128, 127]都可能会存储在常量池中，

```
String a = "123";

String b = "123";
System.out.println(a == b); // true

Integer n1 = 127;
Integer n2 = 127;

System.out.println(n1 == n2); //true
n1 = 128;
n2 = 128;

System.out.println(n1 == n2); // false

n1 = -128;
n2 = -128;
System.out.println(n1 == n2); // true

//intern方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中
String myInfo = new String("I love u").intern()；
```

#### 索引

B+树

最左值匹配，遇到范围查询就停止匹配：如索引(a,b)

```
// 能匹配
where a = 1 and b = 2
where b = 2 and a == 1 // 自动调整顺序，满足最优查询
// 不能匹配
where b = 2
// order 情况比较特殊
// 下面需要建立 a索引，order by是最后执行的，不会调整顺序
where a > 1 order by b
```

全局来看，最左边的索引才是有序的，当左边的值确定后，后面的索引才有序，才能利用索引

Innodb行结构

mysql数据页

Compact 结构

#### MySQL 性能优化

1. 开启查询缓存（query_cache_type=on)
2. 只有一行数据是使用limit 1
3. 建立索引，查询字段，排序字段和join字段
4. 避免select *
5. 预编译的SQL，一次解析；二进制传输，网络传输效率高
6. 读写分离

#### 主从复制（异步）

![img](https://upload-images.jianshu.io/upload_images/11414906-1e1d8aaa7a86af96.png?imageMogr2/auto-orient/strip|imageView2/2/w/799/format/webp)

1. 主服务器上面的任何修改都会通过自己的 I/O tread(I/O 线程)保存在二进制日志 `Binary log` 里面。

2. 从服务器上面也启动一个 I/O thread，通过配置好的用户名和密码, 连接到主服务器上面请求读取二进制日志，然后把读取到的二进制日志写到本地的一个`Realy log`（中继日志）里面。

3. 从服务器上面同时开启一个 SQL thread 定时检查 `Realy log`(这个文件也是二进制的)，如果发现有更新立即把更新的内容在本机的数据库上面执行一遍。



#### 分布式session

- 使用cookie来完成（很明显这种不安全的操作并不可靠）
- 使用Nginx中的ip绑定策略，同一个ip只能在指定的同一个机器访问（不支持负载均衡）
- 利用数据库同步session（效率不高）
- 使用tomcat内置的session同步（同步可能会产生延迟）
- 使用token代替session
- redis 分布式缓存

#### 自己动手实现

**LRU**

1. 新数据插入链表头部，如果数据存在就移动到头部，每次只需将尾部的结点移除

**定时器**

延迟队列，每个任务有个延时时间或者绝对执行时间（可以通过堆来实现，最小延时时间在第一个的特点），如果当前时间大于任务执行的时间（或者相对大于），则执行任务，否则等待第一个任务需要运行的时间（Object 的wait方法，可以等待一段时间），然后再继续调度下一个任务

**线程池**

任务队列和多个线程，添加任务到阻塞队列，线程不断从队列中取出任务来运行；一开始创建coreSize数量的线程

### C++指针和引用的区别

（1）引用被创建的同时必须被初始化（指针则可以在任何时候被初始化）

（2）不能有 NULL 引用，引用必须与合法的存储单元关联（指针则可以是 NULL）

（3）一旦引用被初始化，就不能改变引用的关系（指针则可以随时改变所指的对象）

### 消息队列

**rabbitmq**：消峰、异步通信、P/S解耦

**kafka**: