#Java  #SpringBoot 
# 一、Java元注解
元注解是负责对其它注解进行说明的注解，自定义注解时可以使用元注解。[Java](http://c.biancheng.net/java/) 5 定义了 4 个注解，分别是 @Documented、@Target、@Retention 和 @Inherited。Java 8 又增加了 @Repeatable 和 @Native 两个注解。这些注解都可以在 java.lang.annotation 包中找到。下面主要介绍每个元注解的作用及使用。
## @Documented
    

@Documented 是一个标记注解，没有成员变量。用 @Documented 注解修饰的注解类会被 JavaDoc 工具提取成文档。默认情况下，JavaDoc 是不包括注解的，但如果声明注解时指定了 @Documented，就会被 JavaDoc 之类的工具处理，所以注解类型信息就会被包括在生成的帮助文档中。


  ---
## @Target
@Target 注解用来指定一个注解的使用范围，即被 @Target 修饰的注解可以用在什么地方。@Target 注解有一个成员变量（value）用来设置适用目标，value 是 java.lang.annotation.ElementType 枚举类型的数组，下表为 ElementType 常用的枚举常量。
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-31_17-35.png)

---
## @Retention

@Retention 用于描述注解的生命周期，也就是该注解被保留的时间长短。@Retention 注解中的成员变量（value）用来设置保留策略，value 是 java.lang.annotation.RetentionPolicy 枚举类型，RetentionPolicy 有 3 个枚举常量，如下所示。

1.  SOURCE：在源文件中有效（即源文件保留）
2.  CLASS：在 class 文件中有效（即 class 保留）
3.  RUNTIME：在运行时有效（即运行时保留）

  
生命周期大小排序为 SOURCE < CLASS < RUNTIME，前者能使用的地方后者一定也能使用。**如果需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解**；如果要在编译时进行一些预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS 注解；如果只是做一些检查性的操作，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解。

---

## @Inherited

@Inherited 是一个标记注解，用来指定该注解可以被继承。使用 @Inherited 注解的 Class 类，表示这个注解可以被用于该 Class 类的子类。就是说如果某个类使用了被 @Inherited 修饰的注解，则其子类将自动具有该注解。

---

# 二、自定义注解
声明自定义注解使用 @interface 关键字（interface 关键字前加 @ 符号）实现。定义注解与定义接口非常像，如下代码可定义一个简单形式的注解类型。
```java
//自定义一个简单注解
public @inteface test{

}
```

上述代码声明了一个 Test 注解。默认情况下，注解可以在程序的任何地方使用，通常用于修饰类、接口、方法和变量等。  
  
定义注解和定义类相似，注解前面的访问修饰符和类一样有两种，分别是公有访问权限（public）和默认访问权限（默认不写）。**一个源程序文件中可以声明多个注解，但只能有一个是公有访问权限的注解。且源程序文件命名和公有访问权限的注解名一致。  **
  
不包含任何成员变量的注解称为标记注解，例如上面声明的 Test 注解以及基本注解中的 @Override 注解都属于标记注解。根据需要，注解中可以定义成员变量，成员变量以无形参的方法形式来声明，其方法名和返回值定义了该成员变量的名字和类型。代码如下所示：

```java
public @interface MyTag {
	// 定义带两个成员变量的注解
	// 注解中的成员变量以方法的形式来定义
	String name();
	int age();
}
```

如果在注解里定义了成员变量，那么使用该注解时就应该为它的成员变量指定值，如下代码所示。
```java
public class Test {
    // 使用带成员变量的注解时，需要为成员变量赋值
    @MyTag(name="xx", age=6)
  	public void info() {
  	...
  	}
  	...
}
```

注解中的成员变量也可以有默认值，可使用 default 关键字。如下代码定义了 @MyTag 注解，该注解里包含了 name 和 age 两个成员变量。  

```java
 public @interface MyTag {
	// 定义了两个成员变量的注解
	// 使用default为两个成员变量指定初始值
	String name() default "jianhao~";
	int age() default 7;
}
```
>如果为注解的成员变量指定了默认值，那么使用该注解时就可以不为这些成员变量赋值，而是直接使用默认值。
>当然也可以在使用 MyTag 注解时为成员变量指定值，如果为 MyTag 的成员变量指定了值，则默认值不会起作用。