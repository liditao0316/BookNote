# 一、Spring IoC

## 1、IOC理论推导

UserDao接口

```java
public interface UserDao {
    void getUser();
}
```

UserDaoImpl 实现类

```java
public class UserDaoImpl implements UserDao {
    public void getUser() {
        System.out.println("默认获取用户数据");
    }
}
```

UserService 业务接口

```java
public interface UserService {
    void getUser();
}
```

UserServiceImpl 业务实现类

```java
public class UserServiceImpl implements UserService {

    private UserDao userDao = new UserDaoImpl();

    public void getUser() {
        userDao.getUser();
    }
}
```

测试

```java
public class MyTest {
    public static void main(String[] args) {

        //用户实际调用的是业务层，dao层他们不需要接触！
        UserService userService = new UserServiceImpl();
        userService.getUser();
    }
}
```

在我们之前的业务中，用户的需求可能会影响我们原来的代码，我们需要根据用户的需求去修改原代码！如果程序代码量十分大，修改一次的成本代价十分昂贵。

![img](https://img-blog.csdnimg.cn/20201109112133683.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpNjQzOTM3NTc5,size_16,color_FFFFFF,t_70#pic_center)

我们使用一个Set接口实现，已经发生了革命性的变化！

```java
    private UserDao userDao;

    //利用set进行动态实现值的注入！
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
```

- 之前，程序是主动创建对象！控制权在程序猿手上！
- 使用了set注入后，程序不再具有主动性，而是变成了被动的接收对象！

这种思想，从本质上解决了问题，我们程序猿不用再去管理对象的创建了。系统的耦合性大大降低~，可以更加专注的在业务的实现上！这是IOC的原型！

![img](https://img-blog.csdnimg.cn/20201109112157443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpNjQzOTM3NTc5,size_16,color_FFFFFF,t_70#pic_center)

## 2、IOC本质

- **接口的使用，减少了代码量，也易于后期的维护**
- **反转指的是控制权发生了反转，即不是由调用者的程序代码控制，控制权由调用者转移到Spring容器**
- **从Spring容器角度来看，Spring容器负责将被依赖对象赋值给调用者的成员变量，相当于为调用者注入它所依赖的实例，这就是Spring的控制反转**
- **控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是IoC容器，其实现方式是依赖注入。**

控制反转IoC(Inverison of Control)，是一种设计思想，DI (dependency injection依赖注入) 是实现IoC的一种方法。未使用IOC的程序中，我们使用面向对象编程，对象的创建和对象之间的依赖关系完全硬编码在程序中，对象的创建是由程序自己控制的。控制反转就是将对象的创建转移给了第三方。



## 3、Spring IoC容器

实现控制反转的是Spring IoC容器。Spring IoC容器的设计主要是基于BeanFactory和ApplicationContext两个接口。



## 4、依赖注入

​	**依赖：bean对象的创建依赖于容器**

​	**注入：bean对象中的所有属性，由容器来注入**

- 构造器注入

  1. 使用无参构造创建对象，默认！

  2. 假设我们要使用有参构造创建对象。

     - 下标赋值

     ```xml
     <!--第一种方式：下标赋值    -->
     <bean id="user" class="com.kuang.pojo.User">
         <constructor-arg index="0" value="狂神说Java"/>
     </bean>
     ```

     - 类型

     ```xml
     <!--第二种方式：通过类型的创建，不建议使用    -->
     <bean id="user" class="com.kuang.pojo.User">
         <constructor-arg type="java.lang.String" value="lifa"/>
     </bean>
     ```

     - 参数名

     ```xml
     <!--第三种方式：直接通过参数名来设置    -->
     <bean id="user" class="com.kuang.pojo.User">
         <constructor-arg name="name" value="李发"/>
     </bean>
     ```

- Set方式注入

  ```java
  public class Student {
  
      private String name;
      private Address address;
      private String[] books;
      private List<String> hobbies;
      private Map<String,String> card;
      private Set<String> games;
      private String wife;
      private Properties info;
  }
  ```

  ```xml
      <bean id="address" class="com.kuang.pojo.Address">
          <property name="address" value="西安"/>
      </bean>
  
      <bean id="student" class="com.kuang.pojo.Student">
          <!--第一种：普通值注入，value        -->
          <property name="name" value="黑心白莲"/>
  
          <!--第二种：        -->
          <property name="address" ref="address"/>
  
          <!--数组        -->
          <property name="books">
              <array>
                  <value>红楼梦</value>
                  <value>西游记</value>
                  <value>水浒传</value>
                  <value>三国演义</value>
              </array>
          </property>
  
          <!--List        -->
          <property name="hobbies">
              <list>
                  <value>打篮球</value>
                  <value>看电影</value>
                  <value>敲代码</value>
              </list>
          </property>
  
          <!--Map        -->
          <property name="card">
              <map>
                  <entry key="身份证" value="123456789987456321"/>
                  <entry key="银行卡" value="359419496419481649"/>
              </map>
          </property>
  
          <!--Set        -->
          <property name="games">
              <set>
                  <value>LOL</value>
                  <value>COC</value>
                  <value>BOB</value>
              </set>
          </property>
  
          <!--NULL        -->
          <property name="wife">
              <null/>
          </property>
  
          <!--Properties        -->
          <property name="info">
              <props>
                  <prop key="driver">20191029</prop>
                  <prop key="url">102.0913.524.4585</prop>
                  <prop key="user">黑心白莲</prop>
                  <prop key="password">123456</prop>
              </props>
          </property>
  
      </bean>
  
  ```

- 拓展方式注入

  

# 二、Spring Bean

## 1、Bean的作用域

- singleton作用域

  Spring Ioc容器仅生成和管理一个Bean实例

- prototype作用域

  Spring IoC容器将为每次请求创建一个新的实例



## 2、Bean的装配方式

- 基于注解的装配

  1. @Component

     ```java
     package annotation;
     
     @Component()
     /**
     相当于@Component("annotationUser")
     annotationUser为Bean的id，默认为首字母小写的类名
     **/
     public class AnnotationUser(){
       @value("chenheng")
       private String uname;
       /**省略setter和getter方法**/
     }
     ```

     ```xml
     <context:component-scan base-package="annotation"/>
     ```

  2. @Repository

     注解数据访问层（DAO）bean

  3. @Service

     标注一个业务逻辑组件类（Service层）

  4. @Controller

     标注一个控制器组件类（Spring MVC的Controller）

  5. @Autowired

     该注解可以对类的成员变量、方法及构造方法进行标注，完成自动装配的工作。通过使用@Autowired来消除setter和getter方法。如果成员变量是一个类，则不需要搭配@Resource和@Qualifier使用。

     ```java
     package annotation.controller;
     
     @Controller()
     public class TestController{
       @Autowired
       private TestService testService;
       //如果显式定义了Autowired的required属性为false，说明这个对象可以为null否则不允许为空
       @Autowired(required = false)
       private Cat cat
       public void save(){
         testService.save();
       }
     }
     
     ```

     由于annotation.dao、annotation.service、annotation.controller包都属于annotation包的子包，因此不需要在配置文件中配置注解。

  6. @Resource

  7. @Qualifier

     如果bean装配了相同的类，只是id不一样而已，那么使用@Autowired无法正常注入属性，则需要@Qualifier注解的配合

     ```java
     package annotation.controller;
     
     @Controller()
     public class TestController{
       @Autowired
       @Qualifier(value = "cat")
       private TestService testService;
       private Cat cat
       public void save(){
         testService.save();
       }
     }
     ```

     

  8. @Nullable

     ```java
     @Nullable 字段标记了这个注解，允许该值为空
     public People(@Nullable String name){
       this.name = name;
     }
     
     ```

     

  

# 三、Spring AOP

## 1、代理模式

### 介绍

- 意图：为其他对象提供一种代理以控制对这个对象的访问。

- 主要解决：在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

- 何时使用：想在访问一个类时做一些控制。

- 如何解决：增加中间层。

- 关键代码：实现与被代理类组合。

### 静态代理模式

我们将创建一个 *Image* 接口和实现了 *Image* 接口的实体类。*ProxyImage* 是一个代理类，减少 *RealImage* 对象加载的内存占用。

*ProxyPatternDemo* 类使用 *ProxyImage* 来获取要加载的 *Image* 对象，并按照需求进行显示。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211112163729816.png" alt="image-20211112163729816" style="zoom:50%;" />

- Image.java

```java
public interface Image {
   void display();
}
```

- RealImage.java

```java
public class RealImage implements Image {
 
   private String fileName;
 
   public RealImage(String fileName){
      this.fileName = fileName;
      loadFromDisk(fileName);
   }
 
   @Override
   public void display() {
      System.out.println("Displaying " + fileName);
   }
 
   private void loadFromDisk(String fileName){
      System.out.println("Loading " + fileName);
   }
}
```

- ProxyImage.java

```java
public class ProxyImage implements Image{
 
   private RealImage realImage;
   private String fileName;
 
   public ProxyImage(String fileName){
      this.fileName = fileName;
   }
 
   @Override
   public void display() {
      if(realImage == null){
         realImage = new RealImage(fileName);
      }
      realImage.display();
   }
}
```

当被请求时，使用 *ProxyImage* 来获取 *RealImage* 类的对象。

- ProxyPatternDemo.java

```java
public class ProxyPatternDemo {
   
   public static void main(String[] args) {
      Image image = new ProxyImage("test_10mb.jpg");
 
      // 图像将从磁盘加载
      image.display(); 
      System.out.println("");
      // 图像不需要从磁盘加载
      image.display();  
   }
}
```



### 动态代理模式

```java
package com.kuang.demo03;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyInvocationHandler implements InvocationHandler {
    //被代理的接口
    private Object target;

    public void setTarget(Object target){
        this.target = target;
    }

    //生成得到代理类 
    /*
    * @param:
    * loader 类加载器来定义代理类
    * interface 代理类实现的接口列表
    * 
    */
    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(),this);
    }
		 /*
    * @param:
    * proxy 调用该方法的代理实例
    * method 所述方法对应于调用代理实例上的接口方法的实例
    * args 包含的方法调用传递代理实例的参数值的对象的阵列
    */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log(method.getName());
        Object result = method.invoke(target,args);
        return result;
    }

    public void log(String msg){
        System.out.println("执行了"+msg+"方法");
    }
}
```

```java
package com.kuang.demo03;

import com.kuang.demo02.UserServiceImpl;
import com.kuang.demo02.Userservice;

public class Client {
    public static void main(String []args){
        UserServiceImpl userService = new UserServiceImpl();
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        pih.setTarget(userService);
        Userservice proxy = (Userservice) pih.getProxy();
        proxy.update();
    }
}
```

## 2、什么是AOP

AOP（Aspect Oriented Programming）意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211112165118986.png" alt="image-20211112165118986" style="zoom:50%;" />

- 横切关注点：跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。如日志 , 安全 , 缓存 , 事务等等 ....
- 切面（ASPECT）：横切关注点 被模块化 的特殊对象。即，它是一个类。
- 通知（Advice）：切面必须要完成的工作。即，它是类中的一个方法。
- 目标（Target）：被通知对象。
- 代理（Proxy）：向目标对象应用通知之后创建的对象。
- 切入点（PointCut）：切面通知 执行的 “地点”的定义。
- 连接点（JointPoint）：与切入点匹配的执行点。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211112165215778.png" alt="image-20211112165215778" style="zoom:50%;" />

SpringAOP中，通过Advice定义横切逻辑，Spring中支持5种类型的Advice:

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211112165515269.png" alt="image-20211112165515269" style="zoom:50%;" />

### Spring API 实现

首先编写我们的业务接口和实现类

```java
public interface UserService {

   public void add();

   public void delete();

   public void update();

   public void search();

}

```

```java
public class UserServiceImpl implements UserService{

   @Override
   public void add() {
       System.out.println("增加用户");
  }

   @Override
   public void delete() {
       System.out.println("删除用户");
  }

   @Override
   public void update() {
       System.out.println("更新用户");
  }

   @Override
   public void search() {
       System.out.println("查询用户");
  }
}
```

然后去写我们的增强类 , 我们编写两个 , 一个前置增强 一个后置增强

```java
public class Log implements MethodBeforeAdvice {

   //method : 要执行的目标对象的方法
   //objects : 被调用的方法的参数
   //Object : 目标对象
   @Override
   public void before(Method method, Object[] objects, Object o) throws Throwable {
       System.out.println( o.getClass().getName() + "的" + method.getName() + "方法被执行了");
  }
}
```

```java
public class AfterLog implements AfterReturningAdvice {
   //returnValue 返回值
   //method被调用的方法
   //args 被调用的方法的对象的参数
   //target 被调用的目标对象
   @Override
   public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
       System.out.println("执行了" + target.getClass().getName()
       +"的"+method.getName()+"方法,"
       +"返回值："+returnValue);
  }
}
```

spring xml的配置

```xml
<!--aop的配置-->
   <aop:config>
       <!--切入点 expression:表达式匹配要执行的方法-->
       <aop:pointcut id="pointcut" expression="execution(* com.kuang.service.UserServiceImpl.*(..))"/>
       <!--执行环绕; advice-ref执行方法 . pointcut-ref切入点-->
       <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
       <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
   </aop:config>
```

Aop的重要性 : 很重要 . 一定要理解其中的思路 , 主要是思想的理解这一块 .

Spring的Aop就是将公共的业务 (日志 , 安全等) 和领域业务结合起来 , 当执行领域业务时 , 将会把公共业务加进来 . 实现公共业务的重复利用 . 领域业务更纯粹 , 程序猿专注领域业务 , 其本质还是动态代理 . 



### 自定义类

目标业务类不变依旧是userServiceImpl

第一步 : 写我们自己的一个切入类

```java
public class DiyPointcut {

   public void before(){
       System.out.println("---------方法执行前---------");
  }
   public void after(){
       System.out.println("---------方法执行后---------");
  }
   
}
```

spring配置

```xml
<!--第二种方式自定义实现-->
<!--注册bean-->
<bean id="diy" class="com.kuang.config.DiyPointcut"/>

<!--aop的配置-->
<aop:config>
   <!--第二种方式：使用AOP的标签实现-->
   <aop:aspect ref="diy">
       <aop:pointcut id="diyPonitcut" expression="execution(* com.kuang.service.UserServiceImpl.*(..))"/>
       <aop:before pointcut-ref="diyPonitcut" method="before"/>
       <aop:after pointcut-ref="diyPonitcut" method="after"/>
   </aop:aspect>
</aop:config>
```



### 使用注解实现

第一步：编写一个注解实现的增强类

```java
package com.kuang.config;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class AnnotationPointcut {
   @Before("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void before(){
       System.out.println("---------方法执行前---------");
  }

   @After("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void after(){
       System.out.println("---------方法执行后---------");
  }

   @Around("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void around(ProceedingJoinPoint jp) throws Throwable {
       System.out.println("环绕前");
       System.out.println("签名:"+jp.getSignature());
       //执行目标方法proceed
       Object proceed = jp.proceed();
       System.out.println("环绕后");
       System.out.println(proceed);
  }
}
```

第二步：在Spring配置文件中，注册bean，并增加支持注解的配置

```xml
<!--第三种方式:注解实现-->
<bean id="annotationPointcut" class="com.kuang.config.AnnotationPointcut"/>
<aop:aspectj-autoproxy/>
```

aop:aspectj-autoproxy：说明

```xml
通过aop命名空间的<aop:aspectj-autoproxy />声明自动为spring容器中那些配置@aspectJ切面的bean创建代理，织入切面。当然，spring 在内部依旧采用AnnotationAwareAspectJAutoProxyCreator进行自动代理的创建工作，但具体实现的细节已经被<aop:aspectj-autoproxy />隐藏起来了

<aop:aspectj-autoproxy />有一个proxy-target-class属性，默认为false，表示使用jdk动态代理织入增强，当配为<aop:aspectj-autoproxy  poxy-target-class="true"/>时，表示使用CGLib动态代理技术织入增强。不过即使proxy-target-class设置为false，如果目标类没有声明接口，则spring将自动使用CGLib动态代理。
```



# 四、Spring的事务管理

## 1、Spring的数据库编程

### Spring JDBC 的配置

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/user?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="12345678"/>
    </bean>
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
```

配置JDBC模板是需要将dataSource注入到脚都不错Template，而在数据访问层（Dao类）需要使用jdbcTemplate注入到对应的Bean中

### Spring  JdbcTemplate的常用方法

```java
String insertSql="insert into user values(null,?,?)";
Object param[] = {"chen","男"};
jdbcTemplate.update(sql,param);
```

```java
String selectSql = "select *from user";
RowMapper<MyUser> rowmapper = new BeanPropertyRowMapper<MuUser>(MyUser.class);
List<MyUser> list = jdbcTemplate.query(sql,rowMapper,null);
//rowMapper将结果集映射到用户自定义的类中，前提是自定义类中的属性要与数据表的字段对应
```

## 2、编程式事务管理

## 3、声明式事务管理

​		Spring的声明式事务管理是通过AOP技术实现的事务管理，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者执行回滚事务。

​		与编程式声明事务管理相比，声明式管理唯一不足的地方是最细粒度只能作用到方法级别，无法做到像编程式事务管理那样可以作用到代码级别。

### 基于XML方式的声明式事务管理

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211113203839695.png" alt="image-20211113203839695" style="zoom:50%;" />

StatementController

```java
package com.statement.controller;

import com.statement.service.TestService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller("statementController")
public class StatementController {
    @Autowired
    private TestService testService;
    public String test(){
        String message = "";
        String deleteSql = "delete from user";
        String saveSql = "insert into user value(?,?,?)";
        Object param[] = {6,"chenheng",1233};
        try{
            testService.delete(deleteSql,null);
            testService.save(saveSql,param);
            testService.save(saveSql,param);
        }catch (Exception e){
            message = "主键重复，事务回滚";
            e.printStackTrace();
        }
        return message;
    }
}
```

TestDao

```java
package com.statement.dao;

import org.springframework.stereotype.Repository;


public interface TestDao {
    public int save(String sql,Object param[]);
    public int delete(String sql,Object param[]);
}
```

TestDaoImpl

```java
package com.statement.dao;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository("testDao")
public class TestDaoImpl implements TestDao{

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public int save(String sql, Object[] param) {
        return jdbcTemplate.update(sql,param);
    }

    @Override
    public int delete(String sql, Object[] param) {
        return jdbcTemplate.update(sql,param);
    }
}
```

TestService

```java
package com.statement.service;

public interface TestService {
    public int save(String sql, Object param[]);
    public int delete(String sql,Object param[]);
}
```

TestServiceImpl

```java
package com.statement.service;

import com.statement.dao.TestDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service("testService")
public class TestServiceImpl implements TestService{
    @Autowired
    private TestDao testDao;

    @Override
    public int save(String sql, Object[] param) {
        return testDao.save(sql,param);
    }

    @Override
    public int delete(String sql, Object[] param) {
        return testDao.delete(sql,param);
    }
}
```

Application.xml

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd 
        http://www.springframework.org/schema/context 
        https://www.springframework.org/schema/context/spring-context.xsd">
  
    <context:component-scan base-package="com.statement"/>
  
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/user?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="12345678"/>
    </bean> 
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
  <!--重点-->
  <!--为数据添加事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--编写通知声明事务-->
 	  <!--需要指定事务管理器-->
    <tx:advice id="myAdvice" transaction-manager="txManager">
        <!--指定事务的细节,指定哪些方法执行事务-->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
		<!--编写AOP，让Spring自动对目标对象生成代理，需要使用AspectJ的表达式-->
    <aop:config>
      	<!--定义切入点-->
        <aop:pointcut id="txPointCut" expression="execution(* com.statement.service.*.*())"/>
      	<!--切面：将切入点与通知关联-->
        <aop:advisor advice-ref="myAdvice" pointcut-ref="txPointCut"/>
    </aop:config>
</beans>
```



### 基于@Transactional注解



## 4、核心依赖包

```xml
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
  <version>1.8.14</version>
</dependency>
```







# 五、Mybatis

Mybatis项目结构

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211113094303538.png" alt="image-20211113094303538" style="zoom:50%;" />

UserMapper

```java
package com.kuang.mapper;

import com.kuang.pojo.User;

import java.util.List;

public interface UserMapper {
    public List<User> selectUser();
}
```

UserMapper.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.mapper.UserMapper">
    <select id="selectUser" resultType="user">
        select * from user.user;
    </select>
</mapper>
```

User

```java
package com.kuang.pojo;

import lombok.Data;

@Data
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

Mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="com.kuang.pojo"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/user?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="12345678"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper class="com.kuang.mapper.UserMapper"/>
    </mappers>
</configuration>
```

Pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>Spring</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-mybatis</artifactId>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.18.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.4</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
        </dependency>
    </dependencies>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>

</project>
```



## 1、入门

本章将会以简略的步骤告诉你如何安装和配置 MyBatis-Spring，并构建一个简单的具备事务管理功能的数据访问应用程序。

### 安装

要使用 MyBatis-Spring 模块，只需要在类路径下包含 `mybatis-spring-2.0.6.jar` 文件和相关依赖即可。

如果使用 Maven 作为构建工具，仅需要在 pom.xml 中加入以下代码即可：

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>2.0.6</version>
</dependency>
```

### 快速上手

要和 Spring 一起使用 MyBatis，需要在 Spring 应用上下文中定义至少两样东西：一个 `SqlSessionFactory` 和至少一个数据映射器类。

在 MyBatis-Spring 中，可使用 `SqlSessionFactoryBean`来创建 `SqlSessionFactory`。 要配置这个工厂 bean，只需要把下面代码放在 Spring 的 XML 配置文件中：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
```

```java
@Configuration
public class MyBatisConfig {
  @Bean
  public SqlSessionFactory sqlSessionFactory() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource());
    return factoryBean.getObject();
  }
}
```

注意：`SqlSessionFactory` 需要一个 `DataSource`（数据源）。这可以是任意的 `DataSource`，只需要和配置其它 Spring 数据库连接一样配置它就可以了。

假设你定义了一个如下的 mapper 接口：

```java
public interface UserMapper {
  @Select("SELECT * FROM users WHERE id = #{userId}")
  User getUser(@Param("userId") String userId);
}
```

那么可以通过 `MapperFactoryBean` 将接口加入到 Spring 中:

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

需要注意的是：所指定的映射器类**必须**是一个接口，而不是具体的实现类。在这个示例中，通过注解来指定 SQL 语句，但是也可以使用 MyBatis 映射器的 XML 配置文件。

配置好之后，你就可以像 Spring 中普通的 bean 注入方法那样，将映射器注入到你的业务或服务对象中。`MapperFactoryBean` 将会负责 `SqlSession` 的创建和关闭。 如果使用了 Spring 的事务功能，那么当事务完成时，session 将会被提交或回滚。最终任何异常都会被转换成 Spring 的 `DataAccessException` 异常。

使用 Java 代码来配置的方式如下：

```java
@Configuration
public class MyBatisConfig {
  @Bean
  public UserMapper userMapper() throws Exception {
    SqlSessionTemplate sqlSessionTemplate = new SqlSessionTemplate(sqlSessionFactory());
    return sqlSessionTemplate.getMapper(UserMapper.class);
  }
}
```

要调用 MyBatis 的数据方法，只需一行代码：

```java
public class FooServiceImpl implements FooService {

  private final UserMapper userMapper;

  public FooServiceImpl(UserMapper userMapper) {
    this.userMapper = userMapper;
  }

  public User doSomeBusinessStuff(String userId) {
    return this.userMapper.getUser(userId);
  }
}
```

## 2、SqlSessionFactoryBean

在基础的 MyBatis 用法中，是通过 `SqlSessionFactoryBuilder` 来创建 `SqlSessionFactory` 的。而在 MyBatis-Spring 中，则使用 `SqlSessionFactoryBean` 来创建。

### 设置

要创建工厂 bean，将下面的代码放到 Spring 的 XML 配置文件中：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
```

需要注意的是 `SqlSessionFactoryBean` 实现了 Spring 的 `FactoryBean` 接口（参见 Spring 官方文档 3.8 节 [通过工厂 bean 自定义实例化逻辑](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-extension-factorybean) ）。 这意味着由 Spring 最终创建的 bean **并不是** `SqlSessionFactoryBean` 本身，而是工厂类（`SqlSessionFactoryBean`）的 getObject() 方法的返回结果。这种情况下，Spring 将会在应用启动时为你创建 `SqlSessionFactory`，并使用 `sqlSessionFactory` 这个名字存储起来。

等效的 Java 代码如下：

```java
@Configuration
public class MyBatisConfig {
  @Bean
  public SqlSessionFactory sqlSessionFactory() {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource());
    return factoryBean.getObject();
  }
}
```

通常，在 MyBatis-Spring 中，你不需要直接使用 `SqlSessionFactoryBean` 或对应的 `SqlSessionFactory`。 相反，session 的工厂 bean 将会被注入到 `MapperFactoryBean` 或其它继承于 `SqlSessionDaoSupport` 的 DAO（Data Access Object，数据访问对象）中。

### 属性

`SqlSessionFactory` 有一个唯一的必要属性：用于 JDBC 的 `DataSource`。这可以是任意的 `DataSource` 对象，它的配置方法和其它 Spring 数据库连接是一样的。

一个常用的属性是 `configLocation`，它用来指定 MyBatis 的 XML 配置文件路径。它在需要修改 MyBatis 的基础配置非常有用。通常，基础配置指的是 `<settings>` 或 `<typeAliases>`元素。

需要注意的是，这个配置文件**并不需要**是一个完整的 MyBatis 配置。确切地说，任何环境配置（`<environments>`），数据源（`<DataSource>`）和 MyBatis 的事务管理器（`<transactionManager>`）都会被**忽略**。 `SqlSessionFactoryBean` 会创建它自有的 MyBatis 环境配置（`Environment`），并按要求设置自定义环境的值。

如果 MyBatis 在映射器类对应的路径下找不到与之相对应的映射器 XML 文件，那么也需要配置文件。这时有两种解决办法：第一种是手动在 MyBatis 的 XML 配置文件中的 `<mappers>` 部分中指定 XML 文件的类路径；第二种是设置工厂 bean 的 `mapperLocations` 属性。

`mapperLocations` 属性接受多个资源位置。这个属性可以用来指定 MyBatis 的映射器 XML 配置文件的位置。属性的值是一个 Ant 风格的字符串，可以指定加载一个目录中的所有文件，或者从一个目录开始递归搜索所有目录。比如:

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />
</bean>
```

这会从类路径下加载所有在 `sample.config.mappers` 包和它的子包中的 MyBatis 映射器 XML 配置文件。

在容器管理事务的时候，你可能需要的一个属性是 `transactionFactoryClass`。请参考事务一章的相关章节。

如果你使用了多个数据库，那么需要设置 `databaseIdProvider` 属性：

```xml
<bean id="databaseIdProvider" class="org.apache.ibatis.mapping.VendorDatabaseIdProvider">
  <property name="properties">
    <props>
      <prop key="SQL Server">sqlserver</prop>
      <prop key="DB2">db2</prop>
      <prop key="Oracle">oracle</prop>
      <prop key="MySQL">mysql</prop>
    </props>
  </property>
</bean>
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />
  <property name="databaseIdProvider" ref="databaseIdProvider"/>
</bean>
```

**提示** 自 1.3.0 版本开始，新增的 `configuration` 属性能够在没有对应的 MyBatis XML 配置文件的情况下，直接设置 `Configuration` 实例。例如：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="configuration">
    <bean class="org.apache.ibatis.session.Configuration">
      <property name="mapUnderscoreToCamelCase" value="true"/>
    </bean>
  </property>
</bean>
```

## 3、使用 SqlSession

在 MyBatis 中，你可以使用 `SqlSessionFactory` 来创建 `SqlSession`。 一旦你获得一个 session 之后，你可以使用它来执行映射了的语句，提交或回滚连接，最后，当不再需要它的时候，你可以关闭 session。 使用 MyBatis-Spring 之后，你不再需要直接使用 `SqlSessionFactory` 了，因为你的 bean 可以被注入一个线程安全的 `SqlSession`，它能基于 Spring 的事务配置来自动提交、回滚、关闭 session。

### SqlSessionTemplate

`SqlSessionTemplate` 是 MyBatis-Spring 的核心。作为 `SqlSession` 的一个实现，这意味着可以使用它无缝代替你代码中已经在使用的 `SqlSession`。 `SqlSessionTemplate` 是线程安全的，可以被多个 DAO 或映射器所共享使用。

当调用 SQL 方法时（包括由 `getMapper()` 方法返回的映射器中的方法），`SqlSessionTemplate` 将会保证使用的 `SqlSession` 与当前 Spring 的事务相关。 此外，它管理 session 的生命周期，包含必要的关闭、提交或回滚操作。另外，它也负责将 MyBatis 的异常翻译成 Spring 中的 `DataAccessExceptions`。

由于模板可以参与到 Spring 的事务管理中，并且由于其是线程安全的，可以供多个映射器类使用，你应该**总是**用 `SqlSessionTemplate` 来替换 MyBatis 默认的 `DefaultSqlSession` 实现。在同一应用程序中的不同类之间混杂使用可能会引起数据一致性的问题。

可以使用 `SqlSessionFactory` 作为构造方法的参数来创建 `SqlSessionTemplate` 对象。

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

```java
@Configuration
public class MyBatisConfig {
  @Bean
  public SqlSessionTemplate sqlSession() throws Exception {
    return new SqlSessionTemplate(sqlSessionFactory());
  }
}
```

现在，这个 bean 就可以直接注入到你的 DAO bean 中了。你需要在你的 bean 中添加一个 SqlSession 属性，就像下面这样：

```java
public class UserDaoImpl implements UserDao {

  private SqlSession sqlSession;

  public void setSqlSession(SqlSession sqlSession) {
    this.sqlSession = sqlSession;
  }

  public User getUser(String userId) {
    return sqlSession.selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
  }
}
```

按下面这样，注入 `SqlSessionTemplate`：

```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
  <property name="sqlSession" ref="sqlSession" />
</bean>
```

`SqlSessionTemplate` 还有一个接收 `ExecutorType` 参数的构造方法。这允许你使用如下 Spring 配置来批量创建对象，例如批量创建一些 SqlSession：

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
  <constructor-arg index="1" value="BATCH" />
</bean>
```

```java
@Configuration
public class MyBatisConfig {
  @Bean
  public SqlSessionTemplate sqlSession() throws Exception {
    return new SqlSessionTemplate(sqlSessionFactory(), ExecutorType.BATCH);
  }
}
```

现在所有的映射语句可以进行批量操作了，可以在 DAO 中编写如下的代码

```java
public class UserService {
  private final SqlSession sqlSession;
  public UserService(SqlSession sqlSession) {
    this.sqlSession = sqlSession;
  }
  public void insertUsers(List<User> users) {
    for (User user : users) {
      sqlSession.insert("org.mybatis.spring.sample.mapper.UserMapper.insertUser", user);
    }
  }
}
```

注意，只需要在希望语句执行的方法与 `SqlSessionTemplate` 中的默认设置不同时使用这种配置。

这种配置的弊端在于，当调用这个方法时，不能存在使用不同 `ExecutorType` 的进行中的事务。要么确保对不同 `ExecutorType` 的 `SqlSessionTemplate` 的调用处在不同的事务中，要么完全不使用事务。

### SqlSessionDaoSupport

`SqlSessionDaoSupport` 是一个抽象的支持类，用来为你提供 `SqlSession`。调用 `getSqlSession()` 方法你会得到一个 `SqlSessionTemplate`，之后可以用于执行 SQL 方法，就像下面这样:

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {
  public User getUser(String userId) {
    return getSqlSession().selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
  }
}
```

在这个类里面，通常更倾向于使用 `MapperFactoryBean`，因为它不需要额外的代码。但是，如果你需要在 DAO 中做其它非 MyBatis 的工作或需要一个非抽象的实现类，那么这个类就很有用了。

`SqlSessionDaoSupport` 需要通过属性设置一个 `sqlSessionFactory` 或 `SqlSessionTemplate`。如果两个属性都被设置了，那么 `SqlSessionFactory` 将被忽略。

假设类 `UserMapperImpl` 是 `SqlSessionDaoSupport` 的子类，可以编写如下的 Spring 配置来执行设置：

```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```



## 4、步骤

1. 编写数据源配置
2. sqlSessionFactory
3. sqlSessionTemplate
4. 需要给接口加实现类
5. 将自己写的实现类，注入到spring中



## 5、实例

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211113160146698.png" alt="image-20211113160146698" style="zoom:50%;" />

UserMapper

```java
package com.kuang.mapper;

import com.kuang.pojo.User;

import java.util.List;

public interface UserMapper {
    public List<User> selectUser();
}
```

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.mapper.UserMapper">
    <select id="selectUser" resultType="user">
        select * from user.user;
    </select>
</mapper>
```

其他操作（删除、插入、更新）

```xml
<mapper namespace="work7.dao.UserDao">
    <select id="findAll" resultType="user">
        select * from user.user;
    </select>
   <select id="findOne" parameterType="Integer" resultType="user">
       select * from user.user where id = #{id}
   </select>
    <insert id="addUser" parameterType="user">
        insert into user(name,account) values(#{name},#{account});
    </insert>
    <delete id="delUser" parameterType="Integer">
        delete from user where id=#{id}
    </delete>
    <update id="updateUser" parameterType="work7.pojo.UserAccount">
        update user set account=account+#{m} where id = #{uid}
    </update>
</mapper>
```



UserMapperImpl

```java
package com.kuang.mapper;

import com.kuang.pojo.User;
import org.apache.ibatis.session.SqlSession;

import java.util.List;

public class UserMapperImpl implements UserMapper{
    private SqlSession sqlSession;

    public void setSqlSession(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public List<User> selectUser() {
        UseMapper mapper = sqlSession.getMapper(UseMapper.class);
        return mapper.selectUser();
    }
}
```

User

```java
package com.kuang.pojo;

import lombok.Data;

@Data
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

MyTest

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class MyTest {

    @Test
    public void test() throws IOException {
        ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        UserMapper userMapper = (UserMapper) context.getBean("userMapper");
        for(User user: userMapper.selectUser()){
            System.out.println(user);
        }
    }
}
```

Application.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">
    <import resource="spring-dao.xml"/>

    <bean id="userMapper" class="com.kuang.mapper.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>
</beans>
```

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="com.kuang.pojo"/>
    </typeAliases>

</configuration>
```

Spring-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/user?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="12345678"/>
    </bean>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="mapperLocations" value="classpath:com/kuang/mapper/*.xml"/>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
</beans>
```

Pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>Spring</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-mybatis</artifactId>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.18.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.4</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
        </dependency>
    </dependencies>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>

</project>
```

## 6、Mybatis核心依赖包

```xml
	<dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis</artifactId>
       <version>3.5.2</version>
   </dependency>
   <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis-spring</artifactId>
       <version>2.0.2</version>
   </dependency>
	<dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>5.1.47</version>
   </dependency>
```



# 六、Spring核心依赖包

```xml
<dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-webmvc</artifactId>
       <version>5.1.9.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-jdbc</artifactId>
       <version>5.1.9.RELEASE</version>
   </dependency>
```

