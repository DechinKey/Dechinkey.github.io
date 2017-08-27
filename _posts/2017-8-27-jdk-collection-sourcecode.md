---
date: 2017-08-27
layout:     post
comments:   yes
code:        yes
title:      jdk集合类源码实现总结
category:   blog
tags: [jdk,java,源码,HashMap]
---
待完善，来自http://ypj.me
各种集合类的总结：

* 实现原理描述，看完后再去看代码就会简单的多。网上的很多文章都是开始就贴上并直接分析源码，但是如果提前先了解了关键的（但不是一句话的）实现原理之后再看代码就会变得轻松一些。
* 基于JDK 8U112的源码分析
* 对于不常用的集合类，介绍了它们的应用场景，不常见但有时很有用的API
* 各种需要注意的地方，这个不属于实现原理，而是根据实现原理的特征得到的结果

GO

* `ArrayList`
  * 线程不安全，使用数组实现
  * 数组扩充逻辑：在需要增加元素但原数组已经容纳不下时，需要扩充。新数组的长度取“原长度的1.5倍”和“本次需要增加的元素个数（`addAll`一次性会增加多个）+原数组长度”的较大值。
  * 通过无参构造器`new ArrayList()`，数组默认长度为10，但是创建`ArrayList`时不会立即创建数组，而是在第一次添加元素时创建，只有在第一次添加小于等于10个元素时，第一次初始化的数组长度才为10。
  * 通过`new ArrayList(n)`可以指定初始大小，创建时会立即分配。所以在可预知数组大小或大概大小时，可以指定此值，避免空间浪费和频繁扩充数组。
  * `ListIterator listIterator()`相比传统`Iterator iterator()`多了添加，删除的方法，可以在遍历中途进行增删。在遍历过程中不能进行增删（调用`ListIterator`的API除外），否则此`Iterator`对象再执行`next()`时，会抛出`ConcurrentModificationException`，`for (X i : list)` 用于集合时其实也是调用的`Iterator`。
  * `subList()`返回一个`SubList`对象，该对象的各种增删查方法都会被代理到原`List`中去，会相互影响，注意。当然有时用`SubList`时会有好处，相当于返回一个视图，操作视图会比对着原对象操作有时会直观一些。

* `LinkedList`
  * 线程不安全，使用双向链表实现（双向的好处，在做基于索引的操作时，如果目标位不靠中间时，可以从离的比较近的那头去操作）

