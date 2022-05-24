![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/7f749fc8.png)

### 1. 乐观锁 VS 悲观锁

对于同一数据的并发操作，悲观锁认为自己使用的数据一定会有其他线程来修改，所以在使用数据之前加上锁，确保数据不会被其他线程修改。Java中的synchronized关键字和Lock的实现类就是悲观锁。

乐观锁认为自己在使用数据的时候不会有其他线程来修改数据，因此不会加锁，只有在更新数据的时候才去判断数据是否被其他线程更新。如果这个数据没有被更新，当前线程将自己修改的数据写入。如果这个数据已经被其他线程更新，则根据不同的实现方式完成不同的操作（比如报错 或 重试）。

乐观锁在Java中是通过使用无锁编程来实现的，最常用的是采用CAS算法，Java原子类中的递增操作就是通过CAS自旋方式实现的。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/c8703cd9.png)



#### 1.1 乐观锁和悲观锁的代码示例：

```java
// ------------------------- 悲观锁的调用方式 -------------------------
// synchronized
public synchronized void testMethod() {
	// 操作同步资源
}
// ReentrantLock
private ReentrantLock lock = new ReentrantLock(); // 需要保证多个线程使用的是同一个锁
public void modifyPublicResources() {
	lock.lock();
	// 操作同步资源
	lock.unlock();
}

// ------------------------- 乐观锁的调用方式 -------------------------
private AtomicInteger atomicInteger = new AtomicInteger();  // 需要保证多个线程使用的是同一个AtomicInteger
atomicInteger.incrementAndGet(); //执行自增1
```



#### 1.2 **CAS的技术原理**

CAS全称 Compare And Swap（比较与交换），是一种无锁算法。在不加锁的情况下，实现多线程之间的变量同步。java.util.concurrent包中的原子类就是通过CAS来实现了乐观锁。

CAS涉及到的三个操作数：

- 需要读写的内存位置 V（对象o中的偏移量处的值v）

- 进行比较的值 A

- 要写入的值B

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/v2-d25bfd180fd5d1fc16090af61d813bb4_1440w.jpg" alt="img" style="zoom:60%;" />

当且仅当A的值等于V时，CAS通过原子方式用新值B来更新V的值（比较和更新整体是一个原子操作），否则不会执行任何操作。一般情况下，更新是一个不断重试的操作。



#### 1.3 **源码分析**

原子类AtomicInteger的源码：

```java
private static final long serialVersionUID = 6214790243416807050L;

// setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
  try {
    valueOffset = unsafe.objectFieldOffset
      (AtomicInteger.class.getDeclaredField("value"));
  } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

根据定义我们可以看出各属性的作用：

- unsafe： 获取并操作内存的数据。
- valueOffset： 存储value在AtomicInteger中的偏移量。
- value： 存储AtomicInteger的int值，该属性需要借助volatile关键字保证其在线程间是可见的。



接下来，我们查看AtomicInteger的自增函数incrementAndGet()的源码时，发现自增函数底层调用的是unsafe.getAndAddInt()。

```java
//class文件中的参数名，并不能很好的了解方法的作用，所以我们通过OpenJDK 8 来查看Unsafe的源码：

// ------------------------- JDK 8 -------------------------
// AtomicInteger 自增方法
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

// Unsafe.class
public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
      var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  return var5;
}

