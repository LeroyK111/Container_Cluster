# kubernetes 学习部署笔记

**这里我们以linux平台为主**
官网: https://kubernetes.io/

![](../readme.assets/Pasted%20image%2020240804232958.png)
node: 节点服务器
pod: 调度单位, node节点服务器上的一个虚拟环境.
container: 调度单位中的一个容器

**实现了四层抽象.**
## 自学版

[我们快速过了一遍教程版的基础上，我们根据官方文档再次学习一遍](obsidian://open?vault=cloud&file=K8S%2FTutorialEdition)









- `kubeadm`：用来初始化集群的指令
- `kubelet`：在集群中的每个节点上用来启动pod和容器
- `kubectl`：用来与集群通许的命令行工具









## k3s 轻量级k8s集群框架