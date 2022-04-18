#Java #SpringBoot 

# 一、事件-监听机制
对于SpringApplicationContext(BeanFactory)而言,在整个应用运行过程中(包括应用的启动、销毁), 会发布各种应用事件。开发者也可以实现自己的事件, 从而起到扩展spring框架的作用 。

Spring的事件(Application Event)为 Bean与 Bean之间的消息通信提供了支持。当一个Bean处理完一个任务之后,希望另外一个 Bean知道并能做相应的处理, 这时我们就需要让另外一个 Bean监听当前 Bean所发送的事件。
抛开Java的事件监听机制不谈，现实生活中有很多的监听机制例子
>当有人呼喊你的名字，你会根据这个事件来判断要不要进行应答；
>如果喊的是你的名字，那么你就应答；如果不是，则不回应；
>这个过程其实就是产生事件，别人就是事件发布者，而我就是事件的监听者，在监听到有人喊我的时候，我会做回应；回应就是监听到事件之后所作出的反映。

以上的例子除了可以表示为事件-监听机制外，也可以表示为事件驱动机制（响应式编程）框架Reactor；

# 二、Spring实现的事件-监听机制

## 接口实现
下面是在Spring中实现事件-监听机制的几个接口
-   `ApplicationEvent` 就是 `Spring` 的事件接口
-   `ApplicationListener` 就是 `Spring` 的事件监听器接口，所有的监听器都实现该接口
-   `ApplicationEventPublisher` 是 `Spring` 的事件发布接口，`ApplicationContext` 实现了该接口
-   `ApplicationEventMulticaster` 就是`Spring`事件机制中的事件广播器，默认实现`SimpleApplicationEventMulticaster`

## 事件-监听执行流程

> 当一个事件源产生事件时，它通过事件发布器`ApplicationEventPublisher`发布事件，然后事件广播器`ApplicationEventMulticaster`会去事件注册表`ApplicationContext`中找到事件监听器`ApplicationListnener`，并且逐个执行监听器的`onApplicationEvent`方法，从而完成事件监听器的逻辑。

在Spring中，实现 `ApplicationListener` 接口外，还可以使用注解 `@EventListener` 来监听一个事件，同时注解还支持  [[SpEL]] 表达式，来确定触发该监听器的条件，只有满足表达式才会执行监听器里面的代码。


# 三、示例
事件-监听思路步骤：
- 定义事件类
- 定义事件监听器类
- 发布事件


## 1. 自定义事件类
```java
public class MyEvent extends ApplicationEvent {  
    private String msg;  
    private User user;  
    public MyEvent(Object source,User user) {  
        super(source);  
        this.user = user;  
        this.msg = "my event ~";  
    }  
	
	public MyEvent(User user){  
		super(user);  
		this.user = user;  
	}
  
    public User getUser(){  
        return user;  
    }  
  
    public String getMsg(){  
        return msg;  
    }  
}
```

## 2. 定义监听器类
这里可以选择实现 `ApplicationListener` 接口，也可以使用 `@EventListener` 注解注册监听器

### 实现ApplicationListener接口
1. 实现ApplicationListener接口并重写 onApplicationListenerEvent方法
```java
public class MyListener implements ApplicationListener<MyEvent> {  
  
   @Override  
   public void onApplicationEvent(MyEvent myEvent) {  
       System.out.println("user = " + myEvent.getUser());  
       System.out.println("msg = " + myEvent.getMsg());  
   }  
}
```


2.在主函数中将该监听器注册到 ApplicationContext 中，并且定义一个controller类，定义一个接口，访问的时候发布事件
```java
@SpringBootApplication  
public class client02Application {  
    public static void main(String[] args) {  
        SpringApplication application = new SpringApplication(client02Application.class);  
        application.addListeners(new MyListener());  
        application.run(args);  
    }  

}
```

### 使用@EventListener注解来注册一个监听器类
该注解是作用于方法
推荐使用注解的方式来注册一个监听器类，这样可以免去在主函数中手动注册监听器；并且可以使用@Order注解来调整对同一个事件监听的监听器执行顺序，注解里面的值越小越优先执行
```java
@Component  
public class MyListener2 {  
    @EventListener  
 	@Order(1)  
    public void listenerMyEvent(MyEvent yEvent){  
        System.out.println("======MyListener2======");  
        System.out.println("user = " + myEvent.getUser());  
        System.out.println("msg = " + myEvent.getMsg());  
        System.out.println("======MyListener2======");  
    }  
}

@Component  
public class MyListener3 {  
    @EventListener  
 	@Order(2)  
    public void listenerMyEvent(MyEvent myEvent){  
        System.out.println();  
        System.out.println("======MyListener3======");  
        System.out.println("user = " + myEvent.getUser());  
        System.out.println("msg = " + myEvent.getMsg());  
        System.out.println("======MyListener3======");  
        System.out.println();  
    }  
}

```
> 上面2个监听器类同时监听myEvent类事件，当事件发生时，Order注解中的数字越小越优先执行，所以MyListenr2事件优先执行。
## 3.发布事件
在需要发布事件的文件里面注入 `ApplicationEventPublisher` ，然后调用该对象的 `publkishEvent` 方法，该方法需要入参一个事件对象，发布事件成功。
```java
    @RequestMapping  
    @RestController 
	public class testControlelr{  
        @Autowired  
        private ApplicationEventPublisher applicationEventPublisher;  
  
        @GetMapping("/")  
        public String test(){
			//下列2个发布事件，2选1
            applicationEventPublisher.publishEvent(new MyEvent("",new User("id","man","China")));
            applicationEventPublisher.publishEvent(new MyEvent(new User("id","man","China")));
			return "suceess";  
        } 
    }  
```

