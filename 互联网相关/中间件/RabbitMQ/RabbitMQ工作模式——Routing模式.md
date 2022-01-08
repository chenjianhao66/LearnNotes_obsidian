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
>  //绑定交换机、队列和路由key
>channel.queueBind(tempQueueName,"directExchange","type");  
>channel.queueBind(tempQueueName,"directExchange","type1"); 
> channel.queueBind(tempQueueName,"directExchange","type2"); 
>```

## Topic模式
`Topic`类型的`Exchange`与`Direct`相比，都是可以根据`RoutingKey`把消息路由到不同的队列。只不过`Topic`类型`Exchange`可以让队列在绑定`Routing key` 的时候使用通配符！这种模型`Routingkey` 一般都是由一个或多个单词组成，多个单词之间以”.”分割，例如： `item.insert`

这里面涉及到2个字符：`*` 和 `#`
	* (star) can substitute for exactly one word.    匹配不多不少恰好1个词
	# (hash) can substitute for zero or more words.  匹配一个或多个词
	
示例：
```
audit.#    匹配audit.irs.corporate或者 audit.irs 等
audit.*   只能匹配 audit.irs
```

### 示例

#### 1、定义生产者
```java
public class Provider {  
	 public static void main(String[] args) throws IOException {  
		 //获取连接对象  
		 Connection connection = RabbitmqUtils.getConnection();  
		 Channel channel = connection.createChannel();  

		 //声明交换机  
		 channel.exchangeDeclare("topicText", BuiltinExchangeType.TOPIC);  

		 //声明路由key  
		 String routeKey = "user.delete";  

		 //发布消息  
		 channel.basicPublish("topicText",routeKey, null,("这是topic模型所发出的消息,routeKey -> ["+routeKey+"]").getBytes());  
		 //关闭资源  
		 RabbitmqUtils.close(connection,channel);  
	 }  
}
```

#### 2、定义消费者
```java
public class Customer1 {  
	 public static void main(String[] args) throws IOException {  
		 //获取连接对象  
		 Connection connection = RabbitmqUtils.getConnection();  
		 Channel channel = connection.createChannel();  

		 //声明交换机  
		 channel.exchangeDeclare("topicText", BuiltinExchangeType.TOPIC);  

		 //声明临时队列  
		 String queue = channel.queueDeclare().getQueue();  

		 //绑定队列和交换机  
		 channel.queueBind(queue,"topicText","user.*");  

		 //消费消息  
		 channel.basicConsume(queue,true,new DefaultConsumer(channel){  
			 @Override 
			 public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
				System.out.println("消费者1 - > "+new String(body));  
			 }  
		 });  
	 }  
}
```

消费者2，与消费者1的区别尽在与routekey的不同，消费者2的routeKey是#
```java
public class Customer2 {  
	 public static void main(String[] args) throws IOException {  
		 //获取连接对象  
		 Connection connection = RabbitmqUtils.getConnection();  
		 Channel channel = connection.createChannel();  

		 //声明交换机  
		 channel.exchangeDeclare("topicText", BuiltinExchangeType.TOPIC);  

		 //声明临时队列  
		 String queue = channel.queueDeclare().getQueue();  

		 //绑定队列和交换机  
		 channel.queueBind(queue,"topicText","user.#");  

		 //消费消息  
		 channel.basicConsume(queue,true,new DefaultConsumer(channel){  
			 @Override 
			 public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
				 System.out.println("消费者2 - > "+new String(body));  
			 }  
		 });  
	 }  
}
```

#### 测试
在生产者发送消息的时候，消费者1和消费者2的消费情况
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111302128041.png)

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111302129776.png)

因为此时生产者所使用的routeKey是 `user.delete`，既符合消费者1的*（匹配单个词）也符合消费者2的#（匹配多个词）。
如果将生产者所使用的routeKey修改为 `user.findById.delete`，那么这2个的消费情况又如何呢？

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111302131084.png)
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111302131560.png)

从上图可以看出，在出现多级词的时候消费者1已经消费不了消息了，因为生产者目前所使用的routeKey是 `user.findById.delete`是有3级的，消费者1所使用的通配符 `*` 只能匹配任意一个词所以消费不到信息；
反之消费者2使用的通配符是 `#` ，能够匹配任意多个词所以能消费。