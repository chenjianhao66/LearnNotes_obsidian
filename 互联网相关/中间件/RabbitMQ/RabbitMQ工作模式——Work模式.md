#Java #中间件 #RabbitMQ

# RabbitMQ工作模式——Work模式
在实际环境中，使用简单模式的场景还是比较少的，生产者与消费者一一对应的话很容易消息堆积的情况：
当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。
此时就可以使用work 模型：**让多个消费者绑定到一个队列，共同消费队列中的消息**。队列中的消息一旦消费，就会消失，因此任务是不会被重复执行的。

work模式的工作示意图：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828143951.png)

当有多个消费者时，我们的消息会被哪个消费者消费呢，我们又该如何均衡消费者消费信息的多少呢?  
主要有两种模式：  
1、轮询模式的分发：一个消费者一条，按均分配；  
2、公平分发：根据消费者的消费能力进行公平分发，处理快的处理的多，处理慢的处理的少；按劳分配；

# Work模式的分类——轮询模式
该模式接收消息是当有多个消费者接入时，消息的分配模式是一个消费者分配一条，直至消息消费完成。
> 消费者1和消费者2的消息处理能力不同，但是最后处理的消息条数相同，是“按均分配”。
> 因为消息处理能力不同，经常会出现处理能力快的消费者已经消费完了，而处理能力慢的消费者还在处理消息。

## 示例
示例例步骤如下：
- 创建连接工厂对象，设置对象参数
- 根据连接工厂对象获取连接对象
- 根据连接对象获取通道对象
- 获取通道对象之后，分生产者和消费者两种情况
	- 生产者
		- 创建队列并且设置队列的各种参数
		- 发送消息
		- 关闭通道对象和连接对象
	- 消费者
		- 创建队列并且设置队列的各种参数
		- 接受消息
		- 一直监听队列参数.....


### 1、Maven依赖
**pom.xml**
```Java
<dependency>  
	 <groupId>com.rabbitmq</groupId>  
	 <artifactId>amqp-client</artifactId>  
	 <version>5.7.2</version>  
</dependency>
```

### 2、将连接Rabbit MQ的代码封装成工具类
**RabbitmqUtils类**
```java
public class RabbitmqUtils {  
 private static final ConnectionFactory connectionFactory;  
	 static {  
		 connectionFactory = new ConnectionFactory();  
		 connectionFactory.setHost("localhost");  
		 connectionFactory.setPort(5672);  
		 connectionFactory.setUsername("admin");  
		 connectionFactory.setPassword("admin");  
		 connectionFactory.setVirtualHost("/");  
	 }  
	 //获取连接对象
	 public static Connection getConnection(){  
		 try {  
			 return connectionFactory.newConnection();  
		 } catch (IOException e) {  
			 e.printStackTrace();  
		 } catch (TimeoutException e) {  
			 e.printStackTrace();  
		 }  
		return null;  
	}  

	//关闭连接
	public static void close(Connection var1, Channel var2) {  
		 try {  
			 if(var2 != null) var2.close();  
			 if(var1 != null) var1.close();  
		 } catch (IOException e) {  
			 e.printStackTrace();  
		 } catch (TimeoutException e) {  
			 e.printStackTrace();  
		 }  
	 }  
}
```

### 3、定义生产者

**Work_Producer类**
```java
public class Work_Producer {  
 public static void main(String[] args){  
	 // 1.获取连接对象  
	 Connection connection = RabbitmqUtils.getConnection();  
	 Channel channel = null;  
	 try {  
		 channel = connection.createChannel();  
		 //2.获取通道对象  
		 //3.绑定队列  
		 channel.queueDeclare("work", false, false, false, null);  
		 //4.发布消息  
		 for (int i = 1; i <= 100; i++) {  
			 String msg = "第"+i+"条消息~";  
		 	 channel.basicPublish("", "work", null,msg.getBytes());  
	 	 }  
	 	 RabbitmqUtils.close(connection,channel);  
	 } catch (IOException e) {  
	 	e.printStackTrace();  
	 }  
 }
}
```

