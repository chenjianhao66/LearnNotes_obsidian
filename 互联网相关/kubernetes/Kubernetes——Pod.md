2021-12-02
08:57:27
author:陈建浩
#Kubernetes 

--- 
# 什么是Pod
_Pod_ 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。k8s是通过定义一个Pod的资源，然后在Pod里面运行容器，容器需要指定一个镜像，这样就可以用来运行具体的服务。一个Pod封装一个容器（也可以封装多个容器），**Pod里的容器共享存储、网络**等。也就是说，应该把整个pod看作虚拟机，然后每个容器相当于运行在虚拟机的进程。

_Pod_ （就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） [容器](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/#why-containers)； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的“逻辑主机”，其中包含一个或多个应用容器， 这些容器是相对紧密的耦合在一起的。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于 在同一逻辑主机上运行的云应用。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 [Init 容器](https://kubernetes.io/zh/docs/concepts/workloads/pods/init-containers/)。 你也可以在集群中支持[临时性容器](https://kubernetes.io/zh/docs/concepts/workloads/pods/ephemeral-containers/) 的情况下，为调试的目的注入临时性容器。


![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/image-20210709090640962.png)
Pod是需要调度到k8s集群的工作节点来运行的，具体调度到哪个节点，是根据`scheduler`调度器实现的。

可以把pod看成是一个“豌豆荚”，里面有很多“豆子”（容器）。一个豌豆荚里的豆子，它们吸收着共同的营养成分、肥料、水分等，Pod和容器的关系也是一样，Pod里面的容器共享pod的网络、存储等。

# Pod如何管理多个容器的
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/image-20210709090845912.png)

Pod中可以同时运行多个容器。同一个Pod中的容器会自动的分配到同一个 node 上。同一个Pod中的容器共享资源、网络环境，它们总是被同时调度，在一个Pod中同时运行多个容器是一种比较高级的用法，只有当你的容器需要紧密配合协作的时候才考虑用这种模式。例如，你有一个容器作为web服务器运行，需要用到共享的volume，有另一个`sidecar`容器来从远端获取资源更新这些文件。

一些Pod有`init容器`和应用容器，在应用程序容器启动之前，运行初始化容器。

