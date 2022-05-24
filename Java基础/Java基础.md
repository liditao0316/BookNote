## String、StringBuilder和StringBuffer

>- String类的长度是不可变的，StringBuffer类和StringBuilder类的长度时可变的。
>- StringBuilder类是线程不安全的，StringBuffer类是线程安全的
>- 在使用 StringBuffer 类时，每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象，所以如果需要对字符串进行修改推荐使用 StringBuffer。
>- 由于 StringBuilder 相较于 StringBuffer 有速度优势，所以多数情况下建议使用 StringBuilder 类。



### String和StringBuilder

>String是不可变的对象，每次在对String对象进行修改操作时都会生成新的String对象，然后将指针指向新的String对象。因此对String对象进行大量修改操作时，会产生大量新的String对象，因此对于这种情况是不建议使用的，因为每生成一个新的String对象都有系统性能上的消耗，特别是当大量String对象失去引用时，JVM虚拟机的GC线程就会开始工作，性能就会下降。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/2021-03-01-java-stringbuffer.svg)

以上实例编译运行结果如下：

```
Runoob..
Runoob..!
Runoob..Java!
RunooJava!
```



### StringBuilder的实现原理

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220509093904146.png" alt="image-20220509093904146" style="zoom:50%;" />

#### 数据结构

>StringBuilder采用了char数组的类型，通过count记录字符串的长度。

```java
/**
 * The value is used for character storage.
*/
char[] value;
/**
 * The count is the number of characters used.
*/
int count;
```



### StringBuilder的实现原理

>与StringBuilder相比，StringBuffer使用synchronized实现线程同步，StringBuffer是粒度较粗的锁，锁住的是整个StringBuffer对象。