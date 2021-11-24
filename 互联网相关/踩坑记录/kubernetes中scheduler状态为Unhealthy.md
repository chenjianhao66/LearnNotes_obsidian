[[2021-11-22]]
17:26:58
author:陈建浩
#踩坑记录  #Kubernetes

--- 

# 问题背景
通过 `kubeadm` 工具部署完 `kubernetes` 集群。
# 问题描述
`kubernetes`中 `scheduler` 状态为 `Unhealthy`
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/1756619-20201221105925745-342494989.png)

# 问题分析
1. 初步判断为端口占用

# 解决方法
1. 查看端口是否被占用

```bash
ss -ntlpu |grep 10251 
ss -ntlpu |grep 10252
```
结果表明，这2个端口没有被占用
2. 查看这2个服务是否存在

```bash
kubectl get pod -A |grep kube-system
```
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-22_17-31.png)

状态也均为 `Running`

3. 检查组件yaml配置的端口

```bash
cd /etc/kubernetes/manifests
sudo vim kube-scheduler.yaml

## 将yaml文件中的 spec.command.--port=0 这一设置给删除掉
## 删除之后再重启 kubelet
systemctl daemon-reload
systemctl restart kubelet
```

最后检查状态
```bash
kubectl get cs
```
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-11-22_17-38.png)

问题解决