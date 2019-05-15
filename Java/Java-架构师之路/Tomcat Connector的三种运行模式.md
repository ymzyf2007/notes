#### Tomcat Connector（Tomcat连接器）有bio、nio和apr三种运行模式

-    bio

  ​	bio（blocking I/O，阻塞式I/O操作），表示Tomcat使用的是传统的Java I/O操作（即java.io包及其子包）。默认的模式，性能最差，没有经过任何优化处理和支持。

- nio

  ​	nio（non-blocking I/O），Java SE 1.4及后续版本提供的一种新的I/O操作方式（即java.nio包及其子包）。Java nio是一个基于缓冲区，并能提供非阻塞I/O操作的Java API。拥有比传统I/O操作（bio）更好的并发运行性能。要让tomcat以nio模式来运行，修改配置文件：tomcat/conf/server.xml

  修改以下内容：

  `<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />`

  修改protocol的值为：org.apache.coyote.http11.Http11NioProtocol

  `<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol " connectionTimeout="20000" redirectPort="8443" />`

  重启tomcat后生效

- apr

  ​	apr（Apache Portable Runtime/Apache可移植运行时库），Tomcat将以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地提高Tomcat对静态文件的处理性能。从操作系统级别来解决异步IO问题，大幅度的提高性能。Tomcat apr也是在Tomcat上运行高并发应用的首选模式。