# kubernetes 学习部署笔记

**这里我们以linux平台为主**
官网: https://kubernetes.io/

![](assets/Pasted%20image%2020240804232958.png)
node: 节点服务器
pod: 调度单位, node节点服务器上的一个虚拟环境.
container: 调度单位中的一个容器

**实现了四层抽象.**
## 自学版

[我们快速过了一遍教程版的基础上，我们根据官方文档再次学习一遍](obsidian://open?vault=cloud&file=K8S%2FTutorialEdition)

- schedulet: 调度器, 为node分配任务
- api server 集群网关,.负载均衡
- etcd 键值数据库
	- v2 内存
	- v3 磁盘存储
	- 恢复还原
- replication controller 控制器
- node 服务器节点
	- kind | minikube 简易集群工具, 推荐使用 minikube
	- kubeadm 高级集群管理工具. 这个是生产级别的工具
	- kubectl 命令行工具, 必备
	- kubelet 配置集群管理工具时, 会自动为node安装
	- kube proxy 配置集群管理工具时, 会自动为node安装
	- container 容器: CRI 接口
	- dashboard: b/s访问体系
	- ingress controller: 七层代理, 代理网关.
	- federation: 跨集群中心多k8s统一管理功能
	- Prometheus: k8s集群监控
	- ELK: k8s日志统一接入平台










## k3s 轻量级k8s集群框架