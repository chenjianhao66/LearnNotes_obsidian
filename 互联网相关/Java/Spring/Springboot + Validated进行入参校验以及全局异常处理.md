#Java #SpringBoot 

# 概述
日常开发中，对入参进行参数校验是必不可少的一个环节。 而使用最多的就是Validator框架 。

Validator校验框架遵循了JSR-303 【Java Specification Requests】验证规范 。

这里实践下，在boot项目中如何优雅的集成参数校验框架


# Validator常用校验规则
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050939668.png)


# 使用Validator

## 添加依赖
boot 2.3 以后版本的pom信息如下

``` 
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
</dependencies>
```

springboot 2.3版本之前只需要引入 spring-boot-starter-web 即可 ，已经包含了

## 添加携带有参数校验的实体类
```java
package com.artisan.vo;

import lombok.Data;
import org.hibernate.validator.constraints.Length;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotEmpty;


@Data
public class Artisan {

    private String id;


    @NotBlank(message = "名字为必填项")
    private String name;


    @Length(min = 8, max = 12, message = "password长度必须位于8到12之间")
    private String password;


    @Email(message = "请填写正确的邮箱地址")
    private String email;

    private String sex;

    @NotEmpty(message = "Code不能为空")
    private String code;
}
```

## 添加控制器用于接收请求并验证
```java
package com.artisan.controller;

import com.artisan.vo.Artisan;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.validation.constraints.Email;

@RestController
@Slf4j
@Validated
@RequestMapping("/valid")
public class ArtisanController {


    /**
     * 使用@RequestBody注解，用于接受前端发送的json数据
     *
     * @param artisan
     * @return
     */
    @PostMapping("/testJson")
    public String testJson(@Validated @RequestBody Artisan artisan) {
        log.info("InComing Param  {}", artisan);
        return "testJson valid success";
    }

    /**
     * 模拟表单提交
     *
     * @param artisan
     * @return
     */
    @PostMapping(value = "/testForm")
    public String testForm(@Validated Artisan artisan) {
        log.info("InComing Param is {}", artisan);
        return "testForm valid success";
    }

    /**
     * 模拟单参数提交
     *
     * @param email
     * @return
     */
    @PostMapping(value = "/testParma")
    public String testParma(@Email String email) {
        log.info("InComing Param is {}", email);
        return "testParma  valid success";
    }
}
    
```

## 发起请求

### /testJson接口
测试第一个接口【/testJson】，该接口接收一个JSON数据
请求参数邮箱参数不合格
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050940825.png)

结果：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050942181.png)

可以看到抛出的异常为： org.springframework.web.bind.MethodArgumentNotValidException

### /testFrom
测试表单数据的接口
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050943138.png)

结果：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050943877.png)

可以看到抛出的异常为： org.springframework.validation.BindException

### /testParams
测试参数是在url的数据
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050944408.png)

结果：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050944578.png)

可以看到抛出的异常为：javax.validation.ConstraintViolationException

## 存在的问题
  
且不说好不好看， 不管怎么样，现在是通过Validation框架实现了校验。 当然了，我们的追求肯定不是这样的，Validator校验框架返回的错误提示太臃肿了 ，格式啥的都不一样，很难搞哦， 怎么给前台返回？？？？

#  使用 统一格式 + 全局异常Handler 优化
增加统一返回 和 [[Spring boot的全局异常处理器|全局异常Handler]]，单独拦截参数校验的三个异常：
`javax.validation.ConstraintViolationException`
`org.springframework.validation.BindException`
`org.springframework.web.bind.MethodArgumentNotValidException`


