##### Java序列化之Externalizable

###### 概述

JDK中除了提供Serializable序列化接口外，还提供了另一个序列化接口Externalizable，使用该接口之后，之前基于Serializable接口的序列化机制就将失效。Externalizable的序列化机制优先级要高于Serializable。

###### Externalizable源码分析

~~~java
public interface Externalizable extends java.io.Serializable {
    /**
     * The object implements the writeExternal method to save its contents
     * by calling the methods of DataOutput for its primitive values or
     * calling the writeObject method of ObjectOutput for objects, strings,
     * and arrays.
     *
     * @serialData Overriding methods should use this tag to describe
     *             the data layout of this Externalizable object.
     *             List the sequence of element types and, if possible,
     *             relate the element to a public/protected field and/or
     *             method of this Externalizable class.
     *
     * @param out the stream to write the object to
     * @exception IOException Includes any I/O exceptions that may occur
     */
    void writeExternal(ObjectOutput out) throws IOException;

    /**
     * The object implements the readExternal method to restore its
     * contents by calling the methods of DataInput for primitive
     * types and readObject for objects, strings and arrays.  The
     * readExternal method must read the values in the same sequence
     * and with the same types as were written by writeExternal.
     *
     * @param in the stream to read data from in order to restore the object
     * @exception IOException if I/O errors occur
     * @exception ClassNotFoundException If the class for an object being
     *              restored cannot be found.
     */
    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
~~~

​		从源码中，我们可以看到Externalizable接口继承了Serializable接口。并定义了两个方法writeExternal()和readExternal()方法。

###### Externalizable示例一

**User类**

~~~java
public class User implements Externalizable {

	private String userName;
	private transient String password;
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
	public void writeExternal(ObjectOutput out) throws IOException {
	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
	}

	@Override
    public String toString() {
        return "User [userName=" + userName + ", password=" + password + ", addr=" + addr + "]";
    }

}
~~~

​		User类中有三个熟悉，其中有一个是transient字段的password，并实现了Externalizable接口，writeExternal和readExternal方法都不填写任何逻辑。

​		然后下面的代码，来看下序列化和反序列化User对象的效果。

~~~java
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
~~~

> 执行结果：
>
> User [userName=null, password=null, addr=null]

###### 示例二

实现Externalizable接口的两个方法的试下，代码如下：

~~~java
public class User implements Externalizable {

	private String userName;
	private transient String password;
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
	public void writeExternal(ObjectOutput out) throws IOException {
		out.writeObject(this.userName);
		out.writeObject(this.password);
		out.writeObject(this.addr);
	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		this.userName = in.readObject().toString();
		this.password = in.readObject().toString();
		this.addr = in.readObject().toString();
	}
	
	@Override
    public String toString() {
        return "User [userName=" + userName + ", password=" + password + ", addr=" + addr + "]";
    }

}
~~~

​		在writeExternal方法中写入userName、password和addr字段，在readExternal方法中读取反序列化的信息并赋值给userName、password和addr字段。

​		执行Test main方法

> 执行结果如下：
>
> User [userName=Alex, password=123456, addr=address test]

###### 结论一

* 实现Externalizable接口后，序列化的细节需要由开发人员自己实现。由于writeExternal与readExternal方法未作任何处理，那么该序列化行为将不会保存/读取任何一个字段。这也就是为什么输出结果中所有字段的值均为空。
* 实现Externalizable接口后，属性字段使用transient和不使用没有任何区别。

###### 示例三

~~~java
public class User implements Externalizable, Serializable {

	private String userName;
	private transient String password;
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
	
	public void writeObject(ObjectOutputStream out) throws IOException {
		out.defaultWriteObject();
		out.writeObject("serializable:" + this.password);
	}
	
	public void readObject(ObjectInputStream in) throws ClassNotFoundException, IOException {
		in.defaultReadObject();
		this.password = (String) in.readObject();
	}
	
	@Override
	public void writeExternal(ObjectOutput out) throws IOException {
		out.writeObject(this.userName);
		out.writeObject("externalizable:" + this.password);
		out.writeObject(this.addr);
	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		this.userName = in.readObject().toString();
		this.password = in.readObject().toString();
		this.addr = in.readObject().toString();
	}
	
	@Override
    public String toString() {
        return "User [userName=" + userName + ", password=" + password + ", addr=" + addr + "]";
    }

}
~~~

​		User类既实现了Serializable接口，又实现了Externalizable接口（其实没啥意义，因为Externalizable已经继承了Serializable接口）。

​		在writeObject和readObject方法中实现了序列化和反序列化。

​		在writeExternal和readExternal方法中也实现了序列化和反序列化。

​		当执行Test.main()方法，执行的结果如下：

> User [userName=Alex, password=externalizable:123456, addr=address test]
>
> 可以看出，执行的是 Externalizable ，而不是 Serializable 。

###### 结论二

**Externalizable序列化的优先级比Serializable的优先级高。**

###### 示例四

~~~java
public class User implements Externalizable {

	private String userName;
	private transient String password;
	private String addr;
	
	public User(String userName, String password, String addr) {
		super();
		this.userName = userName;
		this.password = password;
		this.addr = addr;
	}

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
	public void writeExternal(ObjectOutput out) throws IOException {
		out.writeObject(this.userName);
		out.writeObject("externalizable:" + this.password);
		out.writeObject(this.addr);
	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		this.userName = in.readObject().toString();
		this.password = in.readObject().toString();
		this.addr = in.readObject().toString();
	}
	
	@Override
    public String toString() {
        return "User [userName=" + userName + ", password=" + password + ", addr=" + addr + "]";
    }

}
~~~

​		User类实现了Externalizable接口，并且有一个带参数的构造方法。

~~~java
public static void main(String[] args) throws Exception {
    File file = new File("E:\\serializable\\user.txt");
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
    User user1 = new User("Alex", "123456", "address test");
    //		user1.setUserName("Alex");
    //		user1.setPassword("123456");
    //		user1.setAddr("address test");
    oos.writeObject(user1);
    
    System.out.println("-------序列化成功");

    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
    User user2 = (User) ois.readObject();
    System.out.println(user2);
}
~~~

​		执行Test.main()方法，执行结果如下：

> -------序列化成功
>
> Exception in thread "main" java.io.InvalidClassException: ***.***.User; no valid constructor
> 	at java.io.ObjectStreamClass$ExceptionInfo.newInvalidClassException(ObjectStreamClass.java:157)
> 	at java.io.ObjectStreamClass.checkDeserialize(ObjectStreamClass.java:862)
> 	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2038)
> 	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1568)
> 	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:428)

​		发现报错了，序列化成功了，而没有反序列化成功。提示没有有效的构造方法。这是因为使用Externalizable进行反序列化时，需要有默认的构造方法，通过反射先创建出该类的实例，然后再把解析后的属性值，通过反射赋值。

###### 结论

​		使用Externalizable进行序列化时，必须要有默认的构造方法，而Serializable可以没有默认的构造方法。

















