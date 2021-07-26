# Swagger
## 引言

相信无论是前端还是后端开发，都或多或少地被接口文档折磨过。`前端经常抱怨后端给的接口文档与实际情况不一致。后端又觉得编写及维护接口文档会耗费不少精力，经常来不及更新。其实无论是前端调用后端，还是后端调用后端，都期望有一个好的接口文档`。但是这个接口文档对于程序员来说，就跟注释一样，经常会抱怨别人写的代码没有写注释，然而自己写起代码起来，最讨厌的，也是写注释。所以仅仅只通过强制来规范大家是不够的，随着时间推移，版本迭代，接口文档往往很容易就跟不上代码了。

## 1.什么是Swagger

发现了痛点就要去找解决方案。解决方案用的人多了，就成了标准的规范，这就是Swagger的由来。通过这套规范，你只需要按照它的规范去定义接口及接口相关的信息。再通过Swagger衍生出来的一系列项目和工具，就可以做到生成各种格式的接口文档，生成多种语言的客户端和服务端的代码，以及在线接口调试页面等等。这样，如果按照新的开发模式，在开发新版本或者迭代版本的时候，只需要更新Swagger描述文件，就可以自动生成接口文档和客户端服务端代码，做到调用端代码、服务端代码以及接口文档的一致性。

但即便如此，对于许多开发来说，编写这个yml或json格式的描述文件，本身也是有一定负担的工作，特别是在后面持续迭代开发的时候，往往会忽略更新这个描述文件，直接更改代码。久而久之，这个描述文件也和实际项目渐行渐远，基于该描述文件生成的接口文档也失去了参考意义。**所以作为Java届服务端的大一统框架Spring，迅速将Swagger规范纳入自身的标准，建立了Spring-swagger项目，后面改成了现在的Springfox。通过在项目中引入Springfox，可以扫描相关的代码，生成该描述文件，进而生成与代码一致的接口文档和客户端代码**。这种通过代码生成接口文档的形式，在后面需求持续迭代的项目中，显得尤为重要和高效。

- 总结: **Swagger就是一个用来定义接口标准,接口规范,同时能根据你的代码自动生成接口说明文档的一个工具**

## 2. 官方提供的工具

**Swagger Codegen**: 通过Codegen 可以将描述文件生成html格式和cwiki形式的接口文档，同时也能生成多种语言的服务端和客户端的代码。支持通过jar包，docker，node等方式在本地化执行生成。也可以在后面的Swagger Editor中在线生成。

**Swagger UI**:提供了一个可视化的UI页面展示描述文件。接口的调用方、测试、项目经理等都可以在该页面中对相关接口进行查阅和做一些简单的接口请求。该项目支持在线导入描述文件和本地部署UI项目。

**Swagger Editor**: 类似于markendown编辑器的编辑Swagger描述文件的编辑器，该编辑支持实时预览描述文件的更新效果。也提供了在线编辑器和本地部署编辑器两种方式。

**Swagger Inspector**: 感觉和postman差不多，是一个可以对接口进行测试的在线版的postman。比在Swagger UI里面做接口请求，会返回更多的信息，也会保存你请求的实际请求参数等数据。

**Swagger Hub**：集成了上面所有项目的各个功能，你可以以项目和版本为单位，将你的描述文件上传到Swagger Hub中。在Swagger Hub中可以完成上面项目的所有工作，需要注册账号，分免费版和收费版。

**Springfox Swagger**: Spring 基于swagger规范，可以将基于SpringMVC和Spring Boot项目的项目代码，自动生成JSON格式的描述文件。本身不是属于Swagger官网提供的，在这里列出来做个说明，方便后面作一个使用的展开。

## 3.构建Swagger与SpringBoot环境

### 3.1 引入依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

### 3.2 编写Swagger配置类

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .pathMapping("/")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.baizhi.controller"))
                .paths(PathSelectors.any())
                .build().apiInfo(new ApiInfoBuilder()
                        .title("SpringBoot整合Swagger")
                        .description("SpringBoot整合Swagger，详细信息......")
                        .version("9.0")
                        .contact(new Contact("xiaochen","www.baidu.com","1079170090@qq.com"))
                        .license("copyright jianhao")
                        .licenseUrl("http://www.baidu.com.com")
                        .build());
    }
}
```

配置完以上配置之后，启动该 `Springboot` 项目， 访问Swagger提供的ui界面: http://localhost:8080/swagger-ui.html


----

## 4.使用Swagger构建

### 1.开发Controller接口

```java
@RestController
@RequestMapping("user")
public class UserController {
    @GetMapping("findAll")
    public Map<String,Object> findAll(){
        HashMap<String, Object> map = new HashMap<>();
        map.put("msg","查询所有成功");
        map.put("state",true);
        return map;
    }   
}
```

### 2.重启项目访问接口界面

----

## 5.Swagger的注解

### 5.1 @Api

- 作用: 用来指定接口的描述文字

- 修饰范围: 用在类上

  ```java
  @RequestMapping("user")
  @Api(tags = "用户模块接口规范说明")
  public class UserController {
  	....
  }
  ```

### 5.2 @ApiOperation

- 作用: 用来对接口中具体方法做描述

- 修饰范围: 用在方法上

  ```java
  @GetMapping("findAll")
  @ApiOperation(value = "查询所有用户接口",
                notes = "<span style='color:red;'>描述:</span>&nbsp;用来查询所有用户信息的接口")
  public Map<String,Object> findAll(){
    Map<String, Object> map = new HashMap<String,Object>();
    map.put("success","查询所有成功!");
    map.put("state",true);
    return map;
  }
  ```

  **value: 用来对接口的说明**

  **notes:用来对接口的详细描述**

### 5.3 @ApiImplicitParams

- 作用: 用来接口的中参数进行说明

- 修饰范围: 用在方法上

  ```java
  @ApiImplicitParams({
    @ApiImplicitParam(name="id",value = "用户id",dataType = "String",defaultValue = "21"),
    @ApiImplicitParam(name="name",value = "用户姓名",dataType ="String",defaultValue = "张三")
  })
  public Map<String, Object> save(String id, String name) {
    .....
  }
  ```

### 5.4 @ApiResponses

- 作用：用于请求的方法上，表示一组响应

- 修饰范围: 用在方法上

  ```java
  @ApiResponses({
              @ApiResponse(code = 400, message = "请求参数没填好"),
              @ApiResponse(code = 404, message = "请求路径没有或页面跳转路径不对")
  })
  public Map<String, Object> save(String id, String name) {
    .....
  }
  ```

  



   