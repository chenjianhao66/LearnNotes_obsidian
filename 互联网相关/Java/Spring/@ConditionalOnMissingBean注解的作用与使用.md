## @ConditionalOnMissingBean注解
@ConditionalOnMissingBean，它是修饰bean的一个注解。
主要实现的是，当你的bean被注册之后，如果再注册相同类型的bean，就不会成功，它会保证你的bean只有一个，即你的实例只有一个，当你注册多个相同的bean时，会出现异常，以此来告诉开发人员。抛出的异常如图：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-10_13-30-.png)

## 使用方法
该注解的用法如下：
```
@ConditionalOnMissingBean
@ConditionalOnMissingBean(TransactionManager.class)
@ConditionalOnMissingBean(ignored = {DistributedCommandBus.class})
@ConditionalOnMissingBean({EventStorageEngine.class,EventBus.class, EventStore.class})
```

## 代码演示
```java
@Component public class AutoConfig{ 
	@Bean 
	public AConfig aConfig(){ 
		return new AConfig("lind"); 
	} 
	
	@Bean
	@ConditionalOnMissingBean(AMapper.class) 
	public AMapper aMapper1(AConfig aConfig){ 
		return new AMapperImpl1(aConfig); 
	} 
	
	@Bean 
	public AMapper aMapper2(AConfig aConfig){ 
		return new AMapperImpl2(aConfig); 
	} 
}
```

以上这段代码运行起来就会抛异常，因为在aapper1中使用了注解@ConditionalOnMissingBean并表明了注入的类是 `AMapper.class`，而下面还有一个返回是 `AMapper` 的Bean。这就会抛出异常，因为注解是不允许有多个返回同样类型的Bean存在。

当我们把 `@ConditionalOnMissingBean(AMapper.class)` 去掉之后，bean可以注册多次，那么我注入的Bean会是什么？这个时候可以使用@**AutoWried**注解和@**Qualifier**来指定一个Bean来实现注入。

但是如果不使用这两个注解的话，这时需要用的@Primary来确定你要哪个实现；一般来说，对于自定义的配置类，我们应该加上@ConditionalOnMissingBean注解，以避免多个配置同时注入的风险。

```java
 @Bean 
 public AMapper aMapper1(AConfig aConfig) { 
 	return new AMapperImpl1(aConfig); 
 } 
 
 @Bean 
 @Primary 
 public AMapper aMapper2(AConfig aConfig) { 
 	return new AMapperImpl2(aConfig); 
 }
```

这样就可以来明显的区分注入的时候会优先选择哪一个了。
