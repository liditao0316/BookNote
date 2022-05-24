## 一、Spring的核心及其底层原理

### Spring IOC的理解

IoC（inversion of Control）是控制反转的意思，是一种面向对象编程的设计思想。如果不采用这种设计思想，我们要手动维护对象与对象之间的依赖关系，这种方式使得对象与对象之间耦合度过高，不利于代码的升级和维护。IoC则可以帮我们维护对象与对象之间的依赖关系，降低对象之间的耦合度。

IoC的实现原理是工厂模式和反射机制。其实这个工厂模式是对传统的工厂模式的一个优化，优化过后的工厂类能够实现对所有类的创建，因为优化后的工厂类通过反射机制完成对对象的创建工作，同时由于所有的类都继承自Object类，我们通过反射机制创建对象并返回Object对象之后，再将Object对象进行转换，就能完成对象的生成工作。

说到IoC，不得不说DI（dependency Injection），DI是依赖注入的意思，是IoC的一种实现方式，也就是说IoC是通过DI来实现的。实现依赖注入的关键是IoC容器，其注入方式有以下三种：

- 构造方法注入：
- setter注入：
- 接口注入：



### Spring AOP的理解

AOP（Aspect Oriented Programming）是面向切面编程思想，是对OOP的补充，它可以在OOP的基础上进一步提高编程的效率，也就是提高代码的重用。简单来说，它可以统一解决一批组件的共性问题（如权限检查，记录日志，事务管理等）。在AOP思想下，我们可以将解决共性需求的代码独立出来，然后通过配置的方式，声明这些代码在什么地方、什么时机调用。当满足调用条件时，AOP会将该业务代码织入到我们指定位置，从而统一解决了问题。

AOP是可以通过多种方式实现的。JDK动态代理，CGLib动态代理。

- JDK动态代理：这是Java提供的动态代理技术，可以在运行时`创建接口`的代理实例。Spring AOP默认采用这种方式，在接口的代理实例中织入代码。
- CGLib动态代理：采用底层的字节码技术，在运行时创建子类代理的实例。当目标对象不存在接口时，Spring AOP就会采用这种方式，在子类实例中织入代码。

JDK动态代理，也就是我们常说的动态代理模式。代理类的生成并不是在项目编译的时候，而是在运行时才生成代理类。动态代理模式也是通过反射机制实现的。动态代理模式的实现步骤主要分为：

1. 定义真实对象和代理对象的公共接口
2. 真实对象实现公共接口方法
3. 定义一个继承invocationHandler接口的类，重点是重写invoke()方法
4. 通过调用Proxy的newProxyInvocation()方法完成对代理对象的生成

以上是传统的动态代理模式的步骤，但是为了能够实现AOP的前置增强和后置增强的功能，我们还需要增加两个接口，一个是前置增强接口，还有一个是后置增强接口，在继承invocationHandler接口的类中将这两个接口作为该类的成员变量。当然，这是AOP的基本实现，在Spring的底层，要完成其他的大量的工作。



### 既然有有接口都可以采用CGLib，为什么还要考虑用JDK动态代理

- 在性能方面，CGLib创建代理对象比JDK动态代理性能高很多。但是在时间上，CGLib创建代理对象时所花费的时间比JDK动态代理多得多。所以，对于单例的对象因为无需频繁创建代理对象，采用CGLib动态代理比较合理。反之，对于多例的对象可能需要频繁的创建代理对象，则JDK动态代理更合适。

## 二、Spring Bean

### 介绍Bean的作用域

默认情况下，Bean在Spring容器中是单例的，我们可以通过修改@Scope注解修改Bean的作用域。

- singleton 单例模式
- protptype 多例模式
- request：为每一个request请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
- session：与request范围类似，同一个session会话共享一个实例，不同会话使用不同的实例。
- global-session：全局作用域，所有会话共享一个实例。如果想要声明让所有会话共享的存储变量的话，那么这全局变量需要存储在global-session中。

### Spring Bean的生命周期

Spring Bean的生命周期只有四个阶段：实例化 Instantiation --> 属性赋值 Populate --> 初始化 Initialization --> 销毁 Destruction

### Bean的线程安全问题

Spring容器本身并没有对bean提供线程安全策略，因此可以说Spring容器中的Bean本身是不具备线程的特性，但是还是要具体讨论的。

- 对于多例的bean，每次都创建一个新对象，也就是线程之间不存在Bean共享，因此不会有线程安全问题。
- 对于单例的bean，所有的线程都公用一个bean，因此存在线程安全问题。但是单例的bean可以分为，有状态的bean和无状态的bean。
  - 有状态的bean：就是能够存储数据的bean，是非线程安全的。
  - 无状态的bean：不能够存储数据的bean，是线程安全的，比如Controller类，Service类等。

- 对于单例的bean，如何解决线程安全的问题：
  - 采用ThreadLocal解决线程安全问题，为每一个线程提供一个独立的变量副本，不同线程只操作自己线程的副本。



## 三、Spring 容器

### Spring 容器的了解

Spring主要提供了两个容器，分别是BeanFactory和ApplicationContext

BeanFactory：是基础类型的IoC容器，对IoC提供了完整的技术支持。BeanFactory在默认情况下，采用延迟初始化容器的策略，只有当客户端对象需要访问某个容器中的受管对象时，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动速度较快，所需要的资源较少。

ApplicationContext：它是在BeanFactory容器的基础上创建的，是相对高级的容器实现，除了拥有BeanFactory容器的所有支持，ApplicationContext还提供其他高级特性，比如事件发布、国际化信息支持等。ApplicationContext管理的对象，在该类型容器启动之后， 默认全部初始化并完成绑定。所以相比于BeanFactory而言，ApplicationContext的启动速度更慢，所需要的资源也更多。



### BeanFactory的了解

BeanFactory是一个类工厂，与传统类工厂不同，它能创建并管理各种类的对象。在Spring中，这些被创建和管理的对象叫做bean。BeanFactory是Spring容器的底层接口，Spring为BeanFactory提供了多种实现，比如xmlBeanFactory。BeanFactory最主要的方法是getBean()



### Spring Bean的启动流程