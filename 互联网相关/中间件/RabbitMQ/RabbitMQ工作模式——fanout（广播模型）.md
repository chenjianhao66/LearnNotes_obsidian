2021-11-22
22:02:18
author:陈建浩
#Java #中间件 #RabbitMQ 

--- 

# RabbitMQ工作模式——fanout模式（广播模型）
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2263764-20210114134513271-559134819.png)


在广播模式下，消息发送流程是这样的：

-   可以有多个消费者
    
-   每个**消费者有自己的queue**（队列）
    
-   每个**队列都要绑定到Exchange**（交换机）
    
-   **生产者发送的消息，只能发送到交换机**，交换机来决定要发给哪个队列，生产者无法决定。
    
-   交换机把消息发送给绑定过的所有队列
    
-   队列的消费者都能拿到消息。实现一条消息被多个消费者消费



## 示例
使用广播模型的步骤如下：

-   创建连接工厂对象，设置对象参数
-   根据连接工厂对象获取连接对象
-   根据连接对象获取通道对象
-   获取通道对象之后，分生产者和消费者两种情况
	-   生产者
		-   声明交换机以及该交换机的类型（这里为`fanout`）
		-   发送消息
		-   关闭通道对象和连接对象
	-   消费者
		-   声明交换机以及该交换机的类型
		-   获取到一个临时队列（因为广播类型可能就收一次广播消息，不知道对这个队列进行持久化）
		-   通道对象绑定队列名称以及交换机
		-   消费消息



### 1、Maven依赖![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2263764-20210114134513271-559134819.png)


**pom.xml**

```java
<dependency>  
     <groupId>com.rabbitmq</groupId>  
     <artifactId>amqp-client</artifactId>  
     <version>5.7.2</version>  
</dependency>
```

### 2、定义生产者
```java
public class Provider {  
  
  
 public static void main(String[] args) throws IOException {  
		 // 获取连接对线  
		 Connection connection = RabbitmqUtils.getConnection();  
		 Channel channel = connection.createChannel();  

		 //声明交换机  
		 channel.exchangeDeclare("fanoutExchange","fanout");  

		 //发送信息  
		 channel.basicPublish("fanoutExchange","",null,"测试广播类型工作模式".getBytes());  
		 
		 //关闭资源  
		 RabbitmqUtils.close(connection,channel);  
	 }  
}
```

### 3、定义消费者
这里定义两个消费者，代码都一样，为了做区别，只在消费信息的时候标注是哪一个消费者消费。
```java

public class Customer {  
 	public static void main(String[] args) throws IOException {  
		 //获取连接对线  
		 Connection connection = RabbitmqUtils.getConnection();  
		 Channel channel = connection.createChannel();  

		 //通道绑定交换机  
		 channel.exchangeDeclare("fanoutExchange","fanout");  

		 //获取临时队列，广播类型没有必要对队列进行持久化  
		 String tempQueueName = channel.queueDeclare().getQueue();  

		 //绑定交换机和队列  
		 channel.queueBind(tempQueueName,"fanoutExchange","");  

		 channel.basicConsume(tempQueueName,true,new DefaultConsumer(channel){  
			 @Override public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
				 System.out.println("消费者1消费消息-> "+new String(body));  
			 }  
		 });  
	 }  
}
```

### 4、运行结果
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111222241124.png)

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202111222241995.png)

可以看出，生产者发出一条消息后，所有绑定到这个`fanoutExchange` 交换机的消费者都会消费这个消息。

**总结**

要实现广播模式，生产者需要声明交换机，确定把消息发送给哪个交换机，至于交换机把消息给哪个消费者是不会管的；
消费者需要声明一个交换机和临时队列，然后将队列交换机绑定到channel对象，然后就监听并消费消息了。