## 一、SpringBoot简介

## 二、SpringBoot优势

### 1、独立运行

### 2、直接内嵌Tomcat、Jetty等

- Spring Boot项目不需要打包成war包部署到容器中，Spring Boot只要打成一个可执行的jar包就能独立运行，所有以来包都在一个jar包中

### 3、简化配置

- spring- boot- starter- web启动器自动依赖其他组件，减少了maven的配置

### 4、自动配置

- Spring Boot能根据当前类路径下的类、jar包来自动配置bean，如添加一个spring-boot-starter-web启动器就能拥有web的功能，无需其他配置

### 5、无代码生成和XML配置

### 6、应用监控

## 三、基础入门

SpringBoot项目的核心依赖在父包中

![image-20220222184845005](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220222184845005.png)

- 启动器