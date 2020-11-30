InitializingBean接口为bean提供了初始化方法的方式，它只包括afterPropertiesSet方法，凡是继承该接口的类，在初始化bean的时候都会执行该方法。

```
import org.springframework.beans.factory.InitializingBean;
public class TestInitializingBean implements InitializingBean{
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("ceshi InitializingBean");        
    }
    public void testInit(){
        System.out.println("ceshi init-method");        
    }
}

配置文件
```
<bean id="testInitializingBean" class="com.TestInitializingBean" ></bean>

Main函数如下
```
public class Main {
    public static void main(String[] args){
        ApplicationContext context = new FileSystemXmlApplicationContext("/src/main/java/com/beans.xml");
    }
}

测试结果为：
```
ceshi InitializingBean

这说明在spring初始化bean的时候，如果bean实现了InitializingBean接口，会自动调用afterPropertiesSet方法。

那么问题来了，在配置bean的时候使用init-method配置也可以为bean配置初始化方法，那这两个哪个会先执行呢，接下来测试一下，修改配置文件，加上init-method:
```
<bean id="testInitializingBean" class="com.TestInitializingBean" init-method="testInit"></bean>

运行程序，得出结果：
```
ceshi InitializingBean
ceshi init-method

从结果可以看出，在Spring初始化bean的时候，如果该bean实现了InitializingBean接口，并且同时在配置文件中指定了init-method，系统则是先调用afterPropertieSet()方法，然后再调用init-method中指定的方法。

总结：
1、Spring为bean提供了两种初始化bean的方式，实现InitializingBean接口，实现afterPropertiesSet方法，或者在配置文件中通过init-method指定，两种方式可以同时使用。

2、实现InitializingBean接口是直接调用afterPropertiesSet方法，比通过反射调用init-method指定的方法效率要高一点，但是init-method方式消除了对spring的依赖。

3、如果调用afterPropertiesSet方法时出错，则不调用init-method指定的方法。