# RabbitMQ在Docker上的安装

首先去到 [RabbitMQ](https://hub.docker.com/_/rabbitmq)网站上获取命令

## 下载官方镜像
```bash
docker pull rabbitmq
```

### 下载带有管理界面的 TAG 镜像

```bash
docker pull rabbitmq:management
```

## 查看是否拉取镜像成功
```bash
docker images
```

如果执行上面命令出现下图的话，就代表拉取成功
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-27_14-10.png)


## 启动该镜像

使用以下命令将镜像启动起来
```bash
docker run -di --name myrabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabbitmq:management`
```

- name：指定容器名词
- e：指定key-value属性，这里设置rabbit默认用户账户和密码
- p：指定开放端口和端口映射，这里开放主机以及容器的15672、5672、25672、61613端口并一一对应起来
- d：（待补充）
- i：（待补充）

## 访问RabbitMQ的管理界面

访问控制台地址：`http://IP:15672`，ip替换成你的ip，这里一般为 `localhost` 。
访问之后效果如图下：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-08-27_14-22.png)
