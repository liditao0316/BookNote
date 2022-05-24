## 源码分析

```java
/*
    *   初始化hashMap
    *   Constructs an empty HashMap with the default initial capacity (16) and the default load factor (0.75).
    * */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /*
    * 第一步
    * 1、获取获取key的hash值
    * 2、调用putVal()方法
    * */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    /*
    *   注意：将hashCode逻辑右移16位，能够有效减少冲突的发生
    */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    /*
    *   transient Node<K,V>[] table; //table就是HashMap的一个数组，类型是Node[]
    */

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)//如果数组为null时，通过resize()方法生成一个table
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)//通过hash值找到key的位置，如果该位置没有数据，则直接写入数据
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            /*
             *	如果当前索引位置对应链表的第一个元素和准备添加的key的hash值一样
             * 并且满足 下面条件之一：
             *（1）准备加入的key和p指向的Node结点的key是同一只对象
             *（2）两个复合数据类型的值相同
             */
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            /*如果链表上的结点大于或等于TREEIFY_THRESHOLD，将会调用treeifyBin()方法，
                            	对该链表进行树化，转成红黑树或者对数组进行扩容
                            */
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;//指向下一个结点
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;//修改次数增加
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

    /**
     * 扩容
     * threshold //The next size value at which to resize (capacity * load factor)临界值
     * static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
     * static final float DEFAULT_LOAD_FACTOR = 0.75f;//加载因子，避免数组太空浪费存储空间，同时避免数组太满导致链表太长影响查询的速度。就是说使HashSMap空间快被用完的时候进行扩容
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //容量扩大为原来的两倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults初始化
            newCap = DEFAULT_INITIAL_CAPACITY;//初始化容量为16的数组
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

    //在table长度大于等于64时，对链表进行树化，并变成一颗红黑树；或者对table进行扩容
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```



### 注意的问题：

#### 1、扩容机制

- 当table容器的大小为0时

  ```java
  /*
  * 1、初始化容量为16的数组
  * 2、通过DEFAULT_LOAD_FACTOR加载因子，使得扩容边界变量threshold的值小于table容器的大小，当大量线程对table容器进行写入操作，避免由于容器能够写入的空间不足，导致大量线程出现阻塞的情况发生。
  */
  newCap = DEFAULT_INITIAL_CAPACITY;
  newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
  ```

- 当table容器的大小>0时

  ```java
  //移位操作使table容器扩容为原来的两倍，同时扩容边界变量threshold也变成原来的两倍
  else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
  	newThr = oldThr << 1; // double threshold
  ```

- 当table容器的某个结点上的链表大于等于8，同时table的size小于64

  ```java
  //putVal()中的代码
  if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
    /*如果链表上的结点大于或等于TREEIFY_THRESHOLD，将会调用treeifyBin()方法，对该链表进行树化，
      转成红黑树或者对数组进行扩容
  	*/
    treeifyBin(tab, hash);
  break;
  }
  
  //treeifyBin()方法中的代码
  if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
    resize();
  ```



#### 2、何时链表会进行树化

- 当table容器的某个结点上的链表大于等于8，同时table的size小于等于64

- 注意：树化的是某条链表，而不是将整个数组上的所有链表都树化

  ```java
  //putVal()中的代码
  if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
    /*如果链表上的结点大于或等于TREEIFY_THRESHOLD，将会调用treeifyBin()方法，对该链表进行树化，
      转成红黑树或者对数组进行扩容
  	*/
    treeifyBin(tab, hash);
  break;
  }
  
  //treeifyBin()方法中的代码
  if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
    resize();
  else if ((e = tab[index = (n - 1) & hash]) != null) {
    //树化操作
  }
  ```



#### 3、关键代码

```java
/*
* 1、hash值相同，内存地址相同
* 2、hash值相同，重写了equal方法返回true
* 以上两种情况都视为key重复
*/
if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))

/*
* 如果hash值相同，那么会出现hash冲突，HashMap中处理冲突的方法是使用拉链法
*/  
if ((p = tab[i = (n - 1) & hash]) == null)
  tab[i] = newNode(hash, key, value, null);
```



