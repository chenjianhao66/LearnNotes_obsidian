[[2021-11-24]]
08:55:28
author:陈建浩
#踩坑记录 #Kubernetes 

--- 

# 问题背景
在安装完 `kubelet`、`kubeadm`、`kubectl` 这三剑客之后，以当前节点作为 `master` 节点进行集群初始化。

# 问题描述
集群一直无法被初始化

# 问题分析
- 配置文件有问题
# 解决方法
在网上冲浪时发现，`kubelet` 一直重启是因为 `docker`作为 `kubernetes`的容器运行时，需要`docker`的 `cgroup`属性要与 `kubernetes` 保持一致；
该属性在 `/etc/docker/daemon.json` 文件中进行追加设置即可
```bash
# 修改该文件
vim /etc/docker/daemon.json

# 在文件末尾追加内容
{
	"exec-opts": ["native.cgroupdriver=systemd"]
}

# 重启kubelet服务
systemctl daemon-reload
systemctl restart docker
```
至此问题解决。