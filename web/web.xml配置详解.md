##### 一、加载过程

1、启动一个WEB项目的时候，WEB容器会去读取它的配置文件web.xml，读取<listener>和<context-param>两个节点。

2、紧接着，容器创建一个ServletContext（Servlet上下文），这个web项目的所有部分都将共享这个上下文。

3、容器将<context-param>转换为键值对，并交给servletContext。

4、容器创建<listener>中的实例，创建监听器。

##### 二、load-on-startup

​		load-on-startup元素在web应用启动的时候指定了servlet被加载的顺序，它的值必须是一个整数。如果它的值是一个负整数或是这个元素不存在，那么容器会在该servlet被调用的时候才去加载这个servlet。如果值是正整数或零，容器在配置的时候就加载并初始化这个servlet，容器必须保证值小的先被加载。如果值相等，容器可以自动选择先加载谁。

​		例如：在servlet的配置中，<load-on-startup>5</load-on-startup>的含义是：标记容器是否在启动的时候就在加载这个servlet。当值为0或者大于0的时候，表示容器在应用启动的时候就加载并初始化这个servlet；当是一个负数或者没有指定时，则指示容器在该servlet被选择时才加载。正数的值越小，启动该servlet的优先级越高。

##### 三、加载顺序

​		首先可以肯定的是，加载顺序与它们在web.xml文件中的顺序无关。即不会因为filter写在listener的前面而会先加载filter。最终得出的结论就是：

ServletContext(context-param)->listener->filter->servlet

样例：

~~~java
<context-param>
     <param-name>my_param</param-name>
     <param-value>hello</param-value>
</context-param>
~~~

​		在这里设定的参数，可以在servlet中用getServletContext().getInitParameter("my_param ")来获取

~~~java
<listener>
     <listener-class></listener-class>
</listener>
~~~

~~~java
<filter>
    <filter-name>setCharacterEncoding</filter-name>
    <filter-class>com.myTest.setCharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>GB2312</param-value>
    </init-param>
</filter>
~~~

~~~java
<servlet>
    <servlet-name>ShoppingServlet</servlet-name>
    <servlet-class>com.myTest.ShoppingServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ShoppingServlet</servlet-name>
    <url-pattern>/shop/ShoppingServlet</url-pattern>
</servlet-mapping>
~~~

session设置

~~~java
<session-config>
     <session-timeout>30</session-timeout>
</session-config>
~~~

定义欢迎页

~~~java
<welcome-file-list>
     <welcome-file>index.jsp</welcome>
     <welcome-file>index.html</welcome>
</welcome-file-list>
~~~

定义错误页面

~~~java
<error-page>
     <error-code></error-code>     指定错误代码
     <exception-type></exception-type>     指定一个JAVA异常类型
     <location></location>     指定在web站台内的相关资源路径
</error-page>
~~~

比如：

~~~java
<error-page>
    <error-code>404</error-code>
    <location>/error404.jsp</location>
</error-page>
<error-page>
    <exception-type>java.lang.Exception</exception-type>
    <location>/exception.jsp</location>
</error-page>
~~~

