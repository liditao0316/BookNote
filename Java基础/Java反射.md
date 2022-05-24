### 一、反射机制问题

- 如何根据配置文件properties指定信息，创建对象并调用方法

  ```pro
  classfullpath = work.pojo.car
  method = hi
  ```

- 这样的需求在学习框架时特别多，即通过外部文件配置，在不修改源码情况下控制程序，符合设计模式的ocp原则

  

### 二、 快速入门

```java
package work.pojo.test;

import work.pojo.Car;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

public class test {

    public static void main(String[] args) throws Exception {
        String classAllPath = "work.pojo.Car";
        //加载类
        Class<?> cls = Class.forName(classAllPath);
        System.out.println(cls);
        //创建对象
        Car car = (Car) cls.getDeclaredConstructor().newInstance();
        System.out.println(car);
        //获取成员变量对象
        Field brand = cls.getField("brand");
        System.out.println(brand.get(car));
        Car car2 = (Car)cls.getDeclaredConstructor().newInstance();
        brand.set(car,"宝马");
        brand.set(car2,"大众");
        System.out.println(brand.get(car));
        System.out.println(brand.get(car2));

    }
}
```



### 三、反射原理图

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211202203204384-8448333.png" style="zoom:50%;" />

### 四、反射相关类

- Java反射机制可以完成
  1. 在运行时判断任意一个对象所属的类
  2. 在运行时构造任意一个类的对象
  3. 在运行时得到任意一个类所具有的成员变量和方法
  4. 在运行时可调用任意一个对象的成员变量和方法
  5. 生成动态代理



- 反射相关的主要类

  1. java.lang.Class 代表一个类，Class对象表示某个类加载后在堆中的对象

  2. java.lang.reflect.Method 代表类的方法，Method对象表示某个类的方法

  3. Java.lang.reflect.Field 代表类的成员变量，Field对象表示某个类的成员变量

  4. java.lang.reflect.Constructor 代表类的构造方法，Constructor对象表示一个构造器

     ```java
     				//获取构造器对象
             //()中可以指定构造器参数类型
             Constructor<?> constructor = cls.getConstructor();
             Constructor<?> constructor1 = cls.getConstructor(String.class,String.class);
             //通过获取到的构造器对象创建类对象
             Car car3 = (Car) constructor.newInstance();
             System.out.println(car3);
             System.out.println(constructor);
             System.out.println(constructor1);
     ```

     

### 五、反射调用优化

- 反射优点和缺点

  1. 优点：可以动态的创建和使用对象(也是框架底层核心)，使用灵活，没有反射机制，框架技术就失去底层支撑

  2. 缺点：使用反射基本是解释执行，对执行速度有影响

     

- 反射调用优化 -- 关闭访问检查

  1. Method 和 Field、Constructor对象都有setAccessible()方法

  2. setAccessible作用是启动和禁用访问安全检查的开关

  3. 参数值为true，表示反射的对象在使用时取消访问检查，提高反射效率；参数值为false，表示反射的对象执行访问检查

     ```java
     Field brand = cls.getField("brand");
     brand.setAccessible(true);
     //对于性能来说可以稍微提高一点
     ```



### 六、Class类分析

- Class类也是一个java类，也继承Object类

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203000222094.png" alt="image-20211203000222094" style="zoom:50%;" />

- Class类对象不是new出来的，而是系统创建的。因为Class类是通过类的加载器的LoadClass()方法生成的

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203000727811.png" alt="image-20211203000727811" style="zoom:50%;" />

- 对于某个类的Class类对象，在内存中只有一份，因为类只被加载一次

  ```java
  //new类对象的过程中，系统将会完成Class类的加载
  Car car = new Car();
  Class<?> cls = Class.forName(classAllPath);
  ```

- 通过类的实例能够获取到与之相对应的Class类实例

  ![image-20211203002654904](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203002654904.png)

- 通过Class以及相应的API可以完整地获得一个类的结构

- Class对象是存放在堆里的

- 类的字节码二进制数据，是存放在方法区的。字节码二进制数据也称为类的元数据，包括方法代码、变量名、方法名、访问权限等。方法区会引用Class类对象的数据

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203003701603.png" alt="image-20211203003701603" style="zoom:50%;" />

### 七、获取Class对象六种方法

#### 1、编译阶段

- 前提：已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法forName()获取，可能抛出ClassNotFoundException
- 实例：Class cls = Class.forName("java.lang.Cat")
- 应用场景：多用于配置文件，读取类全路径，加载类

​	Class.forName()

#### 2、加载阶段

- 前提：若已知具体的类，通过类的class获取，该方式最为安全可靠，程序性能最高
- 实例：Class cls = Cat.class
- 应用场景：多用于参数传递，比如通过反射得到对应构造器对象

​	类.class

#### 3、Runtime阶段

- 前提：已知某个类的实例，调用该实例的getClass()方法获取Class对象
- 实例：Class cls = 对象.getClass();
- 应用场景：通过创建好的对象，获取Class对象

​	对象.getClass()

#### 4、通过类加载器获取Class对象

```java
//注意每个类的加载器只有一个
//1、先得到类加载器 car
ClassLoader classLoader = car3.getClass().getClassLoader();
//2、通过类加载器得到Class对象
Class cls = classLoader.loadClass(classAllPath);
```

