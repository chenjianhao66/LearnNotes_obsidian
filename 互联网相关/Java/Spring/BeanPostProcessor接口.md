# 接口描述
spring提供了一个接口类-BeanPostProcessor，我们叫他：bean的加工器，应该是在bean的实例化过程中对bean做一些包装处理，里边提供两个方法：
```java
public interface BeanPostProcessor
{ 
public abstract Object postProcessBeforeInitialization(Object obj, String s) throws BeansException; public abstract Object postProcessAfterInitialization(Object obj, String s) throws BeansException;
}
```