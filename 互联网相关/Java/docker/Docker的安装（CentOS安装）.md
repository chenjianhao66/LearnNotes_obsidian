2021-09-02
21:43:15
author:陈建浩
#Docker #容器技术 

--- 

## Centos8系统安装Docker
更新索引命令
```bash
yum makecache fast
```


### 安装依赖

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker。

安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```bash
$ yum install -y yum-utils \  
 device-mapper-persistent-data \  
 lvm2
```

### 设置仓库
**设置国内速度较快的阿里云仓库或者清华大学仓库**

阿里云源
```bash
$ yum-config-manager \  
 --add-repo \  
 http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

清华大学源

```bash
$ yum-config-manager \  
 --add-repo \  
 https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

### 安装Docker
```bash
$ yum install docker-ce docker-ce-cli containerd.io
```

> 上面的方式是默认安装最新版本的docker，如果想安装指定版本的可以参照下面的方法

**要安装特定版本的 Docker Engine-Community，请在存储库中列出可用版本，然后选择并安装：**
1、列出并排序您存储库中可用的版本。此示例按版本号（从高到低）对结果进行排序。
```bash
$ yum list docker-ce --showduplicates | sort -r  
  
docker-ce.x86_64 3:18.09.1-3.el7                     docker-ce-stable  
docker-ce.x86_64 3:18.09.0-3.el7                     docker-ce-stable  
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable  
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```

2、通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1。
```bash
$ yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
>在上面<VERSION_STRING>的占位符中选择自己想要安装的版本填充即可。

### 设置阿里云镜像加速（免费）
进入阿里云界面，登陆之后选择进入个人控制台；

选择**产品与服务**，如下图所示：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210826224223.png)

在下面的产品中，找到**弹性计算**分类中找到**容器镜像服务**，如下图所示：
![[Pasted image 20210826224341.png]]


在弹窗的页面中选择“镜像工具”-> “镜像加速器”，然后选择自己的操作系统，将下面弹出的命令输入到自己的Linux中即可

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210826224517.png)

```bash
sudo mkdir -p /etc/docker

sudotee /etc/docker/daemon.json <<-'EOF' 
{ 
	"registry-mirrors": ["https://lto1zi9i.mirror.aliyuncs.com"] 
} 
EOF

sudo systemctl daemon-reload 
sudo systemctl restart docker
```

## 卸载Docker
删除安装包：

```bash 
yum remove docker-ce
```

删除镜像、容器、配置文件等内容：

```bash
rm -rf /var/lib/docker
```