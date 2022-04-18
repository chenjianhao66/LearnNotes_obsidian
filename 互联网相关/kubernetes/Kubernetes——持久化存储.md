
#Kubernetes 
# k8s的持久化存储
在k8s中部署的应用都是以pod容器的形式运行的，假如我们部署MySQL、Redis等数据库，需要对这些数据库产生的数据做备份。因为Pod是有生命周期的，如果pod不挂载数据卷，那pod被删除或重启后这些数据会随之消失，如果想要长久的保留这些数据就要用到pod数据持久化存储。

在k8s上使用存储卷，需要在对应的节点上提供对应存储系统的驱动，对应运行在该节点上的所有pod就可以使用对应的存储系统；
那么问题来了，pod怎么使用对应的存储系统呢？该怎么向其驱动程序传递参数呢？在k8s上一切皆对象，要在k8s上使用存储卷，还需要把对应的驱动抽象成k8s上的资源；在使用时，我们直接初始化对应的资源为对象即可；

为了在k8s上简化使用存储卷的复杂度，k8s内置了一些存储接口，对于不同类型的存储，其使用的接口、传递的参数也有所不同；除此之外在k8s上也支持用户使用自定义存储，通过csi接口来定义，可以去[k8s官网](https://kubernetes.io/zh/docs/concepts/storage/volumes/)支持的存储类型，也可以使用 `explain` 命令查看；

k8s上支持的存储接口还是很多，每一个存储接口都是一种类型；对于这些存储类型我们大致可以分为云存储，分布式存储，网络存储、临时存储，节点本地存储，特殊类型存储、用户自定义存储等等；
- 云存储
	- awsElasticBlockStore
	- azureDisk
	- azureFile
	- gcePersistentDisk
	- vshperVolume
	- cinder
- 分布式存储
	- cephfs
	- glusterfs
	- rbd
- 临时存储
	- emptyDIR
- 本地存储
	- hostPath
	- local划分为本地存储
- 自定义存储
	- csi
- 特殊存储
	- configMap
	- secret
	- downwardAPId


# 持久化存储示例
## hostPath
`hostPath Volume`是指Pod挂载宿主机上的目录或文件。 hostPath Volume使得容器可以使用宿主机的文件系统进行存储，hostpath（宿主机路径）：节点级别的存储卷，在pod被删除，这个存储卷还是存在的，不会被删除，所以**只要同一个pod被调度到同一个节点上来**，在pod被删除重新被调度到这个节点之后，对应的数据依然是存在的。
声明Yaml文件
```yaml
$ cat hostpath.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - image: nginx:1.16-alpine
    name: nginx-hostpath
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /test-tomcat
      name: test-volume    
  - image: tomcat:8.5-jre8-alpine
    name: tomcat-hostpath
    imagePullPolicy: IfNotPresent
    volumeMounts:
	## 存储卷挂载在容器的目录位置
    - mountPath: /test-tomcat
	## 要挂载哪一个卷，这个卷要声明
      name: test-volume
	##声明一个存储卷  
  volumes:
  - name: test-volume
  ## 存储卷的类型
    hostPath:
	## 被调度到工作节点的挂载目录
      path: /data1
	  ## 该目录如果不存在那就创建它
      type: DirectoryOrCreate    
```
从上面的清单可以看出，使用 `spec.volumes` 字段来声明一个卷，`spec.volumes.xxx`用来声明该卷的类型，比如例子中的类型则是hostPath，如果要用emptyDir类型那就写emptyDir；

`类型.path`字段代表着这个pod被调度到的节点的目录被使用为卷的工作目录，那如果所声明的目录不存在呢？那就可以用 `spec.volumes.卷类型.type` 字段来标识；
这个字段有下面的几个值

| 取值                | 行为                                                         |
| ------------------- | ------------------------------------------------------------ |
|                     | 空字符串（默认）用于向后兼容，这意味着在安装 hostPath 卷之前不会执行任何检查。 |
| `DirectoryOrCreate` | 如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 kubelet 相同的组和属主信息。 |
| `Directory`         | 在给定路径上必须存在的目录。                                 |
| `FileOrCreate`      | 如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 kubelet 相同的组和所有权。 |
| `File`              | 在给定路径上必须存在的文件。                                 |
| `Socket`            | 在给定路径上必须存在的 UNIX 套接字。                         |
| `CharDevice`        | 在给定路径上必须存在的字符设备。                             |
| `BlockDevice`       | 在给定路径上必须存在的                                       |

例子中使用 `DirectoryOrCreate` 值，表示如果所声明的目录不存在那就创建它；

声明了存储卷，那么就在容器里面使用这个卷，使用 `spec.containers.volumeMounts` 字段表示挂载存储卷，`spec.containers.volumeMounts.mountPath` 字段代表挂载带容器内部的目录，`spec.containers.volumeMounts.name` 代表着我要挂载哪一个卷，这个卷要在yaml文件里面声明。

```bash
$ kubectl apply -f hostpath.yaml

# 查看该pod的情况
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
deployment-6cb5dc9785-4p6rm   1/1     Running   0          23h
deployment-6cb5dc9785-8v25m   1/1     Running   0          23h
deployment-6cb5dc9785-tdz77   1/1     Running   0          23h
hostpath-demo                 2/2     Running   0          16h


# 进入其中的一个pod容器内看目录是否挂载成功
# 命令格式 exec -it pod名称 容器名称 -- 要使用的命令
$ kubectl exec -it hostpath-demo nginx-hostpath -- sh
Defaulted container "nginx-hostpath" out of: nginx-hostpath, tomcat-hostpath
/ # ls
bin          etc          lib          mnt          proc         run          srv          test-tomcat  usr
dev          home         media        opt          root         sbin         sys          tmp          var

## 可以看出目录已经挂载成功，另外一个 tomcat-hostpath也是一样挂载成功
## 查看这个pod被调度到哪个节点上

$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
deployment-6cb5dc9785-4p6rm   1/1     Running   0          23h   10.244.1.33   slave-node-1   <none>           <none>
deployment-6cb5dc9785-8v25m   1/1     Running   0          23h   10.244.1.31   slave-node-1   <none>           <none>
deployment-6cb5dc9785-tdz77   1/1     Running   0          23h   10.244.1.32   slave-node-1   <none>           <none>
hostpath-demo                 2/2     Running   0          16h   10.244.2.32   slave-node-2   <none>           <none>

## 跳转到slave-node-2节点上，目录已经创建成功
slave-node-2@slave-node-2:/$ ls
bin  boot  cdrom  data1  dev  etc  home  lib  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  swap.img  sys  tmp  usr  var
slave-node-2@slave-node-2:/$ ls data1/
test1
slave-node-2@slave-node-2:/$ 

## 修改test1的内容为 “测试目录文件是否同步修改”
## 在容器内查看文件是否修改
$ kubectl exec -it hostpath-demo nginx-hostpath -- cat test-tomcat/test1
Defaulted container "nginx-hostpath" out of: nginx-hostpath, tomcat-hostpath
测试目录文件是否同步修改

```

从上面的例子可以看出，使用 `hostpath`类型的存储卷是同一个节点共享使用的；
那我把pod删除之后存储卷的数据还在吗？
```bash
$ kubectl delete pod hostpath-demo

## 再去slave-node-2节点上查看目录，发现目录与数据都在
slave-node-2@slave-node-2:/data1$ ls
test1

## 重新创建一个pod，指定这个pod调度到slave-node-2节点上运行，看存储卷是否能在用上
## 要pod调度到指定节点的话，可以使用spec.nodeName字段来指定，值是工作节点的主机名
$ kubectl apply -f hostpath.yaml

$ kubectl exec -it hostpath-demo nginx-hostpath -- sh
$ ls /test-tomcat
test1
```

经过上面的测试，可以得出结论，多个容器共享一个工作节点的挂载目录，即使这个pod删除了，该工作节点的目录也不会删除；重新创建pod之后要是想继续访问该目录，得保证这个pod被调度到同一个节点之上，这里可以使用 `spec.nodeName` 字段来配置该pod被调度到指定节点；
>[kubernetes官网 hostpath 相关资料](https://kubernetes.io/zh/docs/concepts/storage/volumes/#hostpath)


## emptyDir
当 Pod 分派到某个 Node 上时，`emptyDir` 卷会被创建，并且在 Pod 在该节点上运行期间，卷一直存在。 就像其名称表示的那样，卷最初是空的。 尽管 Pod 中的容器挂载 `emptyDir` 卷的路径可能相同也可能不同，这些容器都可以读写 `emptyDir` 卷中相同的文件。 当 Pod 因为某些原因被从节点上删除时，`emptyDir` 卷中的数据也会被永久删除。

`emptyDir` 的一些用途：

-   缓存空间，例如基于磁盘的归并排序。
-   为耗时较长的计算任务提供检查点，以便任务能方便地从崩溃前状态恢复执行。
-   在 Web 服务器容器服务数据时，保存内容管理器容器获取的文件。


示例
```yaml
## 创建文件
$ cat emptyDir.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - image: nginx:1.16-alpine
    name: nginx-empty
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /emptydir-mount
      name: emptydir-volume
  - image: tomcat:8.5-jre8-alpine
    name: tomcat-empty
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /emptydir-mount
      name: emptydir-volume
  volumes:
  - name: emptydir-volume
    emptyDir: {}
```

发布该文件
```bash
$ kubectl apply -f emptyDir.yaml

## 查看pod
$ kubectl get pods 
NAME                          READY   STATUS    RESTARTS   AGE
deployment-6cb5dc9785-4p6rm   1/1     Running   0          24h
deployment-6cb5dc9785-8v25m   1/1     Running   0          24h
deployment-6cb5dc9785-tdz77   1/1     Running   0          24h
emptydir-demo                 2/2     Running   0          13m
hostpath-demo2                2/2     Running   0          60m

## 进入emptydir-demo的nginx-empty容器内部，查看是否存在挂载的目录
$ kubectl exec -it emptydir-demo nginx-empty -- sh
Defaulted container "nginx-empty" out of: nginx-empty, tomcat-empty
/ # ls -l | grep em
drwxrwxrwx    2 root     root          4096 Dec  8 02:46 emptydir-mount

## 有这个目录，创建文件
/ # cd emptydir-mount/
/emptydir-mount # 
/emptydir-mount # echo nginx-empty >> text
/emptydir-mount # cat text
nginx-empty

## 进入另外一个容器 tomcat-empty查看内容是否更新
$ kubectl exec -it emptydir-demo tomcat-empty -- ls /emptydir-mount
Defaulted container "nginx-empty" out of: nginx-empty, tomcat-empty
text
$ kubectl exec -it emptydir-demo tomcat-empty -- cat /emptydir-mount/text
Defaulted container "nginx-empty" out of: nginx-empty, tomcat-empty
nginx-empty
```
从上面的测试可以看出，emptyDir也是同一个pod内共享目录，但是删除了pod这个目录也会被删除，即使同一个pod调度到同一个节点上，数据也会丢失。

>[kubernetes官网 emptyDir 相关资料](https://kubernetes.io/zh/docs/concepts/storage/volumes/#emptydir)


## nfs存储
`nfs` 卷能将 NFS (网络文件系统) 挂载到你的 Pod 中。 不像 `emptyDir` 那样会在删除 Pod 的同时也会被删除，`nfs` 卷的内容在删除 Pod 时会被保存，卷只是被卸载。 这意味着 `nfs` 卷可以被预先填充数据，并且这些数据可以在 Pod 之间共享。

在pod挂载存储前，我们得在自己机器上搭建nfs服务，下面演示ubuntu服务器搭建nfs服务器的步骤：
```bash
## 安装nfs
$ sudo apt-get install nfs-kernel-server


## 创建共享目录
$ sudo mkdir -p /nfs/data
$ sudo chmod -R 777 /nfs

## 修改/etc/exports文件，增加nfs的共享目录位置、用户访问以及源ip
$ sudo vim /etc/exports

## 共享 /nfs/data 目录，
*代表所有网段都可以访问，rw表示读写权限，no_root_squash表示用户具有根目录的完全管理访问权限
/nfs/data *(rw,no_root_squash)

## 重启nfs服务
$ sudo /etc/init.d/nfs-kernel-server restart


## 跳转到客户端测试nfs服务是否启动成功
## 客户端需要安装 nfs的客户端工具包
$ sudo apt-get install nfs-common

## 创建挂载nfs目录的本机目录
$ sudo mkdir /nfs-client
$ sudo chmod 777 /nfs-client

## 挂载，IP地址自行替换
$ sudo mount -t nfs 10.1.3.205:/nfs/data /nfs/client -o nolock

## 创建文件，回到服务端查看目录是否同步
$ echo text >> tt

## 回到服务端，已经同步成功，至此nfs存储系统已经搭建完成！
```

编写配置文件
```yaml
$ cat nfs.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-demo
spec:
  containers:
  - name: nfs-demo
    image: nginx
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
      protocol: TCP
    volumeMounts:
    - name: nfs-volumes
      mountPath: /usr/share/nginx/html
  volumes:
  - name: nfs-volumes
    nfs:
      path: /nfs/data
      server: 10.1.3.205    

```

发布该配置
```bash
## 发布配置
$ kubectl apply -f nfs.yaml

## 获取pod，可以看出已经能正常运行
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
deployment-6cb5dc9785-4p6rm   1/1     Running   0          28h
deployment-6cb5dc9785-8v25m   1/1     Running   0          28h
deployment-6cb5dc9785-tdz77   1/1     Running   0          28h
nfs-demo                      1/1     Running   0          3s

## 在上面的配置文件中可以看出，挂载的目录是nginx的静态资源，所以只要替换了index.html，那么就可以实现自定义主页

## 在服务端创建 index.html 文件
$ echo hello,this is nfs storage nginx index >> /nfs/data/index.html

## 进入pod中容器的sh命令界面，查看是否已经同步该文件
## 已经同步！
$ kubectl exec -it nfs-demo -- cat /usr/share/nginx/html/index.html
hello,this is nfs storage nginx index


## 在服务端通过curl工具访问该pod容器，验证返回的值
# 获取pod的ip地址
$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
nfs-demo                      1/1     Running   0          97s   10.244.2.37   slave-node-2   <none>           <none>

## 通过 10.244.2.37 这个集群IP访问
$ curl 10.244.2.37
hello,this is nfs storage nginx index

## 访问成功，代表这nfs存储也挂载成功
```

nfs支持多个客户端挂载，可以创建多个pod，挂载同一个nfs服务器共享出来的目录；nfs此时是单点，一旦nfs服务器宕机挂掉，对应pod运行时产生的数据将全部丢失；所以对应外部存储系统，我们应该选择一个对数据有冗余，且k8s集群支持的类型的存储系统，比如cephfs，glusterfs等等；