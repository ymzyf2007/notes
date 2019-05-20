#### JVM内存设置原理和调优

​		JVM主要使用了三种不同类型的内存区域：Permanent Generation space（永久保存区域）、Heap space（堆区域）、Java Stacks（Java栈）。

​		其中永久保存区域主要放Class（类）和Meta的信息，Class第一次被Load的时候放入PermGen space区域，Class需要存储的内容主要包括方法和静态属性。

​		堆区域用来存放Class的实例（即对象）和数组，每次用new创建一个对象实例后，对象实例存储在堆区域中，这部分空间也被JVM的垃圾回收机制管理。

​		Java栈主要保存方法中的基本类型的局部变量，对象的引用，操作栈等。Java程序的每个线程中都有一个独立的堆栈。

​		最容易发生内存溢出的内存空间包括：Permanent Generation space和Heap space。

​		第一种OutOfMemoryError：PermGen space

​		这种情况发生的原因是程序中使用了大量的jar或class，使java虚拟机装载类的空间不够，与Permanent Generation space有关。

​		解决这类问题有以下两种方法：

1、增加java虚拟机中的XX:PermSize和MaxPermSize参数的大小，其中XX:PermSize是初始永久保存区域大小，XX:MaxPermSize是最大永久保存区域大小。

​		在项目中大多数情况下都需要在tomcat6.0的启动文件中catalina.sh或catalina.bat增加下面的一行：

JAVA_OPTS=" -XX:PermSize=64m -XX:MaxPermSize=128m"

2、如果tomcat部署了多个应用，并且这些应用使用了相同的jar，可以将共同的jar移到tomcat的lib下，减少类的重复加载。

​		第二种OutOfMemoryError：Java Heap space

​		发生这种问题的原因是突然创建了太多的对象，虚拟机分配的堆内存空间已经用满了。

​		解决这类问题有两种思路：

1、减少不必要的对象的创建，尽可能不要在循环中重复创建对象。

2、增加Java虚拟机中Xms（初始堆大小）和Xmx（最大堆大小）参数的大小。如：set JAVA_OPTS= -Xms256m -Xmx1024m

3、使用完对象后，把对象设置成null【help gc】

​		第三种OutOfMemoryError：unable to create new native thread

​		这种情况在Java线程个数很多的情况下容易发生。

-Xms：java Heap初始大小， 默认是物理内存的1/64。
-Xmx：ava Heap最大值，不可超过物理内存。
-Xmn：young generation的heap大小，一般设置为Xmx的3、4分之一 。增大年轻代后,将会减小年老代大小，可以根据监控合理设置。
-Xss：每个线程的Stack大小，而最佳值应该是128K,默认值好像是512k。
-XX:PermSize：设定内存的永久保存区初始大小，缺省值为64M。
-XX:MaxPermSize：设定内存的永久保存区最大大小，缺省值为64M。
-XX:SurvivorRatio：Eden区与Survivor区的大小比值,设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10
-XX:+UseParallelGC：F年轻代使用并发收集,而年老代仍旧使用串行收集.
-XX:+UseParNewGC：设置年轻代为并行收集,JDK5.0以上,JVM会根据系统配置自行设置,所无需再设置此值。
-XX:ParallelGCThreads：并行收集器的线程数，值最好配置与处理器数目相等 同样适用于CMS。
-XX:+UseParallelOldGC：年老代垃圾收集方式为并行收集(Parallel Compacting)。
-XX:MaxGCPauseMillis：每次年轻代垃圾回收的最长时间(最大暂停时间)，如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值。
-XX:+ScavengeBeforeFullGC：Full GC前调用YGC,默认是true。
实例如：JAVA_OPTS="-Xms2g -Xmx4g -Xmn1024m -XX:PermSize=320m -XX:MaxPermSize=512m -XX:SurvivorRatio=6″

-verbose:gc：可以看GC的收集情况
在Eclipse中可以通过Run As|Run Configurations|Arguments|VM Arguments进行设置。
使用该命令后输出如下：
[Full GC 1224K->1113K(123584K), 0.0120528 secs]
箭头（->）前后的数据1224K和1113K分别表示垃圾收集GC前后所有存活对象使用的内存容量，说明有1224K–1113K = 111K大小的对象被回收，括号内的数据 123584K为堆内存的总容量，收集所需的实际为 0.0120528秒。
需要注意的是：GC会暂用CPU时间片，会造成程序短暂的停顿。
控制台输出GC信息还可以使用如下命令：
在JVM的启动参数中加入-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime，按照参数的顺序分别输出GC的简要信息，GC的详细信息、GC的时间信息及GC造成的应用暂停的时间。