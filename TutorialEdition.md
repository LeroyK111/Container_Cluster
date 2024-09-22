# 教程版

教程地址：https://www.bilibili.com/video/BV1MT411x7GH/
官网地址:   https://kubernetes.io/zh-cn/docs/concepts/overview/#why-you-need-kubernetes-and-what-can-it-do
快速过一遍网上教程. 结合官方文档去理解.
看看云服务商的逻辑.

![](assets/cloud容器化.svg)
## 容器管理器发展历史

Apache Mesos： 开源分布式资源管理框架, 早期容器集群管理，现在不流行了。
主从结构工作模式: 主节点分配任务, 从节点的Executor执行. zookeeper给主节点提供服务注册, 服务发现能力.

Docker Swarm：简易容器集群架构。跟docker无缝集成, 适合小集群规模.

Kubernetes：主流的容器，部署管理框架。早期名字 Borg 用go语言
- 轻量级
- 开源
- 弹性伸缩
- **负载均衡**: IPVS

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
pod (容器组) 中共享容器卷和网络.

k8s中最小的可部署单元，一个pod 包含了一个或多个容器 container .

![](assets/Pasted%20image%2020240831125612.png)

两种部署方式:

- 自助式pod
	- 无法重启, 本质就是无控制器参与的pod
- 控制器管理的pod
	- 可以被重启, 有控制器参与

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
pod上层的控制逻辑.
![](assets/Pasted%20image%2020240831132522.png)

#### 无状态服务
![](assets/Pasted%20image%2020240831152102.png)

RC控制器已经淘汰.
RS控制器不常使用.
![](assets/Pasted%20image%2020240831134527.png)
目前主流使用 deployment 控制器
![](assets/Pasted%20image%2020240831134623.png)
滚动升级/回滚
![](assets/Pasted%20image%2020240831133207.png)

#### 有状态服务
![](assets/Pasted%20image%2020240831152035.png)
![](assets/Pasted%20image%2020240831152705.png)

#### 守护进程

![](assets/Pasted%20image%2020240831153404.png)
#### 任务/定时器

一次性任务: 一般都是命令的形式, 且是立即执行的动作.

周期性任务: 使用定时器构建 命令执行的周期, 条件, 延时等动作.

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

![](assets/Pasted%20image%2020240831171727.png)

#### Service资源 与 Ingress资源 的配置

![](assets/Pasted%20image%2020240831194324.png)

#### 东西流量的容器网络

![](assets/Pasted%20image%2020240806003244.png)
![](assets/Pasted%20image%2020240831195036.png)

#### 南北流量的网络
![](assets/Pasted%20image%2020240831195141.png)




### Flannel: 提供vxlan方法组建扁平网络

推荐方法
![](assets/Pasted%20image%2020240806003358.png)
![](assets/Pasted%20image%2020240806003507.png)
![](assets/Pasted%20image%2020240806003847.png)
![](assets/Pasted%20image%2020240806004101.png)


## 资源清单

针对不同的kind, 需要设置不同的配置文件yaml.

| 资源类型                            | 描述                                       |     |
| ------------------------------- | ---------------------------------------- | --- |
| **Pod**                         | Kubernetes中最小的部署单元，包含一个或多个容器             | 1   |
| **ReplicaSet**                  | 确保指定数量的Pod副本在任何时间都处于运行状态                 | 2   |
| **Deployment**                  | 提供声明式更新和扩展Pod的机制，支持版本回滚、扩展等              | 3   |
| **StatefulSet**                 | 管理有状态应用程序的资源，确保Pod是有序和稳定的                | 4   |
| **DaemonSet**                   | 确保每个Node上运行一个Pod实例，适用于节点守护进程             | 5   |
| **Job**                         | 一次性任务，负责确保指定数量的Pod成功结束                   | 6   |
| **CronJob**                     | 定时任务管理，类似于Linux的cron任务                   | 7   |
| **Service**                     | 定义Pod之间的网络访问规则，通常用于暴露应用程序                | 8   |
| **ConfigMap**                   | 存储非机密的配置信息，以键值对的形式存在，并可以被Pod使用           | 9   |
| **Secret**                      | 用于存储敏感信息（如密码、OAuth令牌），数据是以Base64编码的形式存储  | 10  |
| **Ingress**                     | 提供HTTP和HTTPS路由到集群内部服务的规则，通常用于暴露外部访问      | 11  |
| **PersistentVolume (PV)**       | 定义持久化存储资源的具体实现，由管理员创建                    | 12  |
| **PersistentVolumeClaim (PVC)** | 用户用于请求存储资源的声明，可以绑定到PV上                   | 13  |
| **Namespace**                   | 在同一个集群中为多个团队或项目提供隔离的工作空间                 | 14  |
| ServiceAccount                  | 为Pod提供身份验证信息，用于访问API服务器。                 | 15  |
| Role                            | 定义在命名空间内的权限。                             | 16  |
| ClusterRole                     | 定义在集群范围内的权限。                             | 17  |
| RoleBinding                     | 将Role绑定到一个或多个用户、组或ServiceAccount。        | 18  |
| ClusterRoleBinding              | 将ClusterRole绑定到一个或多个用户、组或ServiceAccount。 | 19  |
| NetworkPolicy                   | 定义Pod之间的网络访问规则，用于控制流量。                   | 20  |
|                                 |                                          |     |

### Pod配置文件

```yaml
apiVersion: v1          # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: Pod               # kind: 资源的类型，这里定义的是一个Pod。
metadata:               # metadata: 资源的元数据，包含名称、标签等。
  name: my-pod          # name: 资源的名称，在集群中必须是唯一的。
  namespace: default    # namespace: Pod 所在的命名空间，默认是"default"。
  labels:               # labels: 键值对，用于对资源进行标记和选择。
    app: my-app         # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                   # spec: 资源的规格和配置，定义了Pod的期望状态。
  containers:           # containers: 定义Pod中的容器列表。
  - name: my-container  # name: 容器的名称，必须在Pod中唯一。
    image: nginx:1.14.2 # image: 要运行的容器镜像，这里使用的是nginx版本1.14.2。
    ports:              # ports: 暴露的容器端口列表。
    - containerPort: 80 # containerPort: 容器内部的端口号，用于接收流量。
  restartPolicy: Always # restartPolicy: Pod的重启策略，Always表示容器总是重启。
  nodeSelector:         # nodeSelector: 用于选择将Pod调度到特定节点的标签。
    disktype: ssd       # 这里指定Pod只能调度到带有“disktype: ssd”标签的节点上。
  tolerations:          # tolerations: 定义Pod能容忍的污点，用于更灵活的调度策略。
  - key: "key1"         # key: 污点的键。
    operator: "Equal"   # operator: 操作符，Equal表示键和值必须相等。
    value: "value1"     # value: 污点的值。
    effect: "NoSchedule" # effect: Pod的调度行为，这里表示禁止调度到带有该污点的节点上。
  volumes:              # volumes: 定义Pod使用的存储卷列表。
  - name: my-volume     # name: 存储卷的名称。
    emptyDir: {}        # emptyDir: 使用一个空目录作为存储卷，生命周期与Pod一致。
```
### ReplicaSet配置文件

```yaml
apiVersion: apps/v1        # apiVersion: 资源使用的API版本，表示使用了apps组中的v1版本API。
kind: ReplicaSet           # kind: 资源的类型，这里定义的是一个ReplicaSet。
metadata:                  # metadata: 资源的元数据，包含名称、标签等。
  name: my-replicaset      # name: 资源的名称，在集群中必须是唯一的。
  namespace: default       # namespace: ReplicaSet 所在的命名空间，默认是 "default"。
  labels:                  # labels: 键值对，用于对资源进行标记和选择。
    app: my-app            # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                      # spec: 资源的规格和配置，定义了ReplicaSet的期望状态。
  replicas: 3              # replicas: 副本数量，定义要运行的Pod副本的数量。
  selector:                # selector: 选择器，用于选择与该ReplicaSet相关联的Pod。
    matchLabels:           # matchLabels: 根据标签选择Pod，标签必须与Pod模板中的标签匹配。
      app: my-app          # 这个标签匹配规则确保ReplicaSet只管理带有'app: my-app'标签的Pod。
  template:                # template: Pod模板，用于定义由ReplicaSet创建的Pod的结构和内容。
    metadata:              # metadata: Pod模板的元数据，包含Pod的标签等。
      labels:              # labels: 标签，必须与选择器中的标签匹配，ReplicaSet才能管理这些Pod。
        app: my-app        # 与选择器一致的标签，确保Pod属于这个ReplicaSet。
    spec:                  # spec: Pod的规格，定义Pod中容器的具体配置。
      containers:          # containers: 定义Pod中的容器列表。
      - name: my-container # name: 容器的名称，必须在Pod中唯一。
        image: nginx:1.14.2 # image: 要运行的容器镜像，这里使用的是nginx版本1.14.2。
        ports:             # ports: 暴露的容器端口列表。
        - containerPort: 80 # containerPort: 容器内部的端口号，用于接收流量。

```

