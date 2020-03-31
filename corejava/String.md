#### 3月31日早
![@我的宝贝](zzpic6879.jpg)

#### 为什么String是不可变的？
1. java的设计中，String有一个常量池. 如果new一个String时，常量池中已经存在一样的String，则直接返回常量池中的String对象。如果String可变，会影响其他引用的值。
2. 为了缓存hashCode，提高String获取hashCode的效率。如果String对象不可变，则hashCode就可以存储，不需要实时去算。
3. 为了安全。可变字符串，会有反射危机。
4. String 的内部实现，用的是final char[]存储。final决定了不可变。

#### String常量池设计意图是什么？
java中存在大量的String创建，为了节省空间、时间，做一个常量池做缓存。此时要求String不可变、常量池维护String引用，防止GC回收。

#### String常量池在哪里？
方法区中