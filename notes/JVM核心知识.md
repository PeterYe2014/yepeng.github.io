# JVM 核心知识

## JAVA内存模型

JAVA内存模型主要由：方法区、程序计数器、JVM栈、本地方法栈、直接内存以及堆来组成的。其中方法区、堆是共享的内存区域，程序计数器和JVM栈是线程私有的。

1. **方法区**：共享的内存区域，存储类的基本信息，常量和字面量，即时编译后的代码。方法区含有常量池，一般不进行垃圾回收。HotSpot把方法区看成永久代，java8取消了永久代，使用元空间代替（Metaspace）。
2. **程序计数器**：在每个线程里面都有一个程序计数器，表面下一条执行的指令。能够线程切换的时候知道某个线程该执行什么指令。
3. **JVM栈**: 每个线程私有的，存储线程方法里面局部变量表。线程中方法开始会新创建一个栈帧，添加到虚拟机栈，如果栈的深度过大会造成```StackOverflowException```
4. **本地方法栈**:存储Native方法里面的数据
5. **直接内存**：Java提供了新的IO方式，通过Channel和Buffer来操作IO数据。这里就使用了一个```DirectByteBuffer```引用，直接分配了堆外内存，该内存不受JVM虚拟机内存限制。
6. **堆**: 存储Java对象实例的内存区域，GC的主要区域。几乎所有的JAVA对象都分配在这个区域里面。Java堆可以分为：新生代和老年代。更细一些为：Eden空间，From survivor空间，To survivor空间

 最常见的VM参数：

```
# 输出gc日志
-verbose:gc
# 详细日志信息,Eden、from、to
-XX:+PrintGCDetails
# 堆最小值为20M
-Xms20m
# 堆最大值为20M
-Xmx20m
# 堆内新生代的大小
-Xmn10m
# 错误时输出内存快照
-HeapDumpOnOutOfMemoryError
# 设置栈容量
-Xss128k
# 方法区最小和最大
-XX:PermSize=10M
-XX:MaxPermSize=10M
# 直接内存容量
-XX:MaxDirectMemorySize=10M
```

## 对象创建过程

Java中创建对象就是简单使用new关键字，在简单的一句话后面有许多复杂的JVM内存分配过程

```
Person person = new Person("张三");
```

1. **查找符号引用，判断类是否加载**: JVM遇到new指令就会根据指令参数区常量池查找是否有对应类的符号引用，同时判断该类是否加载、解析和初始化，如果没有加载，那么就要加载相关类信息到内存
2. **分配堆内存**，当确定类加载后，就能获取对象需要的内存空间大小，从而分配内存给相应的对象，但是内存空间可能是连续存储的，空闲和非空闲有一个边界，也可能是零散分布的。那么对应整齐规则的空间可以采用*指针碰撞*的方法，直接顺序分配空间给对象。对应离散的空间，就需要一个内存空闲空间分布的表，用于描述哪些空间是空闲的，分配对象内存时，找一个适当大小的内存空间来进行分配。此外内存也是一个竞争资源，可能不同线程对象同时请求分配内存，这里可以通过CAS来进行同步；令一种方式是每个线程单独分配一块区域，避免内存分配冲突，这种方法叫*TLAB*(Thread Local Allocation Buffer)，可以通过```-XX:+/-UseTLAB```来设定。
3. **初始化零值**
4. **设置对象头**（Object header）
5. **初始化对象**，按照程序员的方法初始化
6. 最后 完成对象创建后，```person```会存在JVM栈里面，```Person```对象会存储在堆上。

## 对象的存储

绝大多数的对象都存储在堆内存中，那么对象是怎么存储在内存中的呢？JVM中对象主要由**对象头**，**实例数据**，**对齐字节组成**。

对象头主要存储两种数据。一部分存储对象自身的运行数据，如hashcode、GC分代年龄、锁状态标记、线程持有的锁、偏向锁、偏向时间戳等。这部分被称为**Mark Word**，数据长度为32位或者64位。并且该部分采用变长存储的方式，如果某部分数据位没有使用，就可以用来存储其它部分数据。第二部分是类型指针，该指针指向对象的**类元信息**，用于表示对象的类型，但是该部分不是必须的，只有当需要表明对象实例时什么类型时才需要存储。此外如果对象是个数组，那么对象头中还要存储数组的长度。