### Deployment 配置文件
```yaml
apiVersion: apps/v1          # apiVersion: 资源使用的API版本，表示使用了apps组中的v1版本API。
kind: Deployment             # kind: 资源的类型，这里定义的是一个Deployment，用于管理应用的部署。
metadata:                    # metadata: 资源的元数据，包含名称、标签等。
  name: my-deployment        # name: 资源的名称，在集群中必须是唯一的。
  namespace: default         # namespace: Deployment 所在的命名空间，默认是 "default"。
  labels:                    # labels: 键值对，用于对资源进行标记和选择。
    app: my-app              # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                        # spec: 资源的规格和配置，定义了Deployment的期望状态。
  replicas: 3                # replicas: 副本数量，定义要运行的Pod副本的数量。
  selector:                  # selector: 选择器，用于选择与该Deployment相关联的Pod。
    matchLabels:             # matchLabels: 根据标签选择Pod，标签必须与Pod模板中的标签匹配。
      app: my-app            # 这个标签匹配规则确保Deployment只管理带有'app: my-app'标签的Pod。
  template:                  # template: Pod模板，用于定义由Deployment创建的Pod的结构和内容。
    metadata:                # metadata: Pod模板的元数据，包含Pod的标签等。
      labels:                # labels: 标签，必须与选择器中的标签匹配，Deployment才能管理这些Pod。
        app: my-app          # 与选择器一致的标签，确保Pod属于这个Deployment。
    spec:                    # spec: Pod的规格，定义Pod中容器的具体配置。
      containers:            # containers: 定义Pod中的容器列表。
      - name: my-container   # name: 容器的名称，必须在Pod中唯一。
        image: nginx:1.14.2  # image: 要运行的容器镜像，这里使用的是nginx版本1.14.2。
        ports:               # ports: 暴露的容器端口列表。
        - containerPort: 80  # containerPort: 容器内部的端口号，用于接收流量。
  strategy:                  # strategy: Deployment的更新策略，定义如何处理Pod的更新。
    type: RollingUpdate      # type: 更新策略类型，RollingUpdate表示滚动更新。
    rollingUpdate:           # rollingUpdate: 滚动更新的详细配置。
      maxUnavailable: 1      # maxUnavailable: 更新过程中允许不可用的Pod最大数量，可以是绝对数或百分比。
      maxSurge: 1            # maxSurge: 更新过程中允许超出期望Pod数量的最大Pod数量，可以是绝对数或百分比。
  revisionHistoryLimit: 10   # revisionHistoryLimit: 保留的Deployment历史修订版本的数量，便于回滚。
  progressDeadlineSeconds: 600 # progressDeadlineSeconds: 部署的进度超时时间，以秒为单位。
```


### StatefulSet 配置文件

```yaml
apiVersion: apps/v1            # apiVersion: 资源使用的API版本，表示使用了apps组中的v1版本API。
kind: StatefulSet              # kind: 资源的类型，这里定义的是一个StatefulSet，用于管理有状态应用。
metadata:                      # metadata: 资源的元数据，包含名称、标签等。
  name: my-statefulset         # name: 资源的名称，在集群中必须是唯一的。
  namespace: default           # namespace: StatefulSet 所在的命名空间，默认是 "default"。
  labels:                      # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                          # spec: 资源的规格和配置，定义了StatefulSet的期望状态。
  serviceName: "nginx-service" # serviceName: 与StatefulSet关联的Headless Service的名称，用于Pod网络标识。
  replicas: 3                  # replicas: 副本数量，定义要运行的Pod副本的数量。
  selector:                    # selector: 选择器，用于选择与该StatefulSet相关联的Pod。
    matchLabels:               # matchLabels: 根据标签选择Pod，标签必须与Pod模板中的标签匹配。
      app: my-app              # 这个标签匹配规则确保StatefulSet只管理带有'app: my-app'标签的Pod。
  template:                    # template: Pod模板，用于定义由StatefulSet创建的Pod的结构和内容。
    metadata:                  # metadata: Pod模板的元数据，包含Pod的标签等。
      labels:                  # labels: 标签，必须与选择器中的标签匹配，StatefulSet才能管理这些Pod。
        app: my-app            # 与选择器一致的标签，确保Pod属于这个StatefulSet。
    spec:                      # spec: Pod的规格，定义Pod中容器的具体配置。
      containers:              # containers: 定义Pod中的容器列表。
      - name: nginx-container  # name: 容器的名称，必须在Pod中唯一。
        image: nginx:1.14.2    # image: 要运行的容器镜像，这里使用的是nginx版本1.14.2。
        ports:                 # ports: 暴露的容器端口列表。
        - containerPort: 80    # containerPort: 容器内部的端口号，用于接收流量。
        volumeMounts:          # volumeMounts: 定义容器内的存储卷挂载点。
        - name: my-storage     # name: 与定义的存储卷名称对应。
          mountPath: /usr/share/nginx/html # mountPath: 容器内的挂载路径。
  volumeClaimTemplates:        # volumeClaimTemplates: 定义StatefulSet所用的持久化存储卷模板。
  - metadata:                  # metadata: 存储卷模板的元数据。
      name: my-storage         # name: 存储卷的名称。
    spec:                      # spec: 存储卷的规格和配置。
      accessModes: ["ReadWriteOnce"] # accessModes: 存储卷的访问模式，ReadWriteOnce表示存储卷只能被单个节点读写。
      resources:               # resources: 资源请求，定义了需要的存储大小。
        requests:              # requests: 定义所需资源的请求值。
          storage: 1Gi         # storage: 请求的存储容量大小。
  updateStrategy:              # updateStrategy: StatefulSet的更新策略。
    type: RollingUpdate        # type: 更新策略类型，RollingUpdate表示滚动更新。
  podManagementPolicy: OrderedReady # podManagementPolicy: Pod的管理策略，OrderedReady表示有序部署和终止。

```

### DaemonSet 配置文件

```yaml
apiVersion: apps/v1          # apiVersion: 资源使用的API版本，表示使用了apps组中的v1版本API。
kind: DaemonSet              # kind: 资源的类型，这里定义的是一个DaemonSet，用于在每个节点上运行一个Pod实例。
metadata:                    # metadata: 资源的元数据，包含名称、标签等。
  name: my-daemonset         # name: 资源的名称，在集群中必须是唯一的。
  namespace: default         # namespace: DaemonSet 所在的命名空间，默认是 "default"。
  labels:                    # labels: 键值对，用于对资源进行标记和选择。
    app: my-app              # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                        # spec: 资源的规格和配置，定义了DaemonSet的期望状态。
  selector:                  # selector: 选择器，用于选择与该DaemonSet相关联的Pod。
    matchLabels:             # matchLabels: 根据标签选择Pod，标签必须与Pod模板中的标签匹配。
      app: my-app            # 这个标签匹配规则确保DaemonSet只管理带有'app: my-app'标签的Pod。
  template:                  # template: Pod模板，用于定义由DaemonSet创建的Pod的结构和内容。
    metadata:                # metadata: Pod模板的元数据，包含Pod的标签等。
      labels:                # labels: 标签，必须与选择器中的标签匹配，DaemonSet才能管理这些Pod。
        app: my-app          # 与选择器一致的标签，确保Pod属于这个DaemonSet。
    spec:                    # spec: Pod的规格，定义Pod中容器的具体配置。
      containers:            # containers: 定义Pod中的容器列表。
      - name: my-container   # name: 容器的名称，必须在Pod中唯一。
        image: nginx:1.14.2  # image: 要运行的容器镜像，这里使用的是nginx版本1.14.2。
        ports:               # ports: 暴露的容器端口列表。
        - containerPort: 80  # containerPort: 容器内部的端口号，用于接收流量。
  updateStrategy:            # updateStrategy: DaemonSet的更新策略，定义如何处理Pod的更新。
    type: RollingUpdate      # type: 更新策略类型，RollingUpdate表示滚动更新。
    rollingUpdate:           # rollingUpdate: 滚动更新的详细配置。
      maxUnavailable: 1      # maxUnavailable: 更新过程中允许不可用的Pod最大数量，可以是绝对数或百分比。
  revisionHistoryLimit: 10   # revisionHistoryLimit: 保留的DaemonSet历史修订版本的数量，便于回滚。

```

### Job 配置文件