// ------------------------- OpenJDK 8 -------------------------
// Unsafe.java
//获取内存地址为obj+offset的变量值, 并将该变量值加上delta
public final int getAndAddInt(Object obj, long offset, int delta) {
  int v;
  do {
    //通过对象和偏移量获取变量的值
    //由于volatile的修饰, 所有线程看到的v都是一样的
    v= this.getIntVolatile(obj, offset);
    /*
      while中的compareAndSwapInt()方法尝试修改v的值,具体地, 该方法也会通过obj和offset获取变量的值
      如果这个值和v不一样, 说明其他线程修改了obj+offset地址处的值, 此时compareAndSwapInt()返回false, 继续循环
      如果这个值和v一样, 说明没有其他线程修改obj+offset地址处的值, 此时可以将obj+offset地址处的值改为v+delta,       compareAndSwapInt()返回true, 退出循环
      Unsafe类中的compareAndSwapInt()方法是原子操作, 所以compareAndSwapInt()修改obj+offset地址处的值的时候不会被其他线程中断
      */
  } while(!this.compareAndSwapInt(obj, offset, v, v + delta));

  return v;
}
```

**重点：**由于现在是处于多线程环境下，所以 v= this.getIntVolatile(obj, offset) 执行完这个指令之后，v的值可能会发生了变化，则while(!this.compareAndSwapInt(obj, offset, v, v + delta));这个就会返回false。

根据OpenJDK 8的源码我们可以看出，getAndAddInt()循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。整个“比较+更新”操作封装在compareAndSwapInt()中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

后续JDK通过CPU的cmpxchg指令，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止。



#### 1.4 **CAS虽然高效但是也会存在三个问题：**

1. **ABA问题**：CAS需要在操作值之前检查内存值是否被修改过，没有更新变化才会修改内存值。但是如果内存值原来是A，现在变成了B，接着又变回了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。
   - JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。
2. **循环等待长开销大**：CAS如果操作不成功，会导致一直自旋，给CPU带来巨大的开销。
3. **只能保证一个共享变量的原子操作**：对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。
   - Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。



#### 1.5 什么时候使用乐观锁？

资源提交冲突，其他使用方需要重新读取资源，会增加读的次数，但是可以面对高并发场景，前提是如果出现提交失败，用户是可以接受的。因此一般乐观锁只用在高并发、多读少写的场景。
 其中：GIT,SVN,CVS等代码版本控制管理器，就是一个乐观锁使用很好的场景，例如：A、B程序员，同时从SVN服务器上下载了code.html文件，当A完成提交后，此时B再提交，那么会报版本冲突，此时需要B进行版本处理合并后，再提交到服务器。这其实就是乐观锁的实现全过程。如果此时使用的是悲观锁，那么意味者所有程序员都必须一个一个等待操作提交完，才能访问文件，这是难以接受的。

#### 1.6 什么时候使用悲观锁？

一旦通过悲观锁锁定一个资源，那么其他需要操作该资源的使用方，只能等待直到锁被释放，好处在于可以减少并发，但是当并发量非常大的时候，由于锁消耗资源，并且可能锁定时间过长，容易导致系统性能下降，资源消耗严重。因此一般我们可以在并发量不是很大，并且出现并发情况导致的异常用户和系统都很难以接受的情况下，会选择悲观锁进行。





### 2. 自旋锁 VS 自适应自旋锁

阻塞或者唤醒一个线程需要操作系统切换CPU状态完成，这种状态转换是需要消耗处理机时间的。有可能切换CPU状态的时间比执行同步代码块的时间还要多。

在许多场景中，同步资源的锁定时间很短，为了这一小段时间去切换线程，线程挂起的时间和保护现场的花费会让系统得不偿失。因此，我们可以让后到的线程“稍等一下”，等短任务线程释放锁后就获得锁。“稍等一下”，我们需要当前线程进行自旋，如果自旋完成前面的线程释放了锁就可以获得锁，这样就可以避免线程的阻塞和唤醒，从而避免线程阻塞的开销。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/452a3363.png" alt="img" style="zoom:50%;" />

自旋锁本身是有缺点的，它不能代替阻塞。自旋等待虽然避免了线程切换的开销，但它要占用处理器时间。如果锁被占用的时间很短，自旋等待的效果就会非常好。反之，如果锁被占用的时间很长，那么自旋的线程只会白浪费处理器资源。所以，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数（默认是10次，可以使用-XX:PreBlockSpin来更改）没有成功获得锁，就应当挂起线程。

自旋锁的实现原理同样也是CAS，AtomicInteger中调用unsafe进行自增操作的源码中的do-while循环就是一个自旋操作，如果修改数值失败则通过循环来执行自旋，直至修改成功。

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
      var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  return var5;
}
```