实例数据存储的是对象内部定义的各种成变量的存储，并且JVM总是按照相同类型来进行成员变量的内存分配的。

填充也不是必须的，只是在Hotpot虚拟机中要求对象的起始地址必须是8字节的整数倍。

> 对象存储后，该如何定位呢？
>
> 主要由两种方法，一种是句柄法，引用存储句柄的地址，句柄里面存储指向对象的地址以及对象的类型的指针，第二种方法直接指针，引用就是指向对象类型的指针，这种方法就要要求对象实例在对象头中表明字节的类型。

## 垃圾回收（GC）

Java中主要回收的堆内存，也就是对对象的回收。

### 哪些内存该回收

#### 引用计数法

该种方法给每个对象设置一个引用计数，每当有一个对象引用到该对象实例，该引用计数就加一，当一个引用消除后计数就减一，当引用计数为0时，就标记该对象可回收。不过该方法会出现循环引用的情况，这样两个相互引用的对象就都不会被清除。

#### 可达性分析

该方法将对象之间的引用关系看成一个图，从一系列的**GC Roots**开始，如果某些对象不可达，那么这些对象都被标记为可清除。GC Roots对象包括（正在运行的）：

1. 虚拟机栈中引用的对象
2. 方法区静态属性引用的对象
3. 方法区中常量引用的对象
4. 本地方法栈（JNI）中引用的对象
5. 由系统加载器(bootstrap/system class loader)加载的对象
6. 调用wait、notify或者被synchronized的对象

> 被标记为不可达的对象，不一定会被回收，因为当被标记为不可达的时候，属于第一次标记，如果覆盖了finalize()方法，且该方法没有被执行，那么这时会执行finalize()方法。对象会被放入F-Queue队列，由Finalizer线程去执行，并且GC会对F-Queue队列中的对象再进行小规模标记，也就意味如果对象在finalize()方法中使自己可达了，那么就逃脱了被GC清理了。

### 垃圾回收算法

#### 标记清除（Mark-Sweep）

最基础的算法，标记后然后清除掉标记的对象。优点：简单快捷，确定：会产生大量的内存碎片，如果由新的对象需要内存分配，可能找不到足够的连续空间。

#### 标记整理

标记整理算法基础和标记清除一样，不过最后一步不相同，该方法让所有存活的对象超一端移动，并且清理掉边界以外的内存空间，该方法解决了内存碎片产生的问题，同时也不会浪费存储空间。

#### 复制算法

划分空间为两部分，每次只使用其中的一块。如果该块使用完了，那么就将内存中存活的对象移动到另一块，然后清除原来的空间，这种方法避免了内存碎片的产生，不过可能降低内存的使用率。但是，如果对象很快就被标记为可清除，那么划分区域的比例可以适当的扩大，这样可以提高内存的使用率。HotSpot中就将内存区域分为新生代Eden和两个Survivor，每次使用其中两个，回收时就将两个内存区域中存活的对象移动到另一块Survivor区域中，默认Eden和Survivor空间的比为8:2，如果复制过程中Suivivor的内存不够，那么就需要使用内存担保机制。

#### 分代收集

将内存空间分为新生代和老年代，新生代对象存活率低可以采用复制算法，老年代存活率高，采用标记清除或者标记整理算法。

### 回收算法的实现

回收算法可以分为下面的关键步骤：

1. GC Roots 可达性分析
2. 标记不可达节点
3. 采用不同的算法来进行空间回收

由于回收算法会在一个或者多个线程中进行，那么就需要保证对象不可达状态的一致性，如果除了GC线程，还有其它线程在运行，那么引用链也就会发生变化，这样会导致不一致性，为了保证一致性，我们就需要停止其它线程的运行。

同时，由于可达性分析，要找到内存中所有的引用，这个也需要耗费大量资源。不过JVM提供了一种数据结构OopMap来存储内存中哪些地址存储这对象引用，从而避免全局扫描了。

这里又存在一个问题，每个线程该在什么阶段停止运行呢？如果停止的频率过大，那么就会生成许多临时的OopMap来存储引用关系，浪费空间；但是停止的频率过小，GC等待时间就会很长，用户响应时间就会增加，用户体验不佳。故此选择**具有让程序长时间执行的指令**来进行停止，这些指令往往是循环，条件或者异常处理。JVM称这些位置为安全点。