```yaml
apiVersion: batch/v1           # apiVersion: 资源使用的API版本，表示使用了batch组中的v1版本API。
kind: Job                      # kind: 资源的类型，这里定义的是一个Job，用于运行一次性任务。
metadata:                      # metadata: 资源的元数据，包含名称、标签等。
  name: my-job                 # name: 资源的名称，在集群中必须是唯一的。
  namespace: default           # namespace: Job 所在的命名空间，默认是 "default"。
  labels:                      # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                # 这个标签可以用来选择和过滤与该任务相关的资源。
spec:                          # spec: 资源的规格和配置，定义了Job的期望状态。
  completions: 1               # completions: 完成任务所需成功Pod的数量，默认值是1。
  parallelism: 1               # parallelism: 并行运行的Pod数量，默认值是1。
  backoffLimit: 4              # backoffLimit: Pod失败重试的最大次数。
  template:                    # template: Pod模板，用于定义由Job创建的Pod的结构和内容。
    metadata:                  # metadata: Pod模板的元数据，包含Pod的标签等。
      labels:                  # labels: 标签，必须与选择器中的标签匹配，Job才能管理这些Pod。
        app: my-app            # 与选择器一致的标签，确保Pod属于这个Job。
    spec:                      # spec: Pod的规格，定义Pod中容器的具体配置。
      containers:              # containers: 定义Pod中的容器列表。
      - name: my-container     # name: 容器的名称，必须在Pod中唯一。
        image: busybox         # image: 要运行的容器镜像，这里使用的是busybox镜像。
        args:                  # args: 容器启动时执行的命令和参数。
        - /bin/sh
        - -c
        - "echo Hello, Kubernetes! && sleep 30" # 容器运行的具体命令。
      restartPolicy: Never     # restartPolicy: Pod的重启策略，Job通常设置为Never。
```

### CronJob 配置文件

```yaml
apiVersion: batch/v1           # apiVersion: 资源使用的API版本，表示使用了batch组中的v1版本API。
kind: CronJob                  # kind: 资源的类型，这里定义的是一个CronJob，用于按照预定的时间表周期性地运行Job。
metadata:                      # metadata: 资源的元数据，包含名称、标签等。
  name: my-cronjob             # name: 资源的名称，在集群中必须是唯一的。
  namespace: default           # namespace: CronJob 所在的命名空间，默认是 "default"。
  labels:                      # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                          # spec: 资源的规格和配置，定义了CronJob的期望状态。
  schedule: "*/5 * * * *"      # schedule: Cron表达式，定义任务执行的时间表。这里表示每5分钟运行一次。
  jobTemplate:                 # jobTemplate: 定义要执行的Job模板。
    spec:                      # spec: Job的规格和配置，定义Pod的期望状态。
      completions: 1           # completions: 完成任务所需成功Pod的数量，默认值是1。
      parallelism: 1           # parallelism: 并行运行的Pod数量，默认值是1。
      backoffLimit: 4          # backoffLimit: Pod失败重试的最大次数。
      template:                # template: Pod模板，用于定义由Job创建的Pod的结构和内容。
        metadata:              # metadata: Pod模板的元数据，包含Pod的标签等。
          labels:              # labels: 标签，必须与选择器中的标签匹配，Job才能管理这些Pod。
            app: my-app        # 与选择器一致的标签，确保Pod属于这个Job。
        spec:                  # spec: Pod的规格，定义Pod中容器的具体配置。
          containers:          # containers: 定义Pod中的容器列表。
          - name: my-container # name: 容器的名称，必须在Pod中唯一。
            image: busybox     # image: 要运行的容器镜像，这里使用的是busybox镜像。
            args:              # args: 容器启动时执行的命令和参数。
            - /bin/sh
            - -c
            - "echo Hello, Kubernetes! && sleep 30" # 容器运行的具体命令。
          restartPolicy: OnFailure # restartPolicy: Pod的重启策略，设置为OnFailure表示只在失败时重启。
  successfulJobsHistoryLimit: 3   # successfulJobsHistoryLimit: 要保留的成功Job的历史记录数量。
  failedJobsHistoryLimit: 1       # failedJobsHistoryLimit: 要保留的失败Job的历史记录数量。
  startingDeadlineSeconds: 100    # startingDeadlineSeconds: 若CronJob错过其计划时间的截止时间，以秒为单位。
  concurrencyPolicy: Forbid       # concurrencyPolicy: 并发策略，Forbid表示不允许同时运行多个Job实例。

```
### Service 配置文件

```yaml
apiVersion: v1               # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: Service                # kind: 资源的类型，这里定义的是一个Service，用于将Pod暴露为网络服务。
metadata:                    # metadata: 资源的元数据，包含名称、标签等。
  name: my-service           # name: 资源的名称，在集群中必须是唯一的。
  namespace: default         # namespace: Service 所在的命名空间，默认是 "default"。
  labels:                    # labels: 键值对，用于对资源进行标记和选择。
    app: my-app              # 这个标签可以用来选择和过滤与该服务相关的资源。
spec:                        # spec: 资源的规格和配置，定义了Service的期望状态。
  selector:                  # selector: 用于选择符合条件的Pod。
    app: my-app              # 选择标签为 'app: my-app' 的Pod。
  ports:                     # ports: 定义服务的端口映射。
  - protocol: TCP            # protocol: 端口使用的协议，这里是TCP。
    port: 80                 # port: Service暴露的端口号。
    targetPort: 8080         # targetPort: 容器内部实际接收流量的端口号。
  type: ClusterIP            # type: 服务的类型，默认是ClusterIP，表示只能在集群内部访问。
```
### ConfigMap 配置文件

```yaml
apiVersion: v1               # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: ConfigMap              # kind: 资源的类型，这里定义的是一个ConfigMap，用于存储非机密的配置信息。
metadata:                    # metadata: 资源的元数据，包含名称、标签等。
  name: my-configmap         # name: 资源的名称，在集群中必须是唯一的。
  namespace: default         # namespace: ConfigMap 所在的命名空间，默认是 "default"。
  labels:                    # labels: 键值对，用于对资源进行标记和选择。
    app: my-app              # 这个标签可以用来选择和过滤与该ConfigMap相关的资源。
data:                        # data: 包含键值对形式的配置信息。
  configKey: configValue     # configKey: 一个配置键，configValue是其对应的值。
  anotherConfigKey: anotherConfigValue # anotherConfigKey: 另一个配置键及其值。

```
### Secret 配置文件

```yaml
apiVersion: v1               # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: Secret                 # kind: 资源的类型，这里定义的是一个Secret，用于存储敏感信息，如密码、OAuth令牌等。
metadata:                    # metadata: 资源的元数据，包含名称、标签等。
  name: my-secret            # name: 资源的名称，在集群中必须是唯一的。
  namespace: default         # namespace: Secret 所在的命名空间，默认是 "default"。
  labels:                    # labels: 键值对，用于对资源进行标记和选择。
    app: my-app              # 这个标签可以用来选择和过滤与该Secret相关的资源。
type: Opaque                 # type: Secret的类型，Opaque表示任意数据，可以用于一般用途。
data:                        # data: 包含Base64编码的键值对形式的敏感信息。
  username: YWRtaW4=         # username: "admin" 的 Base64 编码值，用于存储用户名。
  password: cGFzc3dvcmQ=     # password: "password" 的 Base64 编码值，用于存储密码。
```

### Ingress 配置文件

```yaml
apiVersion: networking.k8s.io/v1 # apiVersion: 资源使用的API版本，表示使用了networking.k8s.io组中的v1版本API。
kind: Ingress                   # kind: 资源的类型，这里定义的是一个Ingress，用于管理外部访问到集群内服务的HTTP和HTTPS路由。
metadata:                       # metadata: 资源的元数据，包含名称、标签等。
  name: my-ingress              # name: 资源的名称，在集群中必须是唯一的。
  namespace: default            # namespace: Ingress 所在的命名空间，默认是 "default"。
  labels:                       # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                 # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                           # spec: 资源的规格和配置，定义了Ingress的期望状态。
  rules:                        # rules: 定义Ingress的路由规则。
  - host: myapp.example.com     # host: 指定处理请求的主机名，例如 myapp.example.com。
    http:                       # http: 定义HTTP路由规则。
      paths:                    # paths: 路由的路径列表。
      - path: /                 # path: 指定要匹配的URL路径，这里表示根路径。
        pathType: Prefix        # pathType: 匹配路径的类型，Prefix表示路径前缀匹配。
        backend:                # backend: 定义流量转发的后端服务。
          service:              # service: 指定服务名称和端口，用于将流量路由到该服务。
            name: my-service    # name: 后端服务的名称，即Service的名称。
            port:               # port: 后端服务的端口。
              number: 80        # number: 服务的端口号。
  tls:                          # tls: 定义HTTPS的TLS设置。
  - hosts:                      # hosts: 定义为哪些主机名启用TLS。
    - myapp.example.com         # 需要TLS的主机名。
    secretName: my-tls-secret   # secretName: 包含TLS证书的Secret的名称。
```

### PersistentVolume (PV) 配置文件

