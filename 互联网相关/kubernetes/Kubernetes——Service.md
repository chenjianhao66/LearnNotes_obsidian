2021-12-02
16:14:31
author:陈建浩


--- 
# 为什么要有Service
在kubernetes中，Pod是有生命周期的，如果Pod重启它的IP很有可能会发生变化。如果我们的服务都是将Pod的IP地址写死，Pod挂掉或者重启，和刚才重启的pod相关联的其他服务将会找不到它所关联的Pod。
为了解决这个问题，在kubernetes中定义了service资源对象，**Service 定义了一个服务访问的入口，客户端通过这个入口即可访问服务背后的应用集群实例**，service是一组Pod的逻辑集合，这一组Pod能够被Service访问到，通常是通过`Label Selector`实现的。

# 什么是Service
service是一个固定接入层，客户端可以通过访问service的ip和端口访问到service关联的后端pod，这个service工作依赖于在kubernetes集群之上部署的一个附件，就是kubernetes的dns服务。

**service的名称解析是依赖于dns附件的，因此在部署完k8s之后需要再部署dns附件**，kubernetes要想给客户端提供网络功能，需要依赖第三方的网络插件（flannel，calico等）。每个K8s节点上都有一个组件叫做kube-proxy，kube-proxy这个组件将始终监视着apiserver中有关service资源的变动信息，需要跟master之上的apiserver交互，随时连接到apiserver上获取任何一个与service资源相关的资源变动状态。

**简单来讲，service对象就是工作在节点上的一组iptables或ipvs规则，用于将到达service对象ip地址的流量调度转发至相应endpoint对象指向的ip地址和端口之上**；工作于每个节点的kube-proxy组件通过apiserver持续监控着各service及其关联的pod对象，并将其创建或变动实时反映至当前工作节点上相应的iptables或ipvs规则；

其实service和pod或其他资源的关联，本质上不是直接关联，它依靠一个中间组件endpoint；endpoint主要作用就是引用后端pod或其他资源（比如k8s外部的服务也可以被endpoint引用）；所谓endpoint就是ip地址+端口；


![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/1624.png)

# Service的类型
在k8s上service的类型有4种，第一种是clusterIP，我们在创建service资源时，如果不指定其type类型，默认就是clusterip；第二种是NodePort类型，第三种是LoadBalancer，第四种是ExternalName；不同类型的service，其功能和作用也有所不同；

## clusterIP
这种类型service不能被集群外部客户端所访问，仅能在集群节点上访问，这也是默认的ServiceType。

## NodePort

## LoadBalancer

## ExternalName