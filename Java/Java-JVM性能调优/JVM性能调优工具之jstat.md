##### JVM性能调优工具之jstat

###### 概述

​		Jstat是JDK自带的一个轻量级小工具。全称“Java Virtual Machine statistics monitoring tool”，它位于java的bin目录下，主要利用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对**Heap size**和**垃圾回收状况**的监控。

###### jstat 用法

~~~java
[ls@59-37-131-27 order]$ jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
~~~

* option：参数选项
* -t：可以在打印的列加上Timestamp列，用于显示系统运行的时间
* -h：可以在周期性数据数据的时候，可以在指定输出多少行以后输出一次表头
* vmid：Virtual Machine ID（ 进程的 pid）
* interval：执行每次的间隔时间，单位为毫秒
* count：用于指定输出多少次记录，缺省则会一直打印

~~~java
[ls@59-37-131-27 order]$ jstat -options
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
~~~

**option 可以从下面参数中选择**

* -calss 显示ClassLoad的相关信息；
* -compiler 显示JIT编译的相关信息；
* -gc 显示和gc相关的堆信息；
* -gccapaticy 显示各个代的容量以及使用情况；
* -gcmetacapacity 显示metaspace的大小
* -gcnew 显示新生代信息；
* -gcnewcapacity 显示新生代大小和使用情况；
* -gcold 显示老年代和永久代的信息；
* -gcoldcapacity 显示老年代的大小；
* -gcutil 显示垃圾收集信息；
* -gccause 显示垃圾回收的相关信息（通-gcutil）,同时显示最后一次或当前正在发生的垃圾回收的诱因；
* -printcompilation 输出JIT编译的方法信息；

###### 示例一：-class

显示加载class的数量，及所占空间等信息。

~~~java
[ls@59-37-131-27 order]$ jstat -class 24794
Loaded  Bytes  Unloaded  Bytes     Time   
  9468 17711.6        0     0.0      20.22
~~~

* Loaded：已经装载的类的数量
* Bytes：装载类所占用的字节数
* Unloaded：已经卸载类的数量
* Bytes：卸载类的字节数
* Time：装载和卸载类所花费的时间

###### 示例二：-compiler

显示VM实时编译(JIT)的数量等信息。

~~~java
[ls@59-37-131-27 order]$ jstat -compiler 24794
Compiled Failed Invalid   Time   FailedType FailedMethod
   10854      1       0    55.70          1 sun/misc/URLClassPath$JarLoader getResource
~~~

* Compiled：编译任务执行数量
* Failed：编译任务执行失败数量
* Invalid ：编译任务执行失效数量
* Time ：编译任务消耗时间
* FailedType：最后一个编译失败任务的类型
* FailedMethod：最后一个编译失败任务所在的类及方法

###### 示例三：-gc

显示gc相关的堆信息，查看gc的次数，及时间。

~~~java
[ls@59-37-131-27 order]$ jstat -gc 24794
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
20480.0 19968.0 14328.8  0.0   317952.0 178260.2  165888.0   24336.8   55552.0 54183.1 6912.0 6635.7     10    0.344   2      0.151    0.494
~~~

* S0C：年轻代中第一个survivor（幸存区）的容量 （字节）
* S1C：年轻代中第二个survivor（幸存区）的容量 （字节）
* S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
* S1U：年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
* EC：年轻代中Eden（伊甸园）的容量 (字节)
* EU：年轻代中Eden（伊甸园）目前已使用空间 (字节)
* OC：老年代的容量(字节)
* OU：老年代目前已使用空间 (字节)
* MC：metaspace(元空间)的容量 (字节)
* MU：metaspace(元空间)目前已使用空间 (字节)
* YGC：从应用程序启动到采样时年轻代中gc次数
* YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)
* FGC：从应用程序启动到采样时老年代(全gc)gc次数
* FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
* GCT：从应用程序启动到采样时gc用的总时间(s)

###### 示例四：-gccapacity

可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小

~~~java
[ls@59-37-131-27 order]$ jstat -gccapacity 24794
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC 
358400.0 358400.0 358400.0 19968.0 19968.0 318464.0   165888.0   165888.0   165888.0   165888.0      0.0 1099776.0  56576.0      0.0 1048576.0   6912.0     12     2
~~~

* NGCMN：年轻代(young)中初始化(最小)的大小(字节)
* NGCMX：年轻代(young)的最大容量 (字节)
* NGC：年轻代(young)中当前的容量 (字节)
* S0C：年轻代中第一个survivor（幸存区）的容量 (字节)
* S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
* EC：年轻代中Eden（伊甸园）的容量 (字节)
* OGCMN：老年代中初始化(最小)的大小 (字节)
* OGCMX：老年代的最大容量(字节)
* OGC：老年代当前新生成的容量 (字节)
* OC：老年代的容量 (字节)
* MCMN：metaspace(元空间)中初始化(最小)的大小 (字节)
* MCMX：metaspace(元空间)的最大容量 (字节)
* MC：metaspace(元空间)当前新生成的容量 (字节)
* CCSMN：最小压缩类空间大小
* CCSMX：最大压缩类空间大小
* CCSC：当前压缩类空间大小
* YGC：从应用程序启动到采样时年轻代中gc次数
* FGC：从应用程序启动到采样时老年代(全gc)gc次数

###### 示例五：-gcmetacapacity

metaspace 中对象的信息及其占用量。

~~~java
[ls@59-37-131-27 order]$ jstat -gcmetacapacity 24794
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT   
       0.0  1099776.0    57216.0        0.0  1048576.0     7040.0    18     2    0.151    0.996
