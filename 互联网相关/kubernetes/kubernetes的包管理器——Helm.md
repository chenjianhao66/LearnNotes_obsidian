#Kubernetes 

# 一、Helm是什么
Helm 是 Kubernetes 的包管理器，是查找、分享和使用软件构建 Kubernetes 的最优方式。Helm 帮助用户管理 Kubernetes 应用——Helm 图表，即使是最复杂的 Kubernetes 应用程序，都可以方便的定义，安装和升级。

包管理器类似于我们在 Ubuntu 中使用的apt、Centos中使用的yum 或者Python中的 pip 一样，能快速查找、下载和安装软件包。Helm 由客户端组件 helm 和服务端组件 Tiller 组成, 能够将一组K8S资源打包统一管理, 是查找、共享和使用为Kubernetes构建的软件的最佳方式。

# 二、Helm解决了什么问题
在 Kubernetes中部署一个可以使用的应用，需要涉及到很多的 Kubernetes 资源的共同协作。比如你安装一个 WordPress 博客，用到了一些 Kubernetes (下面全部简称k8s)的一些资源对象，包括 Deployment 用于部署应用、Service 提供服务发现、Secret 配置 WordPress 的用户名和密码，可能还需要 pv 和 pvc 来提供持久化服务。并且 WordPress 数据是存储在mariadb里面的，所以需要 mariadb 启动就绪后才能启动 WordPress。这些 k8s 资源过于分散，不方便进行管理，直接通过 kubectl 来管理一个应用，你会发现这十分蛋疼。  
所以总结以上，我们在 k8s 中部署一个应用，通常面临以下几个问题：

-   如何统一管理、配置和更新这些分散的 k8s 的应用资源文件
-   如何分发和复用一套应用模板
-   如何将应用的一系列资源当做一个软件包管理

# 三、Helm的相关组件和概念
Helm 包含两个部分，分别是 helm 客户端 和 Tiller 服务器；
Helm概念如下：
## heml
一个命令行工具，用于本地开发及管理chart，chart仓库管理等

## Tiller
Helm 的服务端。Tiller 负责接收 Helm 的请求，与 k8s 的 apiserver 交互，根据chart 来生成一个 release 并管理 release
## chart
Helm的打包格式叫做chart，所谓chart就是一系列文件, 它描述了一组相关的 k8s 集群资源；Chart 代表着 Helm 包。
它包含在 Kubernetes 集群内部运行应用程序，工具或服务所需的所有资源定义。你可以把它看作是 Homebrew formula，Apt dpkg，或 Yum RPM 在Kubernetes 中的等价物。
## Repoistory 
是用来存放和共享 charts 的地方。它就像 Perl 的 CPAN 档案库网络或是 Fedora 的 软件包仓库，只不过它是供 Kubernetes 包所使用的。Helm 客户端通过 HTTP 协议来访问存储库中 chart 的索引文件和压缩包。

## Release
使用 helm install 命令在 Kubernetes 集群中部署的 Chart 称为 Release；
_Release_ 是运行在 Kubernetes 集群中的 chart 的实例。一个 chart 通常可以在同一个集群中安装多次。每一次安装都会创建一个新的 _release_。以 MySQL chart为例，如果你想在你的集群中运行两个数据库，你可以安装该chart两次。每一个数据库都会拥有它自己的 _release_ 和 _release name_。

因此，Helm可以解释为：
-   Helm在Kubernetes中安装_Chart_，并且每次安装会创建一个新的_Release_。
-   想查找新chart，可以在Helm Chart _仓库_ 搜索。