### 4、定义消费者
这里定义两个消费者，代码都一样，为了做区别，只在消费信息的时候标注是哪一个消费者消费。

**Customer1类**
```java
public class Customer1 {  
 public static void main(String[] args) {  

	 // 1.获取连接对象  
	 Connection connection = RabbitmqUtils.getConnection();  
	 Channel channel = null;  

	 try {  
		 //2.获取通道对象  
		 channel = connection.createChannel();  
		 //3.绑定队列  
		 channel.queueDeclare("work",false,false,false,null);  
		 //4.消费消息  
		 channel.basicConsume("work", true, new DefaultConsumer(channel) {  
		 @Override  
		 public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
		 	System.out.println("Customer_1 消费消息 --- >"+new String(body));  
		 }  
		 });  
	  } catch (IOException e) {  
	 	e.printStackTrace();  
	  }  
  }  
}
```

**Customer2类**
```java
public class Customer2 {  
 public static void main(String[] args) {  

	 // 1.获取连接对象  
	 Connection connection = RabbitmqUtils.getConnection();  
	 Channel channel = null;  

	 try {  
		 //2.获取通道对象  
		 channel = connection.createChannel();  
		 //3.绑定队列  
		 channel.queueDeclare("work",false,false,false,null);  
		 //4.消费消息  
		 channel.basicConsume("work", true, new DefaultConsumer(channel) {  
		 @Override  
		 public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
		 	System.out.println("Customer_2 消费消息 --- >"+new String(body));  
		 }  
		 });  
	  } catch (IOException e) {  
	 	e.printStackTrace();  
	  }  
  }  
}
```

### 5、运行结果
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828145205.png)
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828145230.png)

这时候可以看到消息都是轮询的发给消费者，你处理一条我处理一条；
但是如果一个消费者处理的慢呢？在消费者1的代码下加一条线程休眠语句
```java
System.out.println("Customer_1 消费消息 --- >"+new String(body));  
try {  
 	Thread.sleep(2000);  
} catch (InterruptedException e) {  
 	e.printStackTrace();  
}
```


**加了休眠语句后的运行结果**
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828145549.png)

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828145611.png)
消费者2已经处理完了分配个它的所有信息，而消费者1还在处理信息，这个时候把消费者1断掉，回到RabbitMQ的管理界面，队列已经没有消息了，造成了消息丢失。


**总结**

上面的情况：消费者2已经处理完了分配个它的所有信息，而消费者1还在处理信息，且消费者在调用接收队列信息的方法时传了一个参数：
```java
channel.basicConsume("work", true, new DefaultConsumer(channel) 
```
第二个布尔值的参数就代表着是否自动确认。
为true时消费者被分到消息时就全部把这些消息自动确认收到并且反馈给MQ；
为false时需要消费者手动向MQ服务进行消息确认被消费。
>举个例子，如果队列里面有100条消息，使用轮询模式分发给2个消费者各50条信息，一个快一个慢；快的消费者已经消费完了慢的消费者还在消费；MQ服务分给消费者的时候已经得到了自动确认的信息，就不再管消息了。
但事实上慢的消费者还没有消费完这些信息，MQ也不再管消息了。如果这个时候慢消费者出现了宕机，消息就出现了丢失。
这时候就要引入消息确认机制以及每次从队列拿取多少条信息机制来实现公平分发模式。

----

# Work模式——公平分发模式

该模式根据消费者的消费能力进行公平分发，处理快的处理的多，处理慢的处理的少；按劳分配；
>那如何定义处理快和处理慢呢？

这取决于部署服务的机器性能有关，我们解决不了，但是可以借助RabbitMQ里面的自动确认消息和每次消费消息个数的机制来确定。
在从上面轮询模式中就可以得知，消费者在调用消费消息方法的时候入参一个布尔值参数，代表着是否自动确认消息。当该参数为true时，队列给你分发了50条信息且该消费者无法短时间处理完这些信息时，也会向MQ服务发送消息已经被确认的信息，这个时候就很容易出现消息丢失的情况。
为了防止服务宕机出现消息丢失的情况，引入了手动消息确认机制和QOS机制；

