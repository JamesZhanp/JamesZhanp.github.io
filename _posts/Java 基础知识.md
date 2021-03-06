---
layout:     post

title:      Java基础知识解读

subtitle:   java, imooc

date:       2020-09-03

author:     JZ

header-img: img/post-bg-swift.jpg

catalog: true

tags:

    - java
---

# Java 基础知识

##  String, Long 源码解析

### String

#### 1.1 不可变性

String的值修改， 其实是将s的引用指向**新的**String

源码可知：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence{
    private final char value[];
}
```

由上可知：

1. String被**final**修饰， 说明该类*不可*被**继承**，也就是对任何对String的操作方法，都不会被继承覆写
2. String中保存数据的是一个**char**的数组value。 value也是被final修饰的，value一旦被赋值， 内存地址是无法修改的，value的权限是**private**的，String也没有对外进行赋值的方法，因此一旦产生，内存地址就根本无法被修改

由上可知，对于String的操作，多数只能返回新的String类型，因此需要变量接受返回的参数

#### 1.2 String的方法

**substring**: substring 有两个方法

`public String substring(int beginIndex, int endIndex)`

`public String substring(int beginIndex)`

substring底层方法使用的是字符数组范围截取的方法:`Arrays.copyOfRange(Arrays, beginIndex, endIndex)`

**equals** and **equalsIgnoreCase**

后者判断的时候忽略大小写

源码：

```java
public boolean equals(Object anObject){
    // 判断是否为相同的地址
    if(this == anObject){
        return true;
    }
    //判断数据类型
    if (anObject instanceof String){
        String anotherString  = (String) anObject;
        int n = value.length;
        // 判断长度是否相等
        if(n ==  anotherString.value.lenght){
            char v1[] = value;
            char v2[] = anotherString.Value;
            
            int i = 0;
            // 依次比较， 若出现不同直接返回不相等
            while(n --){
                if	(v1[i]  != v2[i]){
                    return false;
                }
                i++;
            }
            return true;
        }
   }
    return false;
}
```

**替换， 删除**

replace 并不知识替换一个，而是把所有匹配到的字符出字符串都替换掉

删除可以使用replace("str", "")

**拆分和合并**

拆分使用split方法， 两个参数，

第一个参数是我们拆分的标准字符，第二个参数是一个 int 值，叫 limit，来限制我们需要拆分成几个元素。

### Long

**缓存**

Long的缓存问题，Long自己实现了以重缓存机制， 缓存了从-128到127内的所有Long值，在这个范围内的Long值不会初始化，而是直接从缓存中拿

**为什么使用 Long 时，大家推荐多使用 valueOf 方法，少使用 parseLong 方法**

答：因为 Long 本身有缓存机制，缓存了 -128 到 127 范围内的 Long，valueOf 方法会从缓存中去拿值，如果命中缓存，会减少资源的开销，parseLong 方法就没有这个机制。

## 关键词

### 1. static

static修饰的内容是静态， 全局的，一定范围内是共享的，需要注意并发读写

static 只能修饰类变量， 方法 和代码块

**当static修饰类变量**， 如果变量是public的， 表示该变量任何类都可以直接访问，而且无需初始化类， 需要注意的是**线程安全**问题，当多个线程对同一静态变量进行读写时，会出现并发问题。

**当static修饰方法时**，权限是public，证明该方法与类无关，任一类都可以直接方法。

static的方法只能调用同样被static修饰的静态方法，因此util方法基本使用static进行修饰

static方法的内部变量是不存在线程安全问题的，每一个方法在执行的之后，方法的变量会进入栈里面，栈是线程独立的。

**当static修饰代码块的时候**， 静态块，静态块常常被使用初始化变量

**注意**：

1. 父类的静态变量和静态块比子类**优先**初始化；
2. 静态变量和静态块比类构造器**优先**初始化。

### 2. final

final的意思是不变的， 一般而言用于以下三种场景：

1. 被final修饰的类，该类无法继承
2. final修饰的方法， 该方法无法覆写
3. 被final修饰的变量， 变量在申明的时候，就必须初始化完成，后续也不能修改其**内存地址**

**注意**： 无法修改的是内存地址，而不是值，比如对集合类而言，可以修改其内部的值，却不能修改初始化的内存地址

### 3. try catch finally

先执行finally， 在执行catch捕获的异常处理

最终捕获的异常是catch的异常，try抛出来的异常已经被处理掉

### 4. volatile

volatile的意思是可见的，当变量共享时，若出现变量值被修改，会及时通知去哦他线程，该值被修改。

JMM模型

<img src="https://img1.sycdn.imooc.com/5d5fc0ea000100a312740736.png">

### 5. transient

transient关键字我们常用来修饰变量类， 意思是当前是无需进行序列化的。

## Arrays, Collections, Obejcts 常用方法

工具类的通用特征

1. 构造器必须是**私有**的， 工具类无法被new出来，因为工具类在使用的时候，无需初始化，直接使用即可，所以不能开放构造器
2. 工具类的工具方法必须被static， final关键字修饰。保证方法不可变，可直接使用

### Arrays

#### 1. 排序

Arrays.sort 方法主要用于排序，入参支持 int、long、double 等各种基本类型的数组，也支持自定义类的数组， sort 方法支持两个入参：要排序的数组和外部排序器。

```java
// array 需要排序的数组，SortDTO::getSortTarget获取值
Arrays.sort(array, Comparator.comparing(SortDTO::getSortTarget));
```



#### 2. 二分查找

Arrays.binarySearch主要用于快速从数组中查找出相对应的值。如果找到返回下标值，反之，返回负数。

注意的点：

1. 进行二分查找之前，需要将数组进行排序， 如果是无序的，则二分搜索可能搜索不到
2. 搜索方法返回的是数组的下标值。如果搜索不到，返回的下标值就会是负数，这时我们需要判断一下正负。

#### 3. 拷贝

数组拷贝，比如： ArrayList在add或者remove的时候，会进行一些拷贝。拷贝整个数组可以使用copyOf方法，拷贝部分可以使用copyOfRange方法， 底层使用的是System.arrayCopy这个native方法。

### Collections

Collections是为了方便集合使用的工具类。Collections提供sort和binarySearch方法。

#### 求集合中的最值

提供max方法取得集合中的最大值， min方法取得集合中的最小值。需要实现Comparable接口

#### 多种类型的集合

**线程安全的集合**： 线程安全的集合方法都是synchronized开头的，底层使用Synchronized实现

以SynchronizedList为例， 所有的操作方法都被加上了Synchronized锁，因此当多线程同时操作时是线程安全的

**不可变的集合**： 

得到不可变集合的方法都是以 unmodifiable 开头的。这类方法的意思是，我们会从原集合中，得到一个不可变的新集合，新集合只能访问，无法修改；一旦修改，就会抛出异常。这主要是因为只开放了**查询**方法，其余任何**修改操作都会抛出异常**

## ArrayList

ArrayList的整体结构比较简单，就是一个数组结构。 其默认初始化的长度为10。

- 允许put null值，且会自动扩容（1.5倍）
- size， isEmpty， set， get， add等方法的时间复杂度都是O（1）
- 是非线程安全的，多线程情况下，使用Collenctions.SyschronizedList
- 增强for循环，迭代器迭代过程中若大小改变，会快速失败

### 初始化

初始化有三种办法： 无参数直接初始化、指定大小初始化、指定初始数据初始化

注意点：

1. 无参构造初始化时， ArrayList的默认大小是空数组，而不是10， 10始在第一次add的时候扩容后的数组大小、

#### 新增和扩容的实现

新增就是往数组中添加元素：

1. 判断是否需要扩容，如需要则进行扩容
2. 赋值

```java
public boolean add(E e){
    // 判断是否需要扩容
    ensureCapacityInternal(size + 1);
    // 赋值
    elementData[size++] = e;
    return true;
}
private void ensureCapacityInternal(int minCapacity) {
  //如果初始化数组大小时，有给定初始值，以给定的大小为准，不走 if 逻辑
  if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
    minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
  }
  //确保容积足够
  ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
  //记录数组被修改
  modCount++;
  // 如果我们期望的最小容量大于目前数组的长度，那么就扩容
  if (minCapacity - elementData.length > 0)
    grow(minCapacity);
}
//扩容，并把现有数据拷贝到新的数组里面去
private void grow(int minCapacity) {
  int oldCapacity = elementData.length;
  // oldCapacity >> 1 是把 oldCapacity 除以 2 的意思
  int newCapacity = oldCapacity + (oldCapacity >> 1);

  // 如果扩容后的值 < 我们的期望值，扩容后的值就等于我们的期望值
  if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;

  // 如果扩容后的值 > jvm 所能分配的数组的最大值，那么就用 Integer 的最大值
  if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
 
  // 通过复制进行扩容
  elementData = Arrays.copyOf(elementData, newCapacity);
}
```

注意点：

- 扩容不是翻倍，而是原来容量的1.5倍
- ArrayList的最值是Integer.Max_VALUE, 超过这个值， JVM就不会进行内存分配了

扩容的本质： 使用的是Arrays.copyOf(elementData, newCapacity)， 即首先会创建一个符合预期容量的数组，将老数组的数据拷贝过去

#### 删除

注意点：

删除的时候使用equals进行判断，所以当删除的不是基本数据类型的时候，需要关注equals的具体实现

```java
private void fastRemove(int index) {
  // 记录数组的结构要发生变动了
  modCount++;
  // numMoved 表示删除 index 位置的元素后，需要从 index 后移动多少个元素到前面去
  // 减 1 的原因，是因为 size 从 1 开始算起，index 从 0开始算起
  int numMoved = size - index - 1;
  if (numMoved > 0)
    // 从 index +1 位置开始被拷贝，拷贝的起始位置是 index，长度是 numMoved
    System.arraycopy(elementData, index+1, elementData, index, numMoved);
  //数组最后一个位置赋值 null，帮助 GC
  elementData[--size] = null;
}
```



## LinkedList

LinkedList的底层是一个双向链表，结构如下所示

<img src="https://img1.sycdn.imooc.com/5d5fc67a0001f59212400288.png"></img>

基本概念：

- 每个节点具有prev和next两个属性
- 当数据为空的时候，first和last是一个节点，且前后均为null
- 内存足够大，大小没有限制

#### add

追加节点的时候可以追加到头部或者尾部， add方法是默认添加到尾部的

源码如下：

```java
// 从尾部开始追加节点
void linkLast(E e) {
    // 把尾节点数据暂存
    final Node<E> l = last;
    // 新建新的节点，初始化入参含义：
    // l 是新节点的前一个节点，当前值是尾节点值
    // e 表示当前新增节点，当前新增节点后一个节点是 null
    final Node<E> newNode = new Node<>(l, e, null);
    // 新建节点追加到尾部
    last = newNode;
    //如果链表为空（l 是尾节点，尾节点为空，链表即空），头部和尾部是同一个节点，都是新建的节点
    if (l == null)
        first = newNode;
    //否则把前尾节点的下一个节点，指向当前尾节点。
    else
        l.next = newNode;
    //大小和版本更改
    size++;
    modCount++;
}
```

头部添加类似，将first的prev指向现在的节点

**删除节点**

从头部删除

```java
//从头删除节点 f 是链表头节点
private E unlinkFirst(Node<E> f) {
    // 拿出头节点的值，作为方法的返回值
    final E element = f.item;
    // 拿出头节点的下一个节点
    final Node<E> next = f.next;
    //帮助 GC 回收头节点
    f.item = null;
    f.next = null;
    // 头节点的下一个节点成为头节点
    first = next;
    //如果 next 为空，表明链表为空
    if (next == null)
        last = null;
    //链表不为空，头节点的前一个节点指向 null
    els
        next.prev = null;
    //修改链表大小和版本
    size--;
    modCount++;
    return element;
}
```

尾部删除类似



## HashMap

### 整体架构

HashMap的底层的数据结构主要是：数组 + 链表 + 红黑树

当链表的长度大于8的时候，链表会转化成红黑树，当红黑树的大小小于6的时候，会转化成链表。

<img src="https://img1.sycdn.imooc.com/5d5fc7cc0001ec3211040928.png">

#### 类注释

- HashMap允许null值，但也只允许出现一个null的key， 且是非线程安全的
- load factor（影响因子）默认值是0.75， 是均衡了时间和空间损耗出来的值
- 若在HashMap中要存储大量的值，则容量在一开始就要设置足够大，可防止不断扩容的情况出现，影响性能
- HashMap是非线程安全的，可以自己在外部枷锁，或者使用Collections.synchronizedMap来实现线程安全，Collections.synchronizedMap的实现是在每个方法上都添加了**Synchronized**锁
- 迭代过程中，若修改HashMap的结构，则会快速失败

#### 常见属性

```java
// 初是容量16
static final int DEFAULT_INITIAL_CAPACITY=1<<4;
// 最大容量
static final int MAXIMUM_CAPACITY = 1<<20;
//负载因子默认值
static final float DEFAULT_LOAD_FACTOR=0.75f;
//链表长度上限
static final int TREEIFY_THRESHOLD = 8;
//红黑树的长度下限
static final int UNTREEIFY_THREESHOLD = 6;
// 当数组容量大于64时，链表才会转化成红黑树
static final int MIN_TREEIFY_CAPACITY=64;
// 记录迭代过程中，HashMap的结构是否发生变化
transient int madCount;
// HashMap 的实际大小
transient int size;
// 存放数据的数组
transient Node<K, V>[] table;

int threshold;

//链表的节点
static class Node<K, V> implements Map.Entry<K, V>{}
//红黑树的节点
static final class TreeNode<K, V> extends LinkedHashMap.Entry<K, V>{}
```

