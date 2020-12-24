## Java 大数据与Web

### 常见的算法

**布隆滤波器**

判断一个元素(key)是否在另一个集合里面

1. 二进制比特数组
2. 随机散列函数（对key计算多个hash）
3. 二进制数组对应的多个hash位置置1
4. 判断key存在的方法就是所有的hash位都为1

**Hadoop**

**HDFS**

QA:

**块大小**

HDFS目前默认块大小在Hadoop2.x版本中是128M，老版本中是64M,

**为什么HDFS不支持存储小文件**

存储大量小文件会占用NameNode大量的内存和磁盘来存储文件目录和块信息

**说说hdfs的文件上传的流程**
　　　　1.首先客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。
　　　　2.NameNode返回是否可以上传。
　　　　3.如果文件大于128M则分块存储，客户端请求第一个 Block上传到哪几个DataNode服务器上。
　　　　4.NameNode根据副本储存策略返回3个DataNode节点，假如为dn1、dn2、dn3。
　　　　5.客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求调用dn2，dn2调用dn3，建立通信管道完成,dn1、dn2、dn3逐级应答客户端。(建立通信管道)
　　　　6.客户端以Packet为单位往dn1上传第一个Block数据,dn1收到Packet就会传给dn2，dn2传给dn3；dn1,dn2,dn3每接收packet会放入一个待写队列等待写入数据，落盘。
　　　　7.当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器，重复执行3-6步。

　　**说说文件的下载流程**
　　　　1.客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。
　　　　2.挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。
　　　　3.DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。
　　　　4.客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。

### MapReduce

你可以在 Job 对象上面调用 submit() 方法或者 waitForCompletion() 方法来运行一个 MapReduce 作业

