#Java #SpringBoot 

# 前言
单元测试是软件开发中必不可少的一环，但是在平常开发中往往因为项目周期紧，工作量大而被选择忽略，这样往往导致软件问题层出不穷。线上出现的不少问题其实在有单元测试的情况下就可以及时发现和处理，因此培养自己在日常开发中写单元测试的能力是很有必要的。无论是对自己的编码能力的提高，还是项目质量的提升，都是大有好处。


# 认识Junit 5
  
要说什么是 JUnit 5，首先就得聊下 Java 单元测试框架 JUnit，它与另一个框架 TestNG 占据了 Java领域里单元测试框架的主要市场，其中 JUnit 有着较长的发展历史和不断演进的丰富功能，备受大多数 Java 开发者的青睐。

而说到 JUnit 的历史，JUnit 起源于 1997年，最初版本是由两位编程大师 Kent Beck 和 Erich Gamma 的一次飞机之旅上完成的，由于当时 Java 测试过程中缺乏成熟的工具，两人在飞机上就合作设计实现了 JUnit 雏形，旨在成为更好用的 Java 测试框架。如今二十多年过去了，JUnit 经过各个版本迭代演进，已经发展到了 5.x 版本，为 JDK 8以及更高的版本上提供更好的支持 (如支持 Lambda ) 和更丰富的测试形式 (如重复测试，参数化测试)。

# Junit 5 常见用法介绍
这里直接集成到 `Springboot`、
## Maven依赖
```java
<!--    junit5    -->  
 <dependency>  
	 <groupId>org.springframework.boot</groupId>  
	 <artifactId>spring-boot-starter-test</artifactId>  
	 <scope>test</scope>  
	 <exclusions>  
		 <exclusion>  
			 <groupId>org.junit.vintage</groupId>  
			 <artifactId>junit-vintage-engine</artifactId>  
		 </exclusion>  
	 </exclusions>  
 </dependency>
```


## 常用的Junit 5 注解
**@Test**
被该注解修饰的就是测试方法；

**@BeforeAll**
被该注解修饰的必须是静态方法，会在所有测试方法之前执行，会被子类继承，取代低版本的BeforeClass；

**@AfterAll**
被该注解修饰的必须是静态方法，会在所有测试方法执行之后才被执行，会被子类继承，取代低版本的AfterClass；

**@BeforeEach**
被该注解修饰的方法会在每个测试方法执行前被执行一次，会被子类继承，取代低版本的Before；

**@AfterEach**
被该注解修饰的方法会在每个测试方法执行后被执行一次，会被子类继承，取代低版本的Before；

**@DisplayName**
测试方法的展现名称，在测试框架中展示，支持emoji；

**@Timeout**
超时时长，被修饰的方法如果超时则会导致测试不通过；

**@Disabled**
不执行的测试方法；

环境准备

测试服务接口
```java
/**  
 * 服务接口  
 * @author jianhao  
 */public interface HelloService {  
	 /**  
	 * 输出字符串  
	 * @param name 传参  
	 * @return 将那么打印出来  
	 */  
	 String hello(String name);  
	  
	 /**  
	 * 将 value 自增  
	 * @param value 入参  
	 * @return value++  
	 */ int increase(int value);  
	  
	 /**  
	 * 该方法会休眠一秒钟  
	 * @return boolean  
	 */ boolean sleep();  
}
```

实现类
```java
/**  
 * 服务类  
 * @author jianhao  
 */@Service  
public class HelloServiceImpl implements HelloService{  
  
	 @Override  
	 public String hello(String name) {  
		 return name;  
	 }  
	  
	 @Override  
	 public int increase(int value) {  
		 return ++value;  
	 }  

	 @Override  
	 public boolean sleep() {  
		 try {  
			 Thread.sleep(1000);  
		 } catch (InterruptedException e) {  
			 e.printStackTrace();  
			 return false;  
		 }  
		 return true;  
	 }  
}
```