不过不同的线程又不同的安全点。那么怎么协调不同的线程到各自最近的安全点的方法有两种。第一种：抢占式中断，所有线程中断，如果某个线程中断位置不是安全点，那么就运行到最近的安全点。第二种：主动式中断，在安全点，轮询标志位，如果为真就中断线程。

### 主流的垃圾回收器

#### Serial/Serial Old

最简单的单线程回收器，直接暂停其它线程，收集完成后才继续运行其它线程。新生代使用复制算法，老年代使用标记整理算法

#### ParNew

Serial的多线程版本，JVM Server模式下首选新生代收集器。虽然ParNew是多线程，但是在单CPU环境中，由于上下文的切换，不一定有Serial的性能好

```shell
# 启用ParNew
# 默认启用Parnew
-XX:+UseConcMarkSweepGC
# 显示启用
-XX:+UseParNewGC
```

常见的配置参数有：

```shell
# 限制GC线程数量
-XX:ParallelGCThreads
-XX:SurvivorRatio
-XX:PretenureSizeThreshold
-XX:HandlerPromotionFailure
```

#### Parallel Scavenge/Parallel Old

Parallel Scavenge 收集器与其它收集器最大的不同之处在于该收集器以优化CPU吞吐量（代码运行时间/(代码运行时间+GC时间)）为目标，而其它的收集器是优化线程暂停时间为目标的。

配置参数有：

```
# 设置最大的暂停时间
-XX:MaxGCPauseMills
# 设置吞吐量大小(0,100), 默认为99, 1/(1+99) 允许最大的GC占比为1%
-XX:GCTimeRatio
```

#### CMS

优化停顿时间的收集器，适合用于重视服务响应速度的应用上。主要步骤有：

1. 初始标记，标记GC Roots直接关联的对象 ,STW
2. 并发标记，GC Roots 枚举（最长耗时）
3. 重新标记，标记并发标记过程中新出现的可清除对象, STW
4. 并发清除

上述步骤可以看出，CMS中的GC线程和用户线程其实是并发运行的，故此可以取得很好的暂停优化效果。但是CMS只能允许在老年代上进行收集，需要配合ParNew或者Serial收集器来进行收集。此外CMS还不能收集浮动垃圾，由于CMS的并发性，垃圾会在清理阶段产生，一次GC无法清除的对象，必须在下一次进行清除。这样会导致垃圾累积，如果累积超过内存限制就会报错，所一CMS算法还需要预留一部分内存空间，避免出现"Concurrent Mode Failure"，如果出现了该错误可以使用后备清理器Serial Old算法。同时CMS由于使用了标记清除算法，所以会产生大量的内存碎片，故此需要进行内存整理。



缺点：

1. CMS 无法清理浮动垃圾，清除阶段线程还运行，会有新的垃圾产生，这些垃圾无法在本次回收中被清理

2. 产生内存碎片问题（标记清除算法），可以设置条件启动碎片合并
3. CMS 预留的内存无法满足(Concurrent Mode Failure)，会启动预案使用Serial Old来收集，很慢；增大预留的内存，降低-XX:CMSInitiatingOccupancyFraction，该参数设置CMS老年代使用了多少百分比就触发回收机制

常见的参数：

```shell
# CMS收集触发百分比，1.5默认为68%
-XX:CMSInitiatingOccupancyFraction
# 内存整理开关，默认开启
-XX:UseCMSCompactAtFullCollection
# 执行多少次不压缩的Full GC后再执行压缩的
-XX:CMSFullGCsBeforeCompaction
```

#### G1 (Garbage-First)

面向服务端的收集器。主要特点有：

1. 并行和并发

2. 分代收集

3. 空间整合，整体为标记-整理算法，局部是基于复制算法

4. 可预测的停顿，能够指定在一个长度为M毫秒的时间片内，GC时间不超过N毫秒。G1将内存分为多个大小相等的Region，新生代和老年代都是由若干个Region构成的。**G1跟踪Region的垃圾堆积价值**，并且维护一个优先队列，根据允许的收集时间，优先回收价值最大的Region。

   > 价值最大指的是：回收所获得的空间大小及回收所需时间的经验值

#### 垃圾回收触发条件

**Minor GC**：当Eden区满时，触发Minor GC

**Full GC**:

- System.gc()方法的调用（可能会调用，建议回收）
- 老年代空间不足
- 方法区空间不足
- 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
- 过大的对象移动到老年区
- 老年区内无法找到连续的存储空间（由于内存碎片问题）



