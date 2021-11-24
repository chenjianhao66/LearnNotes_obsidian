2021-11-23
16:20:23
author:陈建浩
#Kubernetes

--- 
## 安装kubernetes

### 安装前的准备

#### 一、安装docker

#### 二、关闭swap交换分区

这里有两种方式，临时关闭和永久关闭

**临时关闭**

```bash
sudo swapoff -a
```

**永久关闭**

编辑/etc/fstab,用# 注释里面涉及 swap 行

```bash
sudo vim /etc/fstab
```

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-19_09-54-1.png)



#### 三、检查hostname

```sh
# 此处 hostname 的输出将会是该机器在 Kubernetes 集群中的节点名字
# 不能使用 localhost 作为节点的名字
cat /etc/hosts


## 获取hostname
hostname

# 修改 hostname
hostnamectl set-hostname your-new-host-name

# 修改host解析，127.0.0.1 设置为自己的hostname，不能是localhost
vim /etc/hostname
```

------



### 安装工具

##### 安装kubeadm、kubelet、kubectl（国内版本）

1. 更新 apt 包索引并安装使用 Kubernetes apt 仓库所需要的包

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

2.添加 Kubernetes apt 仓库

添加apt key以及源

```bash
# 添加key
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

# 添加k8s的阿里源到源列表
# 如果提示没有权限，那么就进入到 /etc/apt/sources.list.d/目录下，新建kubernetes.list文件，将deb语句输入到文件里并保存
echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" >>/etc/apt/sources.list.d/kubernetes.list
```



3. 安装

```bash
sudo apt-get update

## 没有指定版本那就安装最新版本的
sudo apt-get install -y kubelet kubeadm kubectl

## 安装指定版本
sudo apt-get install -y kubelet=1.22.3-00 kubeadm=1.22.3-00 kubectl=1.22.3-00
```

若不指定则默认安装最新版本。更多可用版本列表，通过` sudo apt-cache policy kubelet` 查看

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-17_19-06.png)

------



### 安装kubemetes

在安装完 `kubectl`、`kubeadm`、`kubelet`工具后，就可以通过 `kubeadm`设置k8s集群了，执行以下命令：

```bash
sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.3 \
--pod-network-cidr=10.244.0.0/16
```

`init` 命令可选的参数分别有：

| 参数                          | 描述                                                         | 值                                      |
| ----------------------------- | ------------------------------------------------------------ | --------------------------------------- |
| --image-repository            | 指定镜像的仓库，国内使用阿里云即可。                         | registry.aliyuncs.com/google_containers |
| --kubernetes-version          | 这里需要和本机装的kubectl、kubeadm版本保持一致               | v1.22.3                                 |
| --pod-network-cidr            | 指定pod使用的网段                                            | 10.244.0.0/16                           |
| --cert-dir                    | 指定证书的路径                                               |                                         |
| --dry-run                     | 输出将要执行的操作，不做任何改变                             |                                         |
| --feature-gates               | 指定功能配置键值对，可控制是否启用各种功能                   |                                         |
| -h, --help                    | 输出init命令的帮助信息                                       |                                         |
| --apiserver-advertise-address | 指定API Server地址                                           |                                         |
| --apiserver-cert-extra-sans   | 指定API Server的服务器证书                                   |                                         |
| --ignore-preflight-errors     | 忽视检查项错误列表，例如“IsPrivilegedUser,Swap”，如填写为 'all' 则将忽视所有的检查项错误 |                                         |
| --kubernetes-version          | 指定Kubernetes版本                                           |                                         |
| --node-name                   | 指定节点名称                                                 |                                         |
| --pod-network-cidr            | 指定pod网络IP地址段                                          | 10.244.0.0/16                           |
| --service-cidr                | 指定service的IP地址段                                        |                                         |
| --service-dns-domain          | 指定Service的域名，默认为“cluster.local”                     |                                         |
| --skip-token-print            | 不打印Token                                                  |                                         |
| --token                       | 指定token                                                    |                                         |
| --token-ttl                   | 指定token有效时间，如果设置为“0”，则永不过期                 |                                         |



