~~~java
/**
 * 初始化顺序
 * 1.父类静态变量和父类静态代码块[如果同时有父类静态变量和父类静态代码块则按照代码顺序初始化]
 * 2.子类静态变量和子类静态代码块
 * 3.父类非静态变量和父类非静态代码块
 * 4.父类构造器
 * 5.子类非静态变量和子类非静态代码块
 * 6.子类构造器
 * 
 * 输出结果：
 * Static A
 * Static B
 * I am a Class
 * Hello A
 * I am b Class
 * Hello B

 * 
 */
public class HelloB extends HelloA {

	public HelloB() {
		System.out.println("Hello B");
	}
	
	
	static {
		System.out.println("Static B");
	}
	
	public static void main(String[] args) {
		new HelloB();
	}
	
	{
		System.out.println("I am b Class");
	}
	
}
~~~

~~~java
public class HelloA {
	
	public HelloA() {
		System.out.println("Hello A");
	}
	
	{
		System.out.println("I am a Class");
	}
	
	static {
		System.out.println("Static A");
	}

}
~~~

