## RibbonDemo项目架构

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220513013358286.png" alt="image-20220513013358286" style="zoom:80%;" />

## 客户端

>Eureka自带Ribbon包

### 手写负载均衡算法开发流程

>1. ApplicationContextBean去掉注解@LoadBalanced
>2. LoadBalancer接口
>3. MyLB类
>4. OrderController
>5. 测试

#### LoadBalancer.class

```java
public interface LoadBalancer {
    ServiceInstance instances(List<ServiceInstance> serviceInstances);
}
```

#### MyLB.class

```java
@Component
public class MyLb implements LoadBalancer {

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    public final  int getAndIncrement(){
        int current;
        int next;
        do{
          	//重点学习一下java的魔法类
            current = this.atomicInteger.get();
            next = current >= 2147483647 ? 0 :current + 1;
        }while (!this.atomicInteger.compareAndSet(current,next));
        System.out.println("*****next"+next);
        return next;
    }

    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
        int index = getAndIncrement() % serviceInstances.size();
        return serviceInstances.get(index);
    }
}
```

#### OrderController.class

```java
@GetMapping(value = "/consumer/payment/lb")
public String getPaymentLB(){
  List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
  if(instances == null || instances.size() <= 0){
    return null;
  }
  ServiceInstance serviceInstance = loadBalancer.instances(instances);
  URI uri = serviceInstance.getUri();
  return restTemplate.getForObject(uri+"/payment/lb",String.class);
}
```

#### OrderMain80.class

```java
@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

#### application.yaml

```yaml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)(单位：毫秒)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

