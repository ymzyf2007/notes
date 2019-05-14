#### ArrayList与LinkedList的普通for循环遍历

​		对于大部分Java程序员朋友们来说，可能平时使用得最多的List就是ArrayList，对于ArrayList的遍历，一般用如下写法：

```java
public static void main(String[] args)  {
    List<Integer> arrayList = new ArrayList<Integer>();
	for (int i = 0; i < 100; i++) {
		arrayList.add(i);
	}
	for (int i = 0; i < 100; i++) {
		System.out.println(arrayList.get(i));
	}
}
```

​		如果以后要用到LinkedList了，可能有些朋友就会用一样的方式去遍历LinkedList了：

```java
public static void main(String[] args) {
    List<Integer> linkedList = new LinkedList<Integer>();

    for (int i = 0; i < 100; i++) {
        linkedList.add(i);
    }
    for (int i = 0; i < 100; i++) {
        System.out.println(linkedList.get(i));
    }
}
```

​		请记住：这是一种非常糟糕的做法。这其实已经不是Java的问题，而是数据结构的问题了，我相信语言从Java换成其他的也都一样。

​		下面对ArrayList和LinkedList的普通for循环效率进行测试以及分析原因。

​	**ArrayList和LinkedList使用普通for循环遍历速度对比：**

```java
public class ListIteratorTest {

    private final static int LIST_SIZE = 100000;

    public static void main(String[] args) {
        List<Integer> arrayList = new ArrayList<Integer>();
        List<Integer> linkedList = new LinkedList<Integer>();

        for (int i = 0; i < LIST_SIZE; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }

        long startTime = System.currentTimeMillis();
        for (int i = 0; i < arrayList.size(); i++) {
            arrayList.get(i);
        }
        System.out.println("ArrayList遍历速度：" + (System.currentTimeMillis() - startTime) + "ms");

        startTime = System.currentTimeMillis();
        for (int i = 0; i < linkedList.size(); i++) {
            linkedList.get(i);
        }
        System.out.println("LinkedList遍历速度：" + (System.currentTimeMillis() - startTime) + "ms");
    }
}
```

运行结果：

ArrayList遍历速度：0ms

LinkedList遍历速度：12859ms

​		从运行结果我们看到，按倍数增大List容量，ArrayList的遍历显得比较稳定，而LinkedList的遍历几乎是爆发式的增长，再测试下去已经没有必要了。

​		下面解释一下产生此现象的原因：

​		ArrayList使用普通for循环遍历快的原因，先看一下ArrayList的get方法源代码：

```java
public E get(int index) {
    RangeCheck(index);
    return (E) elementData[index];
}
```

​		看到ArrayList的get方法只是从数组里面拿一个位置上的元素罢了。我们有结论，ArrayList的get方法的时间复杂度是O(1)，O(1)的意思也就是说时间复杂度是一个常数，和数组的大小并没有关系，只要给定数组的位置，直接就能定位到数据。

​		其实熟悉C、C++或者对指针理解的朋友一定很好理解为什么，我解释一下为什么对数组使用get就快。

​		在计算机底层，数据都是有地址的，就像人有住址一样。假设我写了这么一句代码：

`int[3] ints = {1, 3, 5};`

​		在Java中一个int型数据是4个字节，此时计算机内部做的事情是，在内存空间中找到一块连续的、足以存放3个4字节也就是12字节的数组的内存空间，并返回该内存空间的首地址。比方说该内存空间的首地址是0x00吧，那么那么1就放在0x00~0x03中、3就放在0x04~0x07中、5就放在0x08~0x0B中。

​		这时就很简单了，取ints[1]的时候，计算机就会算出ints[1]的数据是存放在以0x04开头，占据4个字节空间的内存中，因此，计算机会从0x04~0x07这块地址空间中读取数据出来。整个过程，和数组有多大，并没有关系，计算机做的只是算出起始地址-->去该地址中取数据而已，因此我们看到使用普通for循环遍历ArrayList的速度很快，也很稳定。

​	LinkedList使用普通for循环遍历慢的原因，再看一下LinkedList的get方法做了什么：

```java
public E get(int index) {
	return entry(index).element;
}
private Entry<E> entry(int index) {
	if (index < 0 || index >= size)
		throw new IndexOutOfBoundsException("Index: "+index+", Size: "+size);
	Entry<E> e = header;
	if (index < (size >> 1)) {
		for (int i = 0; i <= index; i++)
			e = e.next;
	} else {
		for (int i = size; i > index; i--)
			e = e.previous;
	}
	return e;
}
```

​		由于LinkedList是双向链表，因此index < (size >> 1) 的意思是算出i在一半前还是一半后，一半前正序遍历、一半后倒序遍历，这样会快很多，当然，先不管这个，分析一下为什么使用普通for循环遍历LinkedList会这么慢。

​		原因就在两个for循里面，以前者为例：

1、get(0)，直接拿到0位的Node0的地址，拿到Node0里面的数据

2、get(1)，直接拿到0位的Node0的地址，从0位的Node0中找到下一个1位的Node1的地址，找到Node1，拿到Node1里面的数据

3、get(2)，直接拿到0位的Node0的地址，从0位的Node0中找到下一个1位的Node1的地址，找到Node1，从1位的Node1中找到下一个2位的Node2的地址，找到Node2，拿到Node2里面的数据。

​		后面的以此类推。

​		也就是说，LinkedList在get任何一个位置的数据的时候，都会把前面的数据走一遍。假如我有10个数据，那么将要查询1+2+3+4+5+5+4+3+2+1=30次数据，相比ArrayList，却只需要查询10次数据就行了，随着LinkedList的容量越大，差距会越拉越大。其实使用LinkedList到底要查询多少次数据，大家应该已经很明白了，来算一下：按照前一半算应该是（1 + 0.5N） * 0.5N / 2，后一半算上即乘以2，应该是（1 + 0.5N） * 0.5N = 0.25N2 + 0.5N，忽略低阶项和首项系数，得出结论，LinikedList遍历的时间复杂度为O(N2)，N为LinkedList的容量。

​		时间复杂度有以下经验规则：

​		O(1) < O(log2N) < O(n) < O(N * log2N) < O(N2) < O(N3) < 2N < 3N < N!

​		前四个比较好、中间两个一般、后3个很烂。也就是说O(N2)是相对糟糕的一种时间复杂度了，N大一点，程序就会执行得比较慢。

​		**根据以上的分析，各位Java程序员朋友们，切记一定不要使用普通for循环去遍历LinkedList。使用迭代器或者foreach循环（foreach循环的原理就是迭代器）去遍历LinkedList即可，这种方式是直接按照地址去找数据的，将会大大提升遍历LinkedList的效率。**