#### 配置GC收集器的VM参数

```
# 参数前面有+/-的表示布尔参数，+表示启用，-表示不启用
-XX:-UseSerialGC 
-XX:-UseParNewGC
-XX:-UseConcMarkSweepGC # Parnew + CMS + Serial Old
-XX:-UseParallelGC # Parallel Scavenge + serial old
-XX:-UseParallelOldGC # Parallel Scavenge + Parallel Old
-SurvivorRatio # Eden与Survivor比例，默认为8
-PretenureSizeTreshold # 直接晋升到老年代的对象大小
-MaxTenuringTreshold # 晋升到老年代的对象年龄
-UseAdaptiveSizePolicy # 动态调整堆区域大小
-HandlePromotionFailure # 是否yun'x
-ParallelGCThreads
```

#### GC日志

(JDK1.8)设置```JVM```参数为```-verbose:gc -XX:+PrintGCDetails```,然后运行程序，就可以得到详细的GC日志：

```
class Student{
    private String name;
    private String age;

}
public class LearnGC {
    public static void main(String[] args){

        Student s = new Student();
        s = null;
        System.gc();
    }
}
```

GC日志中FULL GC表示这次垃圾收集是存在Stop-The-World停顿的。下面的```PSYoungGen```、```ParOldGen```表示的是收集的空间，这个名称不同的收集器也不同，不同的收集器内存区域的名称关系如下：

| 收集器            | 新生代名称              |
| ----------------- | ----------------------- |
| Serial            | Default New Generation  |
| ParNew            | Parallel New Generation |
| Parallel Scavenge | PSYongGen               |

后面的```[Times:....]```有三个时间，分别表示：用户消耗的CPU时间，内核态消耗的用户时间和操作从开始到结束经过的墙钟时间；CPU时间和墙钟时间的区别在于，CPU时间**不包含非运算等待时间**，如I/O、多线程等待；同时对于多线程程序，CPU时间会叠加，所以CPU时间也可能大于墙钟时间。

```
[GC (System.gc()) [PSYoungGen: 3948K->696K(56832K)] 3948K->696K(186880K), 0.0154845 secs] [Times: user=0.00 sys=0.00, real=0.02 secs] 
```

```
[Full GC (System.gc()) [PSYoungGen: 696K->0K(56832K)] [ParOldGen: 0K->633K(130048K)] 696K->633K(186880K), [Metaspace: 3225K->3225K(1056768K)], 0.0277477 secs] [Times: user=0.00 sys=0.00, real=0.03 secs]
```

这里主要表示内存空间不同分区的使用情况：