使用步骤：
- 关闭消息的自动确认
- 设置消费者每次从队列拿取消息的个数
- 消费者手动确认

整体示例例步骤如下：
- 创建连接工厂对象，设置对象参数
- 根据连接工厂对象获取连接对象
- 根据连接对象获取通道对象
- 获取通道对象之后，分生产者和消费者两种情况
	- 生产者
		- 创建队列并且设置队列的各种参数
		- 发送消息
		- 关闭通道对象和连接对象
	- 消费者
		- 创建队列并且设置队列的各种参数（这里要关闭消息的自动确认）
		- 设置QOS（每次从队列中拿取消息的个数）
		- 接受消息，并且手动确认消息
		- 一直监听队列参数.....



## 示例
### 1、定义生产者
生产者代码与上面代码一样
### 2、定义消费者
假设消费者1处理一条消息需要2秒钟；

**Customer1类**
```java
public class Customer1 {  
 public static void main(String[] args) {  

	 // 1.获取连接对象  
	 Connection connection = RabbitmqUtils.getConnection();  
	 Channel channel = null;  

	 try {  
		 //2.获取通道对象  
		 channel = connection.createChannel();  
		 //3.绑定队列  
		 channel.queueDeclare("work",false,false,false,null);  
		 //设置每次只从队列拿取一条消息消费  
		 channel.basicQos(1);
		 //4.消费消息  并且关闭自动确认消息
		 channel.basicConsume("work", false, new DefaultConsumer(channel) {  
		 @Override  
		 public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
		 	System.out.println("Customer_1 消费消息 --- >"+new String(body)); 
			try {  
				Thread.sleep(2000);  
			} catch (InterruptedException e) {  
				e.printStackTrace();  
			}
			//开启手动确认消息被确认
			channel.basicAck(envelope.getDeliveryTag(),false);
		 }  
		 });  
	  } catch (IOException e) {  
	 	e.printStackTrace();  
	  }  
  }  
}
```


假设消费者2处理一条消息只需要0.2秒
**Customer2类**
```java
public class Customer2 {  
 public static void main(String[] args) {  

	 // 1.获取连接对象  
	 Connection connection = RabbitmqUtils.getConnection();  
	 Channel channel = null;  

	 try {  
		 //2.获取通道对象  
		 channel = connection.createChannel();  
		 //3.绑定队列  
		 channel.queueDeclare("work",false,false,false,null);  
		 //设置每次只从队列拿取一条消息消费  
		 channel.basicQos(1);
		 //4.消费消息  并且关闭自动确认消息
		 channel.basicConsume("work", false, new DefaultConsumer(channel) {  
		 @Override  
		 public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
		 	System.out.println("Customer_2 消费消息 --- >"+new String(body)); 
			try {  
				Thread.sleep(200);  
			} catch (InterruptedException e) {  
				e.printStackTrace();  
			}
			//开启手动确认消息被确认
			channel.basicAck(envelope.getDeliveryTag(),false);
		 }  
		 });  
	  } catch (IOException e) {  
	 	e.printStackTrace();  
	  }  
  }  
}
```

**运行结果**

消费者1
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828161121.png)

消费者2
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828161140.png)

因为因为消费者2的处理时间是0.2秒而消费者1处理时间是2秒，所以从结论上面看已经做到了能者多劳的情况。

公平分发模式比轮询模式多了几句代码


**设置每次从队列拿取多少条消息消费**
```java
//设置每次只从队列拿取一条消息消费  
channel.basicQos(1);
```

**设置关闭消费者自动消息确认**
```java
//4.消费消息  并且关闭自动确认消息
channel.basicConsume("work", false, new DefaultConsumer(channel)
```


**手动确认消息被确认**
```java
//开启手动确认消息被确认 
channel.basicAck(envelope.getDeliveryTag(),false);
```