>2020-02-18 11:07:26.864 ERROR 12160 --- [nio-8080-exec-3] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Handler dispatch failed; nested exception is java.lang.StackOverflowError] with root cause
>
>java.lang.StackOverflowError: null
>at com.hzy.service.UserServiceImpl.queryUserByName(UserServiceImpl.java:20) ~[classes/:na]
>at com.hzy.service.UserServiceImpl.queryUserByName(UserServiceImpl.java:20) ~[classes/:na]
>    at com.hzy.service.UserServiceImpl.queryUserByName(UserServiceImpl.java:20) ~[classes/:na]

**检查service层是否是自己调用了自己，形成了死递归** 



>org.springframework.context.ApplicationContextException: Unable to start web server; nested exception is org.springframework.boot.web.server.WebServerException: Unable to start embedded Tomcat
>	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.onRefresh(ServletWebServerApplicationContext.java:156) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:544) ~[spring-context-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:141) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:747) [spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:397) [spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.SpringApplication.run(SpringApplication.java:315) [spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1226) [spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1215) [spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at com.atguigu.springcloud.EureakApplcation.main(EureakApplcation.java:11) [classes/:na]
>Caused by: org.springframework.boot.web.server.WebServerException: Unable to start embedded Tomcat
>	at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.initialize(TomcatWebServer.java:126) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.<init>(TomcatWebServer.java:88) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory.getTomcatWebServer(TomcatServletWebServerFactory.java:438) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory.getWebServer(TomcatServletWebServerFactory.java:191) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.createWebServer(ServletWebServerApplicationContext.java:180) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.onRefresh(ServletWebServerApplicationContext.java:153) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	... 8 common frames omitted
>Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'traceFilterRegistration' defined in class path resource [org/springframework/cloud/netflix/eureka/server/EurekaServerAutoConfiguration.class]: Unsatisfied dependency expressed through method 'traceFilterRegistration' parameter 0; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'javax.servlet.Filter' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Qualifier(value=httpTraceFilter)}
>	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:798) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:539) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1338) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1177) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:557) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:517) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:323) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:321) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:207) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.ServletContextInitializerBeans.getOrderedBeansOfType(ServletContextInitializerBeans.java:211) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.ServletContextInitializerBeans.getOrderedBeansOfType(ServletContextInitializerBeans.java:202) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.ServletContextInitializerBeans.addServletContextInitializerBeans(ServletContextInitializerBeans.java:96) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.ServletContextInitializerBeans.<init>(ServletContextInitializerBeans.java:85) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.getServletContextInitializerBeans(ServletWebServerApplicationContext.java:253) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.selfInitialize(ServletWebServerApplicationContext.java:227) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.springframework.boot.web.embedded.tomcat.TomcatStarter.onStartup(TomcatStarter.java:53) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5135) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1384) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1374) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at java.util.concurrent.FutureTask.run(FutureTask.java:266) ~[na:1.8.0_91]
>	at org.apache.tomcat.util.threads.InlineExecutorService.execute(InlineExecutorService.java:75) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:134) ~[na:1.8.0_91]
>	at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:909) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.StandardHost.startInternal(StandardHost.java:841) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1384) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1374) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at java.util.concurrent.FutureTask.run(FutureTask.java:266) ~[na:1.8.0_91]
>	at org.apache.tomcat.util.threads.InlineExecutorService.execute(InlineExecutorService.java:75) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:134) ~[na:1.8.0_91]
>	at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:909) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.StandardEngine.startInternal(StandardEngine.java:262) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.StandardService.startInternal(StandardService.java:421) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.core.StandardServer.startInternal(StandardServer.java:930) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.apache.catalina.startup.Tomcat.start(Tomcat.java:459) ~[tomcat-embed-core-9.0.29.jar:9.0.29]
>	at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.initialize(TomcatWebServer.java:107) ~[spring-boot-2.2.2.RELEASE.jar:2.2.2.RELEASE]
>	... 13 common frames omitted
>Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'javax.servlet.Filter' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Qualifier(value=httpTraceFilter)}
>	at org.springframework.beans.factory.support.DefaultListableBeanFactory.raiseNoMatchingBeanFound(DefaultListableBeanFactory.java:1695) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1253) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1207) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:885) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:789) ~[spring-beans-5.2.2.RELEASE.jar:5.2.2.RELEASE]
>	... 53 common frames omitted

**这是因为springboot和spingcloud的版本不匹配导致的。更改springcloud的版本后，重启服务。**

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/b73c1a92d6d14b37825456a7deeaeab7.png)

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/4f06ad8926eb4045a8b87c31f9ba359c.png)