```yaml
apiVersion: v1               # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: PersistentVolume       # kind: 资源的类型，这里定义的是一个PersistentVolume，用于存储持久化数据。
metadata:                    # metadata: 资源的元数据，包含名称、标签等。
  name: my-pv                # name: 资源的名称，在集群中必须是唯一的。
  labels:                    # labels: 键值对，用于对资源进行标记和选择。
    type: local-storage      # 标签可以用来选择和过滤与该存储卷相关的资源。
spec:                        # spec: 资源的规格和配置，定义了PersistentVolume的详细属性。
  capacity:                  # capacity: 定义存储卷的存储容量。
    storage: 10Gi            # storage: 存储卷的大小，这里设置为10GiB。
  accessModes:               # accessModes: 定义存储卷的访问模式。
    - ReadWriteOnce          # ReadWriteOnce: 表示该存储卷可以被单个节点读写。
  persistentVolumeReclaimPolicy: Retain # persistentVolumeReclaimPolicy: 定义存储卷的回收策略。
                                      # Retain表示在删除PersistentVolumeClaim (PVC)后保留数据。
  storageClassName: manual   # storageClassName: 存储类名，用于与PVC匹配。
  hostPath:                  # hostPath: 定义一个主机路径，用作PersistentVolume的实际存储位置。
    path: /mnt/data          # path: 主机路径，表示存储卷的数据将存储在该路径下。
```


### PersistentVolumeClaim (PVC) 配置文件

```yaml
apiVersion: v1                   # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: PersistentVolumeClaim      # kind: 资源的类型，这里定义的是一个PersistentVolumeClaim，用于请求持久化存储卷。
metadata:                        # metadata: 资源的元数据，包含名称、标签等。
  name: my-pvc                   # name: 资源的名称，在集群中必须是唯一的。
  namespace: default             # namespace: PVC 所在的命名空间，默认是 "default"。
  labels:                        # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                  # 这个标签可以用来选择和过滤与该应用相关的资源。
spec:                            # spec: 资源的规格和配置，定义了PVC的期望状态。
  accessModes:                   # accessModes: 定义PVC请求的存储卷访问模式。
    - ReadWriteOnce              # ReadWriteOnce: 表示PVC请求的存储卷可以被单个节点以读写模式挂载。
  resources:                     # resources: 定义资源请求，指定所需的存储大小。
    requests:                    # requests: 资源请求的详细信息。
      storage: 5Gi               # storage: 请求的存储容量大小，这里是5GiB。
  storageClassName: manual       # storageClassName: 指定存储类名，用于与PV进行匹配。
  volumeMode: Filesystem         # volumeMode: 指定卷的类型，默认为Filesystem，可以选择Block。
  selector:                      # selector: 定义PVC选择符合条件的PV的标签。
    matchLabels:                 # matchLabels: 通过标签选择特定的PV。
      type: local-storage        # 标签 'type: local-storage' 用于选择PV。
```


### Namespace 配置文件

```yaml
apiVersion: v1                # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: Namespace               # kind: 资源的类型，这里定义的是一个Namespace，用于为Kubernetes中的资源提供隔离的环境。
metadata:                     # metadata: 资源的元数据，包含名称、标签等。
  name: my-namespace          # name: 资源的名称，在集群中必须是唯一的。
  labels:                     # labels: 键值对，用于对资源进行标记和选择。
    environment: development  # 标签 "environment: development" 用于标记命名空间的用途，例如开发环境。

```


### ServiceAccount 配置文件

```yaml
apiVersion: v1                  # apiVersion: 资源使用的API版本，表示使用了v1版本的核心API。
kind: ServiceAccount            # kind: 资源的类型，这里定义的是一个ServiceAccount，用于在Kubernetes集群中为Pod提供身份验证。
metadata:                       # metadata: 资源的元数据，包含名称、命名空间和标签等。
  name: my-service-account      # name: 资源的名称，在集群中必须是唯一的。
  namespace: default            # namespace: ServiceAccount 所在的命名空间，默认是 "default"。
  labels:                       # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                 # 这个标签可以用来选择和过滤与该ServiceAccount相关的资源。
secrets:                        # secrets: 与ServiceAccount相关联的Secret资源，用于存储凭证（自动生成）。
  - name: my-service-account-token-abcde # name: 关联的Secret的名称，存储ServiceAccount的身份验证信息。

```


### Role 配置文件

```yaml
apiVersion: rbac.authorization.k8s.io/v1  # apiVersion: 资源使用的API版本，表示使用了rbac.authorization.k8s.io组中的v1版本API。
kind: Role                               # kind: 资源的类型，这里定义的是一个Role，用于在命名空间内定义权限。
metadata:                                # metadata: 资源的元数据，包含名称、命名空间和标签等。
  name: my-role                          # name: 资源的名称，在集群中必须是唯一的。
  namespace: default                     # namespace: Role 所在的命名空间，默认是 "default"。
  labels:                                # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                          # 这个标签可以用来选择和过滤与该Role相关的资源。
rules:                                   # rules: 定义Role的权限规则列表。
- apiGroups: [""]                        # apiGroups: 指定资源所属的API组，""表示核心API组。
  resources: ["pods"]                    # resources: 适用的资源类型，这里定义为pods。
  verbs: ["get", "watch", "list"]        # verbs: 允许的操作，例如获取(get)、监视(watch)、列出(list)。
- apiGroups: [""]                        # 另一个规则
  resources: ["secrets"]                 # 适用的资源类型，这里定义为secrets。
  verbs: ["get"]                         # 允许的操作，这里是获取(get)。

```


### ClusterRole 配置文件

```yaml
apiVersion: rbac.authorization.k8s.io/v1  # apiVersion: 资源使用的API版本，表示使用了rbac.authorization.k8s.io组中的v1版本API。
kind: ClusterRole                        # kind: 资源的类型，这里定义的是一个ClusterRole，用于集群范围内定义权限。
metadata:                                # metadata: 资源的元数据，包含名称、标签等。
  name: my-clusterrole                   # name: 资源的名称，在集群中必须是唯一的。
  labels:                                # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                          # 这个标签可以用来选择和过滤与该ClusterRole相关的资源。
rules:                                   # rules: 定义ClusterRole的权限规则列表。
- apiGroups: [""]                        # apiGroups: 指定资源所属的API组，""表示核心API组。
  resources: ["pods", "services"]        # resources: 适用的资源类型，这里定义为pods和services。
  verbs: ["get", "list", "watch"]        # verbs: 允许的操作，例如获取(get)、列出(list)、监视(watch)。
- apiGroups: [""]                        # 另一个规则
  resources: ["namespaces"]              # 适用的资源类型，这里定义为namespaces。
  verbs: ["get", "create", "delete"]     # 允许的操作，例如获取(get)、创建(create)、删除(delete)。

```

### RoleBinding 配置文件

```yaml
apiVersion: rbac.authorization.k8s.io/v1  # apiVersion: 资源使用的API版本，表示使用了rbac.authorization.k8s.io组中的v1版本API。
kind: RoleBinding                        # kind: 资源的类型，这里定义的是一个RoleBinding，用于将Role绑定到指定的用户、组或ServiceAccount。
metadata:                                # metadata: 资源的元数据，包含名称、命名空间和标签等。
  name: my-rolebinding                   # name: 资源的名称，在集群中必须是唯一的。
  namespace: default                     # namespace: RoleBinding 所在的命名空间，默认是 "default"。
  labels:                                # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                          # 这个标签可以用来选择和过滤与该RoleBinding相关的资源。
subjects:                                # subjects: 定义角色绑定的主体列表，可以是用户、组或ServiceAccount。
- kind: ServiceAccount                   # kind: 主体的类型，这里是ServiceAccount。
  name: my-service-account               # name: 主体的名称，表示绑定的ServiceAccount名称。
  namespace: default                     # namespace: 主体所在的命名空间，这里与RoleBinding相同。
roleRef:                                 # roleRef: 定义要绑定的Role或ClusterRole。
  apiGroup: rbac.authorization.k8s.io    # apiGroup: 资源所在的API组，这里是rbac.authorization.k8s.io。
  kind: Role                             # kind: 引用的资源类型，这里是Role，也可以是ClusterRole。
  name: my-role                          # name: 引用的Role的名称，表示要绑定的Role名称。
```


### ClusterRoleBinding 配置文件

```yaml
apiVersion: rbac.authorization.k8s.io/v1  # apiVersion: 资源使用的API版本，表示使用了rbac.authorization.k8s.io组中的v1版本API。
kind: ClusterRoleBinding                 # kind: 资源的类型，这里定义的是一个ClusterRoleBinding，用于将ClusterRole绑定到指定的用户、组或ServiceAccount。
metadata:                                # metadata: 资源的元数据，包含名称、标签等。
  name: my-clusterrolebinding            # name: 资源的名称，在集群中必须是唯一的。
  labels:                                # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                          # 这个标签可以用来选择和过滤与该ClusterRoleBinding相关的资源。
subjects:                                # subjects: 定义角色绑定的主体列表，可以是用户、组或ServiceAccount。
- kind: ServiceAccount                   # kind: 主体的类型，这里是ServiceAccount。
  name: my-service-account               # name: 主体的名称，表示绑定的ServiceAccount名称。
  namespace: default                     # namespace: 主体所在的命名空间。
roleRef:                                 # roleRef: 定义要绑定的ClusterRole。
  apiGroup: rbac.authorization.k8s.io    # apiGroup: 资源所在的API组，这里是rbac.authorization.k8s.io。
  kind: ClusterRole                      # kind: 引用的资源类型，这里是ClusterRole。
  name: my-clusterrole                   # name: 引用的ClusterRole的名称，表示要绑定的ClusterRole名称。
```