# init容器
[Pod](https://kubernetes.io/docs/concepts/abstractions/pod/) 能够具有多个容器，应用运行在容器里面，但是它也可能有一个或多个先于应用容器启动的 Init 容器。

Init 容器与普通的容器非常像，除了如下两点：

-   Init 容器总是运行到成功完成为止。
-   每个 Init 容器都必须在下一个 Init 容器启动之前成功完成。

如果 Pod 的 Init 容器失败，Kubernetes 会不断地重启该 Pod，直到 Init 容器成功为止。然而，如果 Pod 对应的 `restartPolicy` 为 Never，它不会重新启动。

指定容器为 Init 容器，在 PodSpec 中添加 `initContainers` 字段，以 [v1.Container](https://kubernetes.io/docs/api-reference/v1.6/#container-v1-core) 类型对象的 JSON 数组的形式，还有 app 的 `containers` 数组。 Init 容器的状态在 `status.initContainerStatuses` 字段中以容器状态数组的格式返回（类似 `status.containerStatuses` 字段）。

Pod 里面可以有一个或者多个容器，部署应用的容器可以称为主容器，在创建Pod时候，Pod 中可以有一个或多个先于主容器启动的`Init容器`,这个init容器就可以成为初始化容器，初始化容器一旦执行完，它从启动开始到初始化代码执行完就退出了，它不会一直存在，所以在主容器启动之前执行初始化，初始化容器可以有多个，多个初始化容器是要串行执行的，先执行初始化容器1，在执行初始化容器2等，等初始化容器执行完初始化就退出了，然后再执行主容器，主容器一退出，pod就结束了，主容器退出的时间点就是pod的结束点，它俩时间轴是一致的；

Init容器就是做初始化工作的容器。可以有一个或多个，如果多个按照定义的顺序依次执行，只有所有的初始化容器执行完后，主容器才启动。**由于一个Pod里的存储卷是共享的，所以Init Container里产生的数据可以被主容器使用到**，Init Container可以在多种K8S资源里被使用到，如Deployment、DaemonSet, StatefulSet、Job等，但都是在Pod启动时，在主容器启动前执行，做初始化工作。

## init容器与其他容器不同的地方
Init 容器支持应用容器的全部字段和特性，包括资源限制、数据卷和安全设置。 然而，Init 容器对资源请求和限制的处理稍有不同，在下面 [资源](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#resources) 处有说明。 而且 Init 容器不支持 Readiness Probe，因为它们必须在 Pod 就绪之前运行完成。

如果为一个 Pod 指定了多个 Init 容器，那些容器会按顺序一次运行一个。 每个 Init 容器必须运行成功，下一个才能够运行。 当所有的 Init 容器运行完成时，Kubernetes 初始化 Pod 并像平常一样运行应用容器。

## init容器能做什么
因为 Init 容器具有与应用程序容器分离的单独镜像，所以它们的启动相关代码具有如下优势：

-   它们可以包含并运行实用工具，但是出于安全考虑，是不建议在应用程序容器镜像中包含这些实用工具的。
-   它们可以包含使用工具和定制化代码来安装，但是不能出现在应用程序镜像中。例如，创建镜像没必要 `FROM` 另一个镜像，只需要在安装过程中使用类似 `sed`、 `awk`、 `python` 或 `dig` 这样的工具。
-   应用程序镜像可以分离出创建和部署的角色，而没有必要联合它们构建一个单独的镜像。
-   Init 容器使用 Linux Namespace，所以相对应用程序容器来说具有不同的文件系统视图。因此，它们能够具有访问 Secret 的权限，而应用程序容器则不能。
-   它们必须在应用程序容器启动之前运行完成，而应用程序容器是并行运行的，所以 Init 容器能够提供了一种简单的阻塞或延迟应用容器的启动的方法，直到满足了一组先决条件。

# 创建一个Pod
## 通过资源清单创建Pod
资源清单指的的是一个后缀名为 `yaml` 的文件，里面都是 `key/value` 的字段，描述了一个pod的所有信息，可以使用命令 `kubectl explain pod` 来查看定义一个pod需要哪些字段，也可以 `kubectl explain pod.**.**` 来查看子字段信息；
```bash
kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status	<Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

```

通过 `explain` 命令去查看每个字段的含义以及用法
```yaml
apiVersion: v1 #版本号，例如v1 
kind: Pod #资源类型，如Pod 
metadata: #元数据 
	name: string # Pod名字 
	namespace: string # Pod所属的命名空间 
	labels: #自定义标签 
		- name: string #自定义标签名字 
	annotations: #自定义注解列表 
		- name: string
spec: # Pod中容器的详细定义
	containers: # Pod中容器列表
	- name: string #容器名称
	image: string #容器的镜像名称
	imagePullPolicy: [Always | Never | IfNotPresent] #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
	command: [string] #容器的启动命令列表，如不指定，使用打包时使用的启动命令
	args: [string] #容器的启动命令参数列表
	workingDir: string #容器的工作目录
	volumeMounts: #挂载到容器内部的存储卷配置
	- name: string #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
	  mountPath: string #存储卷在容器内mount的绝对路径，应少于512字符
	  readOnly: boolean #是否为只读模式
	ports: #需要暴露的端口号
	- name: string #端口号名称
	  containerPort: int #容器需要监听的端口号
	  hostPort: int #容器所在主机需要监听的端口号，默认与Container相同
	  protocol: string #端口协议，支持TCP和UDP，默认TCP
	env: #容器运行前需设置的环境变量列表
	- name: string #环境变量名称
	  value: string #环境变量的值
	resources: #资源限制和请求的设置
	  limits: #资源限制的设置
	    cpu: string #cpu的限制，单位为core数
		memory: string #内存限制，单位可以为Mib/Gib
	  requests: #资源请求的设置
	    cpu: string #cpu请求，容器启动的初始可用数量
		memory: string #内存请求，容器启动的初始可用内存
	livenessProbe: #对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
	  exec: #对Pod容器内检查方式设置为exec方式
	    command: [string] #exec方式需要制定的命令或脚本
	  httpGet: #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
	    path: string
		port: number
		host: string
		scheme: string
		HttpHeaders:
		- name:string
		  value:string
	restartPolicy: [Always | Never | OnFailure] #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
	nodeSelector: obeject #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
	imagePullSecrets: #Pull镜像时使用的secret名称，以key：secretkey格式指定
	- name: string
	hostNetwork:false #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
	volumes: #在该pod上定义共享存储卷列表
	- name: string #共享存储卷名称 （volumes类型有很多种）
	  emptyDir: {} #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
	  hostPath: string #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
	    path:string
	  secret:      #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
	  configMap: #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
```

# 使用Pod
使用Pod有2种方式，一种是 `自主式` 的pod，一种是通过控制器启动的pod

## 自主式Pod
所谓的自主式Pod，就是直接定义一个Pod资源
```bash
vim pod-tomcat.yaml

apiVersion: v1 
kind: Pod 
metadata: 
	name: tomcat-test 
	namespace: default 
	labels: 
		app: tomcat 
spec: 
	containers: 
		- name: tomcat-java 
          ports: 
   		  - containerPort: 8080 
		  image: tomcat:8.5-jre8-alpine 
		  imagePullPolicy: IfNotPresent
		  
## 应用pod-tomcat.yaml文件
kubectl apply -f pod-tomcat.yaml

## 查看pod是否创建成功
kubectl get pods -o wide -l app=tomcat
NAME          READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
tomcat-test   1/1     Running   0          4m59s   10.244.1.5   slave-node-1   <none>           <none>

## 如果删除掉这个pod
kubectl delete pods tomcat-test

## 再次查看就会看不到这个pod了
kubectl get pods -o wide -l app=tomcat
```

就会发现这个pods已经找不到了，这就是自主式pods的坏处；
而使用控制器管理pod的话，则不会出现这些问题，并且控制器管理的Pod可以确保Pod始终维持在指定的副本数运行。

## 副本控制器

Kubernetes的副本控制器根据用途一共有6种。

根据pod里容器跑的应用程序来分类，可以分为有状态应用和无状态应用控制；
从应用程序是否运行为守护进程我们可以将控制器分为，守护进程和非守护进程控制器；
其中无状态控制器中最常用的有ReplicaSet控制器和Deployment控制；
有状态应用控制器常用的有StatefulSet；守护进程控制器最常用的有daemonSet控制器；
非守护进程控制器有job控制器，对Job类型的控制器，如果要周期性执行的有Cronjob控制器；

以下是各个pod控制器的分类
- 无状态应用控制器
	- ReplicaSe
	- Deployment
- 有状态应用控制器
	- StatefulSet
- 守护进程控制器
	- daemonSet
- 非守护进程控制器
	- job控制器
		- Job
		- CronJob


[[Replicaset]]
[[Deployment]]
[[Daemonset]]
[[StatefulSet]]
[[Job]]
[[CronJob]]

如果对应资源不吻合用户定义的资源状态，它就会尝试重启或重建的方式让其状态和用户定义的状态吻合；也就是说在声明控制器的时候也会声明对该pod种每个容器的期望状态，如果pod内容器没有满足期望状态，那么控制器就会作出操作（增加容器，删除容器，调度容器.....），让容器状态与期望状态相等。

# 标签
标签其实就一对 `key/value` ，被关联到对象上，比如Pod,标签的使用我们倾向于能够表示对象的特殊特点，就是一眼就看出了这个Pod是干什么的，标签可以用来划分特定的对象（比如版本，服务类型等），**标签可以在创建一个对象的时候直接定义，也可以在后期随时修改，每一个对象可以拥有多个标签，但是，key值必须是唯一的。**创建标签之后也可以方便我们对资源进行分组管理。如果对pod打标签，之后就可以使用标签来查看、删除指定的pod。  

 对于pod来讲，我们知道使用pod控制器创建的pod在pod故障以后，重建后的pod它的ip地址和名称是变化的，为了解决pod访问问题，我们特此创建了service，我们访问service的ip地址就可以正常访问到pod；那么问题来了，service是怎样去关联pod的呢？我们知道在k8s上如果使用pod控制创建的pod，在pod发生故障以后，对应pod会被对应的控制器重启或重建，一个pod重建以后，对应的ip地址和名称都是会发生变化的，所以靠ip地址和名称关联pod是不行的；那靠什么关联pod呢？**在k8s上是使用的标签和标签选择器的机制实现资源和资源间相互关联的；**
 
** 为了解决pod在重新调度之后的ip地址和名称变化而无法让Service和pod关联起来的问题，选择使用标签和标签选择器来解决问题。**

在k8s中，大部分资源都可以打标签。

```bash
# 获取一个pod下面的所有标签
kubectl get pods tomcat-test --show-labels
NAME          READY   STATUS    RESTARTS   AGE    LABELS
tomcat-test   1/1     Running   0          3h6m   app=tomcat

# 对一个pod打标签
# 注意，在打标签的时候如果这个pod已经有了这个
kubectl label pods tomcat-test label=test
pod/tomcat-test labeled

# 再次查看
kubectl get pods tomcat-test --show-labels
NAME          READY   STATUS    RESTARTS   AGE    LABELS
tomcat-test   1/1     Running   0          3h7m   app=tomcat,label=test
```

关于查看标签的具体用法
查看标签：
标签较多时，也可以使用-L来指定显示那些标签；
```bash
# 查看默认名称空间下所有pod资源的标签
kubectl get pods --show-labels

# 查看默认名称空间下指定pod具有的所有标签
kubectl get pods pod-first --show-labels

# 列出默认名称空间下标签key是release的pod，不显示标签
kubectl get pods -l release=v1

# 列出默认名称空间下标签key是release的所有pod，并打印对应的标签值
kubectl get pods -L release

# 查看所有名称空间下的所有pod的标签
kubectl get pods --all-namespace --show-labels

# 查看指定名称空间的指定标签
kubectl get pods nginx -L app,nginx

```

增加标签：
```bash
# 对一个pod增加标签，给ngxin添加标签test和test2，value都等于tt
kubectl label pods nginx test=tt test2=ttt
```

删除标签：
```bash
# 对一个pod删除标签
kubectl label pods nginx test- test2-
```
> 删除标签只需要在要删除的标签键名后面加上“-”号即可；

修改标签：
```bash
# 修改pod的标签，强制覆盖已有的app标签
kubectl label pods nginx app=nginx --overwrite
```
> 修改一个已有的标签值，需要加上 `--overwrite` 参数


## 标签选择器
所谓标签选择器是指一组表达式，主要用来表达标签查询条件或选择标准；
在k8s上支持两种类型的标签选择器，一种是基于等值关系的选择器，一种是基于集合关系的选择器；
### 基于等值关系的选择器
基于等值关系的选择器，通常可以用=、==或!=这些操作符来表示关系；前两个表示同一个意思相等，后面的!=表示不等；

### 基于集合关系的选择器
基于集合关系的选择器有3个操作符，分别是：
- in
- notin
- exists

in表示指定键名的值在给定的列表中就满足条件；
notin和in相反；
exists表示是否存在对应的键名；
> 比如key表示所有存在此键名标签的资源；！key表示所有不存在此键名标签的资源；

此外在使用标签选择器时遵循以下逻辑：
- 如果同时指定多个选择器时，选择器之间是逻辑与的关系，表示指定的选择器都要同时满足；
- 使用空值的标签选择器意味着所有带有这个key的资源都被选中；
- 空的标签选择器将无法选出任何资源；空值和空的选择器是两会事，一个是有键名但其值为空，另一个是连键名都没有。

使用标签选择器过滤出标签为app=ngx-dep的资源
```bash
# 显示所有的pod和标签
kubectl get pod --show-labels
NAME                         READY   STATUS    RESTARTS   AGE    LABELS
myapp-dep-5bc4d8cc74-cvkbc   1/1     Running   1          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
myapp-dep-5bc4d8cc74-gmt7w   1/1     Running   2          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
myapp-dep-5bc4d8cc74-gqhh5   1/1     Running   1          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
ngx-dep-5c8d96d457-w6nss     1/1     Running   1          7d3h   app=ngx-dep,pod-template-hash=5c8d96d457

# 使用标签选择器
kubectl get pod -l "app=ngx-dep" -L app
NAME                         READY   STATUS    RESTARTS   AGE    LABELS
ngx-dep-5c8d96d457-w6nss     1/1     Running   1          7d3h   app=ngx-dep,pod-template-hash=5c8d96d457
```
> 使用标签选择器来过滤资源，需要用-l选项来指定标签选择器；


使用标签选择器查看app!=ngx-dep的pod
```bash
# 显示所有的pod和标签
kubectl get pod --show-labels
NAME                         READY   STATUS    RESTARTS   AGE    LABELS
myapp-dep-5bc4d8cc74-cvkbc   1/1     Running   1          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
myapp-dep-5bc4d8cc74-gmt7w   1/1     Running   2          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
myapp-dep-5bc4d8cc74-gqhh5   1/1     Running   1          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
ngx-dep-5c8d96d457-w6nss     1/1     Running   1          7d3h   app=ngx-dep,pod-template-hash=5c8d96d457

# 使用标签选择器
kubectl get pod -l "app!=ngx-dep" -L app
NAME                         READY   STATUS    RESTARTS   AGE    LABELS
myapp-dep-5bc4d8cc74-cvkbc   1/1     Running   1          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
myapp-dep-5bc4d8cc74-gmt7w   1/1     Running   2          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
myapp-dep-5bc4d8cc74-gqhh5   1/1     Running   1          7d2h   app=myapp-dep,pod-template-hash=5bc4d8cc74
```
> 也可以使用多个标签选择器，多个标签选择器之间使用逗号间隔

集合选择器示例
```bash
# 获取 nginx命名空间的所有pod和标签
kubectl get pod  -n nginx --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx-pod-demo    1/1     Running   1          7d      env=testing
nginx-pod-demo3   1/1     Running   0          5m33s   app=nginx,env=test,rel=stable
nginx-pod-demo4   1/1     Running   0          5m57s   app=nginx,env=testing,rel=stable
nginx-pod-demo5   1/1     Running   0          33m     app=ngx,env=testing

# 使用集合选择器
# 查询包含标签app并且取值是ngx或者nginx的pod
kubectl get pod -n testing -l "app in (ngx,nginx)" --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx-pod-demo3   1/1     Running   0          5m33s   app=nginx,env=test,rel=stable
nginx-pod-demo4   1/1     Running   0          5m57s   app=nginx,env=testing,rel=stable
nginx-pod-demo5   1/1     Running   0          33m     app=ngx,env=testing

# 查询app标签不在ngx和nginx范围内的容器
# 没有app标签的pod和有app标签但是取值不是ngx和nginx会被查询到）
kubectl get pod -n testing -l "app notin (ngx,nginx)" --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx-pod-demo    1/1     Running   1          7d      env=testing


# 查询存在app这个键的pod
kubectl get pod -n testing -l "app" --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx-pod-demo3   1/1     Running   0          5m33s   app=nginx,env=test,rel=stable
nginx-pod-demo4   1/1     Running   0          5m57s   app=nginx,env=testing,rel=stable
nginx-pod-demo5   1/1     Running   0          33m     app=ngx,env=testing

# 查询不存在app这个键的pod 
kubectl get pod -n testing -l "!app" --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx-pod-demo    1/1     Running   1          7d      env=testing
```

# node选择器
我们在创建pod资源的时候，pod会根据schduler进行调度，那么默认会调度到随机的一个工作节点，如果我们想要pod调度到指定节点或者调度到一些具有相同特点的node节点，怎么办呢？可以使用pod中的`spec.nodeName`或者`spec.nodeSelector`字段指定要调度到的node节点。

## nodeName
指定pod节点运行在哪个具体node上
```bash
vim pod-node.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: demo-pod
  namespace: default
  labels:
    app: myapp
spec:
  nodeName: slave-node-1 # 指定pod调度到 slave-node-1 上
  containers:
  - name: tomcat-pod-java
  ports:
  - containerPort: 8080
  image: tomcat:8.5-jre8-alpine
  imagePullPolicy: IfNotPresent
```
## nodeSelector
指定pod调度到具有哪些标签的node节点上，如果有多个具有相同标签的node节点，由k8s来决定调度到哪个节点。
```bash
# 给node节点打标签
kubectl label nodes slave-node-2 disk=ceph

vim pod-node.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: demo-pod
  namespace: default
  labels:
    app: myapp
spec:
  nodeSelector: 
    disk: ceph
  containers:
  - name: tomcat-pod-java
  ports:
  - containerPort: 8080
  image: tomcat:8.5-jre8-alpine
  imagePullPolicy: IfNotPresent
```
>尽量让打在node节点上的标签具有唯一性。

# Pod网络
每个 Pod 都在每个地址族中获得一个唯一的 IP 地址。 Pod 中的每个容器共享网络名字空间，包括 IP 地址和网络端口。 _Pod 内_ 的容器可以使用 `localhost` 互相通信。 当 Pod 中的容器与 _Pod 之外_ 的实体通信时，它们必须协调如何使用共享的网络资源 （例如端口）。

在同一个 Pod 内，所有容器共享一个 IP 地址和端口空间，并且可以通过 `localhost` 发现对方。 他们也能通过如 SystemV 信号量或 POSIX 共享内存这类标准的进程间通信方式互相通信。 不同 Pod 中的容器的 IP 地址互不相同，没有 [特殊配置](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/) 就不能使用 IPC 进行通信。 如果某容器希望与运行于其他 Pod 中的容器通信，可以通过 IP 联网的方式实现。

Pod 中的容器所看到的系统主机名与为 Pod 配置的 `name` 属性值相同。 [网络](https://kubernetes.io/zh/docs/concepts/cluster-administration/networking/)部分提供了更多有关此内容的信息。

> 上文中提到的地址族可以在k8s创建集群的时候指定
> ```bash
> registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.3 \
--pod-network-cidr=10.244.0.0/16
>```
>参数 `--pod-network-cidr=10.244.0.0/16` 就是用来指定pod所使用的内部网段

这个时候客户端还不允许直接访问pod下的容器服务，还需要 [[Kubernetes——Service|Service]] 来代理请求，这样才能访问服务。



# 参考文档
[kubernetes官网文档](https://kubernetes.io/zh/docs/concepts/workloads/pods/#pods-and-controllers)