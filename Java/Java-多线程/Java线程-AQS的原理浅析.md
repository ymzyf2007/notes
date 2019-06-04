​		AQS的全称为（AbstractQueuedSynchronizer），这个类也是在java.util.concurrent.locks下面。这个类似乎很不容易看懂，因为它仅仅是提供了一系列公共的方法，让子类来调用。那么要理解意思，就得从子类下手，反过来看才容易看懂。

​		这么多子类，我们看那一个？刚刚提到过锁（Lock），我们就从锁开始吧。这里就先以ReentrantLock排它锁为例开始展开讲解如何利用AQS的，然后再简单介绍读写锁的要点（读写锁本身的实现十分复杂，要完全说清楚需要大量的篇幅来说明）。

​		首先来看看ReentrantLock的构造方法，它的构造方法有两个，如下图所示：

~~~java
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = (fair)? new FairSync() : new NonfairSync();
}
~~~

​		很显然，对象中有一个属性叫sync，有两种不同的实现类，默认是“NonfairSync”来实现，而另一个“FairSync”它们都是排它锁的内部类，不论用那一个都能实现排它锁，只是内部可能有点原理上的区别。先以“NonfairSync”类为例，它的lock()方法是如何实现的呢？

~~~java
final void lock() {
    if (compareAndSetState(0, 1))    // 如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数设置为 1。
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);    // 如果该锁被另一个线程保持，则出于线程调度的目的，禁用当前线程，并且在获得锁之前，该线程将一直处于休眠状态，此时锁保持计数被设置为 1。
}
~~~

​		排它锁的lock方法：lock()方法先通过CAS尝试将状态从0修改为1。若直接修改成功，前提条件自然是锁的状态为0，则直接将线程的OWNER修改为当前线程，这是一种理想情况，如果并发粒度设置适当也是一种乐观情况。若上一个动作未成功，则会间接调用了acquire(1)来继续操作，这个acquire(int)方法就是在AbstractQueuedSynchronizer当中了。这个方法表面上看起来简单，但真实情况比较难以看懂，因为第一次看这段代码可能不知道它要做什么！不急，一步一步来分解。

​		首先看tryAcquire(arg)这里的调用（当然传入的参数是1），在默认的“NonfairSync”实现类中，会这样来实现：

~~~java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();    // 获取当前线程
    int c = getState();    // 获取同步状态    0表示未锁
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
~~~

解析：

1、首先获取这个锁的状态，如果状态为0，则尝试设置状态为传入的参数（这里就是1），若设置成功就代表自己获取到了锁，返回true了。状态为0设置1的动作在外部就有做过一次，内部再一次做只是提升概率，而且这样的操作相对锁来讲不占开销。

2、如果状态不是0，则判定当前线程是否为排它锁的Owner，如果是Owner则尝试将状态增加acquires（也就是增加1），如果这个状态值越界，则会抛出异常提示，若没有越界，将状态设置进去后返回true（实现了类似于偏向的功能，可重入，但是无需进一步征用）。【线程从方法A中lock()再进入方法B中的lock()】

3、如果状态不是0，且自身不是owner，则返回false。

​		tryAcquire()方法会返回false【即获取锁失败】，接着就会再调用方法：acquireQueued(addWaiter(Node.EXCLUSIVE), arg)做相关的操作。这个方法的调用的代码更不好懂，需要从里往外看，这里的Node.EXCLUSIVE是节点的类型，看名称应该清楚是排它类型的意思。接着调用addWaiter()来增加一个排它锁类型的节点，这个addWaiter()的代码是这样写的：

~~~java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
~~~

​		这里创建了一个Node的对象，将当前线程和传入的Node.EXCLUSIVE传入，也就是说Node节点理论上包含了这两项信息。代码中的tail是AQS的一个属性，刚开始的时候肯定是为null，也就是不会进入第一层if判定的区域，而直接会进入enq(node)的代码，那么直接来看看enq(node)的代码。

​		看到了tail就应该猜到了AQS是链表吧，没错，而且它还应该有一个head引用来指向链表的头节点，AQS在初始化的时候head、tail都是null，在运行时来回移动。此时，我们最少至少知道AQS是一个基于状态（state）的链表管理方式。

~~~java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            Node h = new Node(); // Dummy header
            h.next = node;
            node.prev = h;
            if (compareAndSetHead(h)) {
                tail = node;
                return h;
            }
        }
        else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
~~~

