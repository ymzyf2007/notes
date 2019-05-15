#### HashSet实现原理

*    HashSet概述

  ​		HashSet实现Set接口，由哈希表支持。它不保证set的迭代顺序：特别是它不保证该顺序恒久不变。此类允许用null元素。HashSet中不允许有重复元素，这是因为HashSet是根据HashMap实现的，HashSet中的元素都存放在HashMap的key上面，而value中的值都是统一的一个private static final Object PRESENT = new Object();HashSet跟HashMap一样，都是一个存放链表的数组。

  ​		HashSet中的add(E e)方法调用的是底层HashMap中的put(K key, V value)方法，而如果是在HashMap中调用put，首先会判断key是否存在，如果key存在则修改value值，如果key不存在则插入这个key-value。而在set中，因为value值没有用，也就不存在修改value值的说法，因此往HashSet中添加元素，首先判断元素（也就是key）是否存在，如果不存在则插入，如果存在则不插入，这样HashSet中就不存在重复值。

* HashSet的实现

  ​	对于HashSet而言，它是基于HashMap实现的，HashSet底层使用HashMap来保存所有元素，更确切的来说，HashSet中的元素，只是存放在底层的HashMap的key上，而value使用一个static final的Object对象标识。因此HashSet的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成。注意对于HashSet中保存的对象，请注意正确重写其equals和hashCode方法，以保证放入的对象的唯一性。

* HashSet源码