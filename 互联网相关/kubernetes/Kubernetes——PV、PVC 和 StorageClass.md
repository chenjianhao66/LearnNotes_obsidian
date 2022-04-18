
#Kubernetes 
# PV、PVC概念

持久卷（PersistentVolume，PV）是集群中的一块存储，可以由管理员事先供应，或者 使用[存储类（Storage Class）](https://kubernetes.io/zh/docs/concepts/storage/storage-classes/)来动态供应。 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样，也是使用 卷插件来实现的，只是它们拥有独立于任何使用 PV 的 Pod 的生命周期。 此 API 对象中记述了存储的实现细节，无论其背后是 NFS、iSCSI 还是特定于云平台的存储系统。

持久卷申请（PersistentVolumeClaim，PVC）表达的是用户对存储的请求。概念上与 Pod 类似。 Pod 会耗用节点资源，而 PVC 申领会耗用 PV 资源。Pod 可以请求特定数量的资源（CPU 和内存）；同样 PVC 申领也可以请求特定的大小和访问模式 ；

用户在创建pod时使用存储卷只需要关心对应名称空间的pvc对象；而对应pv是需要集群管理管理员定义；后端存储是专门的存储管理员负责管理；pv是k8s上的一种标准资源，全称叫做 `PersistentVolume` 翻译成中文就是持久存储卷；它主要作用是把后端存储中的某个逻辑单元，映射为k8s上的pv资源；pv是集群级别的资源；任意名称空间都可以直接关联某一个pv；关联pv的过程我们叫做绑定pv；而对应名称空间关联某一pv需要使用pvc资源来定义；pvc全称PersistentVolumeClaim的缩写，意思就是持久存储卷申请；

在一个名称空间下创建一个pvc就是把对应名称空间同集群上的某一pv做绑定；一旦一个名称空间绑定了一个pv后，对应的pv就会从available状态转变成bond状态，其他名称空间将不能再使用，只有对应pv是available状态才能正常的被其他名称空间关联绑定；

简单讲pvc和pv的关系是一一对应的，一个pv只能对应一个pvc；至于同一名称空间下的多个pod是否能够同时使用一个PVC取决pv是否允许多路读写，对应pv是否支持多路读写取决后端存储系统；不同类型的存储系统，对应访问模式也有所不同。访问模式有三种，单路读写(ReadWriteOnce简称RWO)，多路读写(ReadWriteMany简称RWX)，多路只读(ReadOnlyMany简称ROX)；

PV、PVC、和POD的关系如下图：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-08_16:31.png)

# 示例
创建pv的yaml文件
```bash
$:~/kubernetes/pv$ cat pv-v1.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv-v1
  labels:
    storsystem: nfs-v1
    rel: stable
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes: ["ReadWriteOnce","ReadWriteMany","ReadOnlyMany"]
  persistentVolumeReclaimPolicy: Retain
  mountOptions:
  - hard
  - nfsvers=4.1
  nfs:
    path: /nfs/data
    server: 10.1.3.205
```

## 创建PV资源以及PV字段详解

### kind
在创建pv的配置文件时， `kind` 字段要指定为 `PersistentVolume` 
### spec.capcity
这个字段是对持久性卷的资源和容量的描述；
`spec.capacity.storage`字段用来描述pv的存储用量；
为了序列化，关于存储容量使用 `storage` 字段来声明，值是使用 `i` 来结尾，比如 `1Gi`、`150Mi`。

### spec.volumeMode
`spec.volumeMode` 字段用来描述对应存储系统提供的存储卷类型接口；该字段有以下这些值：
- Filesystem（文件系统）
- Block（块）

>默认是 `Filesystem`

### spec.accessModes
`spec.accessModes`代表着这个卷在挂载pod上的访问模式，该字段有以下值：

`ReadWriteOnce`
卷可以被一个节点以读写方式挂载。 ReadWriteOnce 访问模式也允许运行在同一节点上的多个 Pod 访问卷。

`ReadOnlyMany`
卷可以被多个节点以只读方式挂载。

`ReadWriteMany`
卷可以被多个节点以读写方式挂载。

