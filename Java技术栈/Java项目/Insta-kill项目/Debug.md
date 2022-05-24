- 字段用``，描述信息用''
- 最后一个不能用","隔开

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220312201214892.png" alt="image-20220312201214892" style="zoom:50%;" />

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220312201246942.png" alt="image-20220312201246942" style="zoom:50%;" />



- 关于Springboot调错篇——Error creating bean with name 'sqlSessionFactoryBean--Mapper映射问题
  - 解决办法，查看Mapper.xml文件是否有错
  - 同时查看配置文件，看看配置文件的路径是否有误

```java
Factory method 'sqlSessionFactory' threw exception; nested exception is java.lang.NoClassDefFoundError: com/system/instakill/pojo/User (wrong name: com/system/instaKill/pojo/User)
```

```java
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'userServiceImpl': Unsatisfied dependency expressed through field 'baseMapper'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'userMapper' defined in file [/Users/me/IdeaProjects/Insta-kill/target/classes/com/system/instaKill/mapper/UserMapper.class]: Unsatisfied dependency expressed through bean property 'sqlSessionFactory'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sqlSessionFactory' defined in class path resource [com/baomidou/mybatisplus/autoconfigure/MybatisPlusAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.ibatis.session.SqlSessionFactory]: Factory method 'sqlSessionFactory' threw exception; nested exception is java.lang.NoClassDefFoundError: com/system/instakill/pojo/User (wrong name: com/system/instaKill/pojo/User)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:659) ~[spring-beans-5.3.16.jar:5.3.16]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:639) ~[spring-beans-5.3.16.jar:5.3.16]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:119) ~[spring-beans-5.3.16.jar:5.3.16]

```



- 

![image-20220313143522182](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220313143522182.png)

## 起因

SpringBoot 2.3.0版本之后就没有引入validation对应的包

## 解决方法

导入Spring Boot Starter Validation，根据自己项目所使用的版本号导入。同时添加如下配置

```xml
<dependency>
  <groupId>javax.validation</groupId>
  <artifactId>validation-api</artifactId>
  <version>2.0.1.Final</version>
</dependency>
```





- 远程无法访问redis的解决方案

  - 检查阿里云的安全组是否加入6379端口

    <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220314084253286.png" alt="image-20220314084253286" style="zoom:50%;" />

  - 检查配置文件

    ```java
      daemonize no
      修改为：
      daemonize yes
    
      注释这一行：
      #bind 127.0.0.1
      
      daemonize no（后台启动）
      修改为
      daemonize yes
    ```

  - 修改服务器的端口号

    ```shell
    #开启防火墙
    systemctl start firewalld.service
    
    #打开6379端口
    firewall-cmd --add-prot = 6379/tcp
    
    #测试端口是否打开
    firewall-cmd ---query-port=6379/tcp
    ```

    



- 使用拦截器无法加载静态资源的解决办法

  - 配置类的优先级大于配置文件的，所以如果有配置类，则在加载容器时，以配置类中的配置为准。因此要在配置类对静态资源 进行处理。

    ```java
    		@Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(loginInterceptor).addPathPatterns("/**")
                    .excludePathPatterns("/login/**")
                    .excludePathPatterns("/**/**.js")
                    .excludePathPatterns("/**/**.css")
                    .excludePathPatterns("/**/**.html");
    
        }
    		//plan B
    		@Override
    		public void addResourceHandlers()
    ```

    





- **Spring @transactional注解和synchronized同时使用出现的并发问题(可重复读问题)和解决办法**

主要原因：事务范围大于锁

```java
@Override
@Transactional
public synchronized CouponsH5Respons getACoupons() {
 		.....
}
```



用jmeter进行压力测试的时候,发现优惠券超发
当时很疑惑,synchronized已经放在了方法级别，不应该出现并发的问题

#### 正常情况

`每个线程的事物都是不同的,T1修改后commit T2读取到新数据`

#### 超发场景

出现超发的情况,T2 读取到了旧数据
可以分析出T2在T1没commit之前读取的数据，因为两者在不同事务，T2读取的一定是旧数据
从而分析出synchronized没生效
：

#### 问题原因(事务范围大于锁)

因为spring的AOP的特性，会在进入方法之前开启事务，之后再加锁，当锁住的代码执行完成后，再提交事务，如果在T1执行commit之前 有其他线程进来，读取的一定是旧数据
如下图：
同步代码块是在事务的内部

![image-20220315105123885](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220315105123885.png)

#### 解决方式：

将synchronized关键字加入到Controller层，使synchronized锁的范围大于事务控制的范围。
目的是让同步代码在事务的外面

```java
Object mLock = new Object();
@RequestMapping(value = "test")
@ResponseBody
public void test() throws Exception {
    synchronized (mLock) {
       service.test();
    }
    responseMessage(ModelResult.CODE_200,ModelResult.SUCCESS);
}

```





- **RabbitMQ无法登陆成功的解决办法**

  - RabbitMQ HTTP API request 401 Unauthorized

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220315142358158.png" alt="image-20220315142358158" style="zoom:50%;" />

  - 解决办法：

    - 添加用户名和密码

    - 授予用户权限

      **新增用户**

      ```shell
      rabbitmqctl add_user admin admin
      ```

      **设置用户分配操作权限**

      ```
      rabbitmqctl set_user_tags admin administrator
      ```





- **前端Cookie的Value字段为undefined**

  - 原因

    domain字段没有正确设置为当前服务器的域名

  - 解决办法

    检查CookieUtil工具类的相应处理办法

  - 以下是Cookie各字段的相应信息

  |     name     |  value   |         domain         |            path            | expires/Max-Age |     Size     |              http               |                secure                 |
  | :----------: | :------: | :--------------------: | :------------------------: | :-------------: | :----------: | :-----------------------------: | :-----------------------------------: |
  | cookie的名字 | cook的值 | 可以访问此cookie的域名 | 可以访问此cookie的页面路径 | cookie超时时间  | cookie的大小 | 前端是否能通过j修改cookie的信息 | 设置是否只能通过https来传递此条cookie |

  