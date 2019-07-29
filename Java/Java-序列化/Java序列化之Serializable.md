##### Java序列化之Serializable

###### 概述

**序列化：**就是把对象转化成字节。

**反序列化：**把字节数据转换成对象。

###### 对象序列化应用场景

**1、对象网络传输**

​		例如：在微服务系统中或给第三方提供接口调用时，使用rpc进行调用，一般会把对象转化成字节序列，才能在网络上传输；接收方则需要把字节序列再转化为java对象。

**2、对象保存至文件中**

​		例如：hibernate的二级缓存：把从数据库中查询出来的对象，序列化转存到硬盘中，下次读取的时候，首先从内存中找是否有该对象，如果没有则在二级缓存（硬盘）中去查找。减少数据库的查询次数，提升性能。

**3、tomcat的钝化和活化**

* **tomcat 的session 钝化和活化之 StandarManager**

  ​	当Tomcat服务器关闭或者重启时tomcat服务器会将当前内存中的session对象钝化到服务器文件系统中；

  ​    另一种情况是web应用程序被重新加载时(其实原理也是重启tomcat)，内存中的session对象也会被钝化到服务器的文件系统中，当系统启动时，会把序列化到硬盘上session重新加载到内存中来。这样用户还保持这登录状态，提供系统的可用性。这样，tomcat重启，如果用户在tomcat重启之前登录过，然后在tomcat重启后可以不需要登录（前提是session没过期前，默认是30分钟过期）。

* **tomcat 的session 钝化和活化之 PersistentManager**

  ​	当网站有大量用户访问的时候，服务器会创建大量的session，会占用大量的服务器内存资源，当用户开着浏览器一分钟不操作页面的话建议将session钝化，将session生成文件放在tomcat工作目录下。

###### 1、Java 序列化 Serializable

​		java中只要对象实现了Serializable就可以进行序列化。

~~~java
import java.io.Serializable;

public class User implements Serializable {

	private String userName;
	private String password;
	private String addr;

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getAddr() {
		return addr;
	}

	public void setAddr(String addr) {
		this.addr = addr;
	}
    
    @Override
    public String toString() {
        return "User [userName=" + userName + ", password=" + password + ", addr=" + addr + "]";
    }

}
~~~

**该User类实现了Serializable接口，那么该类应该怎么序列化和反序列化呢？**

###### 2、ObjectOutputStream和ObjectInputStream

Java IO 包中为我们提供了 ObjectInputStream 和 ObjectOutputStream 两个类。
java.io.ObjectOutputStream 类实现类的序列化功能。
java.io.ObjectInputStream 类实现了反序列化功能。

示例如下：

~~~java
public class Test {
	
	public static void main(String[] args) throws Exception {
		File file = new File("E:\\serializable\\user.txt");
		ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
		User user1 = new User();
		user1.setUserName("Alex");
		user1.setPassword("123456");
		user1.setAddr("address test");
		oos.writeObject(user1);
		
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
		User user2 = (User) ois.readObject();
		System.out.println(user2);
	}

}
~~~

> 输出结果如下：
>
> User [userName=Alex, password=123456, addr=address test]

使用 ObjectOutputStream 把 user1 实例序列化到 E:\serializable\user.txt 文件中。
使用 ObjectInputStream 把 E:\serializable\user.txt 文件中的数据反序列化成 user2 实，并打印。

**如果考虑安全问题，我们不想把密码序列化进行保存，那么该怎么做呢？**

###### 3、transient关键字

当某个字段被声明为transient后，默认序列化机制就会忽略该字段。此处将User类中的password字段声明为transient，如下所示：

~~~java
private String userName;
private transient String password;
private String addr;
......
~~~

> 输出结果如下：
>
> User [userName=Alex, password=null, addr=address test]

*************

**当我们把User对象序列化保存到文件中，这时User类的机构添加了一个新字段，那么它能成功反序列化吗？**

###### 4、serialVersionUID作用

User类中添加一个新属性email字段，如下图：

~~~java
private String userName;
private transient String password;
private String addr;
	
private String email;
~~~

然后再执行反序列化

