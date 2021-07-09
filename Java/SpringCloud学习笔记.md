# SpingCloud & SpringCloud Alibaba 微服务工具集
## 1. 什么是微服务


简而言之，微服务架构 样式是一种将单个应用程序作为套件开发的方法 在小型服务中，每个服务都在自己的进程中运行` 以及与轻量级机制（通常是HTTP资源API）通信。这些服务是围绕业务而建立的 能力` 和`独立地 可展开的` 完全自动化的部署机器`有一个光秃秃的房间(基于) 最小集中(分布式) 管理(管理) 在这些服务中，可以用不同的编程语言编写，并使用不同的数据存储技术-----[摘自官网翻译后]
-   官方定义:**微服务就是由一系列围绕自己业务开发的微小服务构成,他们独立部署运行在自己的进程里,基于分布式的管理**
   
-   通俗定义:**微服务是一种架构，这种架构是将单个的整体应用程序分割成更小的项目关联的独立的服务。一个服务通常实现一组独立的特性或功能，包含自己的业务逻辑和适配器。各个微服务之间的关联通过暴露api来实现。这些独立的微服务不需要部署在同一个虚拟机，同一个系统和同一个应用服务器中。**
   
---
## 2.微服务都有哪些组件
目前业界内有几套主流的微服务主流方案，都包含了包括 `服务注册与发现组件`、`服务配置组件`、`服务熔断与降级组件`、`服务网关组件`、`服务间通信组件`等组件。每一套解决方案都各有优劣，所以企业在制作解决方案的时候都是这个组件在其他厂商表现的好就用哪一家。下面是各个厂商所提出的组件以及优劣情况

|      |  Spring  |  Alibaba  |  推荐  |
| ---- | ---- | ---- | ---- |
|   服务发现与注册组件   |   Eureka   |  nacos  |  nacos  |
|   服务配置组件   |   Config   |  nacos  |  nacos  |
|   服务网关组件   |   Getaway   |   无   |   Getaway   |
| 服务熔断与降级组件 | Hystrix | Sentinel | Sentinel |
| 服务间通信组件 | RestTmplate+Ribbon,Openfeign | 无 | RestTmplate+Ribbon,Openfeign s

最终的解决方案：
```
服务发现与注册组件 ： nacos
服务配置组件：nacos
服务网关组件：Getaway
服务熔断与降级组件：Sentinel
服务间通信组件：RestTmplate+Ribbon,Openfeign
```
## 组件介绍
### 服务发现与注册组件 & 配置中心
nacos，全称 Name Service(服务注册与发现) & Configurations Services(统一配置中心)，nacos就是取 Name Service的首字母、Configurations的前2个字母以及Services的首字母。

[nacos官方网站](https://nacos.io/zh-cn/index.html)
[nacos快速开始](https://nacos.io/zh-cn/docs/what-is-nacos.html)
Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

#### 1、安装nacos的前提条件
- 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。必须满足
- 64 bit JDK 1.8+；[[ubuntu相关配置#安装 Java|下载 & 配置]]；必须满足
- Maven 3.2.x+；[[ubuntu相关配置#Maven配置|下载 & 配置]]；可选


[nacos下载地址](https://github.com/alibaba/nacos/releases )

下载之后进行解压
```
sudo tar -zxvf nacos-server-2.0.2.tar.gz -C ~/file/code
```
解压之后进入解压的目录，进入 `bin` 目录下执行 `startup.sh` 文件

**Linx/Unix/Mac系统**
启动命令（standalone代表着单机模式运行，非集群模式）：
```
cd nacos/bin
sh startup.sh -m standalone
```

如果您使用的是**ubuntu**系统，或者运行脚本报错提示符号找不到，可尝试如下运行：

```
bash startup.sh -m standalone
```

**Windows系统**
启动命令(standalone代表着单机模式运行，非集群模式):

```
startup.cmd -m standalone
```

在执行命令后如果出现以下提示则代表启动成功：
```
/usr/lib/jvm/java-1.8.0-openjdk-amd64/bin/java  -Xms512m -Xmx512m -Xmn256m -Dnacos.standalone=true -Dnacos.member.list= -Djava.ext.dirs=/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre/lib/ext:/usr/lib/jvm/java-1.8.0-openjdk-amd64/lib/ext -Xloggc:/home/yz/file/code/nacos/logs/nacos_gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dloader.path=/home/yz/file/code/nacos/plugins/health,/home/yz/file/code/nacos/plugins/cmdb -Dnacos.home=/home/yz/file/code/nacos -jar /home/yz/file/code/nacos/target/nacos-server.jar  --spring.config.additional-location=file:/home/yz/file/code/nacos/conf/ --logging.config=/home/yz/file/code/nacos/conf/nacos-logback.xml --server.max-http-header-size=524288
nacos is starting with standalone
nacos is starting，you can check the /home/yz/file/code/nacos/logs/start.out