```
Heap
 PSYoungGen      total 56832K, used 491K [0x0000000780f00000, 0x0000000784e00000, 0x00000007c0000000)
  eden space 49152K, 1% used [0x0000000780f00000,0x0000000780f7af88,0x0000000783f00000)
  from space 7680K, 0% used [0x0000000783f00000,0x0000000783f00000,0x0000000784680000)
  to   space 7680K, 0% used [0x0000000784680000,0x0000000784680000,0x0000000784e00000)
 ParOldGen       total 130048K, used 633K [0x0000000702c00000, 0x000000070ab00000, 0x0000000780f00000)
  object space 130048K, 0% used [0x0000000702c00000,0x0000000702c9e528,0x000000070ab00000)
 Metaspace       used 3232K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

#### 内存分配和回收策略

1. 对象优先在Eden空间上分配
2. 大对象直接进入老年代，通过设置参数```-XX:PretenureSizeThreshold```来设置判断大对象的阈值
3. 长期存活的对象将进入老年代，每个对象有个age，对象在Survivor中每熬过依次Minior GC，年龄就会增加1，当年龄大于设置的阈值，就会被复制到老年区；设置阈值使用参数:```-XX:MaxTenuringThreshold```
4. 动态对象年龄判断，如果在Survivor空间中相同年龄所有对象大小总和大于Survivor空间的一半，那么年龄大于或等于该年龄的对象就可以直接进入老年代。
5. 空间分配担保。进行Minor GC前需要计算老年区剩余的空间是否大于新生代的所有对象的空间，如果大于那么就行安全的Minor GC，如果小于那么可以选择直接先Full GC回收老年区的空间；或者尝试冒险进行一次可能失败的Minor GC，如果失败了再采用Full GC；由于Full GC很费时，一般我们选择冒险一次，并且冒险时增加一个冒险成功率的条件：老年区最大可用连续空间大于历次晋升老年代对象的平均大小；可以通过配置参数```HandlePromotionFailure```为true来配置冒险的策略。

> minor gc: 对年轻区进行回收，old gc： 回收老年区； full gc：对整个堆进行回收

### 常用的内存分析工具

jmap 导出内存dump，主要包含堆上的类、实例以及GC ROOTS对象有哪些、堆的空间状态、即将要回收的空间，类加载器的信息，导出的dump文件可以使用j

```
jmap -dump:format,file=a.dump <pid>
```

```powershell
C:\Users\35114>jmap -heap 10744
Attaching to process ID 10744, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.221-b11

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 734003200 (700.0MB)
   NewSize                  = 66060288 (63.0MB)
   MaxNewSize               = 244318208 (233.0MB)
   OldSize                  = 133169152 (127.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 50331648 (48.0MB)
   used     = 46018880 (43.88702392578125MB)
   free     = 4312768 (4.11297607421875MB)
   91.43129984537761% used
From Space:
   capacity = 7864320 (7.5MB)
   used     = 5342272 (5.09478759765625MB)
   free     = 2522048 (2.40521240234375MB)
   67.93050130208333% used
To Space:
   capacity = 7864320 (7.5MB)
   used     = 0 (0.0MB)
   free     = 7864320 (7.5MB)
   0.0% used
PS Old Generation
   capacity = 133169152 (127.0MB)
   used     = 0 (0.0MB)
   free     = 133169152 (127.0MB)
   0.0% used

5163 interned Strings occupying 441744 bytes.
```

jstack 输出当前的线程信息，每个线程包含的信息有线程的状态、线程当前运行的栈状态或者线程阻塞的地方

```powershell
C:\Users\35114>jstack 10744
2020-09-05 09:52:54
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.221-b11 mixed mode):

"DestroyJavaVM" #12 prio=5 os_prio=0 tid=0x00000000026b7800 nid=0x2940 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JPS event loop" #10 prio=5 os_prio=0 tid=0x00000000159a4800 nid=0x101c runnable [0x0000000016e9e000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll0(Native Method)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll(WindowsSelectorImpl.java:296)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.access$400(WindowsSelectorImpl.java:278)
        at sun.nio.ch.WindowsSelectorImpl.doSelect(WindowsSelectorImpl.java:159)
        at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
        - locked <0x00000000f4849f18> (a io.netty.channel.nio.SelectedSelectionKeySet)
        - locked <0x00000000f484cda8> (a java.util.Collections$UnmodifiableSet)
        - locked <0x00000000f4849e48> (a sun.nio.ch.WindowsSelectorImpl)
        at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
        at io.netty.channel.nio.SelectedSelectionKeySetSelector.select(SelectedSelectionKeySetSelector.java:62)
        at io.netty.channel.nio.NioEventLoop.select(NioEventLoop.java:765)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:413)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:909)
        at java.lang.Thread.run(Thread.java:748)

