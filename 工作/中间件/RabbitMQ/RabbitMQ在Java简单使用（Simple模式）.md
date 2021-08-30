#Java #中间件 #RabbitMQ

# Java操作RabbitMQ示例（简单模式）
## 1、Maven引入相关依赖
```Java
<dependency>  
	 <groupId>com.rabbitmq</groupId>  
	 <artifactId>amqp-client</artifactId>  
	 <version>5.7.2</version>  
</dependency>
```

## 2、定义生产者
从前面的文章可以得知，生产者负责发送消息到队列，消费者负责从队列中拉取信息消费；而这两者都有共同的操作逻辑，那就是创建连接队列的容器（交换机），这个消息队列模式是简单模式，不涉及到交换机。那么生产者消费者只需要配置连接工厂，获取连接对象用于连接队列即可。

整体的步骤如下：
- 获取**连接工厂**，设置连接参数；
- 根据连接工厂获取**连接对象**；
- 根据连接对象获取**通道对象**
- 通道对象绑定队列
- 向队列发送消息
- 依次关闭通道对象、连接对象；

> 注意，关闭通道对象和连接对象不是必须的；如果是消费者需要一直监听队列的话，那么这一步可以省略。

定义消费者到test方法中：
```java
@Test  
public void producer(){  
	 // 1.创建一个连接工厂  
	 ConnectionFactory connectionFactory = new ConnectionFactory();  

	 //2.设置连接工厂的各种参数  
	 connectionFactory.setHost("localhost");  
	 connectionFactory.setPort(5672);  
	 connectionFactory.setUsername("admin");  
	 connectionFactory.setPassword("admin");  
	 connectionFactory.setVirtualHost("/");  
  
	 try {  
		 //3.根据连接工厂获取连接对象  
		 Connection connection = connectionFactory.newConnection();  

		 //4.根据连接对象获取通道对象  
		 Channel channel = connection.createChannel();  

		 /*  
		 *  5.根据通道对象来绑定对应的消息队列  
		 * 如果队列不存在，则会创建  
		 *  Rabbitmq不允许创建两个相同的队列名称，否则会报错。  
		 *  @params1： queue 队列的名称  
		 *  @params2： durable 队列是否持久化  
		 *  @params3： exclusive 是否排他，即是否私有的，如果为true,会对当前队列加锁，其他的通道不能访问，并且连接自动关闭  
		 *  @params4： autoDelete 是否自动删除，当最后一个消费者断开连接之后是否自动删除消息。  
		 *  @params5： arguments 可以设置队列附加参数，设置队列的有效期，消息的最大长度，队列的消息生命周期等等。  
		 * */ 
		 channel.queueDeclare("simple", false, false, false, null);  

		 /*  
		 *  6.发送消息  
		 *  @params1: 交换机exchange  
		 *  @params2: 队列名称/routing  
		 *  @params3: 属性配置  
		 *  @params4: 发送消息的内容，需要转字节  
		 */ channel.basicPublish("","simple",null,"RabbitMQ is simple mode".getBytes());  

		 //7.close对象  
		 channel.close();  
		 connection.close();  
	 } catch (IOException e) {  
		 e.printStackTrace();  
	 } catch (TimeoutException e) {  
		 e.printStackTrace();  
	 }  
}


```

> 之所以把生产者定义在test方法中，是因为生产者只需要发送消息，而不需要像消费者那样一直监听队列是否有消息；所以生产者可以定义在test方法中，而消费者得运行在main方法里。

运行代码后转到 RabbitMQ 的web界面查看消息是否已经发送到队列中
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828110735.png)

在上图发现，已经在名字为 `simple` 的队列中发现一条状态为 `Ready` 的消息，而且总数量为 1的消息，证明消息已经成功发送到队列当中。


## 3.定义消费者消费消息

定义消费者的代码与生产者一致，只不过消费者不是发送消息了，而是消费消息；