3. 查看测试结果

当访问接口地址的时候，也将会发布事件，在监听器中拿到这个事件之后作输出操作；
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-31_14-58.png)

拿到了该事件，并且将该事件内的成员变量输出。

---

# 四、异步监听

默认情况下，监听事件都是同步执行的。在需要异步处理时，可以在方法上加上`@Async`进行异步化操作。此时，可以定义一个线程池，同时开启异步功能，加入`@EnableAsync`。

验证下是否是同步，修改代码，将监听器类添加代码，让线程休眠5秒
```java
//控制器类
@GetMapping("/")  
public String test(){  
    System.out.println("1 --- publishEvent before");  
    applicationEventPublisher.publishEvent(new MyEvent("",new User("id","man","China")));  
    System.out.println("5 --- method over");  
    return "suceess";  
}

//监听器类
@EventListener  
@Order(1)  
public void listenerMyEvent(MyEvent myEvent){  
    System.out.println("2 --- listenerMyEvent before");  
    System.out.println("3 --- listener run ~~");  
    try {  
        Thread.sleep(5000);  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
    System.out.println("4 --- listenerMyEvent after");  
}
```

运行的时候，代码会按照输出语句中的 1-5 按顺序执行，这就证明了监听器默认是同步的；
运行效果如下：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-31_15-41.png)

事件发布之后会依次执行监听该事件的监听器类代码，执行完之后才回到事件发布代码处，是同步阻塞的。

**异步监听配置**
配置步骤：
- 配置异步所需要执行的线程池
- 标明哪个方法需要开启异步

示例：

1. 添加异步配置类，配置支持异步所需要的线程池，这个类实现AsyncConfigurer接口
```java
//开启异步，并实现AsyncConfigurer接口，实现getAsyncExecutor方法

@Configuration  
@EnableAsync  
public class AsyncListenerConfig implements AsyncConfigurer {  
    @Override  
 	public Executor getAsyncExecutor() {  
        return Executors.newFixedThreadPool(10);  
    }  
}
```
> AsyncConfigurer 这个接口可以重写2个方法，getAsyncExecutor方法来配置支持该异步操作所需要所需要用到的线程池类型。


2. 在想要异步操作的方法上加上@Async注解

```java
@EventListener  
@Order(1)  
@Async  
public void listenerMyEvent(MyEvent myEvent){  
    System.out.println("2 --- listenerMyEvent before");  
    System.out.println("3 --- listener run ~~");  
    try {  
        Thread.sleep(5000);  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
    System.out.println("4 --- listenerMyEvent after");  
}
```

3. 测试查看结果

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-31_15-51.png)

从结果上可以看的出来，事件在发布之后就直接执行下面的语句，没有等待监听器类的代码；这也就实现了代码的异步操作！

--- 

# 五、精细化控制监听条件
使用`@EventListener`的`condition`可以实现更加精细的事件监听，`condition`支持`SpEL`表达式，可根据事件源的参数来判断是否监听。

沿用上面的代码，现在有一个新需求，我监听一个事件，只有当 `sex == woman` 这种条件下才会被监听到，其他不会执行监听器类的代码
解决方法：
只要在@EventListener注解的condition属性中写 SpEL表达式即可

```java
@EventListener(condition = "#myEvent.user.sex == 'woman'")  
@Order(2)  
public void listenerMyEvent2(MyEvent myEvent){  
    System.out.println();  
    System.out.println("======MyListener2======");  
    System.out.println("user = " + myEvent.getUser());  
    System.out.println("msg = " + myEvent.getMsg());  
    System.out.println("======MyListener2======");  
    System.out.println();  
}
```

> 关键代码：@EventListener(condition = "#myEvent.user.sex == 'woman'")  
> 只有在传进来的事件源中的user属性中的sex属性 == woman 时，才会执行这个监听器类下面的代码！