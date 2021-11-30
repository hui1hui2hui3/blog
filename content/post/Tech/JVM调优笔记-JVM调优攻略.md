---
title: "JVM调优笔记-JVM调优攻略"
date: 2018-01-19
tags: ["JVM"]
draft: false
---

在Java虚拟机的参数中，有3种表示方法（出自：http://www.cnblogs.com/wenfeng762/archive/2011/08/14/2137810.html），用`ps -ef |grep “java”`命令，可以得到当前Java进程的所有启动参数和配置参数：

* 标准参数（-），所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
* 非标准参数（-X），默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
* 非Stable参数（-XX），此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用（但是，这些参数往往是非常有用的）；

（额外的，-DpropertyName=“value”的形式定义了一些全局属性值，下面有介绍。）
本文只重点介绍一些重要和常用的参数，如果想了解全部参数，可以参考下面的文章：
[《Java HotSpot VM Options》](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)
[《JVM启动参数大全》](http://www.blogjava.net/midstr/archive/2008/09/21/230265.html)

## 标准参数
其实标准参数是用过Java的人都最熟悉的，就是你在运行java命令时后面加上的参数，如`java -version`, `java -jar` 等，输入命令`java -help`或`java -?`就能获得当前机器所有java的标准参数列表。
### -client
设置jvm使用client模式，这是一般在pc机器上使用的模式，启动很快，但性能和内存管理效率并不高；多用于桌面应用；
### -server
启动模式|	新生代GC方式|	旧生代和持久代GC的方式
------|------|------
client|	串行|	串行
server|	并行|	并发

### -classpath / -cp
JVM加载和搜索文件的目录路径，多个路径用`;`分隔。注意，如果使用了-classpath，JVM就不会再搜索环境变量中定义的CLASSPATH路径。
JVM搜索路径的顺序为：

 1. 先搜索JVM自带的jar或zip包（Bootstrat，搜索路径可以用System.getProperty(“sun.boot.class.path”)获得）；
 2. 搜索JRE_HOME/lib/ext下的jar包（Extension，搜索路径可以用System.getProperty(“java.ext.dirs”)获得）；
 3. 搜索用户自定义目录，顺序为：当前目录（.），CLASSPATH，-cp；（搜索路径用System.getProperty(“java.class.path”)获得）

### -DpropertyName=value
定义系统的全局属性值，如配置文件地址等，如果value有空格，可以用`-Dname=”space string”`这样的形式来定义，用`System.getProperty(“propertyName”)`可以获得这些定义的属性值，在代码中也可以用`System.setProperty(“propertyName”,”value”)`的形式来定义属性。

### -verbose
#### -verbose:class
输出jvm载入类的相关信息，当jvm报告说找不到类或者类冲突时可此进行诊断。
#### -verbose:gc
输出每次GC的相关情况，后面会有更详细的介绍。
#### -verbose:jni
输出native方法调用的相关情况，一般用于诊断jni调用错误信息。

## 非标准参数
非标准参数，是在标准参数的基础上进行扩展的参数，输入“java -X”命令，能够获得当前JVM支持的所有非标准参数列表（你会发现，其实并不多哦）。
![image_1b4fh891111hb1paab93dgv1hts9.png-229.2kB][1]
### -Xmn
**新生代内存大小的最大值，包括E区和两个S区的总和,大小建议为整个堆栈的3/8**，使用方法如：`-Xmn65535，-Xmn1024k，-Xmn512m，-Xmn1g` (`-Xms,-Xmx`也是种写法)
`-Xmn`只能使用在JDK1.4或之后的版本中，（之前的1.3/1.4版本中，可使用`-XX:NewSize`设置年轻代大小，用`-XX:MaxNewSize`设置年轻代最大值）；
如果同时设置了`-Xmn`和`-XX:NewSize，-XX:MaxNewSize`，则谁设置在后面，谁就生效；
如果同时设置了`-XX:NewSize -XX:MaxNewSize`与`-XX:NewRatio`则实际生效的值是:
`min(MaxNewSize,max(NewSize, heap/(NewRatio+1)))`（参考：http://www.open-open.com/home/space.php?uid=71669&do=blog&id=8891）
**建议：
在开发、测试环境，可以`-XX:NewSize 和 -XX:MaxNewSize`来设置新生代大小。
在线上生产环境，使用`-Xmn`一个即可（推荐），或者将`-XX:NewSize 和 -XX:MaxNewSize`设置为同一个值，这样能够防止在每次GC之后都要调整堆的大小（即：抖动，抖动会严重影响性能）**

### -Xms
**初始堆的大小，也是堆大小的最小值**，默认值是`总共的物理内存/64（且小于1G）`，默认情况下，当堆中可用内存小于40%(这个值可以用`-XX: MinHeapFreeRatio` 调整，如`-XX:MinHeapFreeRatio=30`时，堆内存会开始增加，一直增加到`-Xmx`的大小；

### -Xmx
**堆的最大值**，默认值是`总共的物理内存/64（且小于1G）`，如果Xms和Xmx都不设置，则两者大小会相同，默认情况下，当堆中可用内存大于70%（这个值可以用`-XX: MaxHeapFreeRatio` 调整，如`-XX:MaxHeapFreeRatio=60`）时，堆内存会开始减少，一直减小到-Xms的大小；
`整个堆的大小=年轻代大小+年老代大小`，堆的大小不包含持久代大小，如果增大了年轻代，年老代相应就会减小，官方默认的配置为`年老代大小/年轻代大小=2/1左右`（使用`-XX:NewRatio`可以设置`-XX:NewRatio=5`，表示`年老代/年轻代=5/1`）；
**建议:
在开发测试环境可以用`Xms和Xmx`分别设置最小值最大值
在线上生产环境，`Xms和Xmx`设置的值必须一样，原因与年轻代一样——防止抖动；**

### -Xss
这个参数用于设置**每个线程的栈内存**，默认1M，一般来说是不需要改的。除非代码不多，可以设置的小点，另外一个相似的参数是`-XX:ThreadStackSize`，这两个参数在1.6以前，都是谁设置在后面，谁就生效；1.6版本以后，-Xss设置在后面，则以-Xss为准，-XXThreadStackSize设置在后面，则主线程以-Xss为准，其它线程以-XX:ThreadStackSize为准。

### -Xrs
减少JVM对操作系统信号（OS Signals）的使用（JDK1.3.1之后才有效），当此参数被设置之后，jvm将不接收控制台的控制handler，以防止与在后台以服务形式运行的JVM冲突（这个用的比较少，参考：http://www.blogjava.net/midstr/archive/2008/09/21/230265.html）。

### -Xprof
跟踪正运行的程序，并将跟踪数据在标准输出输出；适合于开发环境调试。

### -Xnoclassgc
关闭针对class的gc功能；因为其阻止内存回收，所以可能会导致OutOfMemoryError错误，慎用；

### -Xincgc
开启增量gc（默认为关闭）；这有助于减少长时间GC时应用程序出现的停顿；但由于可能和应用程序并发执行，所以会降低CPU对应用的处理能力。

### -Xloggc:file
与`-verbose:gc`功能类似，只是将每次GC事件的相关情况记录到一个文件中，文件的位置最好在本地，以避免网络的潜在问题。
若与verbose命令同时出现在命令行中，则以-Xloggc为准。

## 非Stable参数（非静态参数）
以-XX表示的非Stable参数，虽然在官方文档中是不确定的，不健壮的，各个公司的实现也各有不同，但往往非常实用，所以这部分参数对于GC非常重要。JVM（Hotspot）中主要的参数可以大致分为3类（参考http://blog.csdn.net/sfdev/article/details/2063928）：

* 性能参数（ Performance Options）：用于JVM的性能调优和内存分配控制，如初始化内存大小的设置；
* 行为参数（Behavioral Options）：用于改变JVM的基础行为，如GC的方式和算法的选择；
* 调试参数（Debugging Options）：用于监控、打印、输出等jvm参数，用于显示jvm更加详细的信息；

对于非Stable参数，使用方法有4种：

* `-XX:+<option>` 启用选项
* `-XX:-<option>` 不启用选项
* `-XX:<option>=<number>` 给选项设置一个数字类型值，可跟单位，例如 32k, 1024m, 2g
* `-XX:<option>=<string>` 给选项设置一个字符串值，例如-XX:HeapDumpPath=./dump.core

###**行为参数**
参数及其默认值|描述
------|------
-XX:-UseConcMarkSweepGC|使用`ParNew+CMS|Serial Old`组合并发收集，优先使用ParNew+CMS，当用户线程内存不足时，采用备用方案Serial Old收集。
-XX:-UseSerialGC|启用串行GC，即采用Serial+Serial Old模式
-XX:-UseParallelGC|启用并行GC，即采用Parallel Scavenge+Serial Old收集器组合（-Server模式下的默认组合）
-XX:-UseParallelOldGC|使用Parallel Scavenge +Parallel Old组合收集器
-XX:+UseParNewGC|使用ParNew+Serial Old收集器组合
-XX:GCTimeRatio=99|设置用户执行时间占总时间的比例（默认值99，即1%的时间用于GC）
-XX:-DisableExplicitGC|禁止调用System.gc()；但jvm的gc仍然有效
-XX:+ScavengeBeforeFullGC|新生代GC优先于Full GC执行，Full GC前调用YGC，默认true
-XX:+MaxFDLimit | 最大化文件描述符的数量限制
-XX:+UseGCOverheadLimit | 在抛出OOM之前限制jvm耗费在GC上的时间比例
-XX:+UseTLAB | 启用线程本地缓存区（Thread Local）。默认开启
-XX:+UseThreadPriorities | 启用本地线程优先级，默认开启
-XX:+UseAltSigs | 为了防止与其他发送信号的应用程序冲突，允许使用候补信号替代 SIGUSR1和SIGUSR2。限于Solaris，默认启用
-XX:+UseBoundThreads | 限于Solaris, 默认启用绑定所有的用户线程到内核线程。减少线程进入饥饿状态（得不到任何cpu time）的次数。
-XX:+UseLWPSynchronization|限于solaris，默认启用。使用轻量级进程（内核线程）替换线程同步。
-XX:+MaxFDLimit|限于Solaris，默认启用，设置java进程可用文件描述符为操作系统允许的最大值。
-XX:+UseVMInterruptibleIO|限于solaris，默认启用。在solaris中，允许运行时中断线程 。
-XX:+CMSParallelRemarkEnabled | 在使用 UseParNewGC 的情况下 , 尽量减少 mark 的时间
-XX:CMSInitiatingOccupancyFraction | 指示在 old generation 在使用了 n% 的比例后 , 启动 concurrent collector, 默认值是 68, 如 :-XX:CMSInitiatingOccupancyFraction=70
-XX:+UseCMSInitiatingOccupancyOnly | 指示只有在 old generation 在使用了初始化的比例后 concurrent collector 启动收集
-XX:TLABWasteTargetPercent | TLAB占eden区的百分比，默认1%
-XX:+CollectGen0First | FullGC时是否先YGC，默认false
-XX:+AggressiveHeap | 试图使用大量的物理内存
-XX:CMSInitiatingPermOccupancyFraction | 设置Perm Gen使用到达多少比率时触发
-XX:-AllowUserSignalHandlers | 允许为java进程安装信号处理器。限于Linux和Solaris，默认不启用
-XX:AltStackSize=16384 | 备用信号堆栈大小
-XX:-RelaxAccessControlCheck | 在Class校验器里，放松对访问控制的检查，默认不启用。作用与reflection里的setAccessible类似。
-XX:+UseSplitVerifier | 使用新Class类型校验器，	java5之前默认不启用，java6之后默认启用，详情参见[附言：什么是新class类型校验器](https://www.zybuluo.com/huis/note/607881#什么是新class类型校验器)
-XX:+FailOverToOldVerifier | 如果新的Class校验器检查失败，则使用老的校验器，默认启用
-XX:+HandlePromotionFailure | 关闭新生代收集担保，默认开启，详情参见[附言：什么是新生代担保](https://www.zybuluo.com/huis/note/607881#什么是新生代收集担保)
-XX:+UseSpinning | 启用多线程自旋锁优化。默认开启，详情参见[附言：自旋锁优化原理](https://www.zybuluo.com/huis/note/607881#自旋锁优化原理)
-XX:PreBlockSpin=10 | 控制多线程自旋锁优化的自旋次数。默认10

### 垃圾收集(G1)选项
参数|描述
-------|------
-XX:+UseG1GC | 使用G1收集器
-XX:MaxGCPauseMillis=time|设置GC的最大停顿时间（软目标）
-XX:InitiatingHeapOccupancyPercent=45 | 堆占用百分比达到目标后启动并发GC循环
-XX:NewRatio=2|新生代内存容量与老生代内存容量的比例（Old/New=NewRatio）
-XX:SurvivorRatio=8|Eden区域Survivor区的容量比值，默认值为8，如Eden：Survivor1：Survivor2=8:1:1
-XX:MaxTenuringThreshold=15|对象在新生代存活区切换的次数（坚持过MinorGC的次数，每坚持过一次，该值就增加1），大于该值会进入老年代，默认15
-XX:ParallelGCThreads=n|并行垃圾收集器使用的线程数
-XX:ConcGCThreads=n|并发垃圾收集器使用的线程数。
-XX:G1ReservePercent=n | 设置保留为false上限的堆的数量，以减少promotion失败的可能性.
-XX:G1HeapRegionSize=n | 讲G1的堆分为大小均匀的区域。这将设置每个子分区的大小。此参数的默认值是符合人体工程学的基于堆的大小确定。最小值为1MB，最大值是32mb。

###**性能参数**
参数及其默认值|描述
-------|-------
-XX:NewSize=2.125m|新生代对象生成时占用内存的默认值
-XX:MaxNewSize=size|新生代对象能占用堆内存的最大值
-XX:MaxPermSize=64m|永久代最大值
-XX:PermSize=64m|永久代初始内存
-XX:NewRatio=2|新生代内存容量与老生代内存容量的比例（Old/New=NewRatio）
-XX:SurvivorRatio=8|Eden区域Survivor区的容量比值，默认值为8，如Eden：Survivor1：Survivor2=8:1:1
-XX:TargetSurvivorRatio=50 | 清除后期望的幸存空间剩余比例
-XX:MaxHeapFreeRatio=70|GC后，如果发现空闲堆内存占到整个预估堆内存的70%，则收缩堆内存预估最大值。参见详情[附言：什么是预估堆内存](https://www.zybuluo.com/huis/note/607881#什么是预估堆内存)
-XX:MinHeapFreeRatio=40|GC后，如果发现空闲堆内存占到整个预估堆内存的40%，则放大堆内存的预估最大值，但不超过固定最大值。
-XX:ReservedCodeCacheSize= 32m|保留代码占用的内存容量
-XX:ThreadStackSize=512|设置线程栈大小，若为0则使用系统默认值
-XX:LargePageSizeInBytes=4m|设置堆的内存页面大小
-XX:PretenureSizeThreshold= size|大于该值的对象直接晋升入老年代（这种对象少用为好）
-XX:+UseLargePages | 使用大页面内存，详情[Java Support for Large Memory Pages](http://www.oracle.com/technetwork/java/javase/tech/largememory-jsp-137182.html)
-XX:+UseFastAccessorMethods | get,set 方法转成本地代码，使用优化的get原始字段
-XX:-UseISM	| 使用亲密共享内存，详情参见[Intimate Shared Memory](http://www.oracle.com/technetwork/java/ism-139376.html)
-XX:+UseMPSS | 启用多页面支持的w/4mb堆内存，不能ISM同用
-XX:+UseStringCache | 启用字符串缓存,缓存常用的字符串
-XX:AllocatePrefetchLines=1 | 高速缓存线后使用预取指令生成JIT编译的代码的最后一个对象分配负载数量。默认值是1，如果最后分配的对象是实例，如果数组是3。
-XX:AllocatePrefetchStyle=1 | 生成的预读指令的代码风格。0没有预读指令生成，1每次分配后执行预读指令，2使用TLAB分配水印指针时，预取指令执行。
-XX:+AggressiveOpts | 启用JVM开发团队最新的调优成果。例如编译优化，偏向锁，并行年老代收集等，默认启用
-XX:+UseBiasedLocking | 启用偏向锁，详情参见[tuning example](http://www.oracle.com/technetwork/java/tuning-139912.html#section4.2.5)
-XX:SoftRefLRUPolicyMSPerMB | 每兆堆空闲空间中SoftReference的存活时间,默认1s
-XX:CompileThreshold=10000 | JIT将方法编译成机器码的触发阀值，可以理解为调用方法的次数，例如调1000次，这编译为机器码。默认1000
-XX:+UseCompressedStrings | 使用byte[]可以代表纯ASCII
-XX:+OptimizeStringConcat | 优化字符串连接操作
-XX:CMSMaxAbortablePrecleanTime | 当abortable-preclean阶段执行达到这个时间时才会结束
-XX:CMSScheduleRemarkEdenSizeThreshold（默认2m） | 控制abortable-preclean阶段什么时候开始执行，即当eden使用达到此值时，才会开始abortable-preclean阶段
-XX:CMSScheduleRemarkEdenPenetratio（默认50%）|控制abortable-preclean阶段什么时候结束执行

###**调试参数**
参数及其默认值|描述
------|------
-XX:-CITime|打印消耗在JIT编译的时间
`-XX:ErrorFile=./hs_err_pid<pid>.log`|保存错误日志或者数据到文件中
-XX:-ExtendedDTraceProbes|开启solaris特有的dtrace探针
`-XX:HeapDumpPath=./java_pid<pid>.hprof`|指定导出堆信息时的路径或文件名
-XX:-HeapDumpOnOutOfMemoryError|当遭遇OOM时导出此时堆中相关信息
-XX:OnError=”`<cmd args>`;`<cmd args>`” | 出现致命ERROR之后运行自定义命令
-XX:OnOutOfMemoryError=”`<cmd args>;<cmd args>`”|当首次遭遇OOM时执行自定义命令
-XX:-PrintClassHistogram|遇到Ctrl-Break后打印类实例的柱状信息，与`jmap -histo`功能相同
-XX:-PrintConcurrentLocks|遇到Ctrl-Break后打印并发锁的相关信息，与`jstack -l`功能相同
-XX:-PrintCommandLineFlags|打印出JVM初始化完毕后所有跟初始化的默认之不同的参数及他们的值
-XX:-PrintCompilation|当一个方法被编译时打印相关信息
-XX:-PrintGC|每次GC时打印相关信息,输出形式：**[GC 118250K->113543K(130112K), 0.0094143 secs][Full GC 121376K->10414K(130112K), 0.0650971 secs]**
-XX:-PrintGCDetails|每次GC时打印详细信息,输出形式：**[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs][GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]**
-XX:-PrintGCTimeStamps|打印每次GC的时间戳,输出形式：**11.851: [GC 98328K->93620K(130112K), 0.0082960 secs]**
-XX:+PrintGCApplicationConcurrentTime | 打印每次垃圾回收前，程序未中断的执行时间,输出形式：**Application time: 0.5291524 seconds**
-XX:-PrintAdaptiveSizePolicy | 打印有关自适应代的大小信息。
-XX:+PrintGCApplicationStoppedTime | 打印垃圾回收期间程序暂停的时间,输出形式：**Total time for which application threads were stopped: 0.0468229 seconds**
-XX:PrintHeapAtGC| 打印GC前后的详细堆栈信息,参见[附言](https://www.zybuluo.com/huis/note/607881#xxprintheapatgc日志输入形式)
-XX:-PrintTenuringDistribution | 打印对象的存活期限信息。
-XX:+PrintFlagsFinal |显示所有可设置的参数及”参数处理”后的默认值，可是查看不同版本默认值,以及是否设置成功.输出的信息中”=”表示使用的是初始默认值，而”:=”表示使用的不是初始默认值
-XX:+PrintFlagsInitial|在”参数处理”之前所有可设置的参数及它们的值,然后直接退出程序
-XX:-TraceClassLoading|跟踪类的加载信息
-XX:-TraceClassLoadingPreorder|跟踪被引用到的所有类的加载信息
-XX:-TraceClassResolution|跟踪常量池
-XX:-TraceClassUnloading|跟踪类的卸载信息
-XX:-TraceLoaderConstraints|跟踪类加载器约束的相关信息
-XX:+PerfDataSaveToFile | 退出时保存jvmstat二进制文件
-XX:ParallelGCThreads=n|并行垃圾收集器使用的线程数
-XX:+UseCompressedOops | 优化64位堆内存小于32G，使用压缩指针
-XX:+AlwaysPreTouch | java堆在JVM初始化预处理。堆的每一页都是这样要求零初始化而不是增量在应用程序执行期间。
-XX:AllocatePrefetchDistance=n | 设置对象分配预取距离。预读取缓存大小（以字节为单位）超过新分配的大小，则新对象被写入内存。每个java线程都有自己的配置点。默认值不同平台上的JVM运行。
-XX:InlineSmallCode=n | 仅在其生成的本机代码大小小于此值时，将先前编译的方法内联。默认值不同平台上的JVM运行。
-XX:MaxInlineSize=35 | 方法内联的最大字节大小。
-XX:FreqInlineSize=n | 频繁执行的是内联方法的最大字节大小。
-XX:LoopUnrollLimit=n | 循环体展开服务器编译器的中间节点数小于这个值。服务器编译器使用的限制是这个值的函数，而不是实际值.
-XX:InitialTenuringThreshold=7 | 自适应并行新生代的初始阀值。该阈值是对象在年轻被确认到年老带或终身存活次数。
-XX:MaxTenuringThreshold=n | 设置自适应GC的最大阀值。当前最大的值为15。默认值是并行是15，CMS是4。
-XX:-UseGCLogFileRotation | 启用GC日志文件的自动转储，要求Xloggc参数。
-XX:NumberOfGClogFiles=1 | 设置循环日志时使用的文件数，必须为> 1。循环的日志文件将使用以下命名方案，<文件名>。0，<文件名>。1，…，<文件名>。1。
-XX:GCLogFileSize=8K | 日志文件的大小,日志会循环,此时必须> = 8 k。。

主要参考了博客：
http://blog.csdn.net/sfdev/article/details/2063928和http://kenwublog.com/docs/java6-jvm-options-chinese-edition.htm，后一个比较全面，有兴趣的可以仔细研读。

## 收集器搭配
![image_1b4fjeid71q9p6v11g6n1t3q1ov6m.png-141kB][2]
图中两个收集器之间有连线，说明它们可以配合使用
###Serial收集器
Serial收集器是在client模式下默认的新生代收集器，其收集效率大约是100M左右的内存需要几十到100多毫秒；
在client模式下，收集桌面应用的内存垃圾，基本上不影响用户体验。所以，一般的Java桌面应用中，直接使用Serial收集器（不需要配置参数，用默认即可）。

###ParNew收集器
Serial收集器的多线程版本，这种收集器默认开通的线程数与CPU数量相同，`-XX:ParallelGCThreads`可以用来设置开通的线程数。
可以与CMS收集器配合使用，事实上用`-XX:+UseConcMarkSweepGC`选择使用CMS收集器时，默认使用的就是ParNew收集器，所以不需要额外设置`-XX:+UseParNewGC`，设置了也不会冲突，因为会将ParNew+Serial Old作为一个备选方案；
如果单独使用`-XX:+UseParNewGC`参数，则选择的是`ParNew+Serial Old`收集器组合收集器。
一般情况下，在server模式下，如果选择CMS收集器，则优先选择ParNew收集器。

###Parallel Scavenge收集器
关注的是吞吐量（关于吞吐量的含义见上一篇博客），可以这么理解，关注吞吐量，意味着强调任务更快的完成，而如CMS等关注停顿时间短的收集器，强调的是用户交互体验。
在需要关注吞吐量的场合，比如数据运算服务器等，就可以使用Parallel Scavenge收集器。

### Serial Old收集器
在1.5版本及以前可以与 `Parallel Scavenge`结合使用（事实上，也是当时Parallel Scavenge唯一能用的版本），另外就是在使用CMS收集器时的备用方案，发生 Concurrent Mode Failure时使用。

如果是单独使用，Serial Old一般用在client模式中。

### Parallel Old收集器
在1.6版本之后，与 `Parallel Scavenge`结合使用，以更好的贯彻吞吐量优先的思想，如果是关注吞吐量的服务器，建议使用`Parallel Scavenge + Parallel Old` 收集器。

### CMS收集器
这是当前阶段使用很广的一种收集器，国内很多大的互联网公司线上服务器都使用这种垃圾收集器（http://blog.csdn.net/wisgood/article/details/17067203），笔者公司的收集器也是这种，CMS收集器以获取最短回收停顿时间为目标，非常适合对用户响应比较高的B/S架构服务器。

### CMSIncrementalMode
CMS收集器变种，属增量式垃圾收集器，在并发标记和并发清理时交替运行垃圾收集器和用户线程。适用于单CPU

### G1 收集器
面向服务器端应用的垃圾收集器，计划未来替代CMS收集器。

## 建议
### Java桌面应用
建议采用`Serial+Serial Old`收集器组合，即：`-XX:+UseSerialGC`（-client下的默认参数）

### 开发/测试环境
可以采用默认参数，即采用`Parallel Scavenge+Serial Old`收集器组合，即：`-XX:+UseParallelGC`（-server下的默认参数）

### 线上运算优先的环境
建议采用`Parallel Scavenge+Serial Old`收集器组合，即：`-XX:+UseParallelGC`

### 线上服务响应优先的环境
建议采用`ParNew+CMS+Serial Old`收集器组合，即：`-XX:+UseConcMarkSweepGC`

### 吞吐量优先的并行收集器
如上文所述，并行收集器主要以到达一定的吞吐量为目标，适用于科学技术和后台处理等。
典型配置 ：
`java -Xmx3800m -Xms3800m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 `
**-XX:+UseParallelGC** ：选择垃圾收集器为并行收集器。 此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
**-XX:ParallelGCThreads=20** ：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
`java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC`
**-XX:+UseParallelOldGC** ：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
`java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100`
**-XX:MaxGCPauseMillis=100** : 设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
`java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy`
**-XX:+UseAdaptiveSizePolicy** ：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。

### 响应时间优先 的并发收集器
如上文所述，并发收集器主要是保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。
典型配置 ：
`java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC`
**-XX:+UseConcMarkSweepGC** ：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
**-XX:+UseParNewGC** :设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
`java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection `
**-XX:CMSFullGCsBeforeCompaction** ：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
**-XX:+UseCMSCompactAtFullCollection** ：打开对年老代的压缩。可能会影响性能，但是可以消除碎片

另外在选择了垃圾收集器组合之后，还要配置一些辅助参数，以保证收集器可以更好的工作。关于这些参数，请在http://kenwublog.com/docs/java6-jvm-options-chinese-edition.htm中查询其意义和用法，如：

### ParNew收集器
你可能需要配置4个参数： `-XX:SurvivorRatio`, `-XX:PretenureSizeThreshold`, `-XX:+HandlePromotionFailure`,`-XX:MaxTenuringThreshold`；

### Parallel Scavenge收集器（并行收集器）
你可能需要配置的参数： `-XX:MaxGCPauseMillis`，`-XX:GCTimeRatio`， `-XX:+UseAdaptiveSizePolicy` `-XX:ParallelGCThreads=n`；

### CMS收集器（并发收集器）
你可能需要配置的参数： `-XX:CMSInitiatingOccupancyFraction`，
`-XX:+CMSIncrementalMode`,
`-XX:ParallelGCThreads=n`,
`-XX:+UseCMSCompactAtFullCollection`, `-XX:CMSFullGCsBeforeCompaction`；

## 启动内存分配
用`-Xmn`，`-Xmx`，`-Xms`，`-Xss`，`-XX:NewSize`，`-XX:MaxNewSize`，`-XX:MaxPermSize`，`-XX:PermSize`，`-XX:SurvivorRatio`，`-XX:PretenureSizeThreshold`，`-XX:MaxTenuringThreshold`就基本可以配置内存启动时的分配情况
**建议**
1. `-XX:PermSize`尽量比`-XX:MaxPermSize`小，`-XX:MaxPermSize>= 2 * -XX:PermSize`, `-XX:PermSize> 64m`，一般对于4G内存的机器，`-XX:MaxPermSize`不会超过256m；
2. `-Xms =  -Xmx（线上Server模式）`，以防止抖动，大小受操作系统和内存大小限制，如果是`32位`系统，则一般`-Xms设置为1g-2g（假设有4g内存）`，在`64位`系统上，没有限制，不过一般为`机器最大内存的一半左右`；
3. `-Xmn`，在`开发环境`下，可以用`-XX:NewSize和-XX:MaxNewSize`来设置新生代的大小（`-XX:NewSize<=-XX:MaxNewSize`），在`生产环境`，建议只设置`-Xmn`，一般`-Xmn的大小是-Xms的1/2左右`，不要设置的过大或过小，过大导致老年代变小，频繁Full GC，过小导致minor GC频繁。如果不设置-Xmn，可以采用`-XX:NewRatio=2`来设置，也是一样的效果；
4. `-Xss`一般是不需要改的，默认值即可。
5. `-XX:SurvivorRatio`一般设置`8-10`左右，推荐设置为`10`，也即：`Survivor区的大小是Eden区的1/10`，一般来说，普通的Java程序应用，一次minorGC后，至少98%-99%的对象，都会消亡，所以，survivor区设置为Eden区的1/10左右，能使Survivor区容纳下10-20次的minor GC才满，然后再进入老年代，这个与 `-XX:MaxTenuringThreshold`的默认值15次也相匹配的.
6. `-XX:MaxTenuringThreshold`默认为`15`，也就是说，`经过15次Survivor轮换（即15次minor GC），就进入老年代`， 如果设置的小的话，则年轻代对象在survivor中存活的时间减小，提前进入年老代，对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代即被回收的概率。需要注意的是，设置了 `-XX:MaxTenuringThreshold`，并不代表着，对象一定在年轻代存活`15`次才被晋升进入老年代，它只是一个最大值，事实上，存在一个动态计算机制，计算每次晋入老年代的阈值，取`阈值和MaxTenuringThreshold中较小`的一个为准。
7. `-XX:PretenureSizeThreshold`一般采用默认值即可。

## 监控工具和方法
**问题：**
1，minor GC和full GC的频率；
2，执行一次GC所消耗的时间；
3，新生代的对象何时被移到老生代以及花费了多少时间；
4，每次GC中，其它线程暂停（Stop the world）的时间；
5，每次GC的效果如何，是否不理想；

### jps
jps命令用于查询正在运行的JVM进程，常用的参数为：

    -q:只输出LVMID，省略主类的名称
    -m:输出虚拟机进程启动时传给主类main()函数的参数
    -l:输出主类的全类名，如果进程执行的是Jar包，输出Jar路径
    -v:输出虚拟机进程启动时JVM参数

**命令格式:**`jps [option] [hostid]`
![image_1b4fkttqie6p1h5j1j7q1qvm9k613.png-25.9kB][3]

### jstat
jstat可以实时显示本地或远程JVM进程中类装载、内存、垃圾收集、JIT编译等数据（如果要显示远程JVM信息，需要远程主机开启RMI支持）。如果在服务启动时没有指定启动参数-verbose:gc，则可以用jstat实时查看gc情况。
jstat有如下选项：

    -class:监视类装载、卸载数量、总空间及类装载所耗费的时间
    -gc:监听Java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量，以用空间、GC时间合计等信息
    -gccapacity:监视内容与-gc基本相同，但输出主要关注java堆各个区域使用到的最大和最小空间
    -gcutil:监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
    -gccause:与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因
    -gcnew:监视新生代GC状况
    -gcnewcapacity:监视内同与-gcnew基本相同，输出主要关注使用到的最大和最小空间
    -gcold:监视老年代GC情况
    -gcoldcapacity:监视内同与-gcold基本相同，输出主要关注使用到的最大和最小空间
    -gcpermcapacity:输出永久代使用到最大和最小空间
    -compiler:输出JIT编译器编译过的方法、耗时等信息
    -printcompilation:输出已经被JIT编译的方法
**命令格式:**`jstat [option vmid [interval[s|ms] [count]]]`
jstat可以监控远程机器，命令格式中VMID和LVMID特别说明：如果是本地虚拟机进程，VMID和LVMID是一致的，如果是远程虚拟机进程，那么VMID格式是: `[protocol:][//]lvmid[@hostname[:port]/servername]`，如果省略`interval和count`，则只查询一次
![image_1b4fl503n1adb1uknqvs1md6jbn1t.png-120kB][4]
在图中，命令`jstat -gc 1581 1000 5`代表着：搜集vid为1581的java进程的整体gc状态， 每1000ms收集一次，共收集5次；
XXC:表示该区容量, XXU:表示该区使用量，各列解释如下：
S0C：S0区容量（S1区相同，略）
S0U：S0区已使用
EC：E区容量
EU：E区已使用
OC：老年代容量
OU：老年代已使用
*PC：Perm容量（过时）
*PU：Perm区已使用（过时）
MC：方法区大小
MU：方法区使用大小
CCSC：压缩类空间大小
CCSU：压缩类空间使用大小
YGC：Young GC（Minor GC）次数
YGCT：Young GC总耗时
FGC：Full GC次数
FGCT：Full GC总耗时
GCT：GC总耗时
参考：http://blog.csdn.net/maosijunzi/article/details/46049117

### jinfo
用于查询当前运行这的JVM属性和参数的值。
jinfo可以使用如下选项：

    -flag:显示未被显示指定的参数的系统默认值
    -flag [+|-]name或-flag name=value: 修改部分参数
    -sysprops:打印虚拟机进程的System.getProperties()
**命令格式:**`jinfo [option] pid`

### jmap
用于显示当前Java堆和永久代的详细信息（如当前使用的收集器，当前的空间使用率等）

    -dump:生成java堆转储快照
    -heap:显示java堆详细信息(只在Linux/Solaris下有效)
    -F:当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照(只在Linux/Solaris下有效)
    -finalizerinfo:显示在F-Queue中等待Finalizer线程执行finalize方法的对象(只在Linux/Solaris下有效)
    -histo:显示堆中对象统计信息
    -permstat:以ClassLoader为统计口径显示永久代内存状态(只在Linux/Solaris下有效)

**命令格式:**`jmap [option] vmid`
其中前面3个参数最重要，如：
查看堆详细信息：`jmap -heap 309`
生成dump文件： `jmap -dump:file=./test.prof 309`
部分用户没有权限时，采用admin用户：`sudo -u admin -H  jmap -dump:format=b,file=文件名.hprof pid`
查看当前堆中对象统计信息：`sudo  jmap -histo 309`：该命令显示3列，分别为对象数量，对象大小，对象名称，通过该命令可以查看是否内存中有大对象；
有的用户可能没有jmap权限：`sudo -u admin -H jmap -histo 309 | less`

### jhat
用于分析使用jmap生成的dump文件，是JDK自带的工具，使用方法为： `jhat -J -Xmx512m [file]`
不过jhat没有mat好用，推荐使用`mat`（Eclipse插件： http://www.eclipse.org/mat ），mat速度更快，而且是图形界面。

### jstack
用于生成当前JVM的所有线程快照，线程快照是虚拟机每一条线程正在执行的方法,目的是定位线程出现长时间停顿的原因。

    -F:当正常输出的请求不被响应时，强制输出线程堆栈
    -l:除堆栈外，显示关于锁的附加信息
    -m:如果调用到本地方法的话，可以显示C/C++的堆栈
**命令格式:**`jstack [option] vmid`

### -verbosegc
-verbosegc是一个比较重要的启动参数，记录每次gc的日志，下面的表格对比了jstat和-verbosegc：
功能|jstat|-verbosegc
-------|------|------
监控对象|运行在本机的Java应用可以把日志输出到终端上，或者借助jstatd命令通过网络连接远程的Java应用|只有那些把-verbogc作为启动参数的JVM。
输出信息|堆状态（已用空间，最大限制，GC执行次数/时间，等等|执行GC前后新生代和老年代空间大小，GC执行时间。
输出时间|Every designated time|每次设定好的时间。每次GC发生的时候。
用途|观察堆空间变化情况|了解单次GC产生的效果。

与-verbosegc配合使用的一些常用参数为：
`-XX:+PrintGCDetails`，打印GC信息，这是-verbosegc默认开启的选项
`-XX:+PrintGCTimeStamps`，打印每次GC的时间戳
`-XX:+PrintHeapAtGC`：每次GC时，打印堆信息
`-XX:+PrintGCDateStamps (from JDK 6 update 4)` ：打印GC日期，适合于长期运行的服务器
`-Xloggc:/home/admin/logs/gc.log`：制定打印信息的记录的日志位置
每条verbosegc打印出的gc日志，都类似于下面的格式：
```
time [GC [<collector>: <starting occupancy1> -> <ending occupancy1>(total occupancy1), <pause time1> secs] <starting occupancy3> -> <ending occupancy3>(total occupancy3), <pause time3> secs]
```
![image_1b4fncn4n1eer1kglv4lvuiuk22q.png-7.5kB][5]
**选项的意义是：**
time：执行GC的时间，需要添加-XX:+PrintGCDateStamps参数才有；
collector：minor gc使用的收集器的名字。
starting occupancy1：GC执行前新生代空间大小。
ending occupancy1：GC执行后新生代空间大小。
total occupancy1：新生代总大小
pause time1：因为执行minor GC，Java应用暂停的时间。
starting occupancy3：GC执行前堆区域总大小
ending occupancy3：GC执行后堆区域总大小
total occupancy3：堆区总大小
pause time3：Java应用由于执行堆空间GC（包括full GC）而停止的时间。

### 可视化工具
监控和分析GC也有一些可视化工具，比较常见的有JConsole和VisualVM，有兴趣的可以看看下面的文章，在此不再赘述：
http://blog.csdn.net/java2000_wl/article/details/8049707

## 调优方法
**原则：**
1. 多数的Java应用不需要在服务器上进行GC优化；
2. 多数导致GC问题的Java应用，都不是因为我们参数设置错误，而是代码问题；
3. 在应用上线之前，先考虑将机器的JVM参数设置到最优（最适合）；
4. 减少创建对象的数量；
5. 减少使用全局变量和大对象；
6. GC优化是到最后不得已才采用的手段；
7. 在实际使用中，分析GC情况优化代码比优化GC参数要多得多；

**GC优化的目的**有两个（http://www.360doc.com/content/13/0305/10/15643_269388816.shtml）：
1. 将转移到老年代的对象数量降低到最小；
2. 减少full GC的执行时间；

**需要做的事情有：**

 1. 减少使用全局变量和大对象； 
 2. 调整新生代的大小到最合适； 
 3. 设置老年代的大小为最合适； 
 4. 选择合适的GC收集器；

**调优般步骤为：**
### 监控GC的状态
使用各种JVM工具，查看当前日志，分析当前JVM参数设置，并且分析当前堆内存快照和gc日志，根据实际的各区域内存划分和GC执行时间，觉得是否进行优化；
### 分析结果，判断是否需要优化
如果各项参数设置合理，系统没有超时日志出现，GC频率不高，GC耗时不高，那么没有必要进行GC优化；如果GC时间超过1-3秒，或者频繁GC，则必须优化；
**注：如果满足下面的指标，则一般不需要进行GC：**

 1. Minor GC执行时间不到50ms； 
 2. Minor GC执行不频繁，约10秒一次； 
 3. Full GC执行时间不到1s； 
 4. Full GC执行频率不算频繁，不低于10分钟1次；

### 调整GC类型和内存分配
如果内存分配过大或过小，或者采用的GC收集器比较慢，则应该优先调整这些参数，并且先找1台或几台机器进行beta，然后比较优化过的机器和没有优化的机器的性能对比，并有针对性的做出最后选择；

### 不断的分析和调整
通过不断的试验和试错，分析并找到最合适的参数

### 全面应用参数
如果找到了最合适的参数，则将这些参数应用到所有服务器，并进行后续跟踪。

## 调优实例
### 实例1：
笔者昨日发现部分开发测试机器出现异常：`java.lang.OutOfMemoryError: GC overhead limit exceeded`，这个异常代表：GC为了释放很小的空间却耗费了太多的时间，其原因一般有两个：
1，堆太小，
2，有死循环或大对象；
笔者首先排除了第2个原因，因为这个应用同时是在线上运行的，如果有问题，早就挂了。所以怀疑是这台机器中堆设置太小；
使用`ps -ef |grep “java”`查看，发现：
![image_1b4fnplct1gkd14opg23ndgsqd37.png-30.6kB][6]
该应用的堆区设置只有768m，而机器内存有2g，机器上只跑这一个java应用，没有其他需要占用内存的地方。另外，这个应用比较大，需要占用的内存也比较多；
笔者通过上面的情况判断，只需要改变堆中各区域的大小设置即可，于是改成下面的情况：
![image_1b4fnq8k01pct1notkgf16ej9tb3k.png-39.1kB][7]
跟踪运行情况发现，相关异常没有再出现；

### 实例2：
（http://www.360doc.com/content/13/0305/10/15643_269388816.shtml）
一个服务系统，经常出现卡顿，分析原因，发现Full GC时间太长：
```
jstat -gcutil:
S0     S1    E     O       P        YGC YGCT FGC FGCT  GCT
12.16 0.00 5.18 63.78 20.32  54   2.047 5     6.946  8.993

```
分析上面的数据，发现Young GC执行了54次，耗时2.047秒，每次Young GC耗时37ms，在正常范围，而Full GC执行了5次，耗时6.946秒，每次平均1.389s，数据显示出来的问题是：Full GC耗时较长，分析该系统的是指发现，NewRatio=9，也就是说，新生代和老生代大小之比为1:9，这就是问题的原因：
1，新生代太小，导致对象提前进入老年代，触发老年代发生Full GC；
2，老年代较大，进行Full GC时耗时较大；
优化的方法是调整NewRatio的值，调整到4，发现Full GC没有再发生，只有Young GC在执行。这就是把对象控制在新生代就清理掉，没有进入老年代（这种做法对一些应用是很有用的，但并不是对所有应用都要这么做）

### 实例3：
一应用在性能测试过程中，发现内存占用率很高，Full GC频繁，使用`sudo -u admin -H  jmap -dump:format=b,file=文件名.hprof pid` 来dump内存，生成dump文件，并使用Eclipse下的mat差距进行分析，发现：
![image_1b4fofdo775cnrr1c5s1ebpfob41.png-115.8kB][8]
从图中可以看出，这个线程存在问题，队列LinkedBlockingQueue所引用的大量对象并未释放，导致整个线程占用内存高达378m，此时通知开发人员进行代码优化，将相关对象释放掉即可。

### 实例4：CMSInitiatingOccupancyFraction值与Xmn的关系公式
上面介绍了promontion faild产生的原因是EDEN空间不足的情况下将EDEN与From survivor中的存活对象存入To survivor区时,To survivor区的空间不足，再次晋升到old gen区，而old gen区内存也不够的情况下产生了promontion faild从而导致full gc.那可以推断出：`eden+from survivor < old gen`区剩余内存时，不会出现promontion faild的情况，即：
`(Xmx-Xmn)*(1-CMSInitiatingOccupancyFraction/100)>=(Xmn-Xmn/(SurvivorRatior+2)) ` 进而推断出：

`CMSInitiatingOccupancyFraction <=((Xmx-Xmn)-(Xmn-Xmn/(SurvivorRatior+2)))/(Xmx-Xmn)*100`

例如：

当xmx=128 xmn=36 SurvivorRatior=1时 `CMSInitiatingOccupancyFraction<=((128.0-36)-(36-36/(1+2)))/(128-36)*100 =73.913`

当xmx=128 xmn=24 SurvivorRatior=1时 `CMSInitiatingOccupancyFraction<=((128.0-24)-(24-24/(1+2)))/(128-24)*100=84.61`

当xmx=3000 xmn=600 SurvivorRatior=1时  `CMSInitiatingOccupancyFraction<=((3000.0-600)-(600-600/(1+2)))/(3000-600)*100=83.33`

CMSInitiatingOccupancyFraction低于70% 需要调整xmn或SurvivorRatior值。

网上一童鞋推断出的公式是：`(Xmx-Xmn)*(100-CMSInitiatingOccupancyFraction)/100>=Xmn` 这个公式个人认为不是很严谨，在内存小的时候会影响xmn的计算。

### 实例5：promotion failed
垃圾回收时promotion failed是个很头痛的问题，一般可能是两种原因产生，第一个原因是救助空间不够，救助空间里的对象还不应该被移动到年老代，但年轻代又有很多对象需要放入救助空间；第二个原因是年老代没有足够的空间接纳来自年轻代的对象；这两种情况都会转向Full GC，网站停顿时间较长。

**解决方方案一：**

第一个原因我的最终解决办法是去掉救助空间，设置`-XX:SurvivorRatio=65536 -XX:MaxTenuringThreshold=0`即可，第二个原因我的解决办法是设置CMSInitiatingOccupancyFraction为某个值（假设70），这样年老代空间到70%时就开始执行CMS，年老代有足够的空间接纳来自年轻代的对象。

**解决方案一的改进方案：**

又有改进了，上面方法不太好，因为没有用到救助空间，所以年老代容易满，CMS执行会比较频繁。我改善了一下，还是用救助空间，但是把救助空间加大，这样也不会有promotion failed。具体操作上，32位Linux和64位Linux好像不一样，64位系统似乎只要配置MaxTenuringThreshold参数，CMS还是有暂停。为了解决暂停问题和promotion failed问题，最后我设置`-XX:SurvivorRatio=1` ，并把MaxTenuringThreshold去掉，这样即没有暂停又不会有promotoin failed，而且更重要的是，年老代和永久代上升非常慢（因为好多对象到不了年老代就被回收了），所以CMS执行频率非常低，好几个小时才执行一次，这样，服务器都不用重启了。
```
-Xmx4000M -Xms4000M -Xmn600M -XX:PermSize=500M -XX:MaxPermSize=500M -Xss256K -XX:+DisableExplicitGC -XX:SurvivorRatio=1 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=0 -XX:+CMSClassUnloadingEnabled -XX:LargePageSizeInBytes=128M -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=80 -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+PrintClassHistogram -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -Xloggc:log/gc.log
```

## 调优总结
### 年轻代大小选择
#### 响应时间优先的应用
尽可能设大，直到接近系统的最低响应时间限制 （根据实际情况选择）。在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象。
#### 吞吐量优先的应用
尽可能的设置大，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用。
###年老代大小选择
####响应时间优先的应用
年老代使用并发收集器，所以其大小需要小心设置，一般要考虑**并发会话率** 和**会话持续时间** 等一些参数。如果堆设置小了，可以会造成内存碎片、高回收频率以及应用暂停而使用传统的标记清除方式；如果堆大了，则需要较长的收集时间。最优化的方案，一般需要参考以下数据获得：
1. 并发垃圾收集信息
2. 持久代并发收集次数
3. 传统GC信息
4. 花在年轻代和年老代回收上的时间比例
减少年轻代和年老代花费的时间，一般会提高应用的效率
#### 吞吐量优先的应用
一般吞吐量优先的应用都有一个很大的年轻代和一个较小的年老代。原因是，这样可以尽可能回收掉大部分短期对象，减少中期的对象，而年老代尽存放长期存活对象。
### 较小堆引起的碎片问题
因为年老代的并发收集器使用标记、清除算法，所以不会对堆进行压缩。当收集器回 收时，他会把相邻的空间进行合并，这样可以分配给较大的对象。但是，当堆空间较小时，运行一段时间以后，就会出现“碎片”，如果并发收集器找不到足够的空 间，那么并发收集器将会停止，然后使用传统的标记、清除方式进行回收。如果出现“碎片”，可能需要进行如下配置：
**-XX:+UseCMSCompactAtFullCollection** ：使用并发收集器时，开启对年老代的压缩。
**-XX:CMSFullGCsBeforeCompaction=0** ：上面配置开启的情况下，这里设置多少次Full GC后，对年老代进行压缩

## 我的参考资料
[JVM监控与调优](http://www.importnew.com/21441.html)
[Java HotSpot VM Options](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)
[Java虚拟机-JVM各种参数配置大全详细](http://blog.csdn.net/chenleixing/article/details/43230527)
[JVM生产环境参数实例及分析【生产环境实例增加中】](http://www.cnblogs.com/redcreen/archive/2011/05/05/2038331.html)

## 其他的
[JVM启动参数大全](http://www.blogjava.net/midstr/archive/2008/09/21/230265.html)
[JVM系列三:JVM参数设置、分析](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)
[Java 6 JVM参数选项大全（中文版）](http://kenwublog.com/docs/java6-jvm-options-chinese-edition.htm)
[成为JavaGC专家Part II — 如何监控Java垃圾回收机制](http://www.importnew.com/2057.html)
[成为Java GC专家系列(3) — 如何优化Java垃圾回收机制](http://www.importnew.com/3146.html)
[JDK5.0垃圾收集优化之–Don’t Pause](http://calvin.iteye.com/blog/91905)
[Java HOTSPOT VM参数大全](http://tech.sina.com.cn/s/2009-09-23/09561077572.shtml)
[【原】GC的默认方式](http://iamzhongyong.iteye.com/blog/1447314)
[JAVA启动参数大全之三：非Stable参数](http://blog.csdn.net/sfdev/article/details/2063928)
[Java虚拟机学习 – 内存调优](http://blog.csdn.net/java2000_wl/article/details/8090940)
[内存溢出](http://www.open-open.com/home/space.php?uid=71669&do=blog&id=8891)
[如何查看JVM的扩展参数：-X](http://www.blogjava.net/beansoft/archive/2012/03/01/371088.html)
[JVM内存状况查看方法和分析工具](http://hi.baidu.com/kingfly666666/item/e710a4371c60b0f1e7bb7a32)
[虚拟机学习系列 – 附 – 虚拟机参数](http://blog.csdn.net/su1216/article/details/7780924)
[JVM系列四:生产环境参数实例及分析【生产环境实例增加中】](http://www.cnblogs.com/redcreen/archive/2011/05/05/2038331.html)
[垃圾收集器与内存分配策略](http://raging-sweet.iteye.com/blog/1170198)
[JVM垃圾收集器使用调查：CMS最受欢迎 ](http://blog.csdn.net/wisgood/article/details/17067203)
[Xms Xmx PermSize MaxPermSize 区别](http://www.cnblogs.com/mingforyou/archive/2012/03/03/2378143.html)
[Java虚拟机学习 – JDK可视化监控工具](http://blog.csdn.net/java2000_wl/article/details/8049707)
[虚拟机学习系列 – 6 – JDK工具](http://blog.csdn.net/su1216/article/details/7780857)
[JVM监控工具介绍jstack, jconsole, jinfo, jmap, jdb, jstat](http://hi.baidu.com/lotusxyhf/item/9cd8fcb8d6f8c1a5ebba935b)
[JVM 与 jstat](http://blog.sina.com.cn/s/blog_56fcfd620100hdcp.html)

## 附言
### -XX:PrintHeapAtGC日志输入形式
```
34.702: [GC {Heap before gc invocations=7:
def new generation   total 55296K, used 52568K [0x1ebd0000, 0x227d0000, 0x227d0000)
eden space 49152K, 99% used [0x1ebd0000, 0x21bce430, 0x21bd0000)
from space 6144K, 55% used [0x221d0000, 0x22527e10, 0x227d0000)
to   space 6144K,   0% used [0x21bd0000, 0x21bd0000, 0x221d0000)
tenured generation   total 69632K, used 2696K [0x227d0000, 0x26bd0000, 0x26bd0000)
the space 69632K,   3% used [0x227d0000, 0x22a720f8, 0x22a72200, 0x26bd0000)
compacting perm gen total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)
   the space 8192K, 35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)
    ro space 8192K, 66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)
    rw space 12288K, 46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)
34.735: [DefNew: 52568K->3433K(55296K), 0.0072126 secs] 55264K->6615K(124928K)Heap after gc invocations=8:
def new generation   total 55296K, used 3433K [0x1ebd0000, 0x227d0000, 0x227d0000)
eden space 49152K,   0% used [0x1ebd0000, 0x1ebd0000, 0x21bd0000)
from space 6144K, 55% used [0x21bd0000, 0x21f2a5e8, 0x221d0000)
to   space 6144K,   0% used [0x221d0000, 0x221d0000, 0x227d0000)
tenured generation   total 69632K, used 3182K [0x227d0000, 0x26bd0000, 0x26bd0000)
the space 69632K,   4% used [0x227d0000, 0x22aeb958, 0x22aeba00, 0x26bd0000)
compacting perm gen total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)
   the space 8192K, 35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)
    ro space 8192K, 66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)
    rw space 12288K, 46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)
}
, 0.0757599 secs]
```

### 什么是新Class类型校验器？
新Class类型校验器，将老的校验步骤拆分成两步：
1，类型推断。
2，类型校验。

新类型校验器通过在javac编译时嵌入类型信息到bytecode中，省略了类型推断这一步，从而提升了classloader的性能。

Classload顺序
`load -> verify -> prepare -> resove -> init`

### 什么是新生代收集担保？
在一次理想化的minor gc中，活跃对象会从Eden和First Survivor中被复制到Second Survivor。
然而，Second Survivor不一定能容纳所有的活跃对象。

为了确保minor gc能够顺利完成，需要在年老代中保留一块足以容纳所有活跃对象的内存空间。
这个预留的操作，被称之为新生代收集担保（New Generation Guarantee）。当预留操作无法完成时，就会触发major gc(full gc)。

为什么要关闭新生代收集担保？
因为在年老代中预留的空间大小，是无法精确计算的。

为了确保极端情况的发生，GC参考了最坏情况下的新生代内存占用，即Eden+First Survivor。

这种策略无疑是在浪费年老代内存，并从时序角度看，可能提前触发Full GC。

为了避免如上情况的发生，JVM允许开发者关闭新生代收集担保。

在开启本选项后，minotr gc将不再提供新生代收集担保，而是在出现survior或年老代不够用时，抛出promotion failed异常。

### 自旋锁优化原理

大家知道，Java的多线程安全是基于Lock机制实现的，而Lock的性能往往不如人意。
原因是，monitorenter与monitorexit这两个控制多线程同步的bytecode原语，是JVM依赖操作系统互斥(mutex)来实现的。
互斥是一种会导致线程挂起，并在较短的时间内又需要重新调度回原线程的，较为消耗资源的操作。

为了避免进入OS互斥，Java6的开发者们提出了自旋锁优化方法。

自旋锁优化的原理是在线程进入OS互斥前，通过CAS自旋一定的次数来检测锁的释放。

如果在自旋次数未达到预订值前，发现锁已被释放，则会立即持有该锁。
CAS检测锁的原理详见: http://www.bt285.cn/sejishikong/ 
**关联选项：**
`-XX:PreBlockSpin=10`

### 什么是预估堆内存？

预估堆内存是堆大小动态调控的重要选项之一。
堆内存预估最大值一定小于或等于固定最大值(-Xmx指定的数值)。
前者会根据使用情况动态调大或缩小，以提高GC回收的效率。



[1]: http://static.zybuluo.com/huis/elx2qjb0vvbca399azxgx1ot/image_1b4fh891111hb1paab93dgv1hts9.png
[2]: http://static.zybuluo.com/huis/6bjtipzs68oxcjs5t7kd1hp6/image_1b4fjeid71q9p6v11g6n1t3q1ov6m.png
[3]: http://static.zybuluo.com/huis/2tn5ilm5bz741065xsaadsgx/image_1b4fkttqie6p1h5j1j7q1qvm9k613.png
[4]: http://static.zybuluo.com/huis/czt1tq4t2a9rkjm3yuwo16rq/image_1b4fl503n1adb1uknqvs1md6jbn1t.png
[5]: http://static.zybuluo.com/huis/gzkr5tejetj5464u6oqnde9r/image_1b4fncn4n1eer1kglv4lvuiuk22q.png
[6]: http://static.zybuluo.com/huis/lwriss4lir2o82k59lga4a3i/image_1b4fnplct1gkd14opg23ndgsqd37.png
[7]: http://static.zybuluo.com/huis/4th8aaoddh76nwn1hgjyphtr/image_1b4fnq8k01pct1notkgf16ej9tb3k.png
[8]: http://static.zybuluo.com/huis/u98e5nxfgydgz5xkxazbrlhu/image_1b4fofdo775cnrr1c5s1ebpfob41.png