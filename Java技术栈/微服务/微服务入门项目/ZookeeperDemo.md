## ZookeeperDemo项目架构

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220510094636643.png" style="zoom:50%;" />





## Zookeeper安装

#### 1、Zookeeper的下载

[Zookeeper点我下载](https://zookeeper.apache.org/releases.html)

![在这里插入图片描述](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NtYWxsX1lvZ3VydA==,size_16,color_FFFFFF,t_70.png)

#### 2、进入conf目录，修改文件名称

```shell
# 进入conf目录
cd conf/
# 复制一份，名为zoo.cfg
cp zoo_sample.cfg zoo.cfg
```

#### 3、修改zoo.cfg配置文件-可选

- 将dataDir修改为我们自己创建的data目录

  ```text
  /usr/local/apache-zookeeper-3.6.0-bin/data/
  ```

- 修改完成如下
  ![修改配置文件](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/20200410181831584.png)

#### 4、启动测试

```she
# 启动zookeeper服务
./zkServer.sh start

# 查看状态
./zkServer.sh status

# 停止服务
./zkServer.sh stop

```





## 微服务集群

### cloud-provider-payment8004

#### 开发流程

![image-20220510100617117](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220510100617117.png)

#### pom.xml

```xml
<!-- SpringBoot整合zookeeper客户端 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

#### application.yml

```yaml
server:
  port: 8004
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
    	#集群地址列表
      connect-string: 172.16.210.134:2181
```





## 客户端

### 开发流程

![image-20220510100617117](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220510100617117.png)

### pom.xml

```xml
<!-- SpringBoot整合zookeeper客户端 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

### application.yml

```yaml
server:
  port: 80
spring:
  application:
    name: cloud-consumer-order
  cloud:
    zookeeper:
      connect-string: 172.16.210.134:2181
```

### config

```java
//ApplicationContextConfig.class
package com.atguigu.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```

### Controller

```java
//OrderZkController
package com.atguigu.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
@Slf4j
public class OrderZkController {
    public static final String INVOKE_URL = "http://cloud-provider-payment";

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/zk")
    public String paymentInfo(){
        String forObject = restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
        System.out.println(forObject);
        return forObject;
    }
}
```

