##### JVM性能调优工具之jps

###### 概述

​		jps 命令类似与 linux 的 ps 命令，但是它只列出系统中所有的 Java 应用程序。 通过 jps 命令可以方便地查看 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息。

​		如果在 linux 中想查看 java 的进程，一般我们都需要 ps -ef | grep java 来获取进程 ID。

​		如果只想获取 Java 程序的进程，可以直接使用 jps 命令来直接查看。

###### jps 用法

~~~java
[ls@59-37-131-27 order]$ jps -help
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
~~~

**参数说明**

-q：只输出进程 ID

-m：输出传入 main 方法的参数

-l：输出完全的包名，应用主类名，jar的完全路径名

-v：输出jvm参数

-V：输出通过flag文件传递到JVM中的参数

###### 示例一：jps

无参数：显示进程的ID和启动类的名称

~~~java
[ls@59-37-131-27 order]$ jps
19009 jar
28418 jar
18114 jar
31428 jar
15429 jar
16586 OAPServerStartUp
32461 jar
22928 OAPServerStartUp
5909 jar
12824 jar
24794 jar
26717 jar
9502 jar
8864 jar
1571 jar
15972 Elasticsearch
20392 Bootstrap
16683 skywalking-webapp.jar
943 jar
12401 skywalking-webapp.jar
29425 jar
17649 jar
30770 jar
4722 jar
32186 Jps
32571 jar
32315 jar
11132 jar
13566 Bootstrap
[ls@59-37-131-27 order]$ 
~~~

###### 示例二：jps -q

参数 -q 只输出进程ID，而不显示出启动类的名称

~~~java
[ls@59-37-131-27 order]$ jps -q
896
19009
28418
18114
31428
15429
16586
32461
22928
5909
12824
24794
26717
9502
8864
1571
15972
20392
16683
943
12401
29425
17649
30770
4722
32571
32315
11132
13566
[ls@59-37-131-27 order]$ 
~~~

###### 示例三：jps -m

参数 -m 可以输出传递给Java进程（main方法）的参数。

~~~java
[ls@59-37-131-27 order]$ jps -m
19009 jar
28418 jar
18114 jar
31428 jar
15429 jar
16586 OAPServerStartUp
32461 jar
22928 OAPServerStartUp
5909 jar
12824 jar
24794 jar
26717 jar
9502 jar
8864 jar
1571 jar
15972 Elasticsearch
20392 Bootstrap start
16683 skywalking-webapp.jar --spring.config.location=/home/ls/hujh/skywalking/skywalking-test/webapp/webapp.yml --logging.file=/home/ls/hujh/skywalking/skywalking-test/logs/webapp.log
2220 Jps -m
943 jar
12401 skywalking-webapp.jar --spring.config.location=/home/ls/hujh/skywalking/skywalking-prod/webapp/webapp.yml --logging.file=/home/ls/hujh/skywalking/skywalking-prod/logs/webapp.log
29425 jar
17649 jar
30770 jar
4722 jar
32571 jar
32315 jar
11132 jar
13566 Bootstrap start
[ls@59-37-131-27 order]$ 
~~~

###### 示例四：jps -l

参数 -l 可以输出主函数的完整路径（类的全路径）。

~~~java
[ls@59-37-131-27 order]$ jps -l
19009 ls-shortmsg-srv.jar
28418 ls-message-srv.jar
18114 ls-service-file.jar
31428 ls-business-srv.jar
15429 ls-user-boot-0.0.1-SNAPSHOT.jar
16586 org.apache.skywalking.oap.server.starter.OAPServerStartUp
32461 ls-settlement-srv.jar
2573 sun.tools.jps.Jps
22928 org.apache.skywalking.oap.server.starter.OAPServerStartUp
5909 ls-workflow-srv.jar
12824 ls-user-srv.jar
24794 ls-order-srv.jar
26717 ls-institution-srv.jar
9502 ls-warn-srv.jar
8864 ls-account-srv.jar
1571 ls-redisSubscriber-app.jar
15972 org.elasticsearch.bootstrap.Elasticsearch
20392 org.apache.catalina.startup.Bootstrap
16683 /home/ls/hujh/skywalking/skywalking-test/webapp/skywalking-webapp.jar
943 ls-event-srv.jar
12401 /home/ls/hujh/skywalking/skywalking-prod/webapp/skywalking-webapp.jar
29425 ls-cache-srv.jar
17649 ls-userdata-srv.jar
30770 ls-analyze-srv.jar
4722 ls-facade-srv.jar
32571 ls-schedule-app.jar
32315 ls-market-srv.jar
11132 ls-mq-app.jar
13566 org.apache.catalina.startup.Bootstrap
[ls@59-37-131-27 order]$ 
~~~

