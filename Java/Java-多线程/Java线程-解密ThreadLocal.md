##### 概述

​		相信读者在网上也看了很多关于ThreadLocal的资料，很多博客都这样说：ThreadLocal为解决多线程程序的并发问题提供了一种新的思路；ThreadLocal的目的是为了解决多线程访问资源时的共享问题。如果你也这样认为的，那现在给你10秒钟，清空之前对ThreadLocal的错误的认知！

​		看看JDK中的源码是怎么写的：ThreadLocal类是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。

​		举个例子，我出门需要先坐公交再做地铁，这里的坐公交和坐地铁就好比是同一个线程内的两个函数，我就是一个线程，我要完成这两个函数都需要同一个东西：公交卡（北京公交和地铁都使用公交卡），那么我为了不向这两个函数都传递公交卡这个变量（相当于不是一直带着公交卡上路），我可以这么做：将公交卡事先交给一个机构，当我需要刷卡的时候再向这个机构要公交卡（当然每次拿的都是同一张公交卡）。这样就能达到只要是我(同一个线程)需要公交卡，何时何地都能向这个机构要的目的。

​		有人要说了：你可以将公交卡设置为全局变量啊，这样不是也能何时何地都能取公交卡吗？但是如果有很多个人（很多个线程）呢？大家可不能都使用同一张公交卡吧(我们假设公交卡是实名认证的)，这样不就乱套了嘛。现在明白了吧？这就是ThreadLocal设计的初衷：**提供线程内部的局部变量，在本线程内随时随地可取，隔离其他线程**。

##### ThreadLocal基本操作

1、构造函数

~~~~java
/**
 * Creates a thread local variable.
 * @see #withInitial(java.util.function.Supplier)
 */
public ThreadLocal() {
}
~~~~

2、initialValue函数：initialValue函数用来设置ThreadLocal的初始值。

~~~java
protected T initialValue() {
    return null;
}
~~~

​		该函数在调用get函数的时候会被第一次调用，但是如果一开始就调用了set函数，则该函数不会被调用。通常该函数只会被调用一次，除非手动调用了remove函数之后又调用get函数，这种情况下，get函数中还是会调用initialValue函数。该函数是protected类型的，很显然是建议在子类重载该函数的，所以通常该函数都会以匿名内部类的形式被重载，以指定初始值。

3、get函数：该函数用来获取与当前线程关联的ThreadLocal的值。

~~~java
public T get()
~~~

​		如果当前线程没有该ThreadLocal的值，则调用initialValue函数获取初始值返回。

4、set函数：set函数用来设置当前线程的该ThreadLocal的值。

~~~java
public void set(T value)
~~~

5、remove函数：remove函数用来将当前线程的ThreadLocal绑定的值删除。

~~~java
public void remove()
~~~

代码演示

​		学习了最基本的操作之后，我们用一段代码演示ThreadLocal的用法。

~~~java
public class ThreadLocalTest {

    private static final ThreadLocal<Integer> threadLocalValue = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };

    static class MyThread implements Runnable {
        private int index;

        public MyThread(int index) {
            this.index = index;
        }

        @Override
        public void run() {
            System.out.println("线程" + index + "的初始值threadLocalValue：" + threadLocalValue.get());
            for(int i = 0; i < 10; i++) {
                threadLocalValue.set(threadLocalValue.get() + i);
            }
            System.out.println("线程" + index + "的累加threadLocalValue：" + threadLocalValue.get());
        }
    }

    public static void main(String[] args) {
        for(int i = 0; i < 5; i++) {
            new Thread(new MyThread(i)).start();
        }
    }

}
执行结果为：
线程0的初始值threadLocalValue：0
线程0的累加threadLocalValue：45
线程2的初始值threadLocalValue：0
线程2的累加threadLocalValue：45
线程4的初始值threadLocalValue：0
线程4的累加threadLocalValue：45
线程1的初始值threadLocalValue：0
线程1的累加threadLocalValue：45
线程3的初始值threadLocalValue：0
线程3的累加threadLocalValue：45
~~~

​		可以看到，各个线程的value值都是相互独立的，本线程的累加操作不会影响到其他线程的值，真正达到了线程内部隔离的效果。

**如何实现的**

​		看了基本介绍，也看了最简单的效果演示之后，我们更应该好好研究下ThreadLocal内部的实现原理。如果给你设计，你会怎么设计？相信大部分人会有这样的想法：

​		每个ThreadLocal类创建一个Map，然后用线程的ID作为Map的key，实例对象作为Map的value，这样就能达到各个线程的值隔离的效果。

​		简单解析一下，get方法的流程是这样的：

1、首先获取当前线程

2、根据当前线程获取一个Map

3、如果获取的Map不为空，则在Map中以ThreadLocal的引用作为key来在Map中获取对应的value e，否则转到5

4、如果e不为null，则返回e.value，否则转到5

5、Map为空或者e为空，则通过initialValue函数获取初始值value，然后用ThreadLocal的引用和value作为firstKey和firstValue创建一个新的Map