~~~

* MCMN：最小元数据容量
* MCMX：最大元数据容量
* MC：当前元数据空间大小
* CCSMN：最小压缩类空间大小
* CCSMX：最大压缩类空间大小
* CCSC：当前压缩类空间大小
* YGC：从应用程序启动到采样时年轻代中gc次数
* FGC：从应用程序启动到采样时老年代(全gc)gc次数
* FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
* GCT：从应用程序启动到采样时gc用的总时间(s)

###### 示例六：-gcnew

年轻代对象的信息。

~~~java
[ls@59-37-131-27 order]$ jstat -gcnew 24794
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT  
25088.0 26624.0 22824.8    0.0 15  15 26624.0 305152.0 254838.3     18    0.845
~~~

* S0C：年轻代中第一个survivor（幸存区）的容量 (字节)
* S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
* S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
* S1U：年轻代中第二L个survivor（幸存区）目前已使用空间 (字节)
* TT：持有次数限制
* MTT：最大持有次数限制
* DSS：期望的幸存区大小
* EC：年轻代中Eden（伊甸园）的容量 (字节)
* EU：年轻代中Eden（伊甸园）目前已使用空间 (字节)
* YGC：从应用程序启动到采样时年轻代中gc次数
* YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)

###### 示例七：-gcnewcapacity

年轻代对象的信息及其占用量

~~~java
[ls@59-37-131-27 order]$ jstat -gcnewcapacity 24794
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC 
  358400.0   358400.0   358400.0 119296.0  25088.0 119296.0  26624.0   357376.0   305152.0    18     2
~~~

* NGCMN：年轻代(young)中初始化(最小)的大小(字节)
* NGCMX：年轻代(young)的最大容量 (字节)
* NGC：年轻代(young)中当前的容量 (字节)
* S0CMX：年轻代中第一个survivor（幸存区）的最大容量 (字节)
* S0C：年轻代中第一个survivor（幸存区）的容量 (字节)
* S1CMX：年轻代中第二个survivor（幸存区）的最大容量 (字节)
* S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
* ECMX：年轻代中Eden（伊甸园）的最大容量 (字节)
* EC：年轻代中Eden（伊甸园）的容量 (字节)
* YGC：从应用程序启动到采样时年轻代中gc次数
* FGC：从应用程序启动到采样时old代(全gc)gc次数

###### 示例八：-gcold

old代对象的信息

~~~java
[ls@59-37-131-27 order]$ jstat -gcold 24794
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT   
 57216.0  55618.7   7040.0   6675.0    165888.0     24408.8     19     2    0.151    1.061
~~~

* MC：metaspace(元空间)的容量 (字节)
* MU：metaspace(元空间)目前已使用空间 (字节)
* CCSC：压缩类空间大小
* CCSU：压缩类空间使用大小
* OC：Old代的容量 (字节)
* OU：Old代目前已使用空间 (字节)
* YGC：从应用程序启动到采样时年轻代中gc次数
* FGC：从应用程序启动到采样时old代(全gc)gc次数
* FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
* GCT：从应用程序启动到采样时gc用的总时间(s)

###### 示例九：-gcoldcapacity

old代对象的信息及其占用量

~~~java
[ls@59-37-131-27 order]$ jstat -gcoldcapacity 24794
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT   
   165888.0    165888.0    165888.0    165888.0    19     2    0.151    1.061
~~~

* OGCMN：老年代中初始化(最小)的大小 (字节)
* OGCMX：老年代的最大容量(字节)
* OGC：老年代当前新生成的容量 (字节)
* OC：老年代的容量 (字节)
* YGC：从应用程序启动到采样时年轻代中gc次数
* FGC：从应用程序启动到采样时old代(全gc)gc次数
* FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
* GCT：从应用程序启动到采样时gc用的总时间(s)

###### 示例十：-gcutil

统计gc信息

~~~java
[ls@59-37-131-27 order]$ jstat -gcutil 24794
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.00  99.57  25.92  14.71  97.21  94.82     19    0.910     2    0.151    1.061
[ls@59-37-131-27 order]$ jstat -gcutil 24794 1000 10
~~~

* S0：年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
* S1：年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
* E：年轻代中Eden（伊甸园）已使用的占当前容量百分比
* O：old代已使用的占当前容量百分比
* M：perm代已使用的占当前容量百分比
* YGC：从应用程序启动到采样时年轻代中gc次数
* YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)
* FGC：从应用程序启动到采样时old代(全gc)gc次数
* FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
* GCT：从应用程序启动到采样时gc用的总时间(s)

###### 示例十一：-gccause

显示垃圾回收的相关信息（通-gcutil）,同时显示最后一次或当前正在发生的垃圾回收的诱因。

~~~java
[ls@59-37-131-27 order]$ jstat -gccause 24794
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
  0.00  99.57  37.82  14.71  97.21  94.82     19    0.910     2    0.151    1.061 Allocation Failure   No GC   
~~~

* LGCC：最后一次GC原因
* GCC：当前GC原因（No GC 为当前没有执行GC）

###### 示例十二：-printcompilation

当前VM执行的信息

~~~java
[ls@59-37-131-27 order]$ jstat -printcompilation 24794
Compiled  Size  Type Method
   12336    213    1 org/apache/zookeeper/server/ByteBufferInputStream read
~~~

* Compiled：编译任务的数目
* Size：方法生成的字节码的大小
* Type：编译类型
* Method：类名和方法名用来标识编译的方法。类名使用/做为一个命名空间分隔符。方法名是给定类中的方法。上述格式是由-XX:+PrintComplation选项进行设置的