### NetworkPolicy 配置文件

```yaml
apiVersion: networking.k8s.io/v1  # apiVersion: 资源使用的API版本，表示使用了networking.k8s.io组中的v1版本API。
kind: NetworkPolicy               # kind: 资源的类型，这里定义的是一个NetworkPolicy，用于控制Pod之间的网络流量。
metadata:                         # metadata: 资源的元数据，包含名称、命名空间和标签等。
  name: my-network-policy         # name: 资源的名称，在集群中必须是唯一的。
  namespace: default              # namespace: NetworkPolicy 所在的命名空间，默认是 "default"。
  labels:                         # labels: 键值对，用于对资源进行标记和选择。
    app: my-app                   # 这个标签可以用来选择和过滤与该NetworkPolicy相关的资源。
spec:                             # spec: 资源的规格和配置，定义了NetworkPolicy的详细规则。
  podSelector:                    # podSelector: 用于选择策略适用的Pod。
    matchLabels:                  # matchLabels: 根据标签选择Pod。
      role: db                    # 这里选择标签为 'role: db' 的Pod。
  policyTypes:                    # policyTypes: 定义网络策略的类型，可以是 "Ingress"、"Egress" 或两者。
  - Ingress                       # Ingress: 限制进入Pod的流量。
  - Egress                        # Egress: 限制离开Pod的流量。
  ingress:                        # ingress: 定义允许进入的流量规则列表。
  - from:                         # from: 指定允许进入流量的来源。
    - ipBlock:                    # ipBlock: 基于IP地址块的流量规则。
        cidr: 192.168.1.0/24      # cidr: 允许进入的IP地址范围。
        except:                   # except: 排除特定的IP地址范围。
        - 192.168.1.5/32          # 这个IP地址将被排除在允许范围之外。
    - namespaceSelector:          # namespaceSelector: 根据命名空间标签选择允许进入流量的来源Pod。
        matchLabels:
          project: myproject      # 允许来自标签为 'project: myproject' 的命名空间中的Pod的流量。
    - podSelector:                # podSelector: 根据Pod标签选择允许进入流量的来源Pod。
        matchLabels:
          role: frontend          # 允许来自标签为 'role: frontend' 的Pod的流量。
  egress:                         # egress: 定义允许离开的流量规则列表。
  - to:                           # to: 指定允许离开流量的目标。
    - ipBlock:                    # ipBlock: 基于IP地址块的流量规则。
        cidr: 10.0.0.0/24         # cidr: 允许离开的IP地址范围。
```

## 部署K8S


### 常见工具or插件
以下是一些常用的 Kubernetes 第三方应用和插件，它们在不同的方面增强了 Kubernetes 的功能，包括监控、日志管理、网络、安全性等。

一下工具和插件覆盖了 Kubernetes 集群管理的方方面面，帮助提升集群的可观测性、安全性、扩展性以及自动化能力。每个工具在其领域内都有广泛的应用，并与 Kubernetes 生态系统紧密集成。

| 功能类别       | 第三方应用/插件                | 描述                                                    |
| ---------- | ----------------------- | ----------------------------------------------------- |
| **监控**     | Prometheus              | 一个强大的开源系统监控和警报工具，常与 Grafana 一起使用可视化监控数据。              |
| **日志管理**   | ELK Stack               | Elasticsearch、Logstash 和 Kibana 的组合，用于集中化日志收集、分析和可视化。 |
| **日志管理**   | Fluentd                 | 一个开源的数据收集工具，用于日志数据的聚合、处理和路由。                          |
| **网络管理**   | Calico                  | 一个高性能的网络插件，提供了网络策略管理和网络安全功能。                          |
| **网络管理**   | Flannel                 | 一个简单的网络插件，用于在 Kubernetes 中提供 pod 间的网络连接。              |
| **存储管理**   | Longhorn                | 一个用于 Kubernetes 的分布式块存储系统，提供持久存储卷。                    |
| **存储管理**   | Ceph                    | 一个强大的分布式存储系统，支持块存储、对象存储和文件存储。                         |
| **安全性**    | OPA (Open Policy Agent) | 用于策略驱动的控制访问和决策的工具，可以在 Kubernetes 中实施安全和合规策略。          |
| **CI/CD**  | Jenkins X               | 一个专为 Kubernetes 设计的 CI/CD 工具，自动化应用程序的部署和管理。           |
| **CI/CD**  | Argo CD                 | 一个声明式的 GitOps 持续交付工具，专注于 Kubernetes 上的应用交付。           |
| **服务网格**   | Istio                   | 一个流行的服务网格解决方案，用于管理微服务之间的流量、策略和安全。                     |
| **服务网格**   | Linkerd                 | 一个轻量级的服务网格，提供服务间的安全、监控和流量管理功能。                        |
| **API 网关** | Kong                    | 一个开源的 API 网关和微服务管理层，提供负载均衡、安全性和 API 管理功能。             |
| **API 网关** | Ambassador              | 专为 Kubernetes 设计的 API 网关，支持微服务架构。                     |
| **自动伸缩**   | KEDA                    | Kubernetes Event-driven Autoscaling，基于事件的自动扩展工具。      |
| **备份与恢复**  | Velero                  | 用于 Kubernetes 集群和持久卷的备份与恢复的工具。                        |
| **镜像管理**   | Harbor                  | 用于企业级镜像管理, 为continue提供镜像                              |
|            |                         |                                                       |
|            |                         |                                                       |

### 常用部署方式

![](assets/Pasted%20image%2020240904235623.png)

这里我们使用hyper-v搭建三个node节点服务器.
![](assets/Pasted%20image%2020240906015659.png)

### docker-cri实现
```
K8S CRI容器化规范.  v1.24 < version 还在支持docker. 
v1.25 以后就不再支持了 dockershim 组件了. 需要插件手动实现cri协议
```

这里我们就用 docker(cri) + k8s 进行容器集群化部署.

#### 开始安装基本软件
docker 官方文档: https://docs.docker.com/guides/deployment-orchestration/kube-deploy/
docker-cri 文档:  https://mirantis.github.io/cri-dockerd/usage/install/
kubernetes 文档: https://kubernetes.io/zh-cn/docs/setup/

##### 环境准备
三个 ubuntu24 主机

- 创建加载内核文件
```sh
cat << EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF


# 手动加载模块
root@k8smaster:/home/k8smaster# modprobe overlay
root@k8smaster:/home/k8smaster# modprobe br_netfilter

# 查看已加载模块
root@k8smaster:/home/k8smaster# lsmod |egrep "overlay"
overlay               212992  0
root@k8smaster:/home/k8smaster# lsmod |egrep "br_netfilter"
br_netfilter           32768  0
bridge                421888  1 br_netfilter
```

- 开启ipv4转发, 应用在 all node
```sh
# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system

# 使用以下命令验证 net.ipv4.ip_forward 是否设置为 1：
sysctl net.ipv4.ip_forward
```

- 安装常用工具
```sh
apt install update
apt install upgrade

apt install net-tools openssh-server git wget curl make cmake ipset ipvsadm vim
sudo systemctl enable ssh
sudo systemctl start ssh
```

现在我们可以使用ssh工具链接虚拟机了

- 安装ipset和ipvsadm
主要是从iptables转为nftables, 记得试用一下.
```sh
# 配置ipvsadm
root@k8smaster:/home/k8smaster# cat << EOF | tee /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF

# 创建模块加载脚本
cat <<EOF | tee ipvs.sh
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF

# 执行脚本
sh ipvs.sh

# 查看模块挂载情况
lsmod | grep ip_vs
```

- 关闭防护墙? 方便学习罢了
```
sudo ufw status
sudo ufw enable
sudo ufw disable
```

- 某些情况下要关闭 selinux (ubuntu 不用管)
```sh
sestatus
```

- 关闭 swap 禁止使用 (虚拟内存交换空间, 主要是性能考虑)
```sh
sudo swapoff -a 临时禁用
vi /etc/fstab 永久禁用
free -m 查看状态
```
编辑 /etc/fstab 文件，注释掉或删除包含 /swap.img 的行
![](assets/Pasted%20image%2020240907213044.png)