```

启动成功之后通过访问以下链接即可到达登陆界面
[http://localhost:8848/nacos/index.html](http://localhost:8848/nacos/index.html)
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_09-28-nacosStart.png)


首页展示
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_09-28-nacosIndex.png)

#### 2、开发服务并注册到nacos
##### 构建父项目
新建一个空项目，在此空项目的基础上建立一个 `Maven Mouble`，在该模块上的 `pom.xml` 配置文件写内容，该配置文件只管理父依赖，具体的依赖包在下面的子模块进行添加；使用 `dependencyManagement` 标签进行管理
```
<!-- parent project  springBoot-->  
<parent>  
    <artifactId>spring-boot-starter</artifactId>  
    <groupId>org.springframework.boot</groupId>  
    <version>2.2.5.RELEASE</version>  
</parent>  
  
<properties>  
    <!--<maven.compiler.source>8</maven.compiler.source>-->  
 <!--<maven.compiler.target>8</maven.compiler.target>--> <java.version>1.8</java.version>  
    <spring-cloud.version>Hoxton.SR6</spring-cloud.version>  
</properties>  
  
<!--springCloud parent project-->  
<dependencyManagement>  
    <dependencies>  
        <!--spring cloud-->  
 		<dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-dependencies</artifactId>  
            <version>${spring-cloud.version}</version>  
            <type>pom</type>  
            <scope>import</scope>  
        </dependency>  
  
        <!--alibaba cloud-->  
 		<dependency>  
            <groupId>com.alibaba.cloud</groupId>  
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>  
			<!--下面的版本由上面的 properteis标签定义-->
            <version>${spring.cloud.alibaba.version}</version>  
            <type>pom</type>  
            <scope>import</scope>  
        </dependency>  
    </dependencies>  
</dependencyManagement>
```

##### 创建子项目
在副项目的基础上新建一个子项目，设置父项目，继承父项目的依赖，然后在子项目的 `pom.xml` 配置文件中添加 `springboot-web` 依赖；

**子项目的pom.xml 配置文件**
在pom文件中要引入nacos的服务发现依赖 `discovery`
```
<!--父项目-->
<parent>  
    <artifactId>springcloud_parent</artifactId>  
    <groupId>com.jianhao</groupId>  
    <version>1.0-SNAPSHOT</version>  
</parent>  
<modelVersion>4.0.0</modelVersion>  
  
<artifactId>springCloud_02nacosClient8888</artifactId>  
  
<properties>  
    <maven.compiler.source>8</maven.compiler.source>  
    <maven.compiler.target>8</maven.compiler.target>  
</properties>  
  
  <!--子项目的具体依赖-->
<dependencies>  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency>  
	
	<!--关键，要引入nacos的服务发现依赖-->
    <dependency>  
        <groupId>com.alibaba.cloud</groupId>  
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
        <version>2.2.4.RELEASE</version>  
    </dependency>  
</dependencies>
```
**application.properties**
```
# 指定服务端口
server.port=8888  

# 指定服务名称，全局唯一标识
spring.application.name=nacosclient8888  

# 指定nacos服务地址  
spring.cloud.nacos.server-addr=localhost:8848  

## 指定注册中心地址  
spring.cloud.nacos.discovery.server-addr=${spring.cloud.nacos.server-addr}  

## 暴露所有web端点  
management.endpoints.web.exposure.include=*
```
**新建端口用于访问测试**
```
@SpringBootApplication  
public class nacosClient8888 {  
    public static void main(String[] args) {  
        SpringApplication.run(nacosClient8888.class,args);  
    }  
  
  @RestController  
 public class controller{  
        @GetMapping("/")  
        public String test(){  
            return "nacosClient8888 server run success";  
        }  
    }  
}
```

##### 启动项目
启动该项目，然后测试接口是否测通
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_10-22-run-project.png)

测试成功，回到 `nacos 管理界面` 看服务是否已经注册到 `nacos` 上；

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_10-17-nacosDiscovery.png)

至此，服务发布成功！
#### 3、nacos配置中心
将服务的配置都放在nacos配置中心，能让服务启动时找到自己所属的那份配置文件，并且在nacos配置中心动态的修改配置，服务也能实时获取到配置变化。
**添加依赖**
```


```

### 服务间通信组件
#### RestTemplate+Ribbon
#### OpenFeign

### 服务网关组件
### 服务熔断与降级组件