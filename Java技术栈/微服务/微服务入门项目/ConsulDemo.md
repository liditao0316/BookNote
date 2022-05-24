## ConsulDemo架构

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220510102156326.png" alt="image-20220510102156326" style="zoom:50%;" />





## consul安装

[consul下载地址](https://www.consul.io/downloads)

### 启动命令

```shell
./consul agent -dev           # -dev表示开发模式运行，另外还有-server表示服务模式运行
```





## 微服务集群

### cloud-provider-payment

#### 开发流程

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220508173733490.png" alt="image-20220508173733490" style="zoom:50%;" />

#### pom.xml

```xml
 <!--SpringCloud consul-server -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul-discovery</artifactId>
  <version>2.2.2.RELEASE</version>
</dependency>
```

#### application.yml

```yaml
###consul服务端口号
server:
  port: 8005

spring:
  application:
    name: cloud-provider-payment
  ####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

