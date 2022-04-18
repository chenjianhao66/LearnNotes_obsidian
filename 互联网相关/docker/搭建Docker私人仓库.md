#Docker #容器技术 

# 搭建私人仓库
 官方在Docker hub上提供了registry的镜像，我们可以直接使用该registry镜像来构建一个容器，搭建自己的私有仓库服务。
 
 ## 拉取最新版本的registry镜像
 ```bash
 docker pull registry
 ```
 
 ## 运行registry容器
 
```bash
docker run -d --name registry -p 5000:5000 --restart=always \
-v /home/yunzhou/images_repo:/var/lib/registry \
-v config.yaml:/etc/docker/registry/config.yaml \
registry
```
run命令参数如下：
`-v /home/yunzhou/images_repo:/var/lib/registry` 代表把镜像存储目录挂载到本地，方便管理和持久化
`-v config.yaml:/etc/docker/registry/config.yaml` 代表把配置文件挂载到本地，方便修改和保存，挂载的配置文件内容如下：

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true ## 代表该仓库镜像是可删除的，其他都是默认项
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