```java
public class Customer {  
 public static void main(String[] args) {  
	 // 消费者代码  

	 //1.获取连接工厂对象  
	 ConnectionFactory connectionFactory = new ConnectionFactory();  

	 //2.设置连接工厂对象的参数  
	 connectionFactory.setVirtualHost("/");  
	 connectionFactory.setHost("localhost");  
	 connectionFactory.setPort(5672);  
	 connectionFactory.setUsername("admin");  
	 connectionFactory.setPassword("admin");  

	 //3.根据连接工厂获取连接对象  
	 Connection connection = null;  
	 try {  
		 connection = connectionFactory.newConnection();  
		 //4.根据连接对象获取通道对象  
		 Channel channel = connection.createChannel();  
		 
		 //5.通道对象绑定队列  
		 channel.queueDeclare("simple",false,false,false,null);  
		 /*  
		 *  6。获取信息  
		 *  @params1: 队列名称/routing  
		 *  @params2: 是否自动确认信息  
		 *  @params3: 消息回调对象函数  
		 */ 
		 //第一种获取消息的方式  
		//            String consume = channel.basicConsume("simple", true, new DefaultConsumer(channel){  
		//                @Override  
		//                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
		//                    System.out.println("获取到的消息---->"+new String(body));  
		//                }  
		//            });  

		 //第二种获取消息的方式  
		 channel.basicConsume("simple", true, new DeliverCallback() {  
		 @Override  
		 public void handle(String s, Delivery delivery) throws IOException {  
		 System.out.println("获取到的消息-->" + new String(delivery.getBody()));  
		 }  
		 }, s -> System.out.println("s = " + s));  
		 
	 } catch (IOException e) {  
	 	e.printStackTrace();  
	 } catch (TimeoutException e) {  
	 	e.printStackTrace();  
	 }  
  
  
 }}


```


运行主程序代码，就可以消费队列的消息：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210828111126.png)

运行生产者的代码，消费者就会立即消费；
是因为消费者的代码是没有关闭通道和连接的，这样会让消费者一直监听队列里面的信息，有信息就立马消费。


## 绑定队列的参数说明

在上面的代码中，无论是生产者端还是消费者端，绑定队列时都要传入一些参数，下面对参数进行注明：

```java
channel.queueDeclare("simple", false, false, false, null);
```

**参数1**
该通道所绑定的队列名称

**参数2**
该参数需要传入一个布尔值，表示的是该队列是否支持持久化；
持久化说的是队列的持久化，如果在服务运行过程中机器宕机，服务恢复的时候，服务界面还是重新出现队列，并不会消失。
> 需要注意的是，这里的持久化指的是队列的持久化，服务挂掉重启之后，队列里面的消息并不会持久化，这个需要注意。消息的持久化在下面会提。

**参数3**
该参数需要传入一个布尔值，表示的是该队列是否为私有的（排他性）
是否排他，即是否私有的，如果为true,会对当前队列加锁，其他的通道不能访问，并且连接自动关闭。

**参数4**
该参数需要传入一个布尔值，表示的是该队列是否会自动删除
如果该参数为true，在消费者消费完队列里面的消息后，队列的消息数量为空并且最后一个消费者断开与该队列的连接后，那么该队列就会自动删除。
> 注意，该队列是否会自动删除不单单只看该队列的消息是否为0，还要看该队列是否还有消费者连接。
> 即使队列消息数量为0，只有这个队列还有消费者连接，那么这个队列就不会自动删除
> 只要满足（队列消息数量为0）和（该队列没有消费者连接）这两个条件后队列才会自动删除

## 发布消息的参数说明
```java
channel.basicPublish("","simple",null,"RabbitMQ is simple mode".getBytes());
```

**参数1**
该参数需要传入一个字符串，代表的是发布该消息所绑定的交换机（exchage）

**参数2**
该参数需要传入一个字符串，代表的是该消息所要发布的路由key或者队列名

**参数3**
该参数需要传入一个`BasicProperties`对象，表示的是发送消息时的额外设置
这里只需要传入一个 `MessageProperties.PERSISTENT_TEXT_PLAIN` 常量对象即可，表示消息的持久化；这样继续服务重启了之后，消息还能保存
```java
channel.basicPublish("","simple", MessageProperties.PERSISTENT_TEXT_PLAIN,msg.getBytes());
```

还有其他的常量对象：
```java
public class MessageProperties {  
    public static final BasicProperties MINIMAL_BASIC = new BasicProperties((String)null, (String)null, (Map)null, (Integer)null, (Integer)null, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);  
 public static final BasicProperties MINIMAL_PERSISTENT_BASIC = new BasicProperties((String)null, (String)null, (Map)null, 2, (Integer)null, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);  
 public static final BasicProperties BASIC = new BasicProperties("application/octet-stream", (String)null, (Map)null, 1, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);  
 public static final BasicProperties PERSISTENT_BASIC = new BasicProperties("application/octet-stream", (String)null, (Map)null, 2, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);  
 public static final BasicProperties TEXT_PLAIN = new BasicProperties("text/plain", (String)null, (Map)null, 1, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);  
 public static final BasicProperties PERSISTENT_TEXT_PLAIN = new BasicProperties("text/plain", (String)null, (Map)null, 2, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);  
  
 public MessageProperties() {  
    }  
}
```

等待补充.....
简单模式的消息队列演示就这样。