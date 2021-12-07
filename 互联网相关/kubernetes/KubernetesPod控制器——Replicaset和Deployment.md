2021-12-03
09:19:44
author:陈建浩
#Kubernetes 

--- 
#  Replicaset控制器
## Replicaset概述
`ReplicaSet`是kubernetes中的一种副本控制器，简称`rs`，主要作用是控制由其管理的pod，使pod副本的数量始终维持在预设的个数。它的主要作用就是保证一定数量的Pod能够在集群中正常运行，它会持续监听这些Pod的运行状态，在Pod发生故障时重启pod，pod数量减少时重新运行新的 Pod副本。

**官方推荐不要直接使用ReplicaSet，用Deployments取而代之**，Deployments是比ReplicaSet更高级的概念，它会管理ReplicaSet并提供很多其它有用的特性，最重要的是Deployments支持声明式更新，声明式更新的好处是不会丢失历史变更。所以Deployment控制器不直接管理Pod对象，而是由 Deployment 管理ReplicaSet，再由ReplicaSet负责管理Pod对象。

## Replicaset的工作原理
Replicaset核心作用在于用户创建指定数量的pod副本，并确保pod副本一直处于满足用户期望的数量， 起到多退少补的作用，并且还具有自动扩容缩容等制。  
Replicaset控制器主要由三个部分组成：
1、**用户期望的pod副本数**：用来定义由这个控制器管控的pod副本有几个，由`spec.replicas`来控制
2、**标签选择器**：选定哪些pod是自己管理的，由`spec.selector`字段控制，如果通过标签选择器选到的pod副本数量少于我们指定的数量，需要用到下面的组件  
3、**pod资源模板**：如果集群中现存的pod数量不够我们定义的副本中期望的数量怎么办，需要新建pod，这就需要pod模板，新建的pod是基于模板来创建的，由`spec.template`字段来控制