自旋锁在JDK1.4.2中引入，使用-XX:+UseSpinning来开启。JDK 6中变为默认开启，并且引入了自适应的自旋锁（适应性自旋锁）。

自适应意味着自旋的时间（次数）不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

在自旋锁中 另有三种常见的锁形式:TicketLock、CLHlock和MCSlock。

### 3. 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁

![image-20220311194350084](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220311194350084.png)

#### 3.1 对象头、Monitor

Hotspot虚拟机对象头主要分为两个部分：Mark Word（标记字段）、Klass Pointer（类型指针）。其中Klass Pointer是对象指向其类原数据的指针，虚拟机通过这个指针确认该对象是哪个类的实例。Mark Word用于存储对象运行时的数据。

**Mark Word**

Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。Java对象头一般占有两个机器码（在32位虚拟机中，1个机器码等于4字节，也就是32bit），但是如果对象是数组类型，则需要三个机器码，因为JVM虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。下图是Java对象头的存储结构（32位虚拟机）：

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220311150545052.png" alt="image-20220311150545052" style="zoom:50%;" />

对象头信息是与对象自身定义的数据无关的额外存储成本，但是考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据，它会根据对象的状态复用自己的存储空间，也就是说，Mark Word会随着程序的运行发生变化，变化状态如下（32位虚拟机）：

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220311150531344.png" alt="image-20220311150531344" style="zoom:50%;" />

#### 3.2 无锁

无锁没有对资源进行锁定，所有的线程都能修改同一个资源，但同时只有一个线程能修改成功。

无锁的特点就是修改操作在循环内进行，如果没有冲突，则修改成功并退出循环；否则会不断重试，直到修改操作完成。CAS原理及其应用就是无锁的实现。

#### 3.3 偏向锁

HotSpot的作者经过研究证明，大多数情况下，锁不仅不存在多线程竞争，而且总是由同一个线程多次获得。所以出现了偏向锁，目的是为了同一个线程多次执行同步代码块时提高性能。偏向锁是指同一个同步资源被同一个线程多次访问时，能够自动获取锁，降低获取锁的代价。

当一个线程访问同步块并获取锁时，会在对象头的Mark Word的线程ID存储锁偏向的线程ID，以后该线程访问同步块时不需要进行CAS操作进行加锁和解锁操作，只需要简单地验证对象头的Mark Word是否存储着指向当前线程的偏向锁。

**引入偏向锁是为了在多线程竞争的情况下，减少不必要的轻量级锁的执行路径，因为轻量级锁的获取和释放操作需要执行多次的CAS原子指令，而偏向锁只需要在置换TreadID的时候执行一次CAS原子指令**。

拥有偏向锁的线程并不会主动去释放锁，只有遇到其他线程竞争偏向锁时才会释放偏向锁。

**偏向锁的获取过程（理解的时候线程1和线程2是同时创建的）：**

- 访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01——确认为可偏向状态。

- (2) 如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤（5），否则进入步骤（3）。
- (3) 如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行(5)；如果竞争失败，执行（4）。
- (4) 如果CAS获取偏向锁失败，则表示有竞争（CAS获取偏向锁失败说明至少有过其他线程曾经获得过偏向锁，因为线程不会主动去释放偏向锁）。当到达全局安全点（safepoint）时，会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着（因为可能持有偏向锁的线程已经执行完毕，但是该线程并不会主动去释放偏向锁），如果**线程不处于活动状态**，则将对象头设置成无锁状态（标志位为“01”），然后重新偏向新的线程；如果**线程仍然活着**，撤销偏向锁后升级到轻量级锁状态（标志位为“00”），此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁。
- (5) 执行同步代码。

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/10/16484ae78e43d14c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp" alt="img" style="zoom:50%;" />

**偏向锁的释放过程：**

如上步骤（4）

#### 3.4 轻量级锁

是指当锁是偏向锁的时候，被另外一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获得锁，不会阻塞，线程类似于交替执行一样，从而提高性能。

- 在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，官方称之为 Displaced Mark Word。这时候线程堆栈与对象头的状态如下图所示。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/v2-f038bb24708478e76d2240267bec56bf_1440w-20220311182629224.jpg)

