## RabbitMQ高可用集群方案

> RabbitMQ的Cluster模式分为两种：
>
> - 普通模式
> - 集群模式

### Cluster普通模式

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220323214718135.png" alt="image-20220323214718135" style="zoom:50%;" />

元数据包含一下内容：

- 队列元数据：队列的名称及属性
- 交换机：交换机的名称及属性
- 绑定关系元数据：交换机与队列 或者 交换机与交换机