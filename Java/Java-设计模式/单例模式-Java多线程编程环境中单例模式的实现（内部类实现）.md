#### 单例模式-Java多线程编程环境中单例模式的实现（内部类实现）

​        在开发中，如果某个实例的创建需要消耗很多系统资源，那么我们通常会使用惰性加载机制，也就是说只有当使用这个实例的时候才会创建这个实例，这个好处在单例模式中得到了广泛应用。这个机制在single-threaded环境下的实现非常简单，然而在muti-threaded环境下却存在隐患。	

​		通常当我们设计一个单例类的时候，会在类的内部构造这个类（通过构造函数，或者在定义处直接创建），并对外提供一个static getInstance方法提供获取单例对象的途径。例如：

```public class Singleton     
{     
    private static Singleton instance = new Singleton();     
    private Singleton() {     
        …     
    }     
    public static Singleton getInstance() {     
        return instance;
    }     
}
```

​		这样的代码缺点是：第一次加载类的时候会连带着创建Singleton实例，这样的结果与我们所期望的不同，因为创建实例的时候可能并不是我们需要这个实例的时候。同时如果这个Singleton实例的创建非常消耗系统资源，而应用始终都没有使用Singleton实例，那么创建Singleton消耗的系统资源就白白浪费了。

​		为了避免这种情况，我们通常使用惰性加载机制，也就是在使用的时候才去创建。惰性加载代码如下：

```public class Singleton {     
    private static Singleton instance = null;     
    private Singleton() {     
        …     
    }     
    public static Singleton getInstance(){     
        if(instance == null)     
            instance = new Singleton(); 
        return instance;       
    }
}
```

​		这样，当我们第一次调用Singleton.getInstance()的时候，这个单例才被创建，而以后再次调用的时候仅仅返回这个单例就可以了。

​		但是，惰性加载在多线程中的问题：

如果两个线程A和B同时执行了该方法，然后以如下方式执行：

1、A进入if判断，此时instance为null，因此进入if内

2、B进入if判断，此时A还没有创建instance，因此instance也为null，因此B也进入if内

3、A创建了一个instance并返回

4、 B也创建了一个instance并返回

​		此时问题出现了，我们的单例被创建了两次，而这并不是我们所期望的。

​		所以我们可以使用Class锁机制给getInstance方法加上一个synchronized关键字，这样每次只允许一个线程调getInstance方法：

```public static synchronized Singleton getInstance() {     
    if(instance == null)     
        instance = new Singleton();       
    return instance;       
}
```

​		这种解决办法的确可以防止错误的出现，但是它却很影响性能：每次调用getInstance方法的时候都是必须获得对象的锁，而实际上，当单例实例被创建以后，其后的请求没有必要再使用互斥机制了。

​		曾经有人为了解决以上问题，提出了double-checked的方案

```public static Singleton getInstance() {     
    if(instance == null)     
        synchronized(Singleton.class) {     
            if(instance == null)     
                instance = new Singleton();     
        }     
    return instance;       
}
```

​		让我们来看一下这个代码是如何工作的：首先当一个线程发出请求后，会先检查instance是否为null，如果不是则直接返回其内容，这样就避免了synchronized块所需要花费的资源。其次，即使上面提到的情况发生了，两个线程同时进入了第一个if判断，那么它们也必须按照顺序执行synchronized块中的代码，第一个进入代码块的线程会创建一个新的Singleton实例，而后续的线程则因为无法判断if判断，而不会创建多余的实例。

​		上述描述似乎已经解决了我们面临的所有问题，但是实际上，从JVM的角度讲，这些代码仍然可能发生错误。

​		对JVM而言，它执行的是一个个Java指令。在Java指令中创建对象和赋值操作是分开进行的，也就是说instance = new Singleton();语句是分两步执行的。但是JVM并不保证这两个操作的先后顺序，也就是说有可能JVM会为新的Singleton实例分配空间，然后直接赋值给instance成员，然后再去初始化这个Singleton实例。这样就使出错成为了可能。我们仍然以A、B两个线程为例：

1、A、B线程同时进入了第一个if判断

2、A首先进入synchronized块，由于instance为null，所以它执行instance = new Singleton();

3、由于JVM内部的优化机制（指令重排序），JVM先画出了一些分配给Singleton实例的空白内存，并赋值给instance成员（注意此时JVM没有开始初始化这个实例），然后A离开了synchronized块。

4、B进入synchronized块，由于instance此时不是null，因此它马上离开了synchronized块并将结果返回给调用该方法的程序。

5、此时B线程打算使用Singleton实例，却发现它没有被初始化，于是错误发生了。

​		为了实现慢加载，并且不希望每次调用getInstance时都必须互斥执行，最好并且最方便的解决办法如下：（通过内部类实现多线程环境下的单例模式）

public class Singleton {     
    private Singleton() {      
    }

	private static class SingletonContainer {     
		private static Singleton instance = new Singleton();     
	}
	
	public static Singleton getInstance() {     
		return SingletonContainer.instance;     
	}
}

​		JVM内部的机制能够保证当一个类被加载的时候，这个类的加载过程是线程互斥的。这样当我们第一次调用getInstance的时候，JVM能够帮我们保证instance只被创建一次，并且会保证把赋值给instance的内存初始化完毕，这样我们就不用担心上面的问题了。此外该方法也只会在第一次调用的时候使用互斥机制，这样也解决了低效问题。最后instance是在第一次加载SingletonContainer类时被创建的，而SingletonContainer类则在调用getInstance方法的时候才会被加载，因此也实现了惰性加载。