* `CopyOnWriteArrayList`
  * 线程安全，所有增删改动作，都使用`ReentrantLock`给包裹
  * 所有的增删改动作的效率都非常，都会生成新数组将原数组的内容复制过来，然后重新引用新数组
  * 遍历`iterator()`效率非常高，因为`next()`方法都不会做各种确认列表未做修改的检查。
  * 在遍历过程中列表被修改了，此`Iterator`不感知，还是遍历修改之前的数组，不会抛出`ConcurrentModificationException`，因为适用于大量读很少写的情况
  * 如果在第一次设置OK后，如果后面不会再去变化的情况下，使用[Guava的Immutable集合对象](https://github.com/google/guava/wiki/ImmutableCollectionsExplained)会更加的高效

* `Vector`
  * 线程安全，逻辑与`ArrayList`几乎相同，只不过是把几乎所有方法都加上`synchronized`，类似`Collections.synchronizedList()`

* `Stack`
  * 是`Vector`的子类，仅仅加了几个栈的方法例如`push()`、`pop()`、`peek()`

* `HashMap`
  * 线程不安全，支持`null`键和`null`值（因为支持`null`值的特性，需要注意`containsKey(K)`和`get(K) != null`可能结果会不一样）
  * 结构是：散列桶数组`Node[] table`，数组引用的可能情况：
    * `null`，没有数据落入此桶
    * `Node`，单向链表，应用于当此桶的数据比较少时
    * `TreeNode`，红黑树（后面简称RBT）的根节点，应用于当此桶的数据比较多时（`get`时能够比较快的搜索到，比如，此桶有1024个元素，如果是链表结构，最坏的情况是目标元素在链表尾部，需要寻找差不多1024次，而用RBT则最多差不多只需要找`lg 1024 = 10`次）
  * `hash`值，不是简单的`hashCode()`，而是通过原`hashCode()`的高位经过处理扩展到低位，使`hash`值的低位变得更加杂乱和随机。毕竟桶的数量一般不是太多，所以`hash`值的低位才决定了入哪个桶。
  * 插入逻辑：首先根据对象的`hash`值对桶个数取模（由于`HashMap`中桶的数量保持在2的指数，可以可以用取模达到求余的效果，而`Hashtable`的桶数量默认是11，所以只能用`%`求余），然后根据该桶的情况将数据放进去
    * `null`：包裹成`Node`直接赋值，作为链表头。
    * 链表：插入到链表尾部，如果此时链表长度达到8会进行树化。
    * RBT：根据RBT算法插入到RBT里面去。
  * 树化：如果散列桶大小小于64，则进行散列桶扩充，但是此桶不会改成红黑树，因为很有可能在散列桶扩充后，此桶的一些元素会挪到其他桶去。如果散列桶大小已经达到64，则将所有元素重新组织成RBT。
  * 扩充：将散列桶长度翻倍，然后重新组织（假如原来有16个桶，则`hash`值为17和33的元素都在第二个桶，扩充到32个桶后。hash为17的就直接挪到第18个桶了）。触发扩充的条件：当总元素个数达到原桶个数乘以增长因子时（默认初始桶大小是16个，增长因子是0.75，所以到达第12个时就进行扩充）；树化时也可能会进行扩充（例如初始桶个数16个小于64，插入了落到同一个桶8个数据，会触发树化扩充，所以并不一定非得达到增长因子时才会扩充）。
  * 扩充逻辑效率很低（新建比较大桶数组，然后需要将每个桶的数据拆分出去，例如上例中第一个桶需要分出去一部分到第17个，如果原RBT将数据分出去后数量小于6时会退化到链表，新桶分到的元素个数达到8时又会变成RBT），所以在构建`HashMap`时能大概清楚会有多少数据，会不会冲突很多，根据`HashMap`的行为，选定一个初始散列桶大小。
  * 删除元素：删除链表或RBT的一个节点。在RBT节点数减到6时，会将RBT又退化到链表。
  * `Map`的键值是可变对象会是什么情况？在将某个键值对放入`Map`后，这个对象变了，导致新的`hash`值变化了，但是`HashMap`识别不了这种情况，不会重新安插。所以不要将可变对象作为K，或者保证插入后不变。
  * 在桶里面的RBT根据什么去决定Key的顺序？从先到后，如果前者一致的话，试图往下继续比较：
    * `hash`值大小
    * `Comparable`，如果Key实现了此接口的话，如果未实现或比较结果为0，则
    * `getClass().getName()`，字符串比较，不过大部分情况下插入的Key的所有Class都一样，少数可能会有子类存在
    * `System.identityHashCode()`，这个每个对象都不一致不会再重复
    而对于`TreeMap`的话，也是用的RBT，但是`TreeMap `要求Key实现`Comparable`接口，如果没有实现，则需要传一个`Comparator`，所以比较逻辑不会像这么蛋疼。
  * 首先根据`hash`值大小，如果两个`hash`一致时，如果K类实现了`Comparable`接口，会根据比较的值做比较，如果还是一样的话，则比较`getClass().getName()`，还一样的话，则根据`System.identityHashCode()`做比较，反正得找到东西让这两个对象不一样。而`TreeMap`的话就强制要求实现`Comparable`或者传进去一个`Comparator`。

* `Map.Key` 和`Set`这些根据`hashCode`或`equals`或`compareTo`方法等各种手段来决定位置，此对象在插入后如果改变的话导致这些方法的返回值变化，导致逻辑上这些元素的位置“不再正确”，很可能出现各种问题，所以尽量使用`Integer`、`String`这些不可变对象，即使用了可变对象，在插入后不应该再动这些数据。

* `LinkedHashMap`
  * 是`HashMap`的子类，很多特性一致，同样线程不安全，支持`null`键和`null`值
  * 遍历（`iterator`、`keySet`、`entrySet`这些）时，能按照插入的先后顺序遍历
  * 插入删除逻辑与`HashMap`完全一致但是做了扩充，其在散列桶数组之外又单独维护了一个链表，在使用`HashMap`的完成散列桶的操作后，再往自己独立的链表尾部插入此节点。遍历时即遍历此链表。

* `Hashtable`
  * 线程安全：通过像`Vector`那样，简单暴力的将几乎所有方法都加上`synchronized`
  * 不允许`null`键，也不允许`null`值。
  * 机理与`HashMap`基本一致，但是每个桶里面只会是链表，不会出现红黑树，在单个桶里面比较多的时候，查找会比较慢

* `TreeMap`
  * 线程不安全，不支持`null`键，但支持`null`值，内部使用RBT保持节点有序，是有序的`Map`，遍历时会根据顺序进行。
  * 实现了`NavigableMap`和`SortedMap`接口，里面有一些API是跟次序相关的，有些场景就不需要进行遍历，直接用这些API更简单一些。
  * 约束：未通过`new TreeMap(Comparator)`执行`Comparator`时，Key的类必须要实现`Comparator`接口；如果没有实现的话，必须要传一个`Comparator`做RBT节点的比较。如果Key实现了`Comparable`，并且指定了`Comparator`，RBT会根据`Comparator.compare`做比较，忽略`Comparable.compareTo`
  * 如何判断Key重复？同保持有序一样，根据`Comparator`或`Comparable`的比较结果而不是根据`equals()`方法，所以可能会出现`equals`不一样但`compare`或`compareTo`方法返回0的情况，这时，会被理解成一样的Key，如果此时做更新的话，Key不会更新，只有Value更新。

* `ConcurrentHashMap`

  * TO-DO Very difficult​



