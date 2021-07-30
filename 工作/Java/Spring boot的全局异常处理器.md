# Spring的全局异常处理
通常一个web程序在运行过程中，由于用户的操作不当，或者程序的bug，有大量需要处理的异常。其中有些异常是需要暴露给用户的，比如登陆超时，权限不足等等。可以通过弹出提示信息的方式告诉用户出了什么错误。

而这就表示在程序中需要一个机制，去处理这些异常，将程序的异常转换为用户可读的异常。而且最重要的，是要将这个机制统一，提供统一的异常处理。

在项目中实现一个全局异常处理器，通过对抛出的异常进行捕捉，并且进行自定义的处理。
实现方法：@ControllerAdvice和@ExceptionHandler注解。

 - @ControllerAdvice
	异常集中处理，使异常处理与业务逻辑分离；对Controller层所抛出的异常进行捕捉
	 

 - @ExceptionHandler
	 标注在方法外，被标注的方法可以统一处理某一种异常，从而减少代码重复率和复杂率

## @ControllerAdvice注解
该注解的作用是由控制器所产生的异常都被该注解所注解的类处理。

提问
- 该注解怎么使用
	- 该注解使用于类上，与@Controller一个用法
- 该注解可以传入什么参数
	- 可以传入包的结构----> @ControllerAdvice(com.jianhao.controller)
	- 可以传入自定义异常类，代表捕获这个类--->@ControllerAdvice(MyException.class)
	- 可以传入一个注解，代表捕捉被这个注解所标注的所有类抛出的异常
	- 可以不传入值，可以捕获所有类型的异常

- 该注解有什么作用
	- 被该注解所标注的类，配合@ExceptionHandler注解进行异常的处理
	- 比如被该注解标注的类中（类A），有一个方法被@Exception注解所标注（方法A），那么只要被注解标注为@Controller的类中（类B）发生异常，那么该异常就会被类A的方法A所捕获，执行方法A里面的业务代码。


## @ExceptionHandler注解
该注解标注在一个方法上，表示某个异常统一在该注解所标注的方法处理

提问
- 该注解怎么使用
	- 该注解使用于方法上，与@RequestMaping注解一个用法
- 该注解传入什么参数
	- 包括一般的异常或特定的异常（即自定义异常），如果注解没有指定异常类，会默认进行映射。
	- 实例：@ExceptionHandler(MyException.class)

- 该注解有什么作用
	- 该注解标注在一个方法上，表示某个异常统一在该注解所标注的方法处理

## 实例
该实例涉及3个Java文件：
-  自定义异常类：MyException.java
-  全局异常捕获类：MyExceptionHandler.java
-  抛出异常的类：testController.java


### MyException.java
```
public class MyException extends RuntimeException {
	//自定义的枚举类
	private resultEnum resultEnum;
	
	public MyException(resultEnum resultEnum){
		this.resultEnum = resultEnum;
	}

	public MyException(){
	}

	public resultEnum getResultEnum(){
		return resultEnum;
	}
	
	public void setResultEnum(resultEnum resultEnum){
		this.resultEnum = resultEnum;
	}
}
```


### MyExceptionHandler.java
```
@RestControllerAdvice(annotations = RestController.class)
public class MyExceptionHandler {

	@ExceptionHandler(MyException.class)
	public Result<String> test(){
		return new Result<String>().setCode(400).setMsg("测试异常捕捉");
	}

}

```

###  testController.java
```
@RestController
public class configController {
	@GetMapping("test")
	public void test(){
		throw new MyException();
	}
}
```

1. 在我们执行 `testController`控制器的 `test`方法时，该方法会抛出一个 `MyException` 异常，
2. 被 `@RestControllerAdvice` 所标注的 `MyExceptionHandler` 类，传入参数 ==(annotations = RestController.class)== ，代表着这个类捕获所有被标注为 `@RestController` 注解的类所抛出的异常。
3. `MyExceptionHandler` 类中有注解  ==@ExceptionHandler== 的方法，该注解传入一个异常类参数**MyException.class**，发生的这个异常则会统一被该注解所标注的方法所处理；该方法会返回一个自定义的响应结果类。


### 测试结果
使用 **swagger** 接口组件来测试，测试如下：




# 参考文章
[Spring异常处理 @ExceptionHandler](https://www.cnblogs.com/shuimuzhushui/p/6791600.html)

[# Spring 异常处理三种方式 @ExceptionHandler](https://www.cnblogs.com/lvbinbin2yujie/p/10574812.html#type4)