```java

    /**
     * @param e
     * @return
     */
    @ExceptionHandler(value = {BindException.class, ValidationException.class, MethodArgumentNotValidException.class})
    public ResponseEntity<ResponseData<String>> handleValidatedException(Exception e) {
        ResponseData<String> resp = null;

        if (e instanceof MethodArgumentNotValidException) {
            // BeanValidation exception
            MethodArgumentNotValidException ex = (MethodArgumentNotValidException) e;
            resp = ResponseData.fail(HttpStatus.BAD_REQUEST.value(),
                    ex.getBindingResult().getAllErrors().stream()
                            .map(ObjectError::getDefaultMessage)
                            .collect(Collectors.joining("; "))
            );
        } else if (e instanceof ConstraintViolationException) {
            // BeanValidation GET simple param
            ConstraintViolationException ex = (ConstraintViolationException) e;
            resp = ResponseData.fail(HttpStatus.BAD_REQUEST.value(),
                    ex.getConstraintViolations().stream()
                            .map(ConstraintViolation::getMessage)
                            .collect(Collectors.joining("; "))
            );
        } else if (e instanceof BindException) {
            // BeanValidation GET object param
            BindException ex = (BindException) e;
            resp = ResponseData.fail(HttpStatus.BAD_REQUEST.value(),
                    ex.getAllErrors().stream()
                            .map(ObjectError::getDefaultMessage)
                            .collect(Collectors.joining("; "))
            );
        }

        log.error("参数校验异常:{}", resp.getMessage());
        return new ResponseEntity<>(resp, HttpStatus.BAD_REQUEST);
    }


```


使用统一异常处理之后重新测试
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050947937.png)

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050947079.png)

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203050947242.png)




# 参数分组
我们经常会碰到这样的一个场景： 新增的时候某些[字段](https://so.csdn.net/so/search?q=%E5%AD%97%E6%AE%B5&spm=1001.2101.3001.7020)为必填（比如密码）， 更新的时候非必填。
这样该如何做呢；

首先得明确，如果给校验分场景来校验的话，就会出现一个对象的字段上会有若干个注解来校验，同时需要指定这个校验是在指定场景来生效；

后面在代码里执行校验的时候，也需要指定当下场景启用校验的场景是什么。

以上的场景就可以理解为分组的概念。

## 执行步骤

### 定义分组接口
```java

import javax.validation.groups.Default;

public interface CustomValidateGroup extends Default {

    interface Crud extends CustomValidateGroup {
        interface Create extends Crud {

        }

        interface Update extends Crud {

        }

        interface Query extends Crud {

        }

        interface Delete extends Crud {

        }
    }
}
```

定义一个分组接口`CustomValidateGroup` 让其继承`javax.validation.groups.Defaul`t，再在分组接口中定义出多个不同的操作类型，Create，Update，Query，Delete.
>接口里面不需要写内容，这里只是对校验场景进行一个区分。

###  给参数分配分组

```java
/**  
 * @author CJH  
 */@Data  
public class User {  
 private String id;  
  
  
 @NotNull(message = "名字为必填项")  
 private String name;  
  
  
 @Length(min = 8, max = 12, message = "password长度必须位于8到12之间",groups = CustomValidateGroup.Crud.Create.class)  
 @NotNull(groups = CustomValidateGroup.Crud.Create.class,message = "密码不能为空")  
 @Null(groups = CustomValidateGroup.Crud.Update.class)  
 private String password;  
  
  
 @Email(message = "请填写正确的邮箱地址")  
 private String email;  
  
  
 private String sex;  
  
 @NotNull(message = "Code不能为空")  
 private String code;  
}
```

以上代表在对 `password` 字段校验的时候有2个场景，一个在 `Create` 场景下需要校验长度和不能为空，而在 `Update` 场景下该参数可以为空。




### 指定分组

给需要参数校验的方法指定分组

```java
/**  
 * 新增的时候 不能为空  
 * @param user 用户  
 * @return  
 */  
@PostMapping(value = "/addUser")  
public String add(@Validated(value = CustomValidateGroup.Crud.Create.class) @RequestBody User user){  
 log.info("InComing Param is {}", user);  
 return "add valid success";  
}  
  
  
/**  
 * 更新的时候 可以为空  
 * @param user 用户  
 * @return  
 */  
@PostMapping(value = "/updateUser")  
public String update(@Validated(value = CustomValidateGroup.Crud.Update.class) @RequestBody User user){  
 log.info("InComing Param is {}", user);  
 return "update valid success";  
}
```

在创建用户下，不填密码：

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203051500739.png)