- （2）拷贝对象头中的Mark Word复制到锁记录中。
- （3）拷贝成功后，虚拟机将**使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock record里的owner指针指向object mark word**。如果更新成功，则执行步骤（3），否则执行步骤（4）。
- （4）如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word的锁标志位设置为“00”，即表示此对象处于轻量级锁定状态，这时候线程堆栈与对象头的状态如下图所示。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/v2-5d58112c4ead326066a836941a2857c2_1440w.jpg)

- （5）如果这个更新操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行。否则说明多个线程竞争锁，若当前只有一个等待线程，则可通过自旋稍微等待一下，可能另一个线程很快就会释放锁。 **但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止CPU空转，锁标志的状态值变为“10”，Mark Word中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入阻塞状态**。

**轻量级锁的解锁过程：**

- （1）通过CAS操作尝试把线程中复制的Displaced Mark Word对象替换当前的Mark Word。
- （2）如果替换成功，整个同步过程就完成了。
- （3）如果替换失败，说明有其他线程尝试过获取该锁（此时锁已膨胀），那就要在释放锁的同时，唤醒被挂起的线程。



<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/11/164873116f6b3f99~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp" alt="img" style="zoom:50%;" />

指向重量级锁的指针实质是指向monitor对象的指针

#### 3.5 重量级锁

升级为重量级锁时，锁标志的状态值变为“10”，此时Mark Word中存储的是指向重量级锁的指针，此时等待锁的线程都会进入阻塞状态。

#### 3.6 总结

偏向锁通过对比Mark Word解决加锁问题，避免执行大量的CAS操作。而轻量级锁是通过用CAS操作避免线程阻塞和唤醒而影响性能。重量级锁是处于等待队列中的线程可能会饿死，或者等很久才会获得锁。



### 4. 公平锁和非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。

公平锁的优点是等待锁的线程不会饥饿。缺点是整体吞吐效率相对非公平锁要低，等待队列中的线程除了第一个线程以外的线程都处于阻塞状态，CPU唤醒线程锁的开销比非公平锁的开销大。

非公平锁就是处于运行态的线程先去尝试获得非公平锁，如果失败则将该线程加入到阻塞队列中等待被唤醒。但是如果该线程成功获得锁，那么这个线程无需阻塞，能够直接获得锁，减少了CPU唤醒线程的开销。所以非公平锁可能后到的线程先获得锁。

非公平锁的优点是可以减少CPU唤醒线程的开销，整体吞吐率高，因为线程有几率无需阻塞获得锁。缺点是可能处于等待队列中的线程很久才能获得锁，甚至出现饿死的情况。

直接用语言描述可能有点抽象，这里作者用从别处看到的一个例子来讲述一下公平锁和非公平锁。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/a23d746a.png)

如上图所示，假设有一口水井，有管理员看守，管理员有一把锁，只有拿到锁的人才能够打水，打完水要把锁还给管理员。每个过来打水的人都要管理员的允许并拿到锁之后才能去打水，如果前面有人正在打水，那么这个想要打水的人就必须排队。管理员会查看下一个要去打水的人是不是队伍里排最前面的人，如果是的话，才会给你锁让你去打水；如果你不是排第一的人，就必须去队尾排队，这就是公平锁。

但是对于非公平锁，管理员对打水的人没有要求。即使等待队伍里有排队等待的人，但如果在上一个人刚打完水把锁还给管理员而且管理员还没有允许等待队伍里下一个人去打水时，刚好来了一个插队的人，这个插队的人是可以直接从管理员那里拿到锁去打水，不需要排队，原本排队等待的人只能继续等待。如下图所示：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/4499559e.png)

**公平锁和非公平锁的实现方式：以ReentrantLock为例**

默认情况下，ReentrantLock使用的是非公平锁。

从实现来看，公平锁和非功能锁最大的区别是公平锁多了一个hasQueuedPredecessors()，用于检查队列的情况。