#### 4、插入的key存在

```java
/*
* 如何存在key的mapping，那么会将该结点上的value被替换成新插入mapping的value
*/ 
if (e != null) { // existing mapping for key
  V oldValue = e.value;
  if (!onlyIfAbsent || oldValue == null)
    e.value = value;
  afterNodeAccess(e);
  return oldValue;
}
```



#### 5、插入key的查找过程

```java
p = tab[i = (n - 1) & hash];
if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
  e = p;
else if (p instanceof TreeNode)//散列表上的结点类型是TreeNode，利用红黑树的查找技术
  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
else {
  for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {//散列表上的结点类型是Node，利用链表的查找技术
      p.next = newNode(hash, key, value, null);
      if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
        treeifyBin(tab, hash);
      break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
      break;
    p = e;//指向下一个结点
  }
```



### Map结点的继承关系

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220303163857367.png" alt="image-20220303163857367" style="zoom:50%;" />





## HashSet实践

- 定义一个Employee类，该类包含：private成员属性那么，age要求：
  - 创建3个Employee对象放入HashSet中
  - 当name和age的值相同时，认为是相同员工，不能添加到HashSet集合中

```java
//Employee类

/*
	关键是重写equals方法和hashCode方法
*/
import java.util.Objects;

public class Employee {
    private String name;
    private String age;

    public Employee(String name, String age) {
        this.name = name;
        this.age = age;
    }
		
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employee = (Employee) o;
        return Objects.equals(name, employee.name) && Objects.equals(age, employee.age);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

```

## 面试题

### 为什么HasMap红黑树的阈值为什么是8呢？

首先和hashcode碰撞次数的泊松分布有关，主要是为了寻找一种时间和空间的平衡。在负载因子0.75（HashMap默认）的情况下，单个hash槽内元素个数为**8的概率小于百万分之一**，将7作为一个分水岭，等于7时不做转换，**大于等于8才转红黑树，小于等于6才转链表**。链表中元素个数为8时的概率已经非常小，再多的就更少了，所以原作者在选择链表元素个数时选择了8，是根据概率统计而选择的。

红黑树转链表的阈值为6，主要是因为，如果也将该阈值设置于8，那么当hash碰撞在8时，会反生链表和红黑树的不停相互激荡转换，白白浪费资源。

## 知识扩展

### 1、“==” 和 equals的区别

- Java数据类型分为基本数据类型和复合数据类型(类)

- 基本数据类型的比较应该用==

- 复合数据类型(类)

  当我们用==比较对象时，实际上比较的是两个对象在内存的存放地址是否相等。

  复合数据类型即是类，继承了Object类，Object类中有equals方法，其实现方式也是用==实现的。

  但是有些复合数据类型重写了equals方法，比如String，Integer，所以这些复合数据类型使用==和equals比较是有区别的。

  - == 与 equals等效

    复合数据类型中的equals方法未被重写

  - == 与 equals不等效

    复合数据类型中的equals方法被重写

### 2、java字符串缓冲池

- Java虚拟机开辟了一个内存空间来存放字符串常量，而通过new创建字符串对象是存储在堆内存中

- 当创建一个字符串常量时，先重字符串缓冲池中找是否存在该字符串常量，没有则新建一个并返回引用地址，有则返回该字符串的引用地址

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220224170343785.png" alt="image-20220224170343785" style="zoom:50%;" />

### 3、解决Hash冲突的方法

#### （1）开放定址法

- 如果由关键码得到的散列地址产生了冲突，根据下列公式寻找下一个空的散列地址：

  ​				（H(key)+d)%m

  - 线性探测法：d=1，2，...，m-1；
  - 二次探测法：d=1<sup>2</sup>,-1<sup>2</sup>,...，q<sup>2</sup>，-q<sup>2</sup>；

#### （2）拉链法

​			HashSet处理Hash冲突的方法就是拉链法