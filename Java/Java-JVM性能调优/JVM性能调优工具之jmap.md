##### JVM性能调优工具之jmap

###### 概述

​		命令jmap是一个多功能的命令。它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。

###### jmap 用法

~~~java
[ls@59-37-131-27 order]$ jmap -help
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
[ls@59-37-131-27 order]$ 
~~~

**参数**

* option：选项参数。
* pid：需要打印配置信息的进程ID。
* executable：产生核心dump的Java可执行文件。
* core：需要打印配置信息的核心文件。
* server-id：可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器。
* remote server IP or hostname：远程调试服务器的IP地址或主机名。

**option**

* no option：查看进程的内存映像信息,类似 Solaris pmap 命令。
* -heap：显示Java堆详细信息
* -histo[:live]：显示堆中对象的统计信息
* -clstats：打印类加载器信息
* -finalizerinfo：显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
* -dump:<dump-options>：生成堆转储快照
* -F：当-dump没有响应时，使用-dump或者-histo参数. 在这个模式下,live子参数无效.

###### 示例一：no option

命令：jmap pid

描述：查看进程的内存映像信息,类似 Solaris pmap 命令。

​		使用不带选项参数的jmap打印共享对象映射，将会打印目标虚拟机中加载的每个共享对象的起始地址、映射大小以及共享对象文件的路径全称。这与Solaris的pmap工具比较相似。

~~~~java
[ls@59-37-131-27 order]$ jmap 24794
Attaching to process ID 24794, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13
0x0000000000400000      7K      /usr/local/java/jdk1.8.0_121/jre/bin/java
0x00007fb8a0178000      49K     /usr/local/java/jdk1.8.0_121/jre/lib/amd64/libmanagement.so
0x00007fb8a217f000      90K     /usr/local/java/jdk1.8.0_121/jre/lib/amd64/libnio.so
0x00007fb8a2690000      113K    /usr/local/java/jdk1.8.0_121/jre/lib/amd64/libnet.so
0x00007fb8c39c8000      121K    /usr/local/java/jdk1.8.0_121/jre/lib/amd64/libzip.so
0x00007fb8c3be3000      60K     /usr/lib64/libnss_files-2.17.so
0x00007fb8c3df6000      48K     /usr/local/java/jdk1.8.0_121/jre/lib/amd64/libinstrument.so
0x00007fb8c811c000      220K    /usr/local/java/jdk1.8.0_121/jre/lib/amd64/libjava.so
0x00007fb8c8348000      64K     /usr/local/java/jdk1.8.0_121/jre/lib/amd64/libverify.so
0x00007fb8c8556000      42K     /usr/lib64/librt-2.17.so
0x00007fb8c875e000      1110K   /usr/lib64/libm-2.17.so
0x00007fb8c8a60000      16591K  /usr/local/java/jdk1.8.0_121/jre/lib/amd64/server/libjvm.so
0x00007fb8c9a52000      2101K   /usr/lib64/libc-2.17.so
0x00007fb8c9e1f000      18K     /usr/lib64/libdl-2.17.so
0x00007fb8ca023000      99K     /usr/local/java/jdk1.8.0_121/jre/lib/amd64/jli/libjli.so
0x00007fb8ca239000      138K    /usr/lib64/libpthread-2.17.so
0x00007fb8ca455000      159K    /usr/lib64/ld-2.17.so
[ls@59-37-131-27 order]$ 
~~~~

###### 示例二：-heap

命令：jmap -heap pid

描述：显示Java堆详细信息

​		打印一个堆的摘要信息，包括使用的GC算法、堆配置信息和各内存区域内存使用信息

~~~java
[ls@59-37-131-27 order]$ jmap -heap 24794
Attaching to process ID 24794, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13

