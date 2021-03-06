**1.sleep()和wait()分别是哪个类的方法，有什么区别？synchronized底层如何实现的？用在代码块和方法上有什么区别？**

答：

​		sleep()是在Thread类的方法，wait()是Object类的方法。

​		最主要的区别是sleep()不会释放锁，而wait()方法会释放锁，使其它线程可以使用同步块或者同步方法。wait()，notify()和notifyAll()只能在同步控制方法或者同步块里使用，而sleep可以在任何地方使用。

​		synchronized加到方法当中，表示整个方法都需要同步。synchronized加到同步代码块当中，代表这块才需要同步。加入到同步方法当中会影响效率。

**2.一个线程执行互斥代码过程**

答：

一个线程执行互斥代码过程如下：

a.获得同步锁

b.清空工作内存

c.从主内存拷贝对象副本到工作内存

d.执行代码(计算或者输出等)

e.刷新主内存数据

f.释放同步锁

**3.volatile（保证可见性但不能保证原子性）与synchronized（保证可见性和原子性）比较**

答：

​		volatile本质是在告诉jvm当前变量在寄存器中的值是不确定的，需要从主存中读取，synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。

​		volatile仅能使用在变量级，synchronized则可以使用在变量，方法。

​		volatile仅能实现变量的修改可见性，但不具备原子特性，而synchronized则可以保证对变量修改的可见性和原子性。

​		volatile不会造成线程的阻塞，而synchronized可能会造成线程的阻塞。

​		volatile标记的变量不会被编译器优化，而synchronized标记的变量可以被编译器优化（指令重排序相关）

**4.什么是线程池？如果让你设计一个动态大小的线程池，如何设计，应该有哪些方法？**

答：

​		线程池是指在初始化一个多线程应用程序过程中创建一个线程集合，然后需要在执行新的任务时重用这些线程而不是重新创建一个线程。线程池中线程的数量通常取决于应用程序的需求。

​		为什么要用线程池：

1）、降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁的消耗

2）、提高响应速度。当任务到达时不必要去创建线程才去执行，而是能够立即执行

3）、提供线程的可管理型。都在线程池中管理

​		设计线程池：

1）、需要设置corePoolSize核心池的大小

2）、需要设置线程池中最大线程数maximumPoolSize

3）、需要设置没有任务执行时最多保持多久时间会终止，默认情况下，只有当线程池中的线程数大于核心线程数时，这个时间才会起作用

4）、保持持续时间的单位，可以是毫秒，秒，分等单位

5）、阻塞队列，用来存储等待执行的任务

6）、线程工厂threadFactory，主要用来创建线程

7）、还需要当拒绝任务时的策略

**5.newFixedThreadPool此种线程池如果线程数达到最大值后会怎么办，底层原理。**

答：

​		Executors.newFixedThreadPool(5)是创建一个固定大小线程的线程池。超出最大线程数的任务会在LinkedBlockingQueue队列中等待。LinkedBlockingQueue队列的长度大小是Integer - 1;如果线程池的任务缓存队列已满并且池中的线程数目达到最大值，如果还有任务到来就会采取任务拒绝策略。通常有以下策略：AbortPolicy（丢弃任务并抛出RejectedExecutionException异常），DiscardPolicy（丢弃任务，不抛出异常），DiscardOldestPolicy（丢弃队列最前面任务）。

**6.场景：在一个主线程中，要求有大量(很多很多)子线程执行完之后，主线程才执行完成。多种方式，考虑效率。**

答：

第一种方法：
     while(Thread.activeCount > 1) {
         Thread.yield();    // 暂停当前正在执行的线程对象，并执行其他线程。
     }
     第二种方法：
     利用join();