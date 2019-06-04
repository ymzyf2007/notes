​		作为一个Java Web开发人员，很少也不需要去处理线程，因为服务器已经帮我们处理好了。

​		当我们查看JDK API的时候，总会发现一些类说明写着，线程安全或者线程不安全，比如说StringBuilder中，有这么一句，“将StringBuilder的实例用于多个线程是不安全的。如果需要这样的同步，则建议使用StringBuffer。”，那么下面手动创建一个线程不安全的类，然后在多线程中使用这个类，看看有什么效果。

~~~java
class Count {
    private int num;

    public void count() {
        for(int i = 1; i <= 100; i++) {
            num += i;
        }
        System.out.println(Thread.currentThread().getName() + "-" + num);
    }
}
~~~

​		在这个类中的count方法是计算1一直加到100的和，并输出当前线程名和总和，我们期望的是每个线程都会输出5050。

~~~java
public class UnSafeThreadTest {

    public static void main(String[] args) {
        Runnable runnable = new Runnable() {
            Count c = new Count();
            @Override
            public void run() {
                c.count();
            }
        };

        for(int i = 0; i < 10; i++) {
            new Thread(runnable).start();
        }
    }

}
~~~

​		这里启动了10个线程，看一下输出结果：

~~~java
Thread-1-5050
Thread-0-15150
Thread-3-15150
Thread-5-20200
Thread-7-25250
Thread-9-30300
Thread-4-35350
Thread-6-40400
Thread-8-45603
Thread-2-50500
~~~

​		只有Thread-1线程输出的结果是我们期望的，而输出的是每次都累加的，这里累加的原因以后会说明，那么要想得到我们期望的结果，有几种解决方案：

1. 将Count中num变成count方法的局部变量；
2. 将线程类成员变量拿到run方法中，这时count引用是线程内的局部变量；

​		上述测试，我们发现，存在成员变量的类用于多线程时是不安全的，不安全体现在这个成员变量可能发生非原子性的操作，而变量定义在方法内也就是局部变量是线程安全的。想想在使用struts1时，不推荐创建成员变量，因为action是单例的，如果创建了成员变量，就会存在线程不安全的隐患，而struts2是每一次请求都会创建一个action，就不用考虑线程安全的问题。所以，日常开发中，通常需要考虑成员变量或者说全局变量在多线程环境下，是否会引发一些问题。