实例
```java
@SpringBootTest  
@Slf4j  
class HelloServiceImplTest {  
  
 @Autowired  
 HelloServiceImpl helloService;  
  
 static String NAME = "jianhao";  
  
 /**  
 * 在所有测试方法执行前所执行的方法  
 * 被BeforeAll注解所标注的方法必须是静态的  
 */  
 @Test  
 @BeforeAll static void beforeAll(){  
	 log.info("在所有测试方法执行前执行一次被 beforeAll注解所标注的方法");  
 }  
  
 /**  
 * 在所有测试方法执行完毕后所执行的方法  
 * 被AfterAll注解所标注的方法必须是静态的  
 */  
 @Test  
 @AfterAll static void afterAll(){  
	 log.info("在所有测试方法执行完毕后执行一次被 afterAll注解所标注的方法");  
 }  
  
  
 /**  
 * 每个测试方法执行前都会执行一次  
 */  
 @BeforeEach  
 void beforeEach() {  
	 log.info("execute beforeEach");  
 }  
  
 /**  
 * 每个测试方法执行后都会执行一次  
 */  
 @AfterEach  
 void afterEach() {  
	 log.info("execute afterEach");  
 }  
  
  
 @Test  
 @DisplayName("测试service层的hello方法")  
 void hello() {  
 log.info("测试hello方法");  
	 assertEquals(helloService.hello(NAME), NAME+1);  
 }  
  
 @Test  
 @DisplayName("测试service层的increase方法")  
 void increase(){  
 log.info("测试increase方法");  
	 assertEquals(helloService.increase(15), 16);  
 }  
  
 @Test  
 @DisplayName("测试service层的sleep方法")  
 @Timeout(unit = TimeUnit.SECONDS,value = 2)  
 void sleep(){  
	 helloService.sleep();  
 }    
}
```

按照以上的写法，期望的结果是：
在执行所有测试方法前，执行 `beforeAll()` 方法
在执行每一个测试方法前，执行 `beforeEach()`方法
在执行每一个测试方法后，执行 `AfterEach()`方法
在执行所有测试方法后，执行 `AfterAll()` 方法

但实际上，以上的3个方法中，只有sleep和increase方法能够执行成功，hello方法因为对Name执行了+1拼接字符串操作，所以两者不一致不通过测试
测试结果：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-28_15-10.png)

在每一个测试方法的执行前后，都执行了相关的方法
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-28_15-11.png)



## Assertions
断言（assertions）是测试方法中的核心部分，用来对测试需要满足的条件进行验证。这些断言方法都是 org.junit.jupiter.api.Assertions 的静态方法。JUnit 5 内置的断言可以分成如下几个类别：  
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/20201002092432969.jpg)

前面的assertEquals、assertTrue、assertFalse就不多说了
### 测试多个断言
**assertAll** 
用于测试多个判断断言，只有全部通过之后才算通过，如果有未通过的会有对应的提示
```java
/**  
 * 断言类用法  
 */  
@Test  
@DisplayName("断言类测试")  
void assertTest(){  
	 assertAll("断言类测试", ()-> assertTrue(true),  
		 ()-> assertEquals(1, 2),  
		 ()-> assertEquals(1+2, Math.addExact(1, 3))  
	 );  
}
```

## Assumptions类 
在 `Junit 5` 中，有 Assumptions 类，这个类是假设类
假设实际就是指定某个特定条件，假如不能满足假设条件，**假设不会导致测试失败，只是终止当前测试。** 这也是假设与断言的最大区别，因为对于断言而言，会导致测试失败。下面是该类的一些API用法：

**assumeFalse**
验证给定的假设为false，若为true，将终止测试，若为false，则通过测试
```java
/**  
 * 假设类用法  
 */  
@Test  
@DisplayName("假设类测试")  
void assumption(){  
	assumeFalse(true, "测试失败时的提示");  
}
```


**assumingTrue**
验证给定的假设为true，若为true，将终止测试
```java
/**  
 * 假设类用法  
 */  
@Test  
@DisplayName("假设类测试")  
void assumption(){  
	assumeTrue(true, "测试失败时的提示");  
}
```

**assumingThat**
执行提供的可执行Executable，但仅在提供的假设有效时执行。如果假设无效，Executable将不执行。如果Executable抛出异常，它将异常重新抛出，但该异常会被屏蔽为未经检查的异常。
```java
/**  
 * 假设类用法  
 */  
@Test  
@DisplayName("假设类测试")  
void assumption(){  
	assumingThat(true,()-> log.info("当参数1为真时，执行该任务，否则不执行！"));  
}
```
测试结果
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-28_17-48.png)


**assertThrows**

Assertions.assertThrows方法，用来测试Executable实例执行execute方法时是否抛出指定类型的异常；
如果execute方法执行时不抛出异常，或者抛出的异常与期望类型不一致，都会导致测试失败；
写段代码验证一下，如下，1除以0会抛出ArithmeticException异常，符合assertThrows指定的异常类型，因此测试可以通过：
但是第一个参数为写了nullpointerException之后，测试就不通过了，因为任务中会抛出ArithmeticException异常，与期望抛出的异常不一致，所以测试不通过。

```java
    @Test  
 @DisplayName("判断抛出异常类型是否为指定类型的测试")  
 void assertException(){  
 Exception arithmeticException = assertThrows(NullPointerException.class,   
												() -> Math.floorDiv(1, 0),  
												 "实际抛出的异常与期望抛出异常不一致时所打印的字符串");  
 log.info("返回的异常实例：{} ",arithmeticException.getMessage());  
 }
```
