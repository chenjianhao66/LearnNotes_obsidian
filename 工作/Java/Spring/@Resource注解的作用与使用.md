## @Resource注解
该注解是用来装配Bean的，可以写在字段上也可以写在setter方法上。
### 注解属性
该注解有2个重要属性：**name** 和 **type**

spring将name属性解析为bean的名字，而type属性则被解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，如果使用type属性则使用byType的自动注入策略。如果都没有指定，则通过反射机制使用byName自动注入策略。


### 注解的使用方式
**@Resource注解什么属性都不指定**
既不指定name属性，也不指定type属性，则自动按byName方式进行查找。如果没有找到符合的bean，则回退为一个原始类型进行查找，如果找到就注入。
但是如果按照原始类型去寻找，找到多个的话就会抛出异常。

**如果@Resource注解指定了name属性**
那么该注解就会根据 **byName** 去寻找Bean并装配，找不到就会抛出异常。

**如果@Resource注解指定了type属性**
那么该注解就会根据 **byType** 去寻找对应类型的Bena并装配，找不到或者找到多个就会抛出异常。

**如果@Resource注解同时指定了name和type属性**
那么该注解就会在Spring上下文中寻找唯一匹配的Bena并装配，找不到则抛出异常。

示例

```java
## 1.什么都不指定
    @Resource
	private String bucketName;

## 2.指定name
    @Resource（name="buckeName")
	private String bucketName;
	
## 3.指定type
    @Resource(type="default")
	private String bucketName;
	
## 4.指定name和type
    @Resource(name=buckeName,type="default")
	private String bucketName;
```


## @Resource注解与@Autowired注解的区别
这两个注解都是用来依赖注入的，只不过@Resource注解什么都不指定默认是通过 **byName** 来注入，而@AutoWired默认是 **byType** 来注入

@Autowired默认按类型装配（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null 值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下： 
```java
@Autowired() 
@Qualifier("baseDao") 
private BaseDao baseDao;
```

@Resource（这个注解属于J2EE的），默认安照名称进行装配，名称可以通过name属性进行指定，
如果没有指定name属性，当注解写在字段上时，默认取字段名进行按照名称查找，如果注解写在setter方法上默认取属性名进行装配。 当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
```java
@Resource(name="baseDao") 
private BaseDao baseDao;
```