#### 5、基本数据类型

```java
Class<Integer> integerClass = int.class;
Class<Character> characterClass = char.class;
```

#### 6、基本数据类型对应的包装类

```java
Class<Integer> type = Integer.TYPE;
```

```java
Class<String> stringClass = String.class;//外部类
Class<Serializable> serializableClass = Serializable.class;//接口
Class<Integer[]> aClass = Integer[].class;//一维数组
Class<float[][]> aClass1 = float[][].class;//二维数组
Class<Deprecated> deprecatedClass = Deprecated.class;//注解
Class<Thread.State> stateClass = Thread.State.class;//枚举
```



## 八、类加载

### 1、基本说明

反射机制是java实现动态语言的关键，也就是通过反射实现类动态加载

无论是静态加载还是动态加载，都需要经过编译阶段，生成.class文件。区别在于，静态加载在编译阶段就已经完成了类的加载工作，而动态加载在运行阶段才将所需要的类通过类加载器生成相应的Class类对象

- 静态加载：编译时加载相关的类，如果没有则报错，依赖性太强

- 动态加载：运行时加载需要的类，如果运行时不用该类，则不报错，降低了依赖性

  ```java
  Scanner scanner = new Scanner(System.in);
  System.out.println("请输入key");
  String key = scanner.next();
  switch(key){
    case "1":
      Car car_ = new Car();//静态加载，依赖性很强
      car_.hi();
      break;
    case "2":
      Class<?> aClass2 = Class.forName(classAllPath);//动态加载
      Object o = aClass2.getDeclaredConstructor().newInstance();
      Method method = aClass2.getMethod("hi");
      method.invoke(o);
      break;
  }
  ```


### 2、类加载过程

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203151529520-8515731.png" alt="image-20211203151529520-8515731" style="zoom:50%;" />





#### 第一步 加载Loading阶段

- 通过类的全限定名（包名 + 类名），获取到该类的`.class`文件的二进制字节流。
- 将二进制字节流所代表的静态存储结构，转化为方法区运行时的数据结构。
- 在`内存`中生成一个代表该类的`java.lang.Class`对象，作为方法区这个类的各种数据的访问入口。

总结：`加载二进制数据到内存` —> `映射成jvm能识别的结构` —> `在内存中生成class文件`。

####  

#### 第二步 Linking阶段

- 链接是指将上面创建好的class类合并至Java虚拟机中，使之能够执行的过程，可分为`验证`、`准备`、`解析`三个阶段。



##### （1）验证

​			目的是确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全

​			包括：文件格式验证(是否以魔数 oxcafebabe开头)、元数据验证、字节码验证和符号引用验证

​			可以考虑使用 -Xverify:none 参数来关闭大部分的类验证措施，缩短虚拟机类加载的时间



##### （2）准备

JVM会在该阶段对静态变量，分配内存并默认初始化（对应数据类型的默认初始值，如0、0L、null、false等）。这些变量所使用的内存都将在方法区中进行分配。

```java
/**
     * n1是实例属性，不是静态变量，因此在准备阶段，是不会分配内存
     * n2是静态变量，分配内存n2是默认初始化0，而不是20，在初始化阶段才会将20赋值给n2
     * n3是static final，是一个常量，他和静态变量不一样，因为一旦赋值就不变 n3 = 30
     */
    public int n1 = 0;
    public static int n2 = 20;
    public static final int n3 = 30;//准备阶段分配内存空间并赋值
```



##### （3）解析

​	虚拟机将常量池内的符号引用替代为直接引用的过程

```
符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《Java 虚拟机规范》的Class文件格式中。
直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
```



#### 第三步 Initialization(初始化)

​	（1）到初始化阶段，才真正开始执行类中定义的java程序代码(也就是复制等操作)，此阶段是执行 clinit()方法的过程

​	（2）clinit()方法是由编译器**按语句在源文件中出现的顺序**，依次自动收集类中的所有**静态变量**的赋值动作和**静态代码块**中的语句，并进行合并

```java
    /**
     *依次自动收集类中的所有静态变量的赋值动作和静态代码块中的语句
     * clinit(){
     *     System.out.println("B 静态代码块被执行");
     *     num = 300;
     *     num = 100;
     * }
     *	最终num的值为100
     */
    class B{
        static {
            System.out.println("B 静态代码块被执行");
            num = 300;
        }

        static int num = 100;
    }

```

​	（3）虚拟机会保证一个类的 clinit() 方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的 clinit() 方法，其他线程都需要阻塞等待，直到活动线程执行clinit() 完毕



### 九、反射爆破

setAccessible(true)，设置私有类型成员为可访问的，然后就可以获取到类的私有成员变量

```java
cls.getDeclaredConstructor()//获取所有构造器，包括共有的和私有的
cls.getConstructor()//获取共有的构造器
Field name = cls.getDeclaredField("name");//获取私有成员属性
name.setAccessible(true);

//如果要访问静态成员变量
Field n1 = cls.getDeclaredField("n1");
n1.setAccessible(true);
System.out.println(n1.get(null));//obj参数可以为空
```



