## 概述

### **Feign**

Feign是Spring Cloud组件中的一个轻量级`RESTful`的`HTTP服务客户端`

Feign内置了Ribbon，用来做**客户端负载均衡**，去调用服务注册中心的服务

Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务

**Feign本身不支持spring MVC 的注解，它有一套自己的注解**

### **OpenFeign**

OpenFeign是spring cloud在feign的基础上支持了spring MVC注解，如`@RequestMapping`等等。

OpenFeign的`@FeignClient`可以解析SpringMVC的`@RequestMapping`注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。