## Replicaset使用案例
```bash
## 编写yaml文件
$ vim replicaset-demo.yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: nginx
    test: replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        test: replicaset
    spec:
      containers:
      - name: nginx
        image: nginx:1.14-alpine
        imagePullPolicy:  IfNotPresent
		
## 应用该yaml文件
$ kubectl apply -f replicaset-demo.yaml
## 查看pod
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
frontend-2rxms           1/1     Running   0          4h32m
frontend-xv6fl           1/1     Running   0          4h32m
```
yaml文件字段详解：
定义ReplicaSet控制器，`apiVersion`字段的值为apps/v1，`kind`为`ReplicaSet`，这两个字段都是固定的；`spec.replicas`字段用于声明期望拥有几个副本；
`spec.selector`字段用于定义标签选择器，值是一个对象：其中 `matchLabels` 字段表示精确匹配标签，这个字段的值为一个字典；除了精确匹配标签选择器这种方式，还有  `matchExpressions` 表示使用匹配表达式，其值为一个对象，[[Kubernetes——Pod#标签选择器 | 该字段的使用与标签选择器很类似]]；
其次就是定义pod模板，使用 `template` 字段定义，该字段的值为一个对象其中 `metadata`字段用于定义模板的元素据信息，这个元数据信息必须定义标签属性；通常这个标签属性和选择器中的标签相同；
`spec` 字段用于定义pod模板的状态，最重要的是定义pod里容器的名字，镜像等等；

### Replicaset的动态扩容和伸缩
列出目前具有 `app` 标签的pod
```bash
$ kubectl get pod -L app
NAME                     READY   STATUS    RESTARTS   AGE     APP
frontend-2rxms           1/1     Running   0          4h56m   nginx
frontend-xv6fl           1/1     Running   0          4h56m   nginx
nginx                    1/1     Running   0          24h     nginx
tomcat-test              1/1     Running   0          24h     tomcat
```
可以看出目前有 `app` 标签的pod有4个，根据之前的`reolicaset-demo.yaml` 文件中 `spec.replicas` 字段得知我们期望的pod数量是2，所以看出来目前以`frontend`为前缀的pod有2个。
那么现在我要对 `frontend-2rxms` 这个pod的标签进行修改，它会有什么变化
```bash
$ kubectl label pod frontend-2rxms app=nginx11 --overwrite
pod/frontend-2rxms labeled
$ kubectl get pod -L app
NAME                     READY   STATUS    RESTARTS   AGE     APP
frontend-2rxms           1/1     Running   0          4h57m   nginx11
frontend-flklj           1/1     Running   0          7s      nginx
frontend-xv6fl           1/1     Running   0          4h57m   nginx
nginx                    1/1     Running   0          24h     nginx
tomcat-test              1/1     Running   0          24h     tomcat
```
当我对 `frontend-2rxms` 这个pod的`app`标签修改了之后，Replicaset控制器又会根据配置文件中`spec.template` 字段所定义的模板创建了一个pod；
那我把 `frontend-2rxms` 这个pod的标签修改回来，看看对应控制器是否会删除一个pod呢？
```bash
$ kubectl label pod frontend-2rxms app=nginx --overwrite
pod/frontend-2rxms labeled
$ kubectl get pod -L app
NAME                     READY   STATUS    RESTARTS   AGE     APP
frontend-2rxms           1/1     Running   0          4h59m   nginx
frontend-xv6fl           1/1     Running   0          4h59m   nginx
nginx                    1/1     Running   0          24h     nginx
tomcat-test              1/1     Running   0          24h     tomcat

```
当修改标签到原来的值后，新创建的pod就已经被删除了，pod数量也维持到我所期望的2个，这个也是在配置文件里面定义的。

我们也可以查看这个控制器的详情：
```bash
$ kubectl describe rs frontend
Name:         frontend
Namespace:    default
Selector:     app=nginx
Labels:       app=nginx
              test=replicaset
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
           test=replicaset
  Containers:
   nginx:
    Image:        nginx:1.14-alpine
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  11m   replicaset-controller  Created pod: frontend-flklj
  Normal  SuccessfulDelete  10m   replicaset-controller  Deleted pod: frontend-flklj

```
在这个控制器的 `Events` 字段中可以看到自动创建了1个pod以及删除了pod。
> 结论：可以看到当集群中有多余用户期望数量的pod标签时，对应控制器会把多余的相同标签的pod删除；
> 从上面的测试可以看到ReplicaSet控制器是依靠标签选择器来判断集群中pod的数量是否和用户定义的数量吻合，如果不吻合就尝试删除或新建，让对应pod数量精确满足用户期望pod数量；


那如果我要对这个程序容器扩容呢？
有通过 `kubectl scale` 命令行去扩容，也可以直接修改 `yaml` 文件扩容，这里只演示修改 `yaml` 文件的方式；
```bash
# 目前只有3个pod
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
frontend-2rxms           1/1     Running   0          5h28m
frontend-sr7c2           1/1     Running   0          3s
frontend-xv6fl           1/1     Running   0          5h28m

# 修改 replicaset-demo.yaml 配置文件，仅修改 spec.replicas字段的值，修改为5
vim replicaset-demo.yaml

# 应用配置文件
$ kubectl apply -f replicaset-demo.yaml

# 查看pod
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
frontend-2rxms           1/1     Running   0          5h31m
frontend-hb5tv           1/1     Running   0          5s
frontend-qvlzc           1/1     Running   0          5s
frontend-sr7c2           1/1     Running   0          2m6s
frontend-xv6fl           1/1     Running   0          5h31m
```
已经扩容成功！
缩减也同理，修改配置文件的 `spec.replicas` 字段应用即可。


### Pod版本更新
扩容和缩减也实现了，那我要对Pod内容器的镜像版本更新呢？
2个方法，一个是通过 `kubectl` 命令去更改，另外一个就是修改配置文件；这里只演示修改配置文件的方式。
```bash
# 查看目前的nginx版本，版本为1.14
$ kubectl describe pod frontend-2rxms | grep Image
    Image:          nginx:1.14-alpine
    Image ID:       docker-pullable://nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7

# 修改 replicaset-demo.yaml 配置文件，修改版本为 nginx:1.16-alpine
$ vim replicaset-demo.yaml

# 应用配置文件
$ kubectl apply -f replicaset-demo.yaml

# 查看pod版本有没有更新
$ kubectl describe pod frontend-2rxms | grep Image
    Image:          nginx:1.14-alpine
    Image ID:       docker-pullable://nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7
	
# pod的版本并没有更新，那利用自动扩容伸缩的原理，删除一个pod并查看新的pod版本
$ kubectl delete pod frontend-2rxms
pod "frontend-2rxms" deleted
yunzhou-test4@yunzhou-test4:~/kubernetes/Replicaset$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
frontend-st256           0/1     ContainerCreating   0          3s
frontend-xv6fl           1/1     Running             0          5h42m

# 查看版本，发现已经是1.16版本了
$ kubectl describe pod frontend-st256 | grep Image
    Image:          nginx:1.16-alpine
    Image ID:       docker-pullable://nginx@sha256:5057451e461dda671da5e951019ddbff9d96a751fc7d548053523ca1f848c1ad

```
可以看到我们删除了一个pod，对应控制器又新建了一个pod，对应新建的pod镜像版本就成为了新版本的pod；
>结论：从上面测试情况可以看到，对于rs控制器当pod模板中的镜像版本发生更改，如果k8s集群上对应pod数量和用户定义的数量吻合，此时rs控制器不会更新pod；只有新建后的pod才会拥有新版本；也就说如果我们要rs来对pod版本更新，就得删除原有老的pod后才会更新；

# Deploymont控制器
## Deploymont概述
`  
Deployment`是kubernetes中最常用的资源对象，为ReplicaSet和Pod的创建提供了一种`声明式`的定义方法，在Deployment对象中描述一个期望的状态，Deployment控制器就会按照一定的控制速率把实际状态改成期望状态，通过定义一个Deployment控制器会创建一个新的ReplicaSet控制器，通过ReplicaSet创建pod，删除Deployment控制器，也会删除Deployment控制器下对应的ReplicaSet控制器和pod资源

 对于deployment控制来说，它的定义方式和rs控制都差不多，但deploy控制器的功能要比rs强大，它可以实现滚动更新，用户手动定义更新策略；其实deploy控制器是在rs控制器的基础上来管理pod；也就说我们在创建deploy控制器时，它自动会创建一个rs控制器；

Deployment控制器是建立在rs之上的一个控制器，可以管理多个rs，每次更新镜像版本，都会生成一个新的rs，把旧的rs替换掉，多个rs同时存在，但是只有一个rs运行。

Deployment控制器的更新流程

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/12-31631.png)

从上图可以看出，rs v1控制三个pod，删除一个pod，在rs v2上重新建立一个，依次类推，直到全部都是由rs v2控制，如果rs v2有问题，还可以回滚，Deployment是建构在rs之上的，多个rs组成一个Deployment，但是只有一个rs处于活跃状态。

通过Deployment对象，你可以轻松的做到以下事情：

1.  创建ReplicaSet和Pod
2.  滚动升级（不停止旧服务的状态下升级）和回滚应用（将应用回滚到之前的版本）
3.  平滑地扩容和缩容
4.  暂停和继续Deployment



## Deployment使用案例
```bash
# 创建一个名称为 deployment 的deployment
$ cat deployment-demo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      demo: deployment
  template:
    metadata:
      labels:
        demo: deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.14-alpine
        ports:
        - name: http
          containerPort: 80
# 发布
$ kubectl apply -f deployment-demo.yaml
deployment.apps/deployment created

# 查看集群中的deployment
$ kubectl get deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
deployment   3/3     3            3           11m
```

当deployment控制器被创建的时候，也会创建对应的replicaset控制器，然后由replicaset控制器去创建pod；
replicaset控制器名称以及pod名称生成规则如下
```
# replicaset控制器名称
deployment控制器名称-pod模板hash

# pod名称
deployment控制器-pod模板hash-随即字符串
```

查看示例
```bash
# 查看deploy控制器
$ kubectl get deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
deployment   3/3     3            3           26m
mysql        0/1     1            0           2d3h

# 查看rs控制器
$ kubectl get rs
NAME                    DESIRED   CURRENT   READY   AGE
deployment-6ff54bbcb8   3         3         3       26m
frontend                2         2         2       7h23m
mysql-7b9cf5df76        1         1         0       2d3h

# 查看pod
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
deployment-6ff54bbcb8-k72m9   1/1     Running   0          26m
deployment-6ff54bbcb8-nt9sx   1/1     Running   0          26m
deployment-6ff54bbcb8-qp7mz   1/1     Running   0          26m
```


## Deployment控制器和Replicaset控制器的区别

Deployment控制器可以完成Replicaset控制器实现不了的功能，比如实现滚动更新以及手动定义更新策略；
从上面的Reolicaset控制器示例中发现，如果我们更改了配置并期望pod容器进行修改，需要创建新的pod才能实现，而旧的pod是不会更新。但Deployment控制器却可以实现发布就立即更新pod的配置。

[摘自Kubernetes官网](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/#%E7%BF%BB%E8%BD%AC-%E5%A4%9A-deployment-%E5%8A%A8%E6%80%81%E6%9B%B4%E6%96%B0)
>Deployment 控制器每次注意到新的 Deployment 时，都会创建一个 ReplicaSet 以启动所需的 Pods。 如果更新了 Deployment，则控制标签匹配 `.spec.selector` 但模板不匹配 `.spec.template` 的 Pods 的现有 ReplicaSet 被缩容。最终，新的 ReplicaSet 缩放为 `.spec.replicas` 个副本， 所有旧 ReplicaSets 缩放为 0 个副本。
>
当 Deployment 正在上线时被更新，Deployment 会针对更新创建一个新的 ReplicaSet 并开始对其扩容，之前正在被扩容的 ReplicaSet 会被翻转，添加到旧 ReplicaSets 列表 并开始缩容。
>
例如，假定你在创建一个 Deployment 以生成 `nginx:1.14.2` 的 5 个副本，但接下来 更新 Deployment 以创建 5 个 `nginx:1.16.1` 的副本，而此时只有 3 个`nginx:1.14.2` 副本已创建。在这种情况下，Deployment 会立即开始杀死 3 个 `nginx:1.14.2` Pods， 并开始创建 `nginx:1.16.1` Pods。它不会等待 `nginx:1.14.2` 的 5 个副本都创建完成 后才开始执行变更动作。

也就是说更新配置之后，deployment就会重新创建一个rsplicaset，逐步的从这个rs创建pod，让旧的rs控制器缩容，新rs控制器替换掉旧rs控制器；deployment是允许同时出现多个rs控制器，但是最终只有一个rs处于活跃状态。

## Deployment控制器-pod版本升级
示例
```bash
# 首先查看当前版本的容器版本，nginx版本均为 1.14
$ kubectl describe deploy deployment | grep Image
    Image:        nginx:1.14-alpine
$ kubectl describe rs deployment-6ff54bbcb8 | grep Image
    Image:        nginx:1.14-alpine
$ kubectl describe pod deployment-6ff54bbcb8-k72m9 | grep Image
    Image:          nginx:1.14-alpine
    Image ID:       docker-pullable://nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7

# 修改 deployment-demo.yaml文件，版本升级为 1.16
$ vim deployment-demo.yaml

# 发布应用
$ kubectl apply -f deployment-demo.yaml

# 查看deployment的版本，版本已经升级上来了
$ kubectl describe deploy deployment | grep Image
    Image:        nginx:1.16-alpine

# 再去查看对应的replicaset版本，却发现版本并没有升级上来
$ kubectl describe rs deployment-6ff54bbcb8 | grep Image
    Image:        nginx:1.14-alpine

# 查看rs的情况，却发现现在已经有2个rs控制器了
$ kubectl get rs
NAME                    DESIRED   CURRENT   READY   AGE
deployment-6b4966fc98   3         3         3       101s
deployment-6ff54bbcb8   0         0         0       36m

# 再次查看pod的情况，发现这些pod的名称和以前的不一样
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
deployment-6b4966fc98-bg9lz   1/1     Running   0          78s
deployment-6b4966fc98-cdtjs   1/1     Running   0          2m1s
deployment-6b4966fc98-grt2f   1/1     Running   0          79s

# 查看具体某一个pod的版本，版本已经升级
$ kubectl describe pod deployment-6b4966fc98-bg9lz | grep Image
    Image:          nginx:1.16-alpine
    Image ID:       docker-pullable://nginx@sha256:5057451e461dda671da5e951019ddbff9d96a751fc7d548053523ca1f848c1ad
	
# 查看deployment控制器的具体信息，具体看 "Event" 字段
yunzhou-test4@yunzhou-test4:~/kubernetes/deployment$ kubectl describe 
// 不重要的信息省略不看，只看Events字段
// 可以看出k8s在逐步的把旧rs控制器的pod删除掉，然后在新rs控制器下重新创建pod来实现版本更新
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  39m    deployment-controller  Scaled up replica set deployment-6ff54bbcb8 to 3
  Normal  ScalingReplicaSet  4m29s  deployment-controller  Scaled up replica set deployment-6b4966fc98 to 1
  Normal  ScalingReplicaSet  3m47s  deployment-controller  Scaled down replica set deployment-6ff54bbcb8 to 2
  Normal  ScalingReplicaSet  3m47s  deployment-controller  Scaled up replica set deployment-6b4966fc98 to 2
  Normal  ScalingReplicaSet  3m46s  deployment-controller  Scaled down replica set deployment-6ff54bbcb8 to 1
  Normal  ScalingReplicaSet  3m46s  deployment-controller  Scaled up replica set deployment-6b4966fc98 to 3
  Normal  ScalingReplicaSet  3m45s  deployment-controller  Scaled down replica set deployment-6ff54bbcb8 to 0
```
从上面的示例中可以看出deployment发现配置变动了，那么就会重新创建一个rs控制器，由新rs控制器创建pod，逐渐的替换掉旧rs控制器下的pod，实现版本更新；这种更新策略被称为`滚动更新`；

那k8s除了 `滚动更新` 还有哪一种更新策略？
[kubernetes官方关于Deployment的更新策略](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/#strategy)

## Deployment的更新策略
`.spec.strategy` 策略指定用于用新 Pods 替换旧 Pods 的策略。 `.spec.strategy.type` 可以是 “Recreate” 或 “RollingUpdate”。“RollingUpdate” 是默认值。

如果 `.spec.strategy.type==Recreate`，在创建新 Pods 之前，所有现有的 Pods 会被杀死。

Deployment 会在 `.spec.strategy.type==RollingUpdate`时，采取 滚动更新的方式更新 Pods。你可以指定 `maxUnavailable` 和 `maxSurge` 来控制滚动更新 过程。

`.spec.strategy.rollingUpdate.maxUnavailable` 是一个可选字段，用来指定 更新过程中不可用的 Pod 的个数上限。该值可以是绝对数字（例如，5），也可以是 所需 Pods 的百分比（例如，10%）。百分比值会转换成绝对数并去除小数部分。 如果 `.spec.strategy.rollingUpdate.maxSurge` 为 0，则此值不能为 0。 默认值为 25%。

`.spec.strategy.rollingUpdate.maxSurge` 是一个可选字段，用来指定可以创建的超出 期望 Pod 个数的 Pod 数量。此值可以是绝对数（例如，5）或所需 Pods 的百分比（例如，10%）。 如果 `MaxUnavailable` 为 0，则此值不能为 0。百分比值会通过向上取整转换为绝对数。 此字段的默认值为 25%。

```bash
# 取值范围 
数值:两者不能同时为0。 
1. maxUnavailable: [0, 副本数] 
2. maxSurge: [0, 副本数] 
比例:两者不能同时为0。 
1. maxUnavailable: [0%, 100%] 向下取整，比如10个副本，5%的话==0.5个，但计算按照0个； 
2. maxSurge: [0%, 100%] 向上取整，比如10个副本，5%的话==0.5个，但计算按照1个； 

# 建议配置 
1. maxUnavailable == 0 
2. maxSurge == 1 

这是我们生产环境提供给用户的默认配置。即“一上一下，先上后下”最平滑原则：1个新版本pod ready（结合readiness）后，才销毁旧版本pod。此配置适用场景是平滑更新、保证服务平稳，但也有缺点，就是“太慢”了。 

# 总结： maxUnavailable：和期望的副本数比，不可用副本数最大比例（或最大值），这个值越小，越能保证服务稳定，更新越平滑； maxSurge：和期望的副本数比，超过期望副本数最大比例（或最大值），这个值调的越大，副本更新速度越快。
```


# 参考文档
[Replicaset控制器官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/)
[Deployment控制器官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)
[Replicaset控制器和Deployment控制器](https://www.cnblogs.com/qiuhom-1874/p/14149042.html)](https://www.cnblogs.com/qiuhom-1874/p/14149042.html)