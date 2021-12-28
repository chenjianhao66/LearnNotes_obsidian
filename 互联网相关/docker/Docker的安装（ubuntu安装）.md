2021-09-02
21:43:23
author:陈建浩
#Docker #容器技术 

--- 

## ubuntu系统Dcoker的安装

在ubuntu系统中可以通过以下3种方法来安装Docker

- 设置Docker仓库来进行安装

- 下载官方DEB文件进行安装

- 使用官方的便捷安装脚本进行安装


###  1. 使用Docker仓库安装

在安装引擎之前，先设置Docker存储库，在存储库里面进行安装

####  设置Docker仓库
1. 更新软件库并允许从HTTPS使用存储库
```bash
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
	
```

2. 增加官方密钥

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

3. 增加以下命令

```bash
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 安装Docker引擎
1. 更新apt软件源版本，并且安装最新版本的docker引擎
```
 $ sudo apt-get update
 $ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

2. 如果要安装指定版本的引擎，使用以下命令来列出所有可用版本

```
 $ apt-cache madison docker-ce
```

如图
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210804215900.png)

将上一步所获取到的版本中，选择其中一种来替换以下命令中的{docker_version}字符，然后执行
```
 $ sudo apt-get install docker-ce=<docker_version> docker-ce-cli=<docker_version> containerd.io
```

3. 测试是否安装成功

```
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally  
latest: Pulling from library/hello-world  
1b930d010525: Pull complete Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f  
Status: Downloaded newer image for hello-world:latest  
  
  
Hello from Docker!  
This message shows that your installation appears to be working correctly.  
  
  
To generate this message, Docker took the following steps:  
 1. The Docker client contacted the Docker daemon.  
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.  
 (amd64)  
 3. The Docker daemon created a new container from that image which runs the  
 executable that produces the output you are currently reading.  
 4. The Docker daemon streamed that output to the Docker client, which sent it  
 to your terminal.  
  
  
To try something more ambitious, you can run an Ubuntu container with:  
 $ docker run -it ubuntu bash  
  
  
Share images, automate workflows, and more with a free Docker ID:  
 https://hub.docker.com/  
  
  
For more examples and ideas, visit:  
 https://docs.docker.com/get-started/
```
至此，docker安装成功

----
### 2. 下载官方Deb文件进行安装
如果无法使用Docker的存储库安装Docker引擎，则可以下载 `.deb` 文件以获取版本并手动安装。需要每次要升级Docker时下载一个新文件。

1. 从网址中[Index of linux/ubuntu/dists/ docker.com)](https://download.docker.com/linux/ubuntu/dists/)，在目录中选择 `ubuntu` 的版本，然后进入 `/pool/stable/` 目录下，下载相对应版本的包，如图所示

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210804221007.png)
安装 `docker-ce、docker-ce-cli、containerd.io` 三份，必须保持三样东西的版本是一致的。

2. 将以上3个软件包下载到本地之后，使用  `sudo dpkg -i`命令进行安装，

```
 sudo dpkg -i /path/to/package.deb
```


3. 运行 `hello-world` 容器进行测试

```
$ sudo docker run hello-world 
```
至此，安装docker成功

---

### 3. 使用便捷安装脚本进行安装
Docker 在 [get.docker.com](https://get.docker.com/) 和 [test.docker.com](https://test.docker.com/) 上提供了方便脚本，用于将快速安装 Docker Engine-Community 的边缘版本和测试版本。脚本的源代码在 docker-install 仓库中。 不建议在生产环境中使用这些脚本，在使用它们之前，您应该了解潜在的风险：

-   脚本需要运行 root 或具有 sudo 特权。因此，在运行脚本之前，应仔细检查和审核脚本。
    
-   这些脚本尝试检测 Linux 发行版和版本，并为您配置软件包管理系统。此外，脚本不允许您自定义任何安装参数。从 Docker 的角度或您自己组织的准则和标准的角度来看，这可能导致不支持的配置。
    
-   这些脚本将安装软件包管理器的所有依赖项和建议，而无需进行确认。这可能会安装大量软件包，具体取决于主机的当前配置。
    
-   该脚本未提供用于指定要安装哪个版本的 Docker 的选项，而是安装了在 edge 通道中发布的最新版本。
    
-   如果已使用其他机制将 Docker 安装在主机上，请不要使用便捷脚本。
    

本示例使用 [get.docker.com](https://get.docker.com/) 上的脚本在 Linux 上安装最新版本的Docker Engine-Community。要安装最新的测试版本，请改用 test.docker.com。在下面的每个命令，取代每次出现 get 用 test。
```
$ curl -fsSL https://get.docker.com -o get-docker.sh 
$ sudo sh get-docker.sh
```

如果要使用 Docker 作为非 root 用户，则应考虑使用类似以下方式将用户添加到 docker 组：
```
$ sudo usermod -aG docker your-user
```


### 4. 权限操作
如果在安装docker成功之后，执行docker命令却提示没有权限：
```bash
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied

```
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-30_17-48.png)

那么执行以下代码：
```bash
#添加docker用户组 
sudo groupadd docker 

#将登陆用户加入到docker用户组中
sudo gpasswd -a $USER docker 

#更新用户组
newgrp docker  

#测试docker命令是否可以使用sudo正常使用
docker ps 
```

### 5. 开启Docker主机的端口，允许API访问
修改Docker服务文件
```bash
vim /lib/systemd/system/docker.service
```
在 Service字段下，找到ExecStart属性，修改为以下值：
```bash
## 在 -H 选项后面跟的就是自己想要的开放的端口，我这里选择开放8088端口，自行选择

ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:8088 -H unix:///var/run/docker.sock
```
修改之后重新加载配置文件和重启docker服务
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker.service
```