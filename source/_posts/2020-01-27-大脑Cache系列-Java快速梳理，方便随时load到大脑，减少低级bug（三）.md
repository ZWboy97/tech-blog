---
title: 大脑Cache系列 -- Java快速梳理，方便随时load到大脑，减少低级bug（三）
categories:
  - [技术]
  - [大脑Cache]
tags:
  - Java
date: 2020-01-27 20:11:41
---


### 背景

> - Java程序常常会遇到一些蛋疼的bug，最后发现，都是在一些很基础的方面造成的。大量的时间花在调试代码找低级bug上是十分没有性价比的。所以，再系统梳理下Java，是十分必要的。
> - 已经反复学习和使用Java多次了，但只要有段时间没用Java之后，每次使用前都想要重头再梳理一遍。本文章将更注重Java只是的系统性，而不是细节性。
> - 本文是本人大脑的专属Cache，所以逻辑上可能只有我自己能够看懂，见谅。

### 一、目录
- Java泛型
- Java集合
- Java部署

<!--more-->

### Java泛型
- 在不支持泛型之前，Java是通过继承实现对不同类型数据的支持，缺点是每次都需要进行格式转换
- 泛型：**类型参数**

```java
// 泛型类
public class Pair<T, U> { // 类型参数，支持多个
  private T first;
  private T second;

  // 泛型方法，
  public <T> T getFirst() { // 类型参数放在修饰符后边，返回类型前边 
    return first;
  }
}

```

- **泛型方法可以定义在泛型类中，也可以定义在普通类中**

```java
public class Pair {
  private int first;
  public <T> T getFirst(){ // 泛型方法可以定义在普通类中
    return first;
  }
}
```
- **类型变量的限定**

```java
public class Pair <T extends BindType1 & BindType2> // T必须是绑定类型的子类，对类型变量做了限定
```
##### 泛型的虚拟机实现——类型擦除
- **虚拟机中并没有泛型类的对象，所有的对象都属于普通类**
- **类型擦除**
  - **原始类型**：每一个泛型类型，都提供一个相应的原始类型。
  - 原始类型名为泛型类删除类型参数
  - 擦除类中的类型变量，替换为限定类型(第一个限定类型)，没有限定则为Object类型
  - 比如Pair<T>类的原始类型为
```java
public class Pair {
  private Object first;
  private Object getFirst(){
    return first;
  }
}
```
- 程序调用泛型方法的时候，在虚拟机层面实际执行两条指令，多出来的一条为插入的类型转换指令，目的是回复擦除的类型。
  - 多出来的虚拟机指令是编译器添加的，在字节码中添加

##### 泛型的注意点
- **不能用基本类型作为类型参数**，比如int,double，而要改用Integer，Double
- **不对泛型类型对象做类型检查**
  - **getClass返回的总是原始类型，而不是泛型类型**
  - 同样的**a instanceof Pair<String>**，编译错误
- **不能实例化创建参数化类型的数组**
  - `Pair<String> [] table = new Pair<String>[10]; // error`
##### 泛型类型的继承规则
- `Pair<Employee>`和`Pair<Manager>`之间没有继承关系。

### Java集合框架
- 传统的支持：Vector,Stack,HashTable,BitSet,Enumeration接口等

##### Java集合框架将集合的接口与实现分离
- 队列接口Queue(interface)
  - 队尾添加元素，队头删除元素，先进先出，且能够获得队列元素数目
  - 接口方法：add(E element),remove(),size()
  - 集合实现这个接口就可以实现一个队列
- Collection集合接口
  - Java集合类的**基本接口**，主要有两个重要的方法

```java
public interface Collection<E> {
  boolean add(E element); // 添加元素，不允许重复
  Iterator<E> iterator(); // 返回实现了迭代器接口的对象，通过该迭代器对象可以依次访问集合中的元素
  int size(); // 返回元素数目
  boolean isEmpty(); 
  boolean contains(Object obj); // 集合中是否存在该元素
  boolean remove(Object obj); // 如果存在obj，则删除
  void clear(); // 情况集合内容
  Object[] toArray(); // 将集合转化为数组
  ...
}
```

- 迭代器Iterator
  - for each循环需要集合实现iterator接口