# 四、Helm的安装
Helm 提供了几种安装方式，这里只提供官方的脚本一键安装，想要查看更多安装方式，请阅读 Helm 的[官方文档](https://helm.sh/zh/docs/intro/install/)：
```bash
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
bash get_helm.sh
```

## 安装Tiller
安装好 helm 客户端后，就可以通过以下命令将 Tiller 安装在 kubernetes 集群中：
这个地方默认使用 https://kubernetes-charts.storage.googleapis.com 作为缺省的 stable repository 的地址，但由于国内有一张无形的墙的存在，googleapis.com是不能访问的。可以使用阿里云的源来配置：
```bash
helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.9.1  --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

> 注意，上面命令的Tiller版本需要和装在本机的Helm版本保持一致
> 使用 `helm version` 来查看 Helm 版本


执行上面命令后，可以通过 kubectl get po -n kube-system 来查看 tiller 的安装情况。
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-01_13-28.png)

## 初始化配置

### 配置用户权限
由于 kubernetes 从1.6 版本开始加入了 RBAC 授权。当前的 Tiller 没有定义用于授权的 ServiceAccount， 访问 API Server 时会被拒绝，需要给 Tiller 加入授权。
比如：
```bash
helm list

## 执行以上命令时会出现以下错误：
Error: configmaps is forbidden: User "system:serviceaccount:kube-system:default" cannot list resource "configmaps" in API group "" in the namespace "kube-system"
```
  
出现以上错误后需要执行以下命令：
```bash
kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

# 执行命令后使用以下命令验证是否成功
kubectl get deploy -n kube-system tiller-deploy -o yaml | grep serviceAccount

# 出现以下字样就成功
      serviceAccount: tiller
      serviceAccountName: tiller
	  

```
使用 helm list 就不会报错了。

### 初始化仓库

# 五、实践
## 查找一个chart
Helm 自带一个强大的搜索命令，可以用来从两种来源中进行搜索：
```bash
helm search {chartName}

# 示例
$ helm search mysql
NAME                            	CHART VERSION	APP VERSION	DESCRIPTION                                                 
office/mysql                    	1.6.9        	5.7.30     	DEPRECATED - Fast, reliable, scalable, and easy to use op...
office/mysqldump                	2.6.2        	2.4.1      	DEPRECATED! - A Helm chart to help backup MySQL databases...
office/prometheus-mysql-exporter	0.7.1        	v0.11.0    	DEPRECATED A Helm chart for prometheus mysql exporter wit...
stable/mysql                    	0.3.5        	           	Fast, reliable, scalable, and easy to use open-source rel...
office/percona                  	1.2.3        	5.7.26     	DEPRECATED - free, fully compatible, enhanced, open sourc...
office/percona-xtradb-cluster   	1.0.8        	5.7.19     	DEPRECATED - free, fully compatible, enhanced, open sourc...
office/phpmyadmin               	4.3.5        	5.0.1      	DEPRECATED phpMyAdmin is an mysql administration frontend   
stable/percona                  	0.3.0        	           	free, fully compatible, enhanced, open source drop-in rep...
stable/percona-xtradb-cluster   	0.0.2        	5.7.19     	free, fully compatible, enhanced, open source drop-in rep...
office/gcloud-sqlproxy          	0.6.1        	1.11       	DEPRECATED Google Cloud SQL Proxy                           
office/mariadb                  	7.3.14       	10.3.22    	DEPRECATED Fast, reliable, scalable, and easy to use open...
stable/gcloud-sqlproxy          	0.2.3        	           	Google Cloud SQL Proxy                                      
stable/mariadb                  	2.1.6        	10.1.31    	Fast, reliable, scalable, and easy to use open-source rel...
```
搜索是用来发现可用包的一个好办法。一旦找到想安装的 helm 包，便可以通过使用 `helm install` 命令来安装它。

## 安装一个helm包
使用 `helm install` 命令来安装一个新的 helm 包。最简单的使用方法只需要传入两个参数：你命名的release名字和你想安装的chart的名称。
```bash
helm install [releaseName] [chartName]
```
这里用nginx为例：
```bash

```

未完待续。

# 参考文档
[Helm官网网站](https://helm.sh/zh/docs/)
[Helm从入门到实践](https://www.jianshu.com/p/4bd853a8068b)
[kubernetes实战篇之helm填坑与基本命令](https://www.cnblogs.com/tylerzhou/p/11133115.html)
[Helm基础](https://zhuanlan.zhihu.com/p/341405124)