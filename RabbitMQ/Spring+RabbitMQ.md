## Spring+RabbitMQ RPC



### pom.xml

```xml
<dependency>
  <groupId>org.springframework.amqp</groupId>
  <artifactId>spring-amqp</artifactId>
  <version>2.2.5.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework.amqp</groupId>
  <artifactId>spring-rabbit</artifactId>
  <version>2.2.5.RELEASE</version>
</dependency>
```



### 客户端

#### 配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/rabbit
           https://www.springframework.org/schema/rabbit/spring-rabbit.xsd
           http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">


    <rabbit:connection-factory id="connectionFactory" virtual-host="${rabbit.virtual-host}" username="${rabbit.username}"
                               password="${rabbit.password}" port="${rabbit.port}" host="${rabbit.host}"/>

    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory" reply-timeout="10000"/>
    <rabbit:admin connection-factory="connectionFactory"/>

</beans>
```

#### 发送端

```java
package easymall.service;


import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.nio.charset.Charset;
import java.util.HashMap;
import java.util.UUID;

@Service
public class TestServiceImpl{

    @Resource
    private RabbitTemplate rabbitTemplate;

    public String send(String message) {
        // 封装Message，直接发送message对象
        final String corrId = UUID.randomUUID().toString();
        Message newMessage = convertMessage(corrId,message);

        Message result = rabbitTemplate.sendAndReceive("topic-exchange", "order", newMessage);
        String response = "";
        if (result != null) {
            String correlationId = result.getMessageProperties().getCorrelationId();

            // 客户端从回调队列获取消息，匹配与发送消息correlationId相同的消息为应答结果
            if (corrId.equals(correlationId)) {
                // 提取RPC回应内容body
                response = new String(result.getBody());
                System.out.println(response);
            }
        }
        return response;
    }

    /**
     * 将发送消息封装成Message
     *
     * @param message
     * @return org.springframework.amqp.core.Message
     * @Author Liuyongfei
     * @Date 下午1:23 2020/5/27
     **/
    public Message convertMessage(String corrId,String message) {
        MessageProperties mp = new MessageProperties();
        byte[] src = message.getBytes(Charset.forName("UTF-8"));
        // 注意：由于在发送消息的时候，系统会自动生成消息唯一id，因此在这里手动设置的方式是无效的
        // CorrelationData correlationId = new CorrelationData(UUID.randomUUID().toString());
        mp.setCorrelationId(corrId);
        mp.setContentType("application/json");
        mp.setContentEncoding("UTF-8");
        mp.setContentLength((long) message.length());
        return new Message(src, mp);
    }
}

```



### 服务端

#### 配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/rabbit
           https://www.springframework.org/schema/rabbit/spring-rabbit.xsd
           http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <rabbit:connection-factory id="connectionFactory" virtual-host="${rabbit.virtual-host}" username="${rabbit.username}"
    password="${rabbit.password}" port="${rabbit.port}" host="${rabbit.host}"/>

    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory" reply-timeout="10000"/>
    <bean class="easymall.controller.TestServiceImpl" id="testService"/>
    <rabbit:admin connection-factory="connectionFactory" />
    <rabbit:listener-container connection-factory="connectionFactory" acknowledge="manual">
        <rabbit:listener ref="testService" queue-names="order"/>
    </rabbit:listener-container>
</beans>
```

#### 接收端

```java
package easymall.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import easymall.po.Orders;
import easymall.po.RabbitmqData;
import easymall.service.OrderService;
import lombok.SneakyThrows;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageListener;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.listener.api.ChannelAwareMessageListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.nio.charset.Charset;
import java.sql.Timestamp;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

@Component
public class TestServiceImpl implements ChannelAwareMessageListener {

    @Autowired
    private OrderService orderService;

    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        MessageProperties messageProperties = message.getMessageProperties();
        try {
            String msg = new String(message.getBody(), Charset.defaultCharset());

            //TODO 接收到消息业务处理
            String json = "下单成功"; //业务返回的结果
            System.out.println(message);
            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
            ObjectMapper objectMapper = new ObjectMapper();
            RabbitmqData rabbitmqData = objectMapper.readValue(message.getBody(), RabbitmqData.class);
            System.out.println(rabbitmqData);
            String orderId=UUID.randomUUID().toString();
            SimpleDateFormat df =new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String time=df.format(new Date());
            Timestamp timeStamp= Timestamp.valueOf(time);
            Orders myOrder = new Orders(orderId,0.0,rabbitmqData.getReceiverinfo(),0,timeStamp, rabbitmqData.getUser().getId(),0,0);
            orderService.addOrder(rabbitmqData.getCartIds(),myOrder);
          	//----业务逻辑结束---
          	
            AMQP.BasicProperties props = new AMQP.BasicProperties
                    .Builder()
                    .correlationId(messageProperties.getCorrelationId()) //重要
                    .build();
            String replyTo = messageProperties.getReplyTo(); //重要
            channel.basicPublish("", replyTo, props, json.getBytes(Charset.defaultCharset())); //这里的交换机为空
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

```



- exchange模式是topic模式