- 设置hostname 和 ip 的映射关系
```sh
# 三台 nodes 分别设置 别名
hostnamectl set-hostname k8smaster

# 只有 master node 设置 vi /etc/hosts 文件
# 直接用脚本追加
cat >> /etc/hosts << EOF
172.28.128.190 k8smaster
172.28.130.19 k8sslave1
172.28.132.204 k8sslave2
EOF
```
![](assets/Pasted%20image%2020240907215500.png)
配置三个node的静态地址: 
对于hyper-v虚拟机而言, 就不要配置了.
```sh
vi /etc/netplan/50-cloud-init.yaml

# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            dhcp4: no
            addresses: 
              - 172.28.128.190/20
            routes:
              - to: default
                via: 172.28.143.255
            nameservers:
              addresses: [119.29.29.29,8.8.8.8,1.0.0.1]
    version: 2

# 配置立即生效
netplan apply
```


- 配置流量转发
```sh
# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat >> /etc/sysctl.d/k8s.conf  << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system
```

- 检查时间同步配置( 如不同步, 则需要自己配置 )
这里三台node 都是自动同步
![](assets/Pasted%20image%2020240907221555.png)


- 先检查 虚拟机的嵌套虚拟化 功能 是否开启.
```sh
# 非0 则是支持 虚拟化
egrep -c '(vmx|svm)' /proc/cpuinfo

# 一般都是默认开启虚拟化模块 
# modprobe kvm
# modprobe kvm_intel  # Intel processors
# modprobe kvm_amd    # AMD processors
lsmod | grep kvm

# 查看你在那种虚拟机里
root@k8sslave2:/home/k8sslave2# systemd-detect-virt
microsoft
```


##### 安装 docker 和 docker-cri  

Containerd 也是docker的一个变种, 同样推荐. 安装docker时, 也会默认安装它.
![](assets/Pasted%20image%2020240917173502.png)

存在三种安装方式: https://docs.docker.com/engine/install/ubuntu/#installation-methods
- apt 安装:  以下我们采用这种方法安装 docker 
- deb package 安装: 
	- https://docs.docker.com/desktop/install/linux-install/
	- https://docs.docker.com/engine/install/ubuntu/#install-from-a-package
	- `sudo apt-get install ./docker-desktop-<arch>.deb`
	- https://download.docker.com/linux/ 发行版
	- 参考上述文档, 都可以实现 .deb 的下载安装
- shell 脚本安装: https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script

首先我们要卸载冲突版本
```sh
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
###### 采用 apt 安装 docker
- 非特权用户对 docker container 的namespace 会出现问题. 建议使用以下命令
```sh
# 命名空间权限
sudo echo "kernel.apparmor_restrict_unprivileged_userns=0" | sudo tee /etc/sysctl.d/99-docker.conf

# 立即生效
sudo sysctl --system
```

- 设置apt docker 存储库:
```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
install docker
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

configure docker images source
```sh
# 编辑镜像源文件
# nano /etc/docker/daemon.json
# 添加镜像
# {
#    "registry-mirrors": [
#     "https://hub.uuuadc.top",
#     "https://docker.anyhub.us.kg",
#     "https://dockerhub.jobcher.com",
#     "https://dockerhub.icu",
#     "https://docker.ckyl.me",
#     "https://docker.awsl9527.cn"
#   ]
# }


cat >> /etc/docker/daemon.json  << EOF
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "registry-mirrors": [
    "https://hub.uuuadc.top",
    "https://docker.anyhub.us.kg",
    "https://dockerhub.jobcher.com",
    "https://dockerhub.icu",
    "https://docker.ckyl.me",
    "https://docker.awsl9527.cn"
  ]
}
EOF



# 重启docker 服务
sudo systemctl daemon-reload
sudo systemctl restart docker
# 查看镜像源是否配置成功
 docker info | grep -A 10 "Registry Mirrors"

## 默认启用
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

test run container
```sh
sudo docker run hello-world
sudo docker run -it ubuntu bash
```

###### 安装docker-cri
https://mirantis.github.io/cri-dockerd/usage/install/

两种方式安装 cri-docker
- install 二进制文件直接安装 : https://github.com/Mirantis/cri-dockerd/releases 
- 手动安装 Install Manually:  主要是会涉及到很多定制化安装, 我比较烦这个
```sh
install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
install packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable --now cri-docker.socket
```
cri配置文件

![](assets/Pasted%20image%2020240908121125.png)
![](assets/Pasted%20image%2020240908121147.png)

选择二进制文件直接安装
```sh
# 下载 deb 文件
wget -P ~/Downloads https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.15/cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb

# apt 安装
sudo apt install ./Downloads/cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb

# 启用服务
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable cri-docker.socket

sudo systemctl start cri-docker.service
sudo systemctl start cri-docker.socket
#  查看服务
sudo systemctl status cri-docker.service
```

##### 安装 containerd
官网: https://containerd.io/
安装地址: https://github.com/containerd/containerd/releases

```sh
wget https://github.com/containerd/containerd/releases/download/v1.7.22/cri-containerd-1.7.22-linux-amd64.tar.gz 
```

```sh
tar xf cri-containerd-1.7.22-linux-amd64.tar.gz -C /

# 检查是否安装成功
root@k8sslave1:/home/k8sslave1# which containerd
/usr/bin/containerd
root@k8sslave1:/home/k8sslave1# which runc
/usr/bin/runc
```

###### containerd配置文件

```sh
mkdir /etc/containerd
# 生成配置文件
containerd config default > /etc/containerd/config.toml
# 修改配置文件
vi /etc/containerd/config.toml
# 67row
sandbox_image = "registry.k8s.io/pause:3.9"
# 139row
SystemdCgroup = true
```

开机自启动并且立即启动
```sh
systemctl enable --now containerd
```


##### 开始安装 kubectl, kubelet, kubeadm
搭建集群的多种方式, 我们选择生产环境最常用的kubeadm
- kind: 学习环境的 k8s 集群部署 
- [Cluster API](https://cluster-api.sigs.k8s.io/) : 生产级别的 k8s 集群部署 (大规模生产环境)
- [kops](https://kops.sigs.k8s.io/)：生产级别的 k8s 集群部署 (云环境)
- [kubespray](https://kubespray.io/):  使用 Ansible 自动化部署 生产级别 Kubernetes 集群 (云环境)
- minikube: 小规模k8s集群 部署方案.
	- https://kubernetes.io/zh-cn/docs/tutorials/
	- 学习环境都是推荐这个
- kubeadm: 
	- https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/
	- 推荐工具

这里我们使用cri-dockerd 的套接字
![](assets/Pasted%20image%2020240908170404.png)

你需要在每台机器上安装以下的软件包：

- `kubeadm`：用来初始化集群的指令。
    
- `kubelet`：在集群中的每个节点上用来启动 Pod 和容器等。
    
- `kubectl`：用来与集群通信的命令行工具。
https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
每个 Kubernetes 小版本都有一个专用的软件包仓库。 我们必须为apt包管理添加k8s源.

```shell
sudo apt update
# apt-transport-https 可能是一个虚拟包（dummy package）；如果是的话，你可以跳过安装这个包
sudo apt install -y apt-transport-https ca-certificates curl gpg
```

```shell
# 如果 `/etc/apt/keyrings` 目录不存在，则应在 curl 命令之前创建它，请阅读下面的注释。
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```shell
# 此操作会覆盖 /etc/apt/sources.list.d/kubernetes.list 中现存的所有配置。
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```shell
sudo apt update
sudo apt install -y kubelet=1.31.0-1.1 kubeadm=1.31.0-1.1 kubectl=1.31.0-1.1
# 版本锁定
sudo apt-mark hold kubelet kubeadm kubectl
# 解锁 sudo apt-mark unhold kubelet kubeadm kubectl


```

加入配置
```sh
root@k8smaster:/etc/containerd# vi /etc/default/kubelet 
root@k8smaster:/etc/containerd# cat /etc/default/kubelet 
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"


# 设置开机启动
systemctl enable kubelet
```

```sh
# vi /etc/crictl.yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock 
```

###### 配置cgroup驱动
一般情况下默认即可.
https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/
```yaml
# /var/lib/kubelet/config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta4
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
# 一般都是默认systemd 而不是 `cgroupfs`
cgroupDriver: systemd
```

```sh
# /var/lib/kubelet/config.yaml
kubeadm init --config /var/lib/kubelet/config.yaml
```

###### master node 初始化控制器节点
https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
目前我们以单控制节点为例. 

如果要实现高可用, 多控制节点, 后续会补充复杂构建.
```
172.28.128.190 k8smaster
172.28.130.19 k8sslave1
172.28.132.204 k8sslave2
```

```sh
root@k8smaster:/etc/containerd# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"31", GitVersion:"v1.31.0", GitCommit:"9edcffcde5595e8a5b1a35f88c421764e575afce", GitTreeState:"clean", BuildDate:"2024-08-13T07:35:57Z", GoVersion:"go1.22.5", Compiler:"gc", Platform:"linux/amd64"}