~~~java
public static void main(String[] args) throws Exception {
		File file = new File("E:\\serializable\\user.txt");
//		ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
//		User user1 = new User();
//		user1.setUserName("Alex");
//		user1.setPassword("123456");
//		user1.setAddr("address test");
//		oos.writeObject(user1);
		
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
    User user2 = (User) ois.readObject();
    System.out.println(user2);
}
~~~

> 执行结果如下：
>
> Exception in thread "main" java.io.InvalidClassException: local class incompatible: stream classdesc serialVersionUID = -5497822370803383974, local class serialVersionUID = -4323797862049766566

​		执行反序列化报 java.io.InvalidClassException 异常。这是由于User类修改了，也就是修改后的class不兼容了，处于安全机制考虑，程序抛出了错误，并且拒绝载入。从异常信息中可以看出，它是根据serialVersionUID值进行判断类是否修改过。

​		如果在添加新字段email后，还可以继续加载之前的字段怎么办呢？

​		我们可以再类中添加serialVersionUID属性字段。

~~~java
public class User implements Serializable {

	private static final long serialVersionUID = -4323797862049766566L;
	private String userName;
	private transient String password;
	private String addr;
	
	private String email;
    ......
}
~~~

​		serialVersionUID 的值和报错中的 "stream classdesc serialVersionUID" 的值一样就可以反序列化了。

> 如果类中没有显示的声明 serialVersionUID 属性，那么java编译器会自动为我们生成一个 serialVersionUID（应该是根据属性和方法进行摘要算出来的，方法里面内容变动 serialVersionUID的值会改变。）

​		如果User对象升级版本，修改了结构，而且不想兼容之前的版本，那么只需要修改下serialVersionUID的值就可以了。

> **建议，每个需要序列化的对象，都要添加一个serialVersionUID字段。**

​		**如果需要把设置的transient的字段也需要序列化和反序列化，我们应该怎么办？我们需要对密码加密序列化，反序列化后解密处理，又应该怎么做？**

**readObject和writeObject**

~~~java
private void readObject(ObjectInputStream s) throws Exception {
		s.defaultReadObject();
		this.password = crypto((String) s.readObject());
	}
	
	private void writeObject(ObjectOutputStream s) throws Exception {
		s.defaultWriteObject();
		s.writeObject(crypto(this.password));
	}
	
	/**
     * 简单加密加密解密字符串 加密解密思路：先将字符串变成byte数组，再将数组每位与key做位运算，得到新的数组就是加密或解密后的byte数组.
     * 知识：^ 是java位运算，可以百度了解下，a = b ^ skey 反之也成立，即b = a ^ skey
     * 
     * @param str 解密/加密 字符串
     * @return
     * @throws Exception
     */
    private String crypto(String str) {
        try {
            byte skey = (byte) 88; //密钥
            byte[] bytes = str.getBytes("GBK");

            for (int i = 0; i < bytes.length; i++) {
                bytes[i] = (byte) (bytes[i] ^ skey);
            }
            return new String(bytes, "GBK");
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return null;
    }
~~~

​		我们只需要在当前User类中添加readObject()和writeObject()方法，在writeObject方法中实现对password字段的加密，在readObject方法中实现对password字段的解密，并赋值给User对象即可。

> writeObject()和readObject()可以实现对transient和非transient字段进行序列化和反序列化。

###### ArrayList序列化源码分析

​		我们知道，ArrayList是通过数组进行存储数据的，当数组中元素达到数组的最大容量时，会自动生成一个更大的数组，并复制到更大的数组中。

~~~java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
    ......
}
~~~

​		打开ArrayList源码，我们可以知道，数据是存储在Object[] elementData数组中。该属性是transient关键字修饰的，通过上面代码可以知道，用transient关键字修饰的字段，默认是不能被序列化的。ArrayList如果要实现序列化，那么就必须通过writeObject()和readObject()方法来实现序列化和反序列化，那么他这是多此一举吗？

###### writeObject()方法

~~~java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
~~~

​		通过源码，我们可以看到，ArrayList序列化数组元素时做了优化（注意size）。

​		因为ArrayList的elementData数组大小，不是ArrayList的实际容量，这里只把实际存储在elementData中的数据，进行序列化。这样就减少了序列化的流大小。

~~~java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
~~~