```java
public interface Iterator<E> {
  // 类比编辑软件的光标、插入以及删除效果
  E next(); // 越过当前元素，迭代到下一个（光标后移），并返回越过元素的内容
  boolean hasNext(); // 判断是否能够越过当前元素
  void remove(); // 删除光标之前的元素
  default void forEachRemaining(Consumer<? super E> action); // Java8
}
// 使用,比如foreach的实际实现
Collection<String> str = ...
Iterator<String> iter = str.iterator();
while(iter.hasNext()){
  String element = iter.next();
  dosomething...
}
// Java8中直接通过forEachRemaining和lamabda表达式实现
itertor.forEachRemaining(element -> {
  do something with element
})
```
  - 元素的迭代顺序由具体的集合来决定
  - 与C++中的迭代器不同（基于索引下标，不需要查找操作，直接通过索引i++），Java中的迭代器是**查找操作与位置变更是紧密相连的，查找一个元素的唯一方法是调用next(),在执行查找操作的同时，迭代器已经就指向下一个元素了，返回上一个元素**
  - **next和remove方法的调用具有互相依赖性，在调用remove之前，如果没有调用next，将会抛出一个非法异常**
  - **删除两个相邻的元素的元素不能直接连续remove，必须调用next越到下一个元素**

- Java集合框架中各种集合接口的关系

```java
- Iterable        可迭代接口
  - Collection    集合类的基础类
    - List        有序集合，能够有序访问和随机访问
    - Set 
      - SortSet
    - Queue
      - Deque
- Map             key value 映射
  - SortedMap
- Iterator
  - ListIterator  针对于List的迭代器
```

##### Java中的具体集合类
- 以Map结尾的都实现了Map接口，没有的都实现了Collection接口

```java
- AbstractCollection
  - AbstractList        有序集合
    - ArrayList         支持动态增长和缩减的索引序列
    - LinkedList        链表类，支持高效插入和删除
  - AbstractSet         数学中的集合，没有重复元素
    - HashSet           没有重复元素的无序集合
      - LinkedHashSet   可以记住元素插入顺序的集合
    - TreeSet           有序集合
    - EnumSet           包含枚举类型值的集合
  - AbstractQueue
    - PriorityQueue     优先级队列，高效删除最小元素的集合
    - ArrayDequeue      循环数组实现的双端队列
- AbstractMap           映射表
  - HashMap             普通常用的键值对
    - LinkedHashMap     在能记录键值对的添加次序
  - TreeMap             键值有序排列
  - EnumMap             键值为枚举类型的映射表
  - WeakHashMap         一种当其值无用武之地的时候，可以被垃圾回收器回收的引射表
  - IdentityHashMap     一种用===而不是queals比较键值的映射表
```

##### 链表LinkedList
- 数组Array和数组列表类对象ArrayList有个缺陷，在数组中间位置添加和删除的代价很大
- Java的链表实际上是**双向链表**，有找到前向和后继的引用
- 链表实现了Collection接口，但Collection接口的add()方法只支持向链表最后添加元素，不支持链表的任意位置添加元素
- 在中间位置插入元素，通过ListIterator接口提供的方法实现。

```java
// 面向LinkedList链表的迭代器
interface ListIterator extends Iterator<E> {
  void add(E element); // 链表的插入方式，在当前迭代位置后加入一个元素
  ...
  E previous(); // 反向遍历链表，访问上一个元素
  boolean hasPrevious(); // 与hasNext()类似
}
// 使用链表，并从中间加入元素
List<String> staff = new LinkedList<>();
staff.add("1");
staff.add("3");
staff.add("4");
ListIterator<String> iter = staff.listIterator();
iter.next(); // 跳过第一个元素，没有next的话，就是作为链表头
iter.add("2"); // 最终的结果，1234
// n长度的链表，有n+1个插入位置
|ABC
A|BC
AB|C
ABC|
// 链表提供了一个set()方法来替换值，不用先删除再添加了
ListIteartor<String> iter = new ListIteartor<>();
String oldValue = iter.next(); // 得到第一个元素
iter.set(newvalue); // 将第一个元素赋予新值
```
- 可以创建多个链表迭代器，比如用于并发，但是同时只能有一个迭代器用来修改链表集合，其他迭代器只能读取，多个迭代器同时修改会抛出异常。
- 链表作为一个List，也是支持有序访问和通过索引访问的的，但对于链表不推荐使用get(i)和set(i,value)

##### 数组列表ArrayList
- 有序列表，动态扩展和收缩
- ArrayList不是同步的，是线程不安全的，适合在非并发的时候使用
- Vector是同步的，线程安全的，但是同步操作上会耗费大量的时间，只在并发的时候使用