# 生成配置文件
root@k8smaster:~# kubeadm config print init-defaults > kubeadm-config.yaml
root@k8smaster:~# ls
kubeadm-config.yaml  snap
```


最简操作:  kubeadm-config.yaml 

```sh
root@k8smaster:~# cat kubeadm-config.yaml 
apiVersion: kubeadm.k8s.io/v1beta4
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  # master 节点绝对地址
  advertiseAddress: 172.28.128.190
  bindPort: 6443
nodeRegistration:
  # cri 接口类型
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  imagePullSerial: true
  # 名字
  name: k8smaster
  taints: null
timeouts:
  controlPlaneComponentHealthCheck: 4m0s
  discovery: 5m0s
  etcdAPICall: 2m0s
  kubeletHealthCheck: 4m0s
  kubernetesAPICall: 1m0s
  tlsBootstrap: 5m0s
  upgradeManifests: 5m0s
---
apiServer: {}
apiVersion: kubeadm.k8s.io/v1beta4
caCertificateValidityPeriod: 87600h0m0s
certificateValidityPeriod: 8760h0m0s
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
encryptionAlgorithm: RSA-2048
etcd:
  local:
    dataDir: /var/lib/etcd
# 替换成阿里
# registry.aliyuncs.com/google_containers
imageRepository: registry.k8s.io
kind: ClusterConfiguration
kubernetesVersion: 1.31.0
networking:
  dnsDomain: cluster.local
  # 服务资源地址
  serviceSubnet: 10.96.0.0/12
  # pod 组网CNI插件地址范围 一般都是Fiannel 每个node都会从这里分配一个子网, node中的每个pod都会分配一个ip
  podSubnet: 10.244.0.0/16
proxy: {}
scheduler: {}
```

k8s 镜像社区
```
# 查看社区镜像
root@k8smaster:~# kubeadm config images list 
registry.k8s.io/kube-apiserver:v1.31.0
registry.k8s.io/kube-controller-manager:v1.31.0
registry.k8s.io/kube-scheduler:v1.31.0
registry.k8s.io/kube-proxy:v1.31.0
registry.k8s.io/coredns/coredns:v1.11.1
registry.k8s.io/pause:3.10
registry.k8s.io/etcd:3.5.15-0


# 拉取社区镜像, 强行指定--cri-socket 为 containerd 不然docker会干涉这个容器基座
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers --cri-socket unix:///var/run/containerd/containerd.sock

# crictl 可以使用了
root@k8smaster:~# crictl images
IMAGE                                                             TAG                 IMAGE ID            SIZE
registry.aliyuncs.com/google_containers/coredns                   v1.11.1             cbb01a7bd410d       18.2MB
registry.aliyuncs.com/google_containers/etcd                      3.5.15-0            2e96e5913fc06       56.9MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.31.0             604f5db92eaa8       28.1MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.31.0             045733566833c       26.2MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.31.0             ad83b2ca7b09e       30.2MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.31.0             1766f54c897f0       20.2MB
registry.aliyuncs.com/google_c


# 记得重新标记镜像名
# 标记 kube-apiserver
sudo ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.31.0 registry.k8s.io/kube-apiserver:v1.31.0

# 标记 kube-controller-manager
sudo ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.31.0 registry.k8s.io/kube-controller-manager:v1.31.0

# 标记 kube-scheduler
sudo ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.31.0 registry.k8s.io/kube-scheduler:v1.31.0

# 标记 kube-proxy
sudo ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/kube-proxy:v1.31.0 registry.k8s.io/kube-proxy:v1.31.0

# 标记 etcd
sudo ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/etcd:3.5.15-0 registry.k8s.io/etcd:3.5.15-0

# 标记 coredns
sudo ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/coredns:v1.11.1 registry.k8s.io/coredns/coredns:v1.11.1


# 标记 pause
sudo ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/pause:3.10 registry.k8s.io/pause:3.10

```

```sh
# 清理之前的初始化
kubeadm reset --cri-socket unix:///var/run/containerd/containerd.sock

# 利用配置文件进行初始化 
kubeadm init --config ./kubeadm-config.yaml --upload-certs --v=9 
```
这里有大坑, 就是因为使用hyper-v的原因.
![](assets/Pasted%20image%2020240921210434.png)
可以看到api服务无法启动, 这就是hyper-v的不支持导致的. 

###### 🕳如何解决坑?

说明 --cri-socket 没有指定好
```sh
root@k8smaster:~# kubeadm config images pull
found multiple CRI endpoints on the host. Please define which one do you wish to use by setting the 'criSocket' field in the kubeadm configuration file: unix:///var/run/containerd/containerd.sock, unix:///var/run/cri-dockerd.sock
To see the stack trace of this error execute with --v=5 or higher

# 重启containerd
systemctl daemon-reload 
systemctl restart containerd
```

还有超时问题, 国内干运维真的扯
```sh
root@k8smaster:~# kubeadm config images pull --cri-socket unix:///var/run/containerd/containerd.sock
failed to pull image "registry.k8s.io/kube-apiserver:v1.31.0": failed to pull image registry.k8s.io/kube-apiserver:v1.31.0: rpc error: code = DeadlineExceeded desc = failed to pull and unpack image "registry.k8s.io/kube-apiserver:v1.31.0": failed to resolve reference "registry.k8s.io/kube-apiserver:v1.31.0": failed to do request: Head "https://europe-west2-docker.pkg.dev/v2/k8s-artifacts-prod/images/kube-apiserver/manifests/v1.31.0": dial tcp 64.233.189.82:443: i/o timeout
To see the stack trace of this error execute with --v=5 or higher

# 直接指定镜像源
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers --cri-socket unix:///var/run/containerd/containerd.sock
```




看到这里就成功了.
![](assets/Pasted%20image%2020240921231629.png)

调用kubectl默认命令
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# root下, 可以只运行这一条命令
export KUBECONFIG=/etc/kubernetes/admin.conf
```

###### 基本指令
```sh
# 查看所有的nodes服务器
kubectl get nodes
```

![](assets/Pasted%20image%2020240921232945.png)


```sh
# 其他node节点, 加入master管理
kubeadm join 192.168.10.140:6443 --token abcdef.0123456xxxxxx --discovery-token-ca-cert-hash sha256:xxxxx.....
```


```sh
# 老方法
kubectl get cs
# 查看组件状态 新方法
kubectl get pods -n kube-system -o wide

NAME                                 READY   STATUS    RESTARTS   AGE
etcd-k8smaster                       1/1     Running   0          10m
kube-apiserver-k8smaster             1/1     Running   0          10m
kube-controller-manager-k8smaster    1/1     Running   0          10m
kube-scheduler-k8smaster             1/1     Running   0          10m
```

![](assets/Pasted%20image%2020240921234517.png)

```sh
# 静态的kube
ls /etc/kubernetes/manifeste/
```

##### CNI插件(pod网络插件)
可以看到有两个节点没有准备好, 则无法调度他们
![](assets/Pasted%20image%2020240921234847.png)
这是由于pod网络插件未部署, 导致的.

各种网络插件: https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

