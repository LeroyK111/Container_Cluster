# 教程版

教程地址：
https://www.bilibili.com/video/BV1w4411y7Go/?spm_id_from=333.337.search-card.all.click&vd_source=79d9fc8b255740f50c9e13219951b7bc

快速过一遍网上教程，希望跟 docker swarm 不会区别太大。

## 容器管理器发展历史

Apache Mesos： 开源分布式资源管理框架, 早期容器集群管理，现在不流行了。

Docker Swarm：简易容器集群架构，功能少。

Kubernetes：主流的容器，部署管理框架。早期名字 Borg 用go语言
- 轻量级
- 开源
- 弹性伸缩
- 负载均衡: IPVS

![](assets/Pasted%20image%2020240805230153.png)
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

`高可用管理节点最好是>=3`
## Pod 

- 自助式pod
	- 无法重启
- 控制器管理的pod
	- 可以被重启

pod中共享容器卷和网络.

### 基本概念

- 资源副本参数:
![](assets/Pasted%20image%2020240805233849.png)
deployment 可以回滚, 推荐使用.

- HPA
水平扩展pod
![](assets/Pasted%20image%2020240805234710.png)
- statefullset
有状态服务
![](assets/Pasted%20image%2020240805234758.png)
启动顺序很重要, 关乎 procedure

- DaemonSet
![](assets/Pasted%20image%2020240805235117.png)


- cron job
job 批处理任务: 在多个pod中处理脚本
cron job: 高级批处理任务
![](assets/Pasted%20image%2020240805235259.png)

标准用法: 

![](assets/Pasted%20image%2020240806000610.png)
然后就使mysql和web server 有了高可用
### 生命周期


### 控制器





## 控制器类型



## 网络通讯模式

### node之间的pod网络
![](assets/Pasted%20image%2020240806000928.png)
可以直接链接的扁平化网络空间.
- 物理链接
	- 交换机
	- 路由器
	- 这是机房最常使用的方法
 - 使用插件CNI, 构建网络策略.
	 - Calico: 提供BGP和IPIP等虚拟隧道组网方案
	 - Flannel: 提供vxlan方法组建扁平网路
	 - Weave: 中小集群专用


### pod内部容器的网络
![](assets/Pasted%20image%2020240806003244.png)

### Flannel: 提供vxlan方法组建扁平网络

推荐方法
![](assets/Pasted%20image%2020240806003358.png)
![](assets/Pasted%20image%2020240806003507.png)
![](assets/Pasted%20image%2020240806003847.png)
![](assets/Pasted%20image%2020240806004101.png)


## 资源清单


### 安装方式

`拉取工具,镜像,文件等需要科学上网.`









## 服务发现

service原理: svc原理及构建方式

### 存储

服务分类:
- 有状态服务: 有数据库dbms
- 无状态服务: 无数据库 lvs apache


## 调度器

调度器原理, 根据要求, 把pod移动到对应的node服务器



## 集群安全机制

### 原理

### 认知

### 鉴权


### 访问控制


## HELM 包管理

Helm原理

Helm模板


## 运维



### 源码修改

#### 证书更新




### 高可用


