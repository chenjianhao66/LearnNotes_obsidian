2021-09-10
21:56:27
author:陈建浩


--- 


# Spring WebFlux框架
Spring WebFlux是随Spring 5推出的响应式Web框架。与Spring MVC不同，它不需要Servlet API，是完全异步且非阻塞的，并且通过Reactor项目实现了Reactive Streams规范。

Spring WebFlux 用于创建基于事件循环执行模型的完全异步且非阻塞的应用程序。

> 所谓异步非阻塞是针对服务端而言的，是说服务端可以充分利用CPU资源去做更多事情，这与客户端无关，客户端该怎么请求还是怎么请求。

Reactive Streams是一套用于构建高吞吐量、低延迟应用的规范。而Reactor项目是基于这套规范的实现，它是一个完全非阻塞的基础，且支持背压。Spring WebFlux基于Reactor实现了完全异步非阻塞的一套web框架，是一套响应式堆栈。

【spring-webmvc + Servlet + Tomcat】命令式的、同步阻塞的

【spring-webflux + Reactor + Netty】响应式的、异步非阻塞的

## Flux和Mono
`Spring WebFlux` 是基于响应式流的，因此可以用来建立异步的、非阻塞的、事件驱动的服务。它采用 `Reactor` 作为首选的响应式流的实现库，不过也提供了对RxJava的支持。所以 `WebFlux` 框架中也有 `Flux` 和 `Mono` 的概念;

Reactor中的发布者（Publisher）由`Flux`和`Mono`两个类定义，它们都提供了丰富的操作符（operator）。一个Flux对象代表一个包含0…N个元素的响应式序列，而一个Mono对象代表一个包含零/一个（0…1）元素的结果。

既然是“数据流”的发布者，Flux和Mono都可以发出三种“数据信号”：元素值、错误信号、完成信号，错误信号和完成信号都是终止信号，完成信号用于告知下游订阅者该数据流正常结束，错误信号终止数据流的同时将错误传递给下游订阅者。
下图是 `Flux` 类型的数据流：

![[Pasted image 20210910221146.png]]

而下图是 `Mono` 类型的数据流：

![[Pasted image 20210910221325.png]]



需要注意的是，无论是哪一种类型的数据流，都包含三种不同类型的消息通知：
- 正常的包含元素的消息
- 序列结束的消息
- 序列出错的消息。


Flux和Mono提供了多种创建数据流的方法，`just`就是一种比较直接的声明数据流的方式，其参数就是数据元素。

```java
Flux.just(1, 2, 3, 4, 5, 6); 
Mono.just(1);
```

除了just方法，还可以通过如下方式声明（分别基于数组、集合和Stream生成）：
```java
Integer[] array = new Integer[]{1,2,3,4,5,6}; 
Flux.fromArray(array); 
List<Integer> list = Arrays.asList(array); 
Flux.fromIterable(list); 
Stream<Integer> stream = list.stream(); 
Flux.fromStream(stream);
```

不过，这三种信号都不是一定要具备的：

-   首先，错误信号和完成信号都是终止信号，二者不可能同时共存；
-   如果没有发出任何一个元素值，而是直接发出完成/错误信号，表示这是一个空数据流；
-   如果没有错误信号和完成信号，那么就是一个无限数据流。


比如，对于只有完成/错误信号的数据流：

```java
// 只有完成信号的空数据流 
Flux.just(); 
Flux.empty(); 
Mono.empty(); 
Mono.justOrEmpty(Optional.empty()); 

// 只有错误信号的数据流 
Flux.error(new Exception("some error")); 
Mono.error(new Exception("some error"));
```

现在数据流有了，那就可以对这些数据流进行操作，比如现在想把数据流的所有元素都打印出来：
```java
Flux.just(1, 2, 3, 4, 5, 6).subscribe(System.out::print); 
System.out.println(); 
Mono.just(1).subscribe(System.out::println);

// 执行结果
123456
1
```

可见，`subscribe`方法中的lambda表达式作用在了每一个数据元素上。此外，Flux和Mono还提供了多个`subscribe`方法的变体：
```java
// 订阅并触发数据流 
subscribe(); 

// 订阅并指定对正常数据元素如何处理 
subscribe(Consumer<? super T> consumer); 

// 订阅并定义对正常数据元素和错误信号的处理 
subscribe(Consumer<? super T> consumer, Consumer<? super Throwable> errorConsumer);

// 订阅并定义对正常数据元素、错误信号和完成信号的处理 
subscribe(Consumer<? super T> consumer, Consumer<? super Throwable> errorConsumer, Runnable completeConsumer); 

// 订阅并定义对正常数据元素、错误信号和完成信号的处理，以及订阅发生时的处理逻辑
subscribe(Consumer<? super T> consumer, Consumer<? super Throwable> errorConsumer, Runnable completeConsumer, Consumer<? super Subscription> subscriptionConsumer);
```