常用的一般是[Flannel](https://github.com/flannel-io/flannel#deploying-flannel-manually) , Calico, Cilium

###### Calico 部署
https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
- yaml部署
- operator部署

安装Tigera Calico操作符和自定义资源定义。
```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/tigera-operator.yaml
```
![](assets/Pasted%20image%2020240922002109.png)

下载calico的配置文件
```sh
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/custom-resources.yaml
# 修改配置文件
vi custom-resources.yaml
# 修改成为pod的网络
cidr: 10.244.0.0/16
```
![](assets/Pasted%20image%2020240922002442.png)
通过创建必要的自定义资源来安装Calico。有关此清单中可用的配置选项的更多信息，
```sh
kubectl create -f custom-resources.yaml
```
![](assets/Pasted%20image%2020240922002705.png)

- 这样一来nodes就可以被调度了
![](assets/Pasted%20image%2020240922002903.png)

##### 域名服务组件 Coredns

![](assets/Pasted%20image%2020240922003409.png)

测试dns服务组件, 能否解析这个域名
![](assets/Pasted%20image%2020240922003533.png)
偶尔出现问题, 可以选择注释nameserver, 添加其他dns服务器 8.8.8.8
![](assets/Pasted%20image%2020240922003612.png)

###### 部署nginx

```sh
vi nginx.yaml
# 通过 `kubectl get deployments` 可以查看集群中的 `Deployment` 资源，它们负责管理和控制对应pod
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxweb
spec:
  selector:
    matchLabels:
      app: nginxweb1
  replicas: 2
  template:
    metadata:
      labels:
        app: nginxweb1
    spec:
      containers:
        - name: nginxwebc
          image: nginx:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
---
# service 资源配置 将pod公开
apiVersion: v1
kind: Service
metadata:
  name: nginxweb-service
spec:
  externalTrafficPolicy: Cluster
  selector:
    app: nginxweb1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort

```
直接部署
```sh
kubectl apply -f nginx.yaml
```
![](assets/Pasted%20image%2020240922004443.png)

然后就可以通过其他 nodes 的ip 访问这个暴露的端口了

### HELM 包管理器

Helm原理

Helm模板





### 常用运维方式

#### k8s 常用命令
http://docs.kubernetes.org.cn/
https://www.kubernetes.org.cn/docs

| **类别**           | **命令**                                                                              | **说明**                                       |
| ---------------- | ----------------------------------------------------------------------------------- | -------------------------------------------- |
| **基础命令**         | `kubectl version`                                                                   | 显示 kubectl 客户端和服务器版本信息                       |
|                  | `kubectl cluster-info`                                                              | 显示集群的集群信息                                    |
|                  | `kubectl api-resources`                                                             | 列出集群中所有可用的 API 资源                            |
|                  | `kubectl get namespaces`                                                            | 列出所有命名空间                                     |
|                  | `kubectl config get-contexts`                                                       | 列出所有可用的 kubectl 上下文                          |
|                  | `kubectl config use-context <context_name>`                                         | 切换到指定的 kubectl 上下文                           |
| **资源管理**         | `kubectl get all --all-namespaces`                                                  | 列出所有命名空间的所有资源                                |
|                  | `kubectl get pods --namespace=<namespace>`                                          | 列出特定命名空间中的所有 Pod                             |
|                  | `kubectl get deployment <deployment_name> -o yaml`                                  | 以 YAML 格式显示特定 Deployment 的详情                 |
|                  | `kubectl get services --namespace=<namespace>`                                      | 列出特定命名空间中的所有服务                               |
|                  | `kubectl get ingress --all-namespaces`                                              | 列出所有命名空间中的所有 Ingress                         |
| **创建资源**         | `kubectl create deployment <deployment_name> --image=<image>`                       | 创建一个新的 Deployment 并指定镜像                      |
|                  | `kubectl create service clusterip <service_name> --tcp=8080:80`                     | 创建一个 ClusterIP 服务                            |
|                  | `kubectl create secret generic <secret_name> --from-literal=<key>=<value>`          | 创建一个通用的 Secret                               |
|                  | `kubectl create configmap <configmap_name> --from-file=<file_path>`                 | 从文件创建一个 ConfigMap                            |
|                  | `kubectl run <pod_name> --image=<image>`                                            | 从指定镜像启动一个 Pod                                |
| **删除资源**         | `kubectl delete pod <pod_name>`                                                     | 删除特定 Pod                                     |
|                  | `kubectl delete service <service_name>`                                             | 删除指定服务                                       |
|                  | `kubectl delete deployment <deployment_name>`                                       | 删除指定 Deployment                              |
|                  | `kubectl delete namespace <namespace_name>`                                         | 删除指定命名空间                                     |
|                  | `kubectl delete all --all --namespace=<namespace>`                                  | 删除指定命名空间下的所有资源                               |
| **描述和查看详情**      | `kubectl describe pod <pod_name>`                                                   | 查看 Pod 的详细信息                                 |
|                  | `kubectl describe service <service_name>`                                           | 查看服务的详细信息                                    |
|                  | `kubectl describe deployment <deployment_name>`                                     | 查看 Deployment 的详细信息                          |
|                  | `kubectl describe node <node_name>`                                                 | 查看节点的详细信息                                    |
|                  | `kubectl describe ingress <ingress_name>`                                           | 查看 Ingress 的详细信息                             |
| **日志和调试**        | `kubectl logs <pod_name>`                                                           | 查看 Pod 的日志                                   |
|                  | `kubectl logs <pod_name> -c <container_name>`                                       | 查看指定容器的日志                                    |
|                  | `kubectl logs --previous <pod_name>`                                                | 查看 Pod 前一个实例的日志                              |
|                  | `kubectl exec <pod_name> -- <command>`                                              | 在 Pod 内运行指定命令                                |
|                  | `kubectl port-forward pod/<pod_name> 8080:80`                                       | 将本地端口转发到 Pod 中的端口                            |
|                  | `kubectl debug node/<node_name> --image=busybox`                                    | 调试节点，使用指定镜像启动调试容器                            |
| **滚动更新和回滚**      | `kubectl set image deployment/<deployment_name> <container_name>=<new_image>`       | 更新 Deployment 中容器的镜像版本                       |
|                  | `kubectl rollout restart deployment/<deployment_name>`                              | 重启 Deployment                                |
|                  | `kubectl rollout status deployment/<deployment_name>`                               | 查看 Deployment 的滚动更新状态                        |
|                  | `kubectl rollout undo deployment/<deployment_name>`                                 | 回滚 Deployment 到之前的版本                         |
| **扩展与缩减**        | `kubectl scale deployment <deployment_name> --replicas=<num>`                       | 手动扩展或缩减 Deployment 的副本数量                     |
|                  | `kubectl autoscale deployment <deployment_name> --min=2 --max=5 --cpu-percent=80`   | 设置 HPA（水平 Pod 自动扩展），根据 CPU 使用情况动态调整 Pod 副本数量 |
| **资源导出与备份**      | `kubectl get <resource> -o yaml > <file.yaml>`                                      | 将资源导出为 YAML 文件                               |
|                  | `kubectl get <resource> -o json > <file.json>`                                      | 将资源导出为 JSON 文件                               |
|                  | `kubectl apply -f <file>`                                                           | 应用指定文件中的资源配置                                 |
| **资源更新**         | `kubectl edit <resource_type> <resource_name>`                                      | 使用默认编辑器编辑指定资源                                |
|                  | `kubectl patch <resource_type> <resource_name> --patch '{"key":"value"}'`           | 使用 JSON 格式补丁更新资源                             |
| **事件查看与监控**      | `kubectl get events --sort-by=.metadata.creationTimestamp`                          | 查看按时间排序的事件                                   |
|                  | `kubectl get events -n <namespace>`                                                 | 查看特定命名空间的事件                                  |
| **Node 节点管理**    | `kubectl drain <node_name>`                                                         | 驱逐节点上的 Pod，用于维护                              |
|                  | `kubectl cordon <node_name>`                                                        | 标记节点为不可调度状态                                  |
|                  | `kubectl uncordon <node_name>`                                                      | 取消节点的不可调度状态                                  |
| **ConfigMap 管理** | `kubectl get configmaps`                                                            | 查看集群中的 ConfigMap                             |
|                  | `kubectl describe configmap <configmap_name>`                                       | 查看特定 ConfigMap 的详细信息                         |
|                  | `kubectl create configmap <configmap_name> --from-literal=<key>=<value>`            | 创建 ConfigMap，从键值对创建                          |
| **Secrets 管理**   | `kubectl get secrets`                                                               | 查看集群中的 Secret                                |
|                  | `kubectl describe secret <secret_name>`                                             | 查看指定 Secret 的详细信息                            |
|                  | `kubectl create secret tls <secret_name> --cert=<path_to_cert> --key=<path_to_key>` | 创建 TLS Secret                                |
| **Network 管理**   | `kubectl get services`                                                              | 查看所有服务                                       |
|                  | `kubectl describe service <service_name>`                                           | 查看指定服务的详细信息                                  |
|                  | `kubectl get ingress`                                                               | 查看所有 Ingress 资源                              |
| **存储管理**         | `kubectl get pvc`                                                                   | 查看所有 PersistentVolumeClaim                   |
|                  | `kubectl describe pvc <pvc_name>`                                                   | 查看指定 PVC 的详细信息                               |
| **Helm 集成**      | `helm install <release_name> <chart>`                                               | 使用 Helm 安装 Chart                             |
|                  | `helm list`                                                                         | 列出所有 Helm 释放                                 |
|                  | `helm upgrade <release_name> <chart>`                                               | 升级指定 Helm 释放                                 |
|                  | `helm uninstall <release_name>`                                                     | 卸载 Helm 释放                                   |
| **资源命名空间管理**     | `kubectl get namespaces`                                                            | 列出所有命名空间                                     |
|                  | `kubectl create namespace <namespace_name>`                                         | 创建命名空间                                       |
|                  | `kubectl delete namespace <namespace_name>`                                         | 删除命名空间                                       |
| **Pod 网络调试**     | `kubectl exec -it <pod_name> -- /bin/sh`                                            | 进入 Pod 的 shell                               |
|                  | `kubectl port-forward svc/<service_name> 8080:80`                                   | 将服务端口转发到本地                                   |
|                  | `kubectl get endpoints`                                                             | 查看服务的端点 IP 地址                                |
| **角色和权限管理**      | `kubectl get roles --namespace=<namespace>`                                         | 列出命名空间内的角色                                   |
|                  | `kubectl describe rolebinding <rolebinding_name>`                                   | 查看指定 RoleBinding 的详细信息                       |
| **用户配置与凭证**      | `kubectl config set-context <context_name>`                                         | 设置当前 kubectl 使用的上下文                          |
|                  | `kubectl config delete-context <context_name>`                                      | 删除指定上下文                                      |

#### 源码修改

#### 证书更新

#### 高可用


