2021-11-30
21:43:17
author:陈建浩
#Java #中间件 #RabbitMQ 

--- 

# SpringBoot整合RabbitMQ
### 1、配置Maven依赖
```xml
<dependency>  
	 <groupId>org.springframework.boot</groupId>  
	 <artifactId>spring-boot-starter-amqp</artifactId>  
</dependency>
```

### 2、配置SpringBoot配置文件
```xml
## 与RabbitMQ的配置
spring.application.name=rabbitmq  
spring.rabbitmq.addresses=localhost  
spring.rabbitmq.port=5672  
spring.rabbitmq.username=admin  
spring.rabbitmq.password=admin  
spring.rabbitmq.virtual-host=/
```

### 3、