在启动之后会等待片刻，出现以下字样就代表安装成功。

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-19_15-16.png)



在出现以上字样之后需要将kubectl的文件移动到HOME目录下，这样才能使用 `kubectl` 命令

执行以下命令

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

获取 `kubernetes` 集群各个组件的状态以及节点状态

```bash
# 获取master节点名和worker节点状态
kubectl get nodes

# 获取所有pod状态
kubectl get pods --all-namespaces
```

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-19_15-21.png)



在上图可以看到，获取pod状态时，`coredns`节点的状态不是 `Running`而是 `Pending`，这是因为集群目前还没有安装网络插件，K8S没有准确的定义一个网络插件，而是定义了接口，实现了该接口的网络插件K8S就可以运行；



#### 安装网络插件

K8S没有准确的定义一个网络插件，而是定义了接口，实现了该接口的网络插件K8S就可以运行；K8S网络插件接口各大厂商都有进行[实现](https://kubernetes.io/zh/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)，这里使用的是 `flannel` 。

网络插件你可以从网上下载一个插件的配置文件然后应用，也可以直接引用网上的插件；

> 第一种方法适合网络不好的场景，第二种适合网络好的。

##### 下载文件

```bash
## 获取flannel配置文件
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

## 应用该配置文件
kubectl apply -f kube-flannel.yml
```



##### 直接应用网络上的配置文件

> 这种方式无需下载文件到本地，但需要网络好

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```



这两种方式运行之后再获取通过命令 `kubectl get pods --all-namespaces` 去查看各个组件状态，发现 `coredns` 组件依然还是 `Pending` 状态，这是因为再应用网络插件时，这个插件也在下载仓库里的镜像进行应用，这里可以使用以下命令去验证：

```bash
kubectl describe pods kube-flannel-ds-amd64-vcmx9 -n kube-system
```

> 命令中的 `kube-flannel-ds-amd64-vcmx9` 需要配置自己的网络插件名称，通过 `kubectl get pods --all-namespaces` 获取到。

使用命令后可以获取到这个组件的详细信息，详细信息的前面不需要看，到最底下的 `Evnets` 标签，步骤会卡在 `"quay.io/coreos/flannel:v0.15.1"`这个镜像的拉取上，所以等待拉取镜像成功即可。

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-22_09-44.png)

如果等不了，可以取巧；将这个镜像名复制下来，然后去到阿里云将相同的镜像下载下来，然后进行改成与事件中相同的镜像名称。这个时候再重新部署网络插件的话发现本地就有这个镜像就不会去网络上拉取镜像，而是直接应用本地镜像；

```bash
docker pull registry.cn-shenzhen.aliyuncs.com/myownmirrors/quay.io_coreos_flannel:v0.15.1
docker tag registry.cn-shenzhen.aliyuncs.com/myownmirrors/quay.io_coreos_flannel:v0.15.1 quay.io/coreos/flannel:v0.12.0-amd64
docker rmi registry.cn-shenzhen.aliyuncs.com/myownmirrors/quay.io_coreos_flannel:v0.15.1
```

执行完以上步骤之后就可以发现，网络插件部署成功，`coredns`组件也是 `Running` 状态了

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-22_09-57.png)

------



```bash
kubeadm join 10.1.3.203:6443 --token mk59c0.2jwnb5d7194iry3c \
	--discovery-token-ca-cert-hash sha256:685e1097f492bdbf90aa215810dab32a0311388e423a81fde287656fa947934c
```







## 参考文章

[七、K8S初上手：安装Flannel网络插件](https://juejin.cn/post/6894457482635116551#heading-3)

[使用 Kubeadm 安装 Kubernetes 集群](https://windcoder.com/shiyong-kubeadm-anzhuang-kubernetes-jiqun)

[k8s官网](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)

[kubeadm部署Kubernetes 网络插件flannel、Calico、weave 并设置集群角色](https://blog.csdn.net/cojn52/article/details/109449828)