##### 散列集HashSet
- 不关注元素的顺序，只想要能够快速查找到元素，且无重复元素的集合Set
- 通过散列表来实现，散列表为每个对象计算一个散列码HashCode，对象的hashCode()方法在这个时候使用
- 装填因子，散列函数，桶，散列冲突等概念
- 对散列集的遍历顺序是随机的

```java
Set<String> words = new HashSet<String>();
words.add("hello");
Iterator<String> iter = words.iterator();
iter.hasNext();
String word = iter.next();
```

##### 树集TreeSet
- 有序集合，内部是树结构，按照树的顺序进行遍历（红黑树）
- 每次添加元素，都将元素添加到树中的正确位置。

##### 队列Queue与双端队列ArrayDeque
- Queue

```java
  // 队尾添加
  - boolean add(E element) 
  - boolean offer(E element) 
  // 对首删除并返回
  - E remove()
  - E poll()
  // 只返回对首元素，不删除
  - E element()
  - E peek()
```
- Deque

```java
void addFirst(E element)
void addLast(E element)
boolean offerFirst(E element)
boolean offerLast(E element)
E removeFirst()
...
```

##### 优先级队列PriorityQueue
- 支持任意顺序的插入，而每次remove，返回的都是队列中的最小值
- 内部通过堆来实现，是一个能够自我调整的二叉树，在对树进行add和remove操作中，能始终保证最小的元素移动到根
- 优先级队列用于任务调度

```java
// E 必须是支持比较的元素类型
PriorityQueue<E> queue = new PriorityQueue<>();
queue.add(element1); // 按照随机顺序加入，自动调整排序
queue.add(element2);
queue.add(element3);
E min = queue.remove(); // 返回最小值
```

##### 映射Map
- HashSet是以对象副本来查找元素，不好用
- 映射通过键值对来查找元素
- 有`HashMap`和`TreeMap`两种
  - `HashMap`，通过对键进行散列，进而访问值
  - `TreeMap`，利用键的顺序，对元素进行排序
- 和集合一样，散列稍微快一些，如果不需要按照排列顺序访问键，推荐使用散列映射
- Map的键使用的是字符串
- 添加键值对：`V put(K key,V value)`
  - 重复用同一个key添加，会覆盖之前添加的值,返回之前的值
  - 添加一个新的键，返回的值为null
- 查找键值对:`V get(Object key)`
  - 如果不存在该key，则返回null
  - `V getOrDefault(Object key, V defaultValue)`
- 删除键值对：`remove(Object key)`
- 判断是否有键：`boolean containsKey(Object key)`
- 判断是否有值：`boolean containerValue(Object value)`
- 遍历键值对：`hashMap.forEach((k,v)={do...})`
- 三个视图View: keySet,valueSet,entrySet

```java
HashMap<String,E> hashMap = new HashMap<String,E>();
hashMap.put("first",element1);
hashMap.put("second",element2);
hashMap.get("first");
Set<String> keySet = hashMap.keySet();// {first,second}
Set<E> valueSet = hashMap.valueSet();
Set<Map.Entry<String,E>> entrySet = hashMap.entrySet();
```

##### 弱散列集合WeakHashMap
- 能够自动回收那些无用的键值对
- 如果有一个值，对应的键已经不再使用了，那么该键值对将无法从Map中删除
- 对于长期存活的Map，需要手动删除，或者直接使用WeakHashMap
- 内部实现中，WeakHashMap使用的弱引用(wek references)来保存键。
  - 弱引用对象将引用保存到另外一个对象中，在这里，就是散列键
  - 如果某个对象，只有WeakReference引用，那么垃圾回收器可以回收它。

##### 链接散列集与映射LinkedHashSet与LinkedHashMap
- 在实现HashSet与HashMap的基础上，还能够记录元素的添加顺序，即迭代遍历顺序
- 实现方式是在Hash表的基础上，再通过链表的方式将添加的元素顺序链接起来。

##### 枚举集与映射EnumSet
- 如果key的类型为枚举，那么可以使用EnumSet
- 使用位序列进行了优化

##### 表示散列映射IdentityHashMap
- 特殊用途：**对象遍历算法**
- hash计算不再通过对象的hashCode计算，而是通过System.identityHashCode计算，返回的是根据对象的内存地址计算的Hash值
- 所以不能用equal()来判断相等了，而是用==号来判断
- **不同的键值对，即便内容一样，也被视为不一样的对象**