###### 示例五：jps -v

参数 -v 可以显示传递给Java虚拟机的参数。

~~~java
[ls@59-37-131-27 order]$ jps -v
19009 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
28418 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
18114 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
31428 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
15429 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
16586 OAPServerStartUp -Xms256M -Xmx512M -Doap.logDir=/home/ls/hujh/skywalking/skywalking-test/logs
32461 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
22928 OAPServerStartUp -Xms256M -Xmx512M -Doap.logDir=/home/ls/hujh/skywalking/skywalking-prod/logs
5909 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=ls-workflow-srv
12824 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=ls-user-srv
24794 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=ls-order-srv
26717 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=ls-institution-srv
9502 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
8864 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
1571 jar -Xms512M -Xmx512M -Xmn256M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
15972 Elasticsearch -Xms2g -Xmx2g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/tmp/elasticsearch-4739594460995557590 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:logs/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m -Des.path.home=/home/ls/hujh/skywalking/elasticsearch-6.6.1 -Des.path.conf=/home/ls/hujh/skywalking/elasticsearch-6.6.1/config -Des.distribution.flavor=default -Des.distribution.type=tar
20392 Bootstrap -Djava.util.logging.config.file=/usr/local/appserver/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.net.preferIPv4Stack=true -Xms1596m -Xmx1596m -XX:NewSize=748m -XX:MaxNewSize=748m -XX:PermSize=64m -XX:MaxPermSize=128m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.endorsed.dirs=/usr/local/appserver/endorsed -Dcatalina.base=/usr/local/appserver -Dcatalina.home=/usr/local/appserver -Djava.io.tmpdir=/usr/local/appserver/temp
2986 Jps -Denv.class.path=.:/usr/local/java/jdk1.8.0_121//lib/dt.jar:/usr/local/java/jdk1.8.0_121//lib/tools.jar -Dapplication.home=/usr/local/java/jdk1.8.0_121 -Xms8m
16683 skywalking-webapp.jar -Xms256M -Xmx512M
943 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
12401 skywalking-webapp.jar -Xms256M -Xmx512M
29425 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=ls-cache-srv
17649 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
30770 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
4722 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=ls-facade-srv
32571 jar -Xms512M -Xmx512M -Xmn256M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
32315 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps
11132 jar -Xms512M -Xmx512M -Xmn350M -XX:PermSize=64M -XX:MaxPermSize=128M -XX:CICompilerCount=4 -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCDateStamps -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=ls-mq-app
13566 Bootstrap -Djava.util.logging.config.file=/usr/local/business/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.net.preferIPv4Stack=true -Xms1596m -Xmx1596m -XX:NewSize=748m -XX:MaxNewSize=748m -XX:PermSize=64m -XX:MaxPermSize=128m -Djdk.tls.ephemeralDHKeySize=2048 -javaagent:/home/ls/hujh/agent/skywalking-agent.jar -Dskywalking.agent.service_name=business-web -Djava.endorsed.dirs=/usr/local/business/endorsed -Dcatalina.base=/usr/local/business -Dcatalina.home=/usr/local/business -Djava.io.tmpdir=/usr/local/business/temp
[ls@59-37-131-27 order]$ 
~~~

**注意：一般都会几个参数选项合在一起来用，比如：jps -l -v**

###### 获取远程服务器jps信息

jps 支持查看远程服务上的 jvm 进程信息。如果需要查看其他机器上的 jvm 进程，需要在待查看机器上启动 jstatd 服务。

###### 开启jstatd服务

启动 jstatd 服务，需要有足够的权限。 需要使用 Java 的安全策略分配相应的权限。

这个一般很少生产会用这种情况，先忽略这个。