using thread-local object allocation.
Parallel GC with 13 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 536870912 (512.0MB)
   NewSize                  = 367001600 (350.0MB)
   MaxNewSize               = 367001600 (350.0MB)
   OldSize                  = 169869312 (162.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 309329920 (295.0MB)
   used     = 253860088 (242.09984588623047MB)
   free     = 55469832 (52.90015411376953MB)
   82.06774436821372% used
From Space:
   capacity = 28835840 (27.5MB)
   used     = 21119456 (20.141082763671875MB)
   free     = 7716384 (7.358917236328125MB)
   73.24030095880681% used
To Space:
   capacity = 28835840 (27.5MB)
   used     = 0 (0.0MB)
   free     = 28835840 (27.5MB)
   0.0% used
PS Old Generation
   capacity = 169869312 (162.0MB)
   used     = 29685888 (28.3106689453125MB)
   free     = 140183424 (133.6893310546875MB)
   17.475721571180557% used

18857 interned Strings occupying 1711064 bytes.
[ls@59-37-131-27 order]$ 
~~~

###### 示例三：-histo[:live]

命令：jmap -histo:live pid

描述：显示堆中对象的统计信息

​		其中包括每个Java类、对象数量、内存大小(单位：字节)、完全限定的类名。打印的虚拟机内部的类名称将会带有一个’*’前缀。如果指定了live子选项，则只计算活动的对象。

~~~java
[ls@59-37-131-27 order]$ jmap -histo:live 24794

 num     #instances         #bytes  class name
----------------------------------------------
   1:         10250        5789296  [B
   2:         60214        5561576  [C
   3:         33523        2190344  [Ljava.lang.Object;
   4:         58356        1400544  java.lang.String
   5:         10299        1143128  java.lang.Class
   6:          1235         652080  [Lorg.apache.skywalking.apm.dependencies.io.netty.handler.codec.http2.HpackHeaderField;
   7:          8786         639224  [I
   8:         19395         620640  java.util.concurrent.ConcurrentHashMap$Node
   9:          5815         511720  java.lang.reflect.Method
  10:         30799         492784  java.lang.Object
  11:          9712         466176  java.util.HashMap
  12:         13602         435264  java.lang.StackTraceElement
  13:          4483         411176  [Ljava.util.HashMap$Node;
  14:         12819         410208  java.util.HashMap$Node
  15:          5040         405008  [S
  16:         14543         349032  java.util.ArrayList
  17:          8645         276640  java.util.concurrent.atomic.LongAdder
  18:          1235         266760  org.apache.skywalking.apm.dependencies.io.grpc.internal.ManagedChannelImpl
  19:          7229         231328  java.lang.ref.WeakReference
  20:          4083         228648  java.util.LinkedHashMap
  21:           326         201928  [Ljava.util.concurrent.ConcurrentHashMap$Node;
  22:          1283         171616  [Ljava.nio.channels.SelectionKey;
  23:          2504         160592  [Lorg.apache.skywalking.apm.dependencies.io.netty.util.internal.PriorityQueueNode;
  24:          4003         160120  java.lang.ref.SoftReference
  25:          2492         158568  [Ljava.lang.StackTraceElement;
  26:          1235         158080  org.apache.skywalking.apm.dependencies.io.grpc.netty.NettyClientHandler
  27:          2410         154240  org.apache.skywalking.apm.dependencies.io.grpc.internal.DnsNameResolver
  28:          1250         140000  sun.nio.ch.SocketChannelImpl
  29:          1235         138320  org.apache.skywalking.apm.dependencies.io.grpc.internal.InternalSubchannel
  30:          8645         138320  org.apache.skywalking.apm.dependencies.io.grpc.internal.ReflectionLongAdderCounter
  31:          1235         138320  org.apache.skywalking.apm.dependencies.io.grpc.netty.NettyClientTransport
  32:          2470         138320  org.apache.skywalking.apm.dependencies.io.netty.handler.codec.http2.DefaultHttp2Connection$DefaultEndpoint
  33:          1235         128440  org.apache.skywalking.apm.dependencies.io.netty.channel.socket.nio.NioSocketChannel
  34:          5292         127008  java.util.ArrayDeque
  35:          5099         122376  java.util.concurrent.atomic.AtomicLong
  36:          4940         118560  org.apache.skywalking.apm.dependencies.io.grpc.internal.LogId
~~~

###### 示例四：-clstats

命令：jmap -clstats pid

描述：打印类加载器信息

​		-clstats是-permstat的替代方案，在JDK8之前，-permstat用来打印类加载器的数据，打印Java堆内存的永久保存区域的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。

~~~java
[ls@59-37-131-27 order]$ jmap -clstats 24794
Attaching to process ID 24794, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13
finding class loader instances ..done.
computing per loader stat ..done.
please wait.. computing liveness.liveness analysis may be inaccurate ...
class_loader    classes bytes   parent_loader   alive?  type

<bootstrap>     2371    4208558   null          live    <internal>
0x00000000e148c948      1       1473    0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e0ff3af8      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e148bf40      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e148f540      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e0ff1af0      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e0ff4cf0      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e148d758      1       1473    0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e148e958      1       880       null          dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e148f158      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e0373820      1       880       null          dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e0ff2ce8      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e148a150      1       1471    0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e148c350      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e01ee400      8       15673     null          dead    sun/misc/Launcher$ExtClassLoader@0x000000010000fa48
0x00000000e0ff3ee0      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e1489d68      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
0x00000000e0ff1ed8      1       880     0x00000000e0020628      dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8
~~~

###### 示例五：-finalizerinfo

命令：jmap -finalizerinfo pid

描述：打印等待终结的对象信息

~~~java
[ls@59-37-131-27 order]$ jmap -finalizerinfo 24794
Attaching to process ID 24794, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13
Number of objects pending for finalization: 0
~~~

Number of objects pending for finalization: 0 说明当前F-QUEUE队列中并没有等待Fializer线程执行final

###### 示例六：-dump:<dump-options>

命令：jmap -dump:format=b,file=filename.hprof pid

描述：生成堆快照dump文件

​		以hprof二进制格式转储Java堆到指定filename的文件中。live子选项是可选的。如果指定了live子选项，堆中只有活动的对象会被转储。想要浏览heap dump，你可以使用jhat(Java堆分析工具)读取生成的文件。

> 这个命令执行，JVM会将整个heap的信息dump写入到一个文件，heap如果比较大的话，就会导致这个过程比较耗时，并且执行的过程中为了保证dump的信息是可靠的，所以会暂停应用， 线上系统慎用。

~~~java
[ls@59-37-131-27 order]$ jmap -dump:format=b,file=20190718-order.hprof 24794
Dumping heap to /home/ls/service/order/20190718-order.hprof ...
Heap dump file created
[ls@59-37-131-27 order]$ ll
总用量 1155580
-rw-rw-r--. 1 ls ls      1380 7月  17 10:46 1.txt
-rw-------. 1 ls ls 369093811 7月  18 01:14 20190718-order.hprof
~~~