- 公平锁

  ```java
  static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;
  
    final void lock() {
      acquire(1);
    }
  
    /**
    * Fair version of tryAcquire.  Don't grant access unless
    * recursive call or no waiters or is first.
    * 返回true说明获得锁成功
    */
    protected final boolean tryAcquire(int acquires) {
      final Thread current = Thread.currentThread();
      int c = getState();
      if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {//如果队列为空或者当前线程在队列头，同时获得锁，则设置当前线程为运行线程
          setExclusiveOwnerThread(current);
          return true;
        }
      }
      else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
          throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
      }
      return false;
    }
  }
  ```

- 非公平锁

  ```java
  static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;
  
    /**
    * Performs lock.  Try immediate barge, backing up to normal
    * acquire on failure.
     */
    final void lock() {
      if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
      else
        acquire(1);
    }
  	/**
    * 返回true说明获得锁成功
     */
    protected final boolean tryAcquire(int acquires) {
      return nonfairTryAcquire(acquires);
    }
  }
  ```

- hasQueuedPredecessors()方法

  ```java
  /*
  Returns:
  true if there is a queued thread preceding the current thread, and false if the current thread is at the head of the queue or the queue is empty
  */
  public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
      ((s = h.next) == null || s.thread != Thread.currentThread());
  }
  ```

综上，公平锁通过同步队列来实现多个线程按照申请锁的顺序来获得锁，而非公共锁加锁时不考虑线程排队等待问题，直接尝试获得锁，所以存在后申请但先获得锁的情况。



### 5. 可重入锁和非可重入锁

可重入锁又名递归锁，是指在同一个线程在一个方法获取锁的时候，再进入该线程执行方法的内层方法会自动获得锁（前提锁对象得是同一个对象或者class），不会因为之前获得锁而阻塞。Java中的ReentrantLock和synchronized就是重入锁

```java
public class Widget {
    public synchronized void doSomething() {
        System.out.println("方法1执行...");
        doOthers();
    }

    public synchronized void doOthers() {
        System.out.println("方法2执行...");
    }
}
```

在上面的代码中，类中的两个方法都是被内置锁synchronized修饰的，doSomething()方法中调用doOthers()方法。因为内置锁是可重入的，所以同一个线程在调用doOthers()时可以直接获得当前对象的锁，进入doOthers()进行操作。

如果是一个不可重入锁，那么当前线程在调用doOthers()之前需要将执行doSomething()时获取当前对象的锁释放掉，实际上该对象锁已被当前线程所持有，且无法释放。所以此时会出现死锁。

**源码分析**

首先ReentrantLock和NonReentrantLock都继承父类AQS，其父类AQS中维护了一个同步状态status来计数重入次数，status初始值为0。

当线程尝试获取锁时，可重入锁先尝试获取并更新status值，如果status == 0表示没有其他线程在执行同步代码，则把status置为1，当前线程开始执行。如果status != 0，则判断当前线程是否是获取到这个锁的线程，如果是的话执行status+1，且当前线程可以再次获取锁。而非可重入锁是直接去获取并尝试更新当前status的值，**如果status != 0的话会导致其获取锁失败，当前线程阻塞。因为当前线程阻塞，所以无法释放锁，进而导致死锁发生。**

释放锁时，可重入锁同样先获取当前status的值，在当前线程是持有锁的线程的前提下。如果status-1 == 0，则表示当前线程所有重复获取锁的操作都已经执行完毕，然后该线程才会真正释放锁。而非可重入锁则是在确定当前线程是持有锁的线程之后，直接将status置为0，将锁释放。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/32536e7a.png" alt="img" style="zoom:50%;" />



### 6. 独享锁和共享锁

独享锁也叫排他锁，是指该锁一次只能被一次线程所持有。如果线程T对数据A加上排他锁，那么其他线程就不能对数据A加其他任何类型的锁。获得排他锁的线程既能读数据又能修改数据。JDK中的synchronized和JUC中Lock的实现类就是互斥锁。

共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排他锁。获得共享锁的线程只能读数据，不能修改数据。

#### 深入理解读写锁—ReadWriteLock源码分析