`ReadWriteOncePod`
卷可以被单个 Pod 以读写方式挂载。 如果你想确保整个集群中只有一个 Pod 可以读取或写入该 PVC， 请使用ReadWriteOncePod 访问模式。这只支持 CSI 卷以及需要 Kubernetes 1.22 以上版本。

>在命令行接口（CLI）中，访问模式也使用以下缩写形式：
>
>-   RWO - ReadWriteOnce
>-   ROX - ReadOnlyMany
>-   RWX - ReadWriteMany
> -   RWOP - ReadWriteOncePod


### spec.persistentVolumeReclaimPolicy
该字段代表着一个PV卷被释放时所出的动作和存储卷回收策略，持久卷回收策略有3种：
`Delete`
表示当pvc删除以后，对应pv也随之删除；

`Recycle`
表示当pvc删除以后，对应pv的数据也随之被删除；

`Retain`
表示当pvc被删除以后，pv原封动，即pv也在，对应数据也在
> 该卷如果是手动创建，那回收策略默认是Recycle；
> 如果是动态创建，回收策略则是Delete
### spec.mountOptions
mountOptions字段用来指定挂载选项，具体请看[kubernetes官网-持久卷挂载选项](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#mount-options)

### spec.nodeAffinity
每个 PV 卷可以通过设置节点亲和性来定义一些约束，进而限制从哪些节点上可以访问此卷。 使用这些卷的 Pod 只会被调度到节点亲和性规则所选择的节点上执行。 要设置节点亲和性，配置 PV 卷 `.spec` 中的 `nodeAffinity`。 [持久卷](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/#PersistentVolumeSpec) API 参考关于该字段的更多细节。

### spec.storageClassName
此持久卷卷所属的StorageClass的名称。空值表示此卷不属于任何StorageClass。

想要查看更详细的文件字段信息，请查看[kubernetesAPI文档—PersistentVolumeSpec](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/#PersistentVolumeSpec)

### PV的状态
每个卷会处于以下阶段（Phase）之一：
`Available（可用）`
卷是一个空闲资源，尚未绑定到任何申领；

`Bound（已绑定）`
该卷已经绑定到某申领；

`Released（已释放）`
所绑定的申领已被删除，但是资源尚未被集群回收；

`Failed（失败）`
卷的自动回收操作失败。

### 查看PV状态
使用命令 `describe` 查看PV的详细状态
```bash
$ kubectl describe pv nfs-pv-v1
Name:            nfs-pv-v1
Labels:          rel=stable
                 storsystem=nfs-v1
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWO,ROX,RWX
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:         
Source:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    10.1.3.205
    Path:      /nfs/data
    ReadOnly:  false
Events:        <none>
```
从上面信息可以看出，该PV的状态为`Available`，回收策略是`Retain`，访问模式是`RWO,ROX,RWX`，卷模式是`Filesystem`，容量为 1G，卷类型是 `nfs`，地址是10.1.3.205，path地址是`/nfs/data`

## 创建PVC资源
```bash
$ cat pvc-v1.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-pv
  namespace: default
  labels:
    storsystem: nfs-v1
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 500Mi
    limits:
      storage: 800Mi
  selector:
    matchLabels:
      storsystem: nfs-v1
      rel: stable  
```
### kind
在创建pv的配置文件时， `kind` 字段要指定为 `PersistentVolumeClaim` 

### spec.accessModes
spec.accessModes字段是用来指定其pvc的访问模式，一般这个模式是被pv的accessModes包含，也就说pvc的访问模式必须是pv的子集，即等于小于pv的访问模式；

### spec.volumeMode
与PV资源的配置保持一致

### spec.resources
resources用来描述对应pvc的存储空间限制，requests用来描述对应pvc最小容量限制，limits用来描述最大容量限制；

### spec.selector
标签选择器，声明可以指定标签选择器以进一步筛选卷集。只有标签与选择器匹配的卷才能绑定到声明。选择器可以由两个字段组成
`matchLabels`
卷必须具有具有此值的标签

`matchExpressions`
通过指定键、值列表以及与键和值相关的运算符而制定的要求列表。有效运算符包括In、NotIn、Exists和DOESNOTEXTIST。
>所有的需求，从匹配标签和匹配表达式，都是和在一起的——它们必须全部满足才能匹配。

应用该pvc配置文件，在查看pv和pvc的状态
```bash
## 获取pv信息
$ kubectl get pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   REASON   AGE
nfs-pv-v1   1Gi        RWO,ROX,RWX    Retain           Bound    default/pvc-pv                           23h

## 获取pvc信息
$ kubectl get pvc
NAME     STATUS    VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-pv   Bound     nfs-pv-v1   1Gi        RWO,ROX,RWX                   28m
```

从以上信息可以看出，在应用PVC配置之后，PV的状态就由 `Available` 转变为 `Bound`，已经被PVC所绑定。



## 创建Pod挂载PVC
创建Pod的配置文件
```bash
$ cat pod-pv-nginx-yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pv
  namespace: default
  labels:
    app:  tomcat
spec:
  containers:
  - name:  nginx
    image: nginx:1.14-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /nfs/data
      name: nfs-pv
  volumes: 
  - name: nfs-pv
    persistentVolumeClaim:
      claimName: pvc-pv
```
pod在使用PVC的时候只需要在 `spec.volumes` 字段中声明该卷的类型是PVC，然后填上PVC的名字即可；
应用该配置，进入容器内的/nfs/data目录中写入数据，看nfs卷中是否有相对应的数据出现；

```bash
## 进入容器中
$ kubectl exec -it pod/nginx-pv -- sh
/ # ls /nfs/data
index.html

## 进入挂载目录并且创建文件
/ # cd /nfs/data
/ # touch test-nfs-pv
/ # ls
index.html   test-nfs-pv

## 退出容器，进入nfs挂载目录，看是否已经有对应的test-nfs-pv文件了
$ cd /nfs/data
:/nfs/data$ ls
index.html  test-nfs-pv

## 挂载成功
```

### 删除PV、PVC和Pod
在pod有挂载PVC，PVC又绑定PV的情况下，正确的删除顺序是
1. 删除pod
2. 删除PVC
3. 删除PV

如果没有按照这个顺序去删除的话，会出现一些情况：

`在没有删除pod的情况下，删除pvc`
删除操作会被阻塞，结束操作后在查看pvc，状态为 **Terminating**

`在没有删除pod的情况下，删除pv`
删除操作会被阻塞，状态为 **Terminating**

`正确删除顺序删除pod、pvc、pv`
删除pod后，对应的pvc状态保持不变；删除对应pvc以后，对应pv的状态就从bound状态转变为Released状态，表示等待回收；我们在资源清单中使用的是Retain回收策略，pv和pvc都是我们人工手动回收；


# StorageClass
上面介绍的PV和PVC模式都是需要先创建好PV，然后定义好PVC和pv进行一对一的Bond，但是如果PVC请求成千上万，那么就需要创建成千上万的PV，对于运维人员来说维护成本很高，Kubernetes提供一种**自动创建PV的机制**，叫`StorageClass`，它的作用就是创建PV的模板。k8s集群管理员通过创建storageclass可以动态生成一个存储卷pv供k8s pvc使用。

每个StorageClass都包含字段`provisioner`，`parameters`和`reclaimPolicy`。

具体来说，StorageClass会定义以下两部分：

-   1、PV的属性 ，比如存储的大小、类型等；
-   2、创建这种PV需要使用到的存储插件，比如Ceph、NFS等

Kubernetes就能够根据用户提交的PVC，找到对应的StorageClass，然后Kubernetes就会调用 StorageClass声明的存储插件，创建出需要的PV。

**provisioner**：供应商，storageclass需要有一个供应者，用来确定我们使用什么样的存储来创建pv

常见的供应商列表请到[官网]([https://kubernetes.io/zh/docs/concepts/storage/storage-classes/](https://kubernetes.io/zh/docs/concepts/storage/storage-classes/))查看，下面使用 **Kuboard** 管理界面来创建存储类

进入到 **Kuboard** 界面中，在左侧一级菜单选择
