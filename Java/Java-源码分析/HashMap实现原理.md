#### HashMap实现原理

​		HashMap其实也是用一个线性数组实现的，所以可以理解为其存储数据的容器是一个线性数组。HashMap实际上是一个**链表散列**的数据结构，即数组和链表的结合体。

1、首先HashMap里面实现一个静态内部类Entry，其重要的属性有key，value，next。从属性key，value我们就能很明显的看出Entry就是HashMap键值对实现的一个基础Bean。HashMap的基础就是一个线性数组，这个数组就是Entry<K,V>[]，Map里面的内容都保存在Entry<K,V>[]里面。

2、既然是线性数组，为什么又能随机存取呢？这个就是上面说的HashMap实际上是一个**链表散列**的数据结构，即数组和链表的结合体。 HashMap的功能是通过**键(key)**能够快速的找到**值**。

下面我们分析下HashMap存数据的基本流程

存储时：

``public V put(K key, V value) {
     if (key == null)
          return putForNullKey(value);
     int hash = hash(key);
     int i = indexFor(hash, table.length);
     for (Entry<K,V> e = table[i]; e != null; e = e.next) {
          Object k;
          if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
               V oldValue = e.value;
               e.value = value;
               e.recordAccess(this);
               return oldValue;
          }
     }
     modCount++;
     addEntry(hash, key, value, i);
     return null;
}``

（1）、当调用put(key, value)时，首先获取key的hashcode，再把hachcode通过以下散列运算得到一个int h。

``h ^= (h >>> 20) ^ (h >>> 12);
return h ^ (h >>> 7) ^ (h >>> 4);``

​		为什么要经过这样的运算呢？这就是HashMap的高明之处。先看个例子，一个十进制数32768(二进制1000 0000 0000 0000)，经过上述公式运算之后的结果是35080(二进制1000 1001 0000 1000)。看出来了吗？或许这样还看不出什么，再举个数字61440(二进制1111 0000 0000 0000)，运算结果是65263(二进制1111 1110 1110 1111)，现在应该很明显了，它的目的是让“1”变的均匀一点，散列的本意就是要尽量均匀分布。那这样有什么意义呢？请看下一步。

（2）、得到h之后，把h与HashMap的承载量（HashMap的默认承载量length是16，可以自动变长。在构造HashMap的时候也可以指定一个长度。这个承载量就是数组Entry<K,V>[]的长度）进行逻辑与运算，即h & (length-1)，这样得到的结果就是一个比length小的正数，我们把这个值叫做index，这个index就是索引将要插入的值在数组中的位置。上面那个算法的意义就是希望能够得出均匀的index。还有一点需要说明，HashMap的键可以为null，它的值是放在数组的第一个位置。

（3）、我们用table[index]表示已经找到元素需要存储的位置。先判断该位置上有没有元素（这个元素是HashMap内部定义的一个类Entry，基本结构包含key，value和指向下一个Entry的next），没有的话就创建一个Entry对象，在table[index]位置上插入，这样插入结束；如果有的话，通过链表的遍历方式去遍历，判断是否已经存在的key（e.hash== hash && ((k = e.key) == key || key.equals(k))），有的话用新的value代替老的value；如果没有，则在table[index]插入改Entry，把原来在table[index]位置上的Entry赋值给新的Entry的next，这样插入结束。

**总结：key->hashcode->h->index->遍历链表->插入**

取值时：

``public V get(Object key) {
     if (key == null)
          return getForNullKey();
     Entry<K,V> entry = getEntry(key);
     return null == entry ? null : entry.getValue();
}``

``final Entry<K,V> getEntry(Object key) {
     if (size == 0) {
          return null;
     }
     int hash = (key == null) ? 0 : hash(key);
     for (Entry<K,V> e = table[indexFor(hash, table.length)];e != null; e = e.next) {
          Object k;
          if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
               return e;
     }
     return null;
}``

**扩展问题**

1、为什么要同时复写equals方法和hashcode方法

​		按照散列函数的定义，如果两个对象相同，即obj1.equals(obj2) = true，则它们的hashcode必须相同；但如果两个对象不同，则它们的hashcode不一定不同。如果两个不同对象的hashcode相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的hashCode方法，能加快哈希表的操作。