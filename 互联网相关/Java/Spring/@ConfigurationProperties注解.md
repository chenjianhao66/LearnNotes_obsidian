2021-09-07
16:48:01
author:陈建浩


--- 

# ConfigurationProperties介绍
ConfigurationProperties是一个注解，可以标注在一个Class上，这样Spring Boot会从 Environment中获取其属性对应的属性值给其进行注入。比如下面的代码定义中，Spring Boot在实例化`TestConfigurationProperties`这个bean时就会把从Environment中获取属性名为appName的属性值赋给 `TestConfigurationProperties` 的 `appName` 属性。

```java
@ConfigurationProperties
@Data
public class TestConfigurationProperties {

    private String appName;
    
}
```

所以当你的application.properties文件中定义了`appName=Test`时就会把`Test`赋值给TestConfigurationProperties对象的appName属性。实际上下面的定义和`appName=Test`是等价的。也就是说在从Environment中获取属性值绑定到ConfigurationProperties标注的对象上时，对大小写是不敏感的，而且其中的`-`和`_`都会被剔除。
```properties 
APPname=Test
app-Name=Test
app-name=Test
app_name=Test
```

---

# 指定需要映射的前缀
在application.properties文件中定义的属性通常不是单一名称的属性，而是以`a.b.c.d`这种形式构成的属性，多个层级之间以点分隔，从而形成不同的分类。
这种属性需要绑定到`@ConfigurationProperties`标注的对象属性上时可以指定一个通用的前缀，然后只对去除前缀之后的内容进行绑定。下面的代码指定了绑定属性时的前缀是`test.config`，所以TestConfigurationProperties对象的username属性将绑定配置文件中的`test.config.username`属性，password属性将匹配配置文件中的`test.config.password`属性。
```java
@ConfigurationProperties("test.config")
@Data
public class TestConfigurationProperties {

    private String username;
    
    private String password;
    
}
```

当在application.properties文件中进行了如下定义时，TestConfigurationProperties对象的username属性绑定的值是`u1`，password属性绑定的值是`p1`。
```properties
test.config.username=u1
test.config.password=p1
```

--- 
# 级联绑定
下面的代码中TestConfigurationProperties的inner属性是一个对象，需要对其进行绑定时需要以`.`进行级联绑定。

```java
@ConfigurationProperties("test.config")
@Data
public class TestConfigurationProperties {

    private Inner inner;
    
    @Data
    public static class Inner {
        private String username;
        private String password;
    }
    
}
```

在application.properties文件中进行如下定义会为TestConfigurationProperties对象的inner属性绑定一个Inner对象，其username属性的值是u1，password的值是p1。
```properties
test.config.inner.username=u1
test.config.inner.password=p1
```

在application.yml文件中进行如下定义与上面的定义等价。
```properties
test.config.inner:
  username: u1
  password: p1
```

---

# 集合属性绑定
下面的代码中使用`@ConfigurationProperties`标注的Class有一个List类型的属性。
```java
@ConfigurationProperties("test.config")
@Data
public class TestConfigurationProperties {

    private List<String> list;
    
}
```

需要给List绑定值时，可以通过`[index]`的形式指定值，下面的代码就定义了List中的三个元素，分别是`ABC`、`DEF`和`GHI`。
```properties
test.config.list[0]=ABC
test.config.list[1]=DEF
test.config.list[2]=GHI
```

也可以使用英文逗号分隔List中的多个值，以下配置跟上面的配置是等价的。
```properties
test.config.list=ABC,DEF,GHI
```

在YAML配置文件定义集合类型的值绑定时可以定义为如下这样：
```yaml
test.config.list:
  - ABC
  - DEF
  - GHI
```

如果需要绑定值的集合元素是一个对象怎么办呢？下面的代码中list属性的元素类型就是一个Inner对象，其中Inner对象又有username和password两个属性。
```java
@ConfigurationProperties("test.config")
@Data
public class TestConfigurationProperties {

    private List<Inner> list;
    
    @Data
    public static class Inner {
        private String username;
        private String password;
    }
    
}
```

在application.properties文件中进行如下定义可以为list属性绑定两个Inner对象，其中第一个对象的username属性值为u1，password属性值为p1；第二个对象的username属性值为u2，password属性值为p2。
```properties
test.config.list[0].username=u1
test.config.list[0].password=p1

test.config.list[1].username=u2
test.config.list[1].password=p2
```

--- 
# 绑定Map属性
下面的代码中拥有一个Map类型的map属性，Key和Value都是String类型。
```java
@ConfigurationProperties("test.config")
@Data
public class TestConfigurationProperties {

    private Map<String, String> map;
    
}
```

需要给上面的map属性绑定值时可以使用`key=value`的形式，下面的配置会给map属性绑定两个元素，分别是key1对应value1，key2对应value2。

```properties
test.config.map.key1=value1
test.config.map.key2=value2
```

如果需要绑定的Value是一个对象怎么办呢？比如map属性的定义改为如下这样：
```java
@ConfigurationProperties("test.config")
@Data
public class TestConfigurationProperties {

    private Map<String, Inner> map;
    
    @Data
    public static class Inner {
        private String username;
        private String password;
    }
    
}
```

在application.properties文件中进行如下定义，会绑定两个元素到map，第一个元素的Key是key1，Value是一个Inner对象，其username属性的值是u1，password属性的值是p1；第二个元素的Key是key2，Value的username属性的值是u2，password属性的值是p2。

```properties
test.config.map.key1.username=u1
test.config.map.key1.password=p1

test.config.map.key2.username=u2
test.config.map.key2.password=p2
```



# 注意事项
- 在使用该注解的时候，需要保证被该注解标注的类提供有seter方法，要不然会注入失败！
- 在使用yaml文件注入的时候，保证yaml文件中的key不存在 "__" 下划线。