而在更新用户时，不传密码：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203051502748.png)



> 对于未指定分组的则使用的是默认分组 。 比如由于email属于默认分组，而我们的分组接口CustomValidateGroup已经继承了Default分组，所以也是可以对email字段作参数校验的；
> 
如果CustomValidateGroup没有继承Default分组，那在代码属性上就需要加上@Validated(value = {ValidGroup.Crud.Create.class, Default.class}才能让email字段的校验生效。


# 自定义注解

 Validation允许用户自定义校验，Validation 提供的注解基本上够用，但是复杂的校验，我们还是需要自己定义注解来实现自动校验。
 
自定义注解分为2步，一个是创建自己自己的注解，第二个是创建校验逻辑类；

## 创建注解
```java
/**  
 * @author CJH  
 */@Target({TYPE,FIELD,METHOD,ANNOTATION_TYPE,CONSTRUCTOR,PARAMETER,TYPE_USE})  
@Retention(RUNTIME)  
@Documented  
@Constraint(validatedBy = IDCardValidator.class)//标明由哪个类执行校验逻辑  
public @interface CheckPhone {  
  
 /**  
 * 校验分组，必需  
 * @return  
 */  
 Class<?>[] groups() default {};  
  
 /**  
 * 校验载荷，必需  
 * @return  
 */  
 Class<? extends Payload>[] payload() default {};  
  
 /**  
 * 校验出错时默认返回的消息，必需  
 * @return  
 */  
 String message() default "手机号格式错误";  
  
 /**  
 * 下面是我自己定义属性  
 */  
    
 String phone() default "";  
  
}
```

在自定义注解的时候，`groups`、`payload`、`message`这3个字段是必需添加的，其他的根据业务去添加；
添加在注解上的注解，还需要添加一个注解 `Constraint`，代表着该注解由哪个校验类负责进行校验。

## 创建校验类
校验类需要实现 `ConstraintValidator` 接口，该接口接收2个泛型参数， 第一个参数是 自定义注解类型，第二个参数是 被注解字段的类。
> 校验的类型如果是基本类型那就直接填入类型，如果是多个字段需要校验，那就传入自定义类型；这里直接传入用户对象


该接口中需要实现2个方法，`initialize()`和`isValid()`。顾名思义就是用来初始化注解和进行校验的方法；
`initialize()` 方法用于初始化注解，拿到在使用注解时传入的值，比如校验错误时所提示的信息，就可以在使用注解的时候进行重写。

`isValid()` 方法就用于进行校验，该方法会传入一个值，该值就是被注解所标注的字段的值。

```java
/**  
 * @author CJH  
 */@Slf4j  
public class IDCardValidator implements ConstraintValidator<CheckIDCard,String> {  
  
 private static final Pattern idCard_pattern = Pattern.compile("^(\\d{6})(\\d{4})(\\d{2})(\\d{2})(\\d{3})([0-9]|X)$");  
 private String IDCard;  
  
 /**  
 * 获取注解的值  
 * @param constraintAnnotation 所标注的对象  
 */  
 @Override  
 public void initialize(CheckIDCard constraintAnnotation) {  
 /**  
 * 这里可以拿到注解里面定义的一些值  
 * 比如定义一个value数组，在声明数组的时候需要传入的值必需在数组里面出现  
 * 那么就可以在将数组里面的内容初始化到该类里面，然后在 isValid 方法里面进行校验  
 */  
 log.info("初始化注解......");  
 IDCard = constraintAnnotation.IDCard();  
 }  
  
 /**  
 * 校验逻辑  
 * @param s 实际传入的值  
 * @param constraintValidatorContext 自定义注解上下文  
 * @return  
 */  
 @Override  
 public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {  
 Matcher matcher = idCard_pattern.matcher(s);  
 System.out.println();  
 return matcher.matches();  
 }  
}
```

## 使用注解
在用户类添加身份证字段

```java
@CheckIDCard(message = "身份证格式错误！")  
private String idCard;
```

发起请求

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203051706175.png)