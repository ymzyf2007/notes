##### 1、jsp和servlet的区别

答：

1）jsp经编译后就变成了servlet（jsp的本质就是servlet，jvm只能识别java的类，不能识别jsp的代码，web容器就将JSP的代码编译成JVM能够识别的java类）
2）jsp更擅长表现于页面展示，servlet更擅长于逻辑控制
3）servlet中没有内置对象，jsp的内置对象都是必须通过HttpServletRequest对象、HttpServletResponse对象以及HttpServlet对象得到。

##### 2、servlet默认是线程安全的吗？为什么不是线程安全的？

答：

​		servlet不是线程安全的。

​		Servlet体系结构是建立在Java多线程机制之上的，它的生命周期是由Web容器负责。当客户端第一次请求Servlet时，容器会根据web.xml配置文件来实例化这个Servlet类。当有新的客户端请求该Servlet时，一般不会再实例化该Servlet类，也就是是多个线程在使用这个实例。Servlet容器会自动使用线程池等技术来支持系统的运行。