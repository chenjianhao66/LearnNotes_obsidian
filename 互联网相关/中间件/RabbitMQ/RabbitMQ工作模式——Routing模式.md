2021-11-22
22:49:31
author:陈建浩
#Java #中间件 #RabbitMQ 

--- 

# RabbitMQ工作模式——Routing模式
在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望**不同的消息被不同的队列消费**。这时就要用到Direct类型的Exchange。

该Routing模式又细分为Direct模式和Topic模式

## Direct模式

在Direct模型下：

-   队列与交换机的绑定，不能是任意绑定了，而是要指定一个`RoutingKey`（路由key）
    
-   消息的发送方在 向 Exchange发送消息时，也必须指定消息的 `RoutingKey`。
    
-   **Exchange不再把消息交给每一个绑定的队列**，而是根据消息的`Routing Key`进行判断，只有队列的`Routingkey`与消息的 `Routing key`完全一致，才会接收到消息

Routing模式的工作示意图

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2263764-20210114135128942-500696537.png)

图解：

-   P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。
    
-   X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列
    
-   C1：消费者，其所在队列指定了需要routing key 为 error 的消息
    
-   C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息

### 示例

使用Direct模型的步骤如下：

-   创建连接工厂对象，设置对象参数
-   根据连接工厂对象获取连接对象
-   根据连接对象获取通道对象
-   获取通道对象之后，分生产者和消费者两种情况
    -   生产者
        -   声明交换机以及该交换机的类型（这里为`direct`）
        -   发送消息，绑定路由key
        -   关闭通道对象和连接对象
    -   消费者
        -   声明交换机以及该交换机的类型
        -   获取到一个临时队列
        -   通道对象绑定队列名称以及交换机
        -   消费消息，并且指定消费哪一种路由key的消息

#### 1、Maven依赖

**pom.xml**

```java
<dependency>  
     <groupId>com.rabbitmq</groupId>  
     <artifactId>amqp-client</artifactId>  
     <version>5.7.2</version>  
</dependency>
```

#### 2、定义生产者
```java
public class Provider {  
  
  
 public static void main(String[] args) throws IOException {  
         // 获取连接对线  
         Connection connection = RabbitmqUtils.getConnection();  
         Channel channel = connection.createChannel();  

         //声明交换机  
         channel.exchangeDeclare("directExchange","direct");  
		 
		 //设置key  
		 String key = "type";

         //发送信息  
         channel.basicPublish("directExchange","",null,("这是发送给["+key+"]的消息").getBytes());  
         
         //关闭资源  
         RabbitmqUtils.close(connection,channel);  
     }  
}

```


#### 3、定义消费者
```java

public class Customer {  
    public static void main(String[] args) throws IOException {  
         //获取连接对线  
         Connection connection = RabbitmqUtils.getConnection();  
         Channel channel = connection.createChannel();  

         //通道绑定交换机  
         channel.exchangeDeclare("directExchange","direct");  

         //获取临时队列 
         String tempQueueName = channel.queueDeclare().getQueue();  

         //绑定交换机、队列和路由key
         channel.queueBind(tempQueueName,"directExchange","type");  

         channel.basicConsume(tempQueueName,true,new DefaultConsumer(channel){  
             @Override public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
                 System.out.println("消费者1消费 ->["+new String(body)+"] 消息");
             }  
         });  
     }  
}

```

其他消费者代码无太多变动，只是对路由key做了改变


#### 测试
消费者1所消费的路由key为`type`的消息
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111222321179.png)

消费者2所消费的路由key为`type1`的消息
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111222321915.png)

消费者3所消费的路由key为`type3`的消息
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111222322132.png)

>需要注意的是，消费者可以绑定多个路由key，就是绑定交换机、队列和路由key的时候多次声明
>```java
>         //绑定交换机、队列和路由key
>         channel.queueBind(tempQueueName,"directExchange","type");  
>         channel.queueBind(tempQueueName,"directExchange","type1"); 
>         channel.queueBind(tempQueueName,"directExchange","type2"); 
>```

## Topic模式