>这里需要注意的一点是，`Flux.just(1, 2, 3, 4, 5, 6)`仅仅声明了这个数据流，此时数据元素并未发出，只有`subscribe()`方法调用的时候才会触发数据流。所以，**订阅前什么都不会发生**。

## 常见WebFlux的操作符
  
通常情况下，我们需要对源发布者发出的原始数据流进行多个阶段的处理，并最终得到我们需要的数据。这种感觉就像是一条流水线，从流水线的源头进入传送带的是原料，经过流水线上各个工位的处理，逐渐由原料变成半成品、零件、组件、成品，最终成为消费者需要的包装品。这其中，流水线源头的下料机就相当于源发布者，消费者就相当于订阅者，流水线上的一道道工序就相当于一个一个的操作符（Operator）。

Reactor中提供了非常丰富的操作符，除了以上几个常见的，还有：

-   用于编程方式自定义生成数据流的`create`和`generate`等及其变体方法；
-   用于“无副作用的peek”场景的`doOnNext`、`doOnError`、`doOncomplete`、`doOnSubscribe`、`doOnCancel`等及其变体方法；
-   用于数据流转换的`when`、`and/or`、`merge`、`concat`、`collect`、`count`、`repeat`等及其变体方法；
-   用于过滤/拣选的`take`、`first`、`last`、`sample`、`skip`、`limitRequest`等及其变体方法；
-   用于错误处理的`timeout`、`onErrorReturn`、`onErrorResume`、`doFinally`、`retryWhen`等及其变体方法；
-   用于分批的`window`、`buffer`、`group`等及其变体方法；
-   用于线程调度的`publishOn`和`subscribeOn`方法。



下面介绍一些我们常用的操作符。
**1、just方法**
可以指定序列中包含的全部元素。创建出来的 Flux 序列在发布这些元素之后会自动结束。
```java
Flux.just("hello", "world")  
 .doOnNext((i) -> {  
 	System.out.println("[doOnNext] " + i);  // 1
 })  
 .doOnComplete(() -> System.out.println("[doOnComplete]"))  // 2
 .subscribe(i -> System.out.println("[subscribe] " + i));  // 3
  
 // 执行结果  
[doOnNext] hello  
[subscribe] hello  
[doOnNext] world  
[subscribe] world  
[doOnComplete]

```

以上以 just方法创建了两个字符串元素
（1）、`doOnNext` 方法是对序列中的每一个元素执行操作，并且 `doOnNext` 方法并不会影响到序列元素本身，这里该方法对序列的每一个元素打印出来。
（2）、`doOnComplete` 方法只有该序列元素完成后才会执行；
（3）、`subscribe` 方法是发布元素，并可以对该元素进行操作。

**2、 fromArray()，fromIterable()和 fromStream()**
可以从一个数组、Iterable 对象或 Stream 对象中创建 Flux 对象。
```java
 List<String> arr = Arrays.asList("flux", "mono", "reactor", "core");  
 Flux.fromIterable(arr)  
 .doOnNext((i) -> {  
 	System.out.println("[doOnNext] " + i);  
 })  
 .subscribe((i) -> {  
 	System.out.println("[subscribe] " + i);  
 });  
//执行结果  
[doOnNext] flux  
[subscribe] flux  
[doOnNext] mono  
[subscribe] mono  
[doOnNext] reactor  
[subscribe] reactor  
[doOnNext] core  
[subscribe] core
```

**3、 empty()**
创建一个不包含任何元素，只发布结束消息的序列。
```java
Flux.empty()  
.doOnNext(i -> {  
	System.out.println("[doOnNext] " + i);  
 }).doOnComplete(() -> {  
	System.out.println("[DoOnComplete] ");  
 }).subscribe(i -> {  
	System.out.println("[subscribe] " + i);  
 });  
//执行结果  
[DoOnComplete]
```
该序列里面没有元素，所以只会执行 `doOnComplete` 方法

**4、 error(Throwable error)**
创建一个只包含错误消息的序列。
```java
try {  
 	int []arr = new int[5];  
 arr[10] = 2;  
 } catch (Exception e) {  
 	Flux.error(e).subscribe(i -> {  
 	System.out.println("error subscribe");  
 });  
 }  
//执行结果
error subscribe
```
数组越界，报错

