##### 一、强引用（StrongReference）

​		强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。如下：

~~~java
Object o = new Object();      // 强引用
~~~

​		当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。如果不适用时，要通过如下方式来弱化引用，如下：

~~~java
o = null; // 帮助垃圾收集器回收此对象
~~~

​		显示地设置o为null，或超出对象的生命周期范围，则GC认为该对象不存在引用，这时就可以回收这个对象。具体什么时候收集这要取决于GC的算法。

​		举例：

~~~java
public void test() {
Object o = new Object();
... // 省略其他操作
}
~~~

​		在一个方法的内部有一个强引用，这个引用保存在栈中，而真正的引用内容（Object）保存在堆中。当这个方法运行完成后就会退出方法栈，则引用内容的引用不存在，这个Object就会被回收。

​		但是如果这个引用o是全局的变量时，就需要在不用这个对象时赋值为null，因为强引用不会被垃圾回收。

​		强引用在实际中有非常重要的用处，举个ArrayList的实现源代码：

~~~java
private transient Object[] elementData;

public void clear() {

modCount++;

// Let gc do its work

for(int i = 0; i < size; i++)

elementData[i] = null;

size = 0;

}
~~~

​		在ArrayList类中定义了一个私有的变量elementData数组，在调用方法清空数组时可以看到每个数组内容赋值为null。不同于elementData = null。强引用仍然存在，避免在后续调用add()等方法添加元素时进行重新的内存分配。使用clear()方法中释放内存的方法对数组中存放的引用类型特别适用，这样就可以及时释放内存。

##### 二、软引用（SoftReference）

​		如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

~~~java
String str = new String("abc");      // 强引用
SoftReference<String> softRef = new SoftReference<String>(str);      // 软引用
~~~

​		当内存不足时，等价于：

if(JVM.内存不足()) {
     str = null;      // 转换为软引用
     System.gc();      // 垃圾回收器进行回收
}

​		软引用在实际中有重要的应用，例如浏览器的后退按钮，这个后退时显示的网页内容是重新进行请求还是从缓存中取出呢？这就要看具体的实现策略了。

（1）如果一个网页在浏览结束时就进行内容的回收，则按后退查看前面浏览过的页面时，需要重新构建

（2）如果将浏览过的网页存储到内存中会造成内存的大量浪费，甚至会造成内存溢出

​		这时候就可以使用软引用

~~~java
Browser prev = new Browser(); // 获取页面进行浏览
SoftReference sr = new SorftReference(prev);
// 浏览完毕后置为软引用
if(sr.get() != null) {

rev = (Browser) sr.get();
// 还没有被回收器回收，直接获取

} else {

prev = new Browser();
// 由于内存吃紧，所以对软引用的对象回收了

sr = new SoftReference(prev);
// 重新构建

}
~~~

​		这样就很好的解决了实际的问题。软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

##### 三、弱引用（WeakReference）

​		弱引用与软引用的区别在于：具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

~~~java
String str = new String("abc");
WeakReference<String> abcWeakRef = new WeakReference<String>(str);
str = null;
~~~

​		当垃圾回收器进行扫描回收时等价于：

str = null;
System.gc();

​		如果这个对象是偶尔的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你就应该用WeakReference来记住此对象。

​		下面的代码会让str再次变为一个强引用：

String abc = abcWeakRef.get();

##### 四、虚引用（PhantomReference）

​		虚引用，顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收

##### 五、总结

​		Java的4种引用的级别由高到低依次为：

​		**强引用 > 软引用 > 弱引用 > 虚引用**