![img](https://www.yiibai.com/uploads/allimg/201509/1-150913101012959.png)

![img](https://pic4.zhimg.com/80/v2-35602b384f731eeca77c1f4f5e2e46bd_1440w.jpg)

yarn资源管理基本步骤：

作业提交：向资源管理器请求一个application id， 计算分片，拷贝作业需要的资源，submitApplication 提交作业

作业初始化：yarn调度器为作业分配执行的容器，容器中application master 被启动（接受任务报告的进度和完成情况），application master 从HDFS中获取客户端计算的输入分片。然后它为每个分片创建一个 map 任务，同样创建由 mapreduce.job.reduces 属性控制的多个reduce 任务对象，创建输出目录和临时工作空间

任务分配：application master 为作业中的 map 任务和 reduce 任务向资源管理器请求容器，为 map 任务发送请求，reduce 任务的请求至少有 5% 的 map 任务已经完成才会发出

任务执行：application master 通过连接节点管理器来启动这个容器，任务通过一个主类为 YarnChild 的 Java 应用程序来执行

进度和状态的更新：作业和任务都含有一个状态，包括运行状态、maps 和 reduces 的处理进度，作业计数器的值，以及一个状态消息或描述（可能在用户代码中设置）

作业完成：当 application master 接受到最后一个任务完成的通知，它改变该作业的状态为 “successful”。

![img](https://images0.cnblogs.com/blog/676214/201409/281109297177557.jpg)

inputsplit->map(sort、combiner、partitioner, write)->shuffle->reduce

map在做输出时候会在内存里开启一个**环形内存缓冲区**，这个缓冲区专门用来输出的，默认大小是100mb，并且在配置文件里为这个缓冲区设定了一个阀值，默认是0.80（这个大小和阀值都是可以在配置文件里进行配置的），同时map还会为输出操作启动一个守护线程，如果缓冲区的内存达到了阀值的80%时候，这个守护线程就会把内容写到磁盘上，这个过程叫spill，另外的20%内存可以继续写入要写进磁盘的数据，每次spill操作也就是写入磁盘操作时候就会写一个溢出文件，map输出全部做完后，map会合并这些输出文件

一个Partitioner对应一个reduce作业，Partitioner因此就是reduce的输入分片，这个程序员可以编程控制，主要是根据实际key和value的值，根据实际业务类型或者为了更好的reduce负载均衡要求进行，

map的shuffle包括sort、combiner、partitioner和存储结果

reduce的shuffle包括partition copy、sort（merge）、reduce

最后reduce的结果存储在HDFS上



*NameNode*，管理元数据（文件块存储的datanode信息，文件位于的blockid），起文件系统的作用

![图 3. NameNode 的元数据存储目录结构](https://developer.ibm.com/developer/default/articles/os-cn-hadoop-name-node/images/img003.png)

1. 内存中有一份元数据，用于客户端的读取
2. NameNode会定期对元数据进行镜像，存储在文件fsimage_xxx里面，fsimage_${end_txid}
3. Editlog，文件写操作时，会记录操作记录到Editlog，Editlog会被分为多个段，正在写的段状态为inprogress，文件名如editinprogress\${startxid}  \${startxid}表示段的起始事务id，写入完成的segment文件名为edits\${startxid}-\${endxid}

通过镜像文件和Editlog可以恢复NameNode状态，先将fsiamge加载到内存，再将Editlog中，镜像文件结束事务id之后的写日志加载到内存中

*DataNode*,  文件块具体存储的节点，该节点启动时，会通知NameNode，自己存储的文件信息以及文件块列表

#### NameNode HA

**主备选举**

NameNode 单点故障怎么解决，提供HA（高可用）支持，Active节点负责具体的通知，Slave节点用于备份，系统刚启动时通过ActiveStandbyElector （利用Zookeeper）进行主备选举，通过HealthMonitor来检测NameNode状态（初始化、正常、不正常如硬盘不足，检测异常）,如果出现异常调用ZKFailoverController 来进行主备切换，切换主要通过ActiveStandbyElector 与ZK交互来完成选举，选举完成后通知节点的它是主节点还是备用节点

**ZooKeeper （CP）**

*Znode* ZooKeeper 的存储结构是一颗类似文件的路径树，路径由Znode名和 /组成，例如 /hbase/master，每个Znode都存储了数据信息，同时节点分为持久和临时，临时节点和客户端会话保持一致

ZK支持下面的特性：

1. 事务操作，每个事务都会被分配一个事务ID
2. Watcher机制，节点上注册Watcher，事件触发时，通知客户端（一次性，通知后就删除）
3. ACL权限控制
4. 写一致性

应用场景：

- 分布式锁，通过创建临时的Znode节点，如果创建成功就拥有锁，否则就注册Watcher到该Znode，节点使用完锁后主动删除Znode或者关闭客户端连接；这时其它节点就能获取到通知，重新竞争锁
- 配置中心，数据库配置信息，服务发现机器列表信息都可以存储在ZK上
- Master选举，多个机器同时创建同一个Znode节点，谁成功谁是Master，并且还要在该节点上注册Watcher，当master故障是，可以通知其它机器进行重新选举
- 心跳检测，每个客户端建立一个临时Znode，并且注册watcher事件，当没有心跳时（session失效就会触发Znode删除事件通知客户端）

异常处理：

- 脑裂  每次切换都生成一个任期ID，优先选择任期大的那个节点作为服务节点

基础命令：

```shell
./zkServer.sh start # 启动
./zkServer.sh status # 状态
get /zookeeper # 获取节点信息
create /merryyou merryyou # 创建节点
create -e  /merryyou/temp merryyou # 创建临时节点
stat /longfei watch # 设置watcher
get /longfei watch # 设置watcher
```

**共享内存 QJM**

除了主备选举之外，在主备之间还需要进行数据同步，不然会导致数据的不一致性，NameNode采用QJM算法实现（基本思想为Paxo），具体使用多个JournalNode来保存EditLog数据，大多数写成功即写成功

![图 5. 基于 QJM 的共享存储的数据同步机制 ](https://developer.ibm.com/developer/default/articles/os-cn-hadoop-name-node/images/img005.png)

Active NameNode 写EditLog时，同时要将写的内容通过RPC发送给JournalNode集群，如果集群中大多数都写成功，那么就返回客户端写成功，否则NameNode关闭退出进程

Standby NameNode ， 通过EditLogTailer 线程周期性的从JournalNode上同步Editlog，将其写入自己的内存镜像（同步的只有finalized的EditLog段）

还有两个问题：1. Active故障 journalNode不一致，切换时如何同步 2. Standby如何同步inprogress的EditLog段

通过Paxos算法来进行日志同步

问题1：A：设置epoch值，每次切换时先获取大多数JournalNode的 lastEpoch然后选择最大的那个epoch+1，再发送newEpoch RPC给JournalNode，当新的epoch大于当前JournalNode的lastEpoch时，返回接收新的epoch，并且设置lastEpoch为新Epoch，同时返回当前JournalNode上最新的EditLog的Segment事务起始编号，如果大多数JurnalNode返回成功时，设置新epoch成功（确保当前的切换有效：大多数节点的数据都一样新；同时获取每个节点最新的数据位置）B：

问题2：TODO

**YARN ResourceManger**

hadoop集群资源管理器系统：yarn基本思想；一个全局的资源管理器resourcemanager和与每个应用对用的ApplicationMaster，Resourcemanager和NodeManager组成全新的通用系统，以分布式的方式管理应用程序。

![img](https://img2018.cnblogs.com/blog/1271254/201910/1271254-20191008160008965-1100694847.png)



**HBase**

Hbase数据库

 ① 它介于 nosql 和 RDBMS 之间，仅能通过主键(row key)和主键的 range 来检索数据，仅支持单行事务(可通过 hive 支持来实现多表 join 等复杂操作)。 

 ② Hbase 查询数据功能很简单， 不支持 join 等复杂操作

 ③ 不支持复杂的事务（行级的事务）

 ④ Hbase 中支持的数据类型： byte[]

 ⑤ 主要用来存储结构化和半结构化的松散数据。

   结构化:数据结构字段含义确定,清晰,典型的如数据库中的表结构.
   半结构化:具有一定结构,但语义不够确定,典型的如 HTML 网页,有些字段是确定的(title), 有些不确定(table)
   非结构化:杂乱无章的数据,很难按照一个概念去进行抽取,无规律性

![img](https://img2020.cnblogs.com/blog/1275415/202007/1275415-20200730210247949-1485773434.png)

1）StoreFile

　　保存实际数据的物理文件，StoreFile 以 HFile 的形式存储在 HDFS 上。每个 Store 会有一个或多个 StoreFile（HFile），数据在每个 StoreFile 中都是有序的。

2）MemStore

　　写缓存，由于 HFile 中的数据要求是有序的，所以数据是先存储在 MemStore 中，排好序后，等到达刷写时机才会刷写到 HFile，每次刷写都会形成一个新的 HFile。

3）WAL

　　由于数据要经 MemStore 排序后才能刷写到 HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做 Write-Ahead logfile 的文件中，然后再写入 MemStore 中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。

其它： Region Split（负载均衡）、StoreFile Compaction（避免文件碎片）

通过ZK查数据表在哪个regionServer，然后再查rowkey在regionserver下的哪个region，写入先写到Mem Store和WAL，然后Mem Store定期刷到HDFS上（每满足128M）写一次HDFS；读取是就从缓存、MemStore还有HFile里查询数据

*此外还有Block cache和meta cache用于中间的读取数据的缓存*

基础命令：

```shell
create 'emp'
put'emp','rw1','col_f1:name','tanggao'
get 'emp','rw1'
scan 'emp'
deleteall 'emp','rw1'
count 'emp'
truncate'emp'
```

**Kafka**

用于日志处理的分布式消息队列

- 发布和订阅数据流，类似于传统消息队列（RabbitMQ，RocketMQ）的功能
- 以容错的方式存储数据流的功能
- 实时处理数据流的功能
- Consumer、Producer、Brocker

#### Dubbo

[^ 1] https://developer.ibm.com/zh/articles/os-cn-hadoop-name-node/

### Spark

#### spark 集群搭建