"Service Thread" #9 daemon prio=9 os_prio=0 tid=0x00000000142fc000 nid=0x27dc runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread2" #8 daemon prio=9 os_prio=2 tid=0x00000000142da800 nid=0x3fe8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #7 daemon prio=9 os_prio=2 tid=0x00000000142d6800 nid=0x15d4 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=2 tid=0x00000000142ca800 nid=0x3c60 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x00000000156b2800 nid=0x1448 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x00000000142b8800 nid=0x3e04 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x00000000027ad800 nid=0x16a8 in Object.wait() [0x000000001560f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000f4876048> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
        - locked <0x00000000f4876048> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x000000001428a000 nid=0x2168 in Object.wait() [0x000000001550f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000f4876278> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000f4876278> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=2 tid=0x0000000014267000 nid=0x22b0 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00000000026ce000 nid=0x3fc8 runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00000000026d0000 nid=0x2e70 runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x00000000026d1800 nid=0x2aac runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x00000000026d3000 nid=0xa70 runnable

"VM Periodic Task Thread" os_prio=2 tid=0x00000000142fe000 nid=0x362c waiting on condition

JNI global references: 305
```

除了命令行工具，还有jvisualvm 可视化工具，可以进行内存和线程的dump，也可以实时监控内存的使用情况，查看内存的对象实例分布等内存分析功能，具体运行情况如下：

**监视程序的内存使用、类和线程的数量**

![image-20200905095630847](C:\Users\35114\AppData\Roaming\Typora\typora-user-images\image-20200905095630847.png)

**线程监视视图**

![image-20200905095754103](C:\Users\35114\AppData\Roaming\Typora\typora-user-images\image-20200905095754103.png)

**可以指定进程导出线程或者堆的dump文件**

![](C:\Users\35114\Desktop\微信截图_20200905102225.png)

jinfo、jstat、jps、jmap、jhat、jstack、JConsole、Mat

```
// 查看gc次数和平均时长
jstat -gcutil 6689 1000

```



### 虚拟机类加载机制

##### 类加载过程

类的生命周期：加载，验证，准备，解析，初始化，使用，卸载，7个阶段。

虚拟机规范规定了有且只有5种初始化类的时刻：

1. 使用new、getstatic、putstatic、invokestatic4条字节码指令时，会触发类的第一次初始化
2. 使用反射时，会触发类的第一次初始化
3. 初始化一个类，如果他的父类没有初始化，那么先初始化其父类
4. 主类含有main函数的类要被初始化
5. 使用动态语言支持，如果一个```java.lang.invoke.MethodHandle``实例的最后解析结果是，REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄时

加载：

1. 通过类的全限定名来获取类的二进制流数据
2. 字节流的静态存储结构转为方法区的运行时数据结构（？静态存储结构和运行时存储结构区别）
3. 内存种生成一个代表这个类的Class对象

验证：包含字节流的格式验证，元数据验证，字节码验证和符号引用验证

准备：为类变量（静态变量）分配内存，并且设置零值的过程；例外：ConstantValue属性

解析：将常量池里面符号引用替换为直接引用的过程

初始化：

1. 执行类构造函器\<clinit\>()方法，该方法由编译器自动收集类中所有类变量的赋值动作和静态语句块种的语句合并产生的。
2. 父类的\<clinit\>()方法会先被调用
3. \<clinit\>()方法也会被同步，多个线程执行一个方法时，只有一个线程能够成功执行，其它线程都要被阻塞。

##### 类加载器

类是由类和类加载器来定义其唯一性的。

类加载器可以分为三种：

1）启动类加载器（```Bootstrap ClassLoader```），主要加载$JAVA_HOME/lib/下的类，该加载器程序员无法自己使用

2）扩展类加载器（Extension ClassLoader），主要加载$JAVA_HOME/lib/ext中的类文件

3）应用程序类加载器（Application ClassLoader）,加载用户类路径中的类库。

类加载器的**双亲委派模型**,每个类加载器都有一个父加载器，通过属性```parent```来表示，当一个类加载器需要加载一个类时要先交给父类加载器加载，如果父类不存在时就交给启动加载器加载，或者父类加载器无法加载类时就用当前的类加载器的```findClass```方法l来进行加载。这种方式能够很好的让一些基础类被统一的类加载器加载，避免类加载的混乱。

> 双亲委派模型也有被打破的时候，当父加载器需要主动让子加载器去加载一些类时就会出现问题，这个可以通过设置线程里面的上下文类加载器方法```setContextClassLoader()```来实现。许多JNDI、JDBC等采用的都是这种方法。

Java9 破坏了双亲委派，通过模块加载的方法，减少加载不必要的类库，提高加载的速度，如果找不到模块加载器才使用parent去加载

#### 调优实战

![img](https://img2018.cnblogs.com/blog/885859/201905/885859-20190501151027147-2068463607.png)

Q：频繁大对象生成，造成FULL GC过多

A1：增大堆的内存；优化代码，避免分配过大的临时对象

A2:  32位JDK集群部署（32位性能要优于64位）

Q: 堆/Perm 区不断增长, 没有下降趋势(回收速度赶不上增长速度), 最后不断触发FullGC, 甚至crash.

​	每次FullGC后, 堆/Perm 区在慢慢的增长, 最后不断触发FullGC, 甚至crash(如下图: 示意图)

A: 排查内存溢出，是否存在不使用的对象没有被回收； dump heap, 查看类和实例的数量，是否存在异常的数量情况，通过MAT定位代码，优化代码

Q: Full GC 或者 Young GC 是否时间过长

A：检查线程的阻塞情况，哪些情况阻塞比较多，采用无锁编程，控制锁的粒度

