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

# 使用kubernetes
systemctl daemon-reload

## 默认启用
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

test run container
```sh
sudo docker run hello-world
sudo docker run -it ubuntu bash
```

```sh
# 升级docker 很容易
apt upgrade docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
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

`一般情况下 /etc/systemd/system/cri-docker.service 不用修改`

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

###### 老版方法

- 问题一: cgroup driver 不是systemd
```sh
docker info|grep Driver
```

![](assets/Pasted%20image%2020240922210825.png)
![](assets/Pasted%20image%2020240922210844.png)
修改docker的管理模式
```
"exec-opts": ["native.cgroupdriver=systemd"]
```
![](assets/Pasted%20image%2020240922210907.png)
![](assets/Pasted%20image%2020240922211436.png)

- 问题二: init失败, 导致cni认证失败
![](assets/Pasted%20image%2020240922211459.png)
```
# reset后,重新init
kubeadm reset .....
kubeadm init .....
```
![](assets/Pasted%20image%2020240922211856.png)

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
推荐使用配置文件初始化init, 如下:

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
  # 污点
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
# etcd地址
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




看到这里就成功了. 记得截图记录一下token
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
kubectl get no
```

![](assets/Pasted%20image%2020240921232945.png)


```sh
# 查看token是否过期
kubeadm token list
# 生成token
kubeadm token create

# 其他node节点, 加入master管理
kubeadm join 192.168.10.140:6443 --token abcdef.0123456xxxxxx --discovery-token-ca-cert-hash sha256:xxxxx.....
```

```sh
# 获取完整的token 
```
![](assets/Pasted%20image%2020240922212607.png)


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
```sh
sed -i 's#docker.io/##g' calico.yaml
```
![](assets/Pasted%20image%2020240922214758.png)
![](assets/Pasted%20image%2020240922215234.png)
```sh
# 查看详情.
kubectl describe po xxxxx
```


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

```sh
# 简写
kubectl get po
```


- 这样一来nodes就可以被调度了
![](assets/Pasted%20image%2020240922002903.png)


##### 域名服务组件 Coredns

![](assets/Pasted%20image%2020240922003409.png)

测试dns服务组件, 能否解析这个域名
![](assets/Pasted%20image%2020240922003533.png)
偶尔出现问题, 可以选择注释nameserver, 添加其他dns服务器 8.8.8.8
![](assets/Pasted%20image%2020240922003612.png)




###### 部署nginx

- yaml文件部署

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
- 指令部署

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --type=NodePort

kubectl get pod,svc 

# 所有的node与虚拟网段,都可以访问这个端口
curl 192.168.113.120:30903
```
![](assets/Pasted%20image%2020240922220315.png)


#### kubectl 详解

记得先查看api版本, 可能三种工具的版本完全不一样. 导致api接口失效or异常.
-  Alpha: v1alpha1  不推荐使用
- Beta: v2beta3 可以一试
- Stable: vX 生产环境必须使用

##### 在任意节点使用kubectl.
 - copy master节点上的admin.conf 文件到 其他服务器上
 ![](assets/Pasted%20image%2020240922221018.png)
![](assets/Pasted%20image%2020240922221129.png)

##### apply

`kubectl apply` 是Kubernetes 中用于将资源文件应用到集群的命令。管理kubernetes资源的核心命令之一, 可以创建,更新,配置资源. 常用于将yaml, json格式的资源定义文件部署到k8s中.

```sh
kubectl apply -f <file-path>
kubectl apply -f deployment.yaml -f service.yaml
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
kubectl apply -k ./kustomize/

# 测试文件是否正确
kubectl apply -f deployment.yaml --dry-run=client
# 记录历史变动
kubectl apply -f deployment.yaml --record
# 递归文件夹中的配置
kubectl apply -R -f ./my-configs/

# 强制应用
kubectl apply -f deployment.yaml --force
# 获取应用状态
kubectl get <resource-type> <resource-name>

```
##### autoscale
本质就是HPA自动扩缩容.
在 Kubernetes 中，`kubectl autoscale` 命令用于为现有的部署（Deployment）或其他控制器（如 ReplicaSet、StatefulSet）创建水平 Pod 自动扩缩容（Horizontal Pod Autoscaler，HPA）。HPA 会根据指定的指标（例如 CPU 利用率、内存利用率等）自动调整 Pod 的副本数量，以应对负载变化。

```bash
kubectl autoscale <resource> <name> --min=<min_replicas> --max=<max_replicas> --cpu-percent=<target_cpu_utilization>
```

- **`<resource>`**：要扩缩容的资源类型，例如 `deployment` 或 `replicaset`。
- **`<name>`**：要扩缩容的资源的名称。
- **`--min=<min_replicas>`**：指定最少的副本数量。
- **`--max=<max_replicas>`**：指定最大的副本数量。
- **`--cpu-percent=<target_cpu_utilization>`**：指定 CPU 的目标利用率（百分比）。


```bash
kubectl autoscale deployment nginx-deployment --min=2 --max=5 --cpu-percent=50
```

这个命令会为 `nginx-deployment` 部署创建一个水平 Pod 自动扩缩容器，设置 Pod 副本数的最小值为 2，最大值为 5，当 CPU 利用率超过 50% 时会增加 Pod 的副本数量。

1. **`--min`**：
   - 最少的副本数量，即自动扩缩容器会保证至少有这么多的 Pod 运行。

2. **`--max`**：
   - 最多的副本数量，自动扩缩容器可以增加副本数量，但不会超过该数量。

3. **`--cpu-percent`**：
   - 目标 CPU 利用率百分比。当平均 CPU 利用率高于设定的阈值时，自动扩缩容器会增加 Pod 副本数量。当低于阈值时，自动缩减 Pod 数量。


- **处理高并发请求**：当负载波动较大时，例如有大量用户请求，HPA 可以自动增加 Pod 副本数量以提高服务的处理能力。
- **降低成本**：在负载较低时，减少 Pod 数量，从而降低资源的占用，减少运行成本。

创建 HPA 后，可以使用以下命令查看其状态：

```bash
kubectl get hpa
```

该命令会显示当前集群中的所有 HPA 及其监控的指标和副本数量等信息。

除了使用 `kubectl autoscale` 命令，还可以使用 YAML 文件来创建 HPA。例如，为 `nginx-deployment` 创建一个 HPA：

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 50
```

使用以下命令应用该配置：

```bash
kubectl apply -f hpa.yaml
```

在 Kubernetes 1.18 及以上版本中，可以使用 `autoscaling/v2` 或 `autoscaling/v2beta2` 版本的 HPA，允许基于更丰富的指标（如内存、定制指标）来扩缩容。例如：

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi
```

在这个例子中，HPA 将根据 CPU 利用率和内存消耗来调整 Pod 的数量。

如果不再需要自动扩缩容，可以使用以下命令删除 HPA：

```bash
kubectl delete hpa nginx-hpa
```


1. **创建 HPA**：
   ```bash
   kubectl autoscale deployment <deployment-name> --min=<min> --max=<max> --cpu-percent=<percent>
   ```

2. **查看 HPA**：
   ```bash
   kubectl get hpa
   ```

3. **查看详细信息**：
   ```bash
   kubectl describe hpa <hpa-name>
   ```

4. **删除 HPA**：
   ```bash
   kubectl delete hpa <hpa-name>
   ```


##### edit

编辑现有资源配置. 通过使用kubectl edit 你可以直接修改运行中的kubernetes资源, 而无需应用新的yaml文件.

```sh
# 在线编辑资源
kubectl edit <resource-type> <resource-name>
kubectl edit deployment nginx-deployment
# 指定命名空间
kubectl edit deployment nginx-deployment -n my-namespace
# 现有资源转配置文件
kubectl edit deployment nginx-deployment -o yaml
```
与patch的区别

- **`kubectl edit`**：使用文本编辑器直接编辑整个资源对象. 生产环境不推荐
- **`kubectl patch`**：使用json或yaml补丁来修改特定字段, 适合用于脚本和自动化操作.

```sh
# 修改副本数
kubectl patch deployment nginx-deployment -p '{"spec": {"replicas": 5}}'
```

##### label
kubectl label 命令用于在 Kubernetes 中为资源（如 Pod、Node、Service、Deployment 等）添加、更新或删除标签。标签（label）是键值对，用于标识和选择 Kubernetes 资源。标签在 Kubernetes 中的应用非常广泛，主要用于管理、筛选和选择资源，便于自动化运维和编排。

```sh
kubectl label <resource-type> <resource-name> <label-key>=<label-value> [--overwrite]

```

- resource-type: 资源类型node pod service deployment
- resource-name: 要标记的资源名称
- label-key: 要更新的键值对 k=v 可以多个标签
- --overwrite 强制覆盖重名标签

```sh
# 直接打标签
kubectl label deployment nginx-deployment environment=production
# 为app=nginx的所有pod打标签
kubectl label pods -l app=nginx environment=production
# 强制覆盖
kubectl label deployment nginx-deployment environment=production --overwrite
# 删除标签
kubectl label deployment nginx-deployment environment --remove
# 查看标签
kubectl get pods -L environment
# 查看更info
kubectl describe pod <pod-name>
# 通过组合标签来查找想要的pod
kubectl get pods -l environment=production,tier=frontend
```

控制pod调度

```sh
# 为node-1 添加 标签
kubectl label nodes node-1 disktype=ssd
# 为nginx-deployment标记信息
kubectl label deployment nginx-deployment version=1.0
# 强制覆盖标签去更新
kubectl label deployment nginx-deployment version=2.0 --overwrite
```

Deployment 控制器会检测到 `PodTemplate` 中的标签发生了变更，因此会自动执行滚动更新。滚动更新意味着旧的 Pod 会逐渐被删除，新的 Pod 会被创建, 直到所有的pod都是新版本为止.


```sh
# 查看更新状态
kubectl rollout status deployment nginx-deployment
```

##### patch
是edit的高阶版本, 可以进行脚本化和自动化运维.

```sh
# 资源类型, 资源名, 字段, 参数说明
kubectl patch <resource-type> <resource-name> -p '<patch-data>' [--type=<patch-type>]

# 这里为nginx-deployment加入元标签
kubectl patch deployment nginx-deployment -p '{"metadata": {"labels": {"environment": "production"}}}' --type=merge

# 更新镜像版本
kubectl patch deployment nginx-deployment -p '{"spec": {"template": {"spec": {"containers": [{"name": "nginx", "image": "nginx:1.16.1"}]}}}}'

# 精确更新 
kubectl patch pod my-pod -p '[{"op": "remove", "path": "/metadata/labels/environment"}]' --type=json

# 添加端口
kubectl patch service my-service -p '{"spec": {"ports": [{"port": 8080, "targetPort": 8081}]}}'

```

##### replace

`kubectl replace` 是 Kubernetes 中用于更新现有资源的命令。它通过使用新的配置文件替换现有资源的配置，是一种替换资源的命令式操作方式。

`kubectl replace` 和 `kubectl apply` 的作用看似相似，但它们的使用场景略有不同：
- **`kubectl replace`**：适用于需要完全替换资源配置的情况，要求资源已存在，否则会报错。
- **`kubectl apply`**：适用于声明式管理资源的整个生命周期，支持新建和更新资源。

基本用法

```bash
kubectl replace -f <file-path>
```

- **`-f <file-path>`**：指定资源定义的 YAML 或 JSON 文件路径，`kubectl replace` 将会根据该文件的内容更新资源。

- 替换 Deployment 配置

假设有一个名为 `nginx-deployment.yaml` 的文件，定义了 `nginx-deployment` Deployment：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

执行以下命令替换现有的 `nginx-deployment`：

```bash
kubectl replace -f nginx-deployment.yaml
```

- 强制替换

在某些情况下，资源可能会被其他对象锁定或者存在资源冲突，`kubectl replace` 可以配合 `--force` 标志进行强制替换（注意，这个过程会删除并重新创建资源）：

```bash
kubectl replace -f nginx-deployment.yaml --force
```

**注意**：强制替换会删除并重新创建资源，可能会导致短暂的中断，需谨慎使用，尤其是在生产环境。

- 使用 `--cascade` 替换有依赖的资源

当替换资源时，可能存在子资源依赖的情况，例如替换 Deployment 时，相关的 ReplicaSet 和 Pod 也可能受到影响。使用 `--cascade` 标志来控制删除和替换操作是否级联作用于子资源：

```bash
kubectl replace -f nginx-deployment.yaml --cascade=true
```

- **`--cascade=true`**（默认值）：会级联删除相关的子资源（如 ReplicaSet、Pod 等）。
- **`--cascade=false`**：仅替换父资源，不影响子资源。

- **`kubectl replace`**：完全替换现有资源，适用于更新所有配置的场景。替换时需要保证资源已存在，否则会报错。
- **`kubectl apply`**：声明式管理资源的更新，适用于资源的增量更新。如果资源不存在，`kubectl apply` 会自动创建新资源。

注意事项

1. **资源必须存在**：`kubectl replace` 只能更新已有的资源，如果资源不存在则会报错。使用 `kubectl create` 或 `kubectl apply` 来创建资源。
  
2. **完全替换**：使用 `kubectl replace` 时，新的资源定义文件会完全替换旧的资源定义，这意味着如果你没有包含某些字段（比如标签），这些字段会被删除。

3. **小心使用强制替换**：强制替换（`--force`）会导致资源重新创建，可能会造成服务短暂中断。在生产环境中需要格外小心。

示例场景

更新 Deployment 的镜像版本

假设你有一个 `nginx-deployment.yaml` 文件，用来定义 `nginx-deployment`，现在你想更新 Nginx 的镜像版本为 `1.16.0`。你可以编辑 `nginx-deployment.yaml` 文件，将 `image` 字段更改为 `nginx:1.16.0`，然后使用以下命令进行替换：

```bash
kubectl replace -f nginx-deployment.yaml
```

替换 Service 的类型

假设有一个名为 `my-service` 的 Service，最初配置为 `ClusterIP` 类型，现在你需要将它改为 `NodePort` 类型。可以编辑 Service 配置文件，然后使用 `kubectl replace` 进行替换：

```bash
kubectl replace -f my-service.yaml
```

这样会将原有的 Service 更新为新的类型。

- `kubectl replace` 用于完全替换 Kubernetes 资源的配置。
- 如果资源不存在，`kubectl replace` 会报错，因此只能用于现有资源的更新。
- 强制替换（`--force`）会先删除资源再重新创建，因此可能会导致短暂的服务中断，慎用。
- 与 `kubectl apply` 相比，`replace` 更适合需要完全覆盖现有资源的场景，而 `apply` 更适合声明式管理。

##### rollout
`kubectl rollout` 是 Kubernetes 中用于管理资源的更新过程的命令，主要用于 `Deployment`、`DaemonSet`、`StatefulSet` 等资源的滚动更新和回滚操作。它提供了管理资源的更新、查看更新状态、以及执行回滚等功能。

常用子命令

`kubectl rollout` 命令有几个重要的子命令，分别可以查看和控制 Kubernetes 中 `Deployment` 等控制器的滚动更新。

1. `kubectl rollout status`
用于查看某个 Deployment、DaemonSet 或 StatefulSet 的滚动更新状态。

```bash
kubectl rollout status deployment/nginx-deployment
```

- **状态输出**：该命令会输出该资源当前的更新状态，例如是否完成、是否存在错误等。

2. `kubectl rollout history`
用于查看某个资源的历史版本，包括每次更新的修改信息。

```bash
kubectl rollout history deployment/nginx-deployment
```

- 这条命令会列出 `nginx-deployment` 的历史修订版本。
- 如果想查看某个修订版本的详细信息，可以使用 `--revision` 参数：

  ```bash
  kubectl rollout history deployment/nginx-deployment --revision=2
  ```

3. `kubectl rollout undo`
用于回滚某个 Deployment、DaemonSet 或 StatefulSet 到之前的一个历史版本。

```bash
kubectl rollout undo deployment/nginx-deployment
```

- 该命令会回滚 `nginx-deployment` 到上一个修订版本。
- 如果想回滚到特定版本，可以使用 `--to-revision` 参数：

  ```bash
  kubectl rollout undo deployment/nginx-deployment --to-revision=3
  ```

4. `kubectl rollout pause`
用于暂停某个 Deployment、DaemonSet 或 StatefulSet 的滚动更新。

```bash
kubectl rollout pause deployment/nginx-deployment
```

- 该命令在当前的更新过程中暂停滚动更新，这样你可以在调整配置或进行某些检查后再恢复。

5. `kubectl rollout resume`
用于恢复已暂停的 Deployment、DaemonSet 或 StatefulSet 的滚动更新。

```bash
kubectl rollout resume deployment/nginx-deployment
```

- 当你暂停更新后，可以使用该命令恢复更新过程。

- 创建一个 Deployment 并进行滚动更新

创建一个 `nginx` Deployment，初始镜像为 `nginx:1.14.2`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

应用该配置：

```bash
kubectl apply -f nginx-deployment.yaml
```

- 查看 Deployment 状态

查看滚动更新状态：

```bash
kubectl rollout status deployment/nginx-deployment
```

- 滚动更新 Deployment

修改 `nginx-deployment.yaml`，将镜像更新为 `nginx:1.16.1`，并再次应用：

```yaml
image: nginx:1.16.1
```

```bash
kubectl apply -f nginx-deployment.yaml
```

此时，Kubernetes 会执行滚动更新，将旧版本的 Pod 替换为新版本。

- 查看更新历史

可以查看 `nginx-deployment` 的更新历史：

```bash
kubectl rollout history deployment/nginx-deployment
```

输出类似于：

```
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Updated nginx version to 1.16.1
```
- 回滚更新

如果新版本存在问题，可以回滚到上一个版本：

```bash
kubectl rollout undo deployment/nginx-deployment
```

也可以指定具体的版本来回滚：

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

- 暂停和恢复更新

当你正在进行更新，但需要暂停更新操作，可以执行：

```bash
kubectl rollout pause deployment/nginx-deployment
```

在暂停后，你可以进行某些检查或配置更改，然后继续更新：

```bash
kubectl rollout resume deployment/nginx-deployment
```

- 使用场景

1. **更新状态监控**：使用 `kubectl rollout status` 来查看更新的实时状态，确保更新过程正常进行，没有出错。

2. **快速回滚**：通过 `kubectl rollout undo` 可以快速回滚到之前的工作版本，特别适合于在生产环境中进行更新时发生问题的情况。

3. **暂停/恢复更新**：如果在更新过程中需要进行额外检查或确保某些资源准备好，可以使用 `kubectl rollout pause` 来暂停更新，检查完毕后使用 `resume` 恢复。

4. **历史记录追踪**：`kubectl rollout history` 提供了所有更新历史，帮助你了解 Deployment 的变更情况，尤其是了解每次更新的变更原因。

- 总结

- `kubectl rollout` 是 Kubernetes 中用于管理 Deployment、DaemonSet、StatefulSet 等资源更新的工具。
- 通过 `status` 查看滚动更新的状态，通过 `history` 查看历史版本，通过 `undo` 回滚到以前的版本。
- 支持暂停（`pause`）和恢复（`resume`）更新操作，用于更精细的控制更新过程。
  
`kubectl rollout` 是 Kubernetes 中非常重要的命令，帮助运维人员在生产环境中安全、可靠地进行更新和版本控制。如果你对某些子命令有具体问题或想要了解更多细节，随时告诉我！


##### scale
`kubectl scale` 是 Kubernetes 中用于水平伸缩（Horizontal Scaling）资源的命令。它允许你动态地调整 Deployment、ReplicaSet、StatefulSet 等资源的副本数量，从而增加或减少 Pod 的数量来适应工作负载的变化。

基本用法

```bash
kubectl scale <resource-type> <resource-name> --replicas=<number>
```

- **`<resource-type>`**：资源类型，例如 `deployment`、`replicaset`、`statefulset` 等。
- **`<resource-name>`**：要伸缩的资源名称。
- **`--replicas=<number>`**：指定目标副本数量。

1. 水平伸缩 Deployment

假设有一个名为 `nginx-deployment` 的 Deployment，当前有 3 个副本，现在想要增加到 5 个副本：

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

执行这条命令后，Kubernetes 会增加 2 个新的 Pod，最终 Pod 副本数为 5。

2. 水平伸缩 ReplicaSet

如果你有一个名为 `nginx-replicaset` 的 ReplicaSet，可以使用以下命令将其副本数量减少到 2 个：

```bash
kubectl scale replicaset nginx-replicaset --replicas=2
```

3. 水平伸缩 StatefulSet

同样地，可以对 StatefulSet 进行伸缩，例如将名为 `nginx-statefulset` 的 StatefulSet 副本数设置为 4：

```bash
kubectl scale statefulset nginx-statefulset --replicas=4
```

通过标签选择器伸缩多个资源

你也可以使用标签选择器来批量伸缩具有相同标签的多个资源。例如，所有具有标签 `app=nginx` 的 Deployment，伸缩它们的副本数量到 6：

```bash
kubectl scale deployment -l app=nginx --replicas=6
```

适用场景

- **负载增加时增加副本数量**：当负载增加时，可以使用 `kubectl scale` 增加 Deployment 或 StatefulSet 的副本数量，以增加处理能力。
- **负载降低时减少副本数量**：在低峰期减少副本数量，节省集群资源。
- **手动控制副本数**：相比于 HPA（Horizontal Pod Autoscaler）自动调整，`kubectl scale` 适合在手动干预或特殊情况下的副本数量调整。

与 `autoscale` 的区别

- **`kubectl scale`**：用于手动调整资源的副本数量，可以精确指定副本数量。适用于需要手动干预的场景。
- **`kubectl autoscale`**：用于自动调整副本数量，基于 CPU、内存等指标，适用于动态调整副本数量以应对负载变化。

查看伸缩后的结果

使用以下命令查看伸缩后的资源状态：

```bash
kubectl get deployment nginx-deployment
```

输出中会显示 `READY` 和 `REPLICAS` 的信息，指示当前副本的实际状态和目标副本数。

也可以查看具体的 Pod 状态：

```bash
kubectl get pods -l app=nginx
```

这条命令会列出所有带有 `app=nginx` 标签的 Pod，并显示它们的当前状态。

示例操作流程

1. **创建 Deployment**

假设你有一个 Deployment 定义文件 `nginx-deployment.yaml`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

首先，应用该 Deployment：

```bash
kubectl apply -f nginx-deployment.yaml
```

2. **使用 `kubectl scale` 进行伸缩**

通过以下命令将副本数增加到 6：

```bash
kubectl scale deployment nginx-deployment --replicas=6
```

3. **查看 Deployment 状态**

```bash
kubectl get deployment nginx-deployment
```

输出示例：

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   6/6     6            6           10m
```

这表示 6 个副本已经准备好并且可用。

总结

- `kubectl scale` 用于手动调整 Kubernetes 资源（如 Deployment、ReplicaSet、StatefulSet）的副本数量。
- 可以通过指定目标副本数量来增加或减少 Pod 数量，以应对工作负载的变化。
- 适用于手动干预的情况，而自动伸缩则可以使用 `kubectl autoscale`。

通过 `kubectl scale`，你可以灵活地管理应用程序的资源分配，确保它们能够在不同负载下有足够的处理能力。如果你有特定的场景或更多的疑问，随时告诉我！

##### annotate
在 Kubernetes 中，`kubectl annotate` 是用于为资源添加、修改或删除注解（annotations）的命令。注解与标签类似，都是键值对形式，但它们主要用于存储资源的元数据信息，不用于资源选择或筛选。

注解通常用于存储对资源的描述、监控信息、配置管理元数据等，而不影响 Kubernetes 控制器的调度行为。

基本用法

```bash
kubectl annotate <resource-type> <resource-name> <annotation-key>=<annotation-value> [--overwrite]
```

- **`<resource-type>`**：资源类型，例如 `pod`、`node`、`service`、`deployment` 等。
- **`<resource-name>`**：要标注的资源的名称。
- **`<annotation-key>=<annotation-value>`**：注解键值对。
- **`--overwrite`**：用于覆盖已存在的注解。

示例

1. 为资源添加注解

给名为 `nginx-deployment` 的 Deployment 添加一个注解 `description=This is a Nginx deployment`：

```bash
kubectl annotate deployment nginx-deployment description="This is a Nginx deployment"
```

这样，`nginx-deployment` 就被打上了 `description=This is a Nginx deployment` 的注解。

2. 添加多个注解

可以一次性添加多个注解，例如：

```bash
kubectl annotate deployment nginx-deployment author="admin" version="1.0"
```

3. 修改已有注解

如果你想修改一个已有的注解，比如把 `description` 修改为 `Updated description`，可以使用 `--overwrite` 参数：

```bash
kubectl annotate deployment nginx-deployment description="Updated description" --overwrite
```

4. 删除注解

可以使用 `annotation-key-` 来删除注解：

```bash
kubectl annotate deployment nginx-deployment description-
```

这会从 `nginx-deployment` 中删除 `description` 注解。

查看注解

1. **通过 `kubectl get` 命令查看注解**：

   你可以使用 `kubectl get` 命令并添加 `-o yaml` 来查看资源的详细信息，包括注解：

   ```bash
   kubectl get deployment nginx-deployment -o yaml
   ```

   这会输出完整的 YAML 文件，其中包含所有注解。

2. **使用 `kubectl describe` 查看注解**：

   也可以使用 `kubectl describe` 来查看注解：

   ```bash
   kubectl describe deployment nginx-deployment
   ```

   在输出的 `Annotations` 部分，你可以看到当前资源的所有注解。

应用场景

1. **元数据存储**：注解主要用于存储与 Kubernetes 控制器无关的元数据。例如，可以存储应用程序的详细描述、作者信息、版本、操作日志等信息。

2. **自动化工具的标记**：注解可以被用于自动化工具和脚本。例如，CI/CD 工具可以在注解中记录最后一次更新的时间或更新者的名字。

3. **运维和监控**：某些监控系统和运维工具依赖于注解来存储配置、监控信息等，用于追踪资源状态或特定的资源元数据。

4. **外部系统集成**：一些外部系统（如 GitOps 工具）可能使用注解来存储特定的配置信息，例如同步状态、版本控制等。

注解与标签的区别

- **标签（Labels）**：用于选择和过滤 Kubernetes 资源，标签通常被 Kubernetes 的控制器（例如服务选择器）用于执行某些操作。
- **注解（Annotations）**：用于存储元数据信息，注解不用于选择资源，也不影响调度和控制器行为，主要用于描述、追踪和集成外部系统。

示例场景

1. 给 Pod 添加调试信息

你可以给一个 Pod 添加调试相关的信息，例如：

```bash
kubectl annotate pod nginx-pod debug-session="2024-10-07T10:30Z"
```

这个注解可以用于标记某个 Pod 进行过调试，记录调试的时间。

2. 为服务添加更新原因注解

假设你在某个服务的配置变更时，需要记录变更原因，可以使用注解来标记：

```bash
kubectl annotate service my-service update-reason="Changed service type from ClusterIP to NodePort"
```

这样，其他团队成员在查看服务时就能知道为什么服务进行了变更。

总结

- `kubectl annotate` 用于为 Kubernetes 资源添加、修改或删除注解。
- 注解是键值对形式的元数据，主要用于描述资源、存储外部系统的集成信息等。
- 注解不会影响 Kubernetes 的调度和选择，适合用于追踪、描述和记录额外信息。
- 可以使用 `--overwrite` 来修改已有注解，使用 `annotation-key-` 来删除注解。


##### debug

在 Kubernetes 中，`kubectl debug` 是一个非常有用的命令，用于帮助开发者和运维人员对正在运行的 Pod 进行调试和问题诊断。`kubectl debug` 提供了一种便捷的方法来启动调试容器、创建有特定调试环境的 Pod 或者进入某个正在运行的容器，查看其内部状态。

```bash
kubectl debug <target> [options]
```

- **`<target>`**：调试目标，可以是一个正在运行的 Pod 名称，或者你也可以为其他资源（如节点）启动调试会话。

主要用途

1. **在现有 Pod 中启动调试容器**。
2. **基于现有 Pod 的配置创建副本并进行调试**。
3. **调试节点（例如进入 Node 的 Shell）**。

使用场景与示例

1. 在 Pod 中启动调试容器

在 Kubernetes 中，有时候我们可能需要对一个正在运行的 Pod 进行详细调试，比如查看文件系统或者安装调试工具。`kubectl debug` 提供了一种方便的方式启动调试容器，该容器共享目标 Pod 的命名空间，但使用你指定的镜像来进行调试。

例如，你可以为名为 `nginx-pod` 的 Pod 启动一个新容器：

```bash
kubectl debug -it nginx-pod --image=busybox --target=nginx
```

- **`-it`**：以交互式方式进入调试容器。
- **`--image=busybox`**：使用 `busybox` 作为调试容器的镜像。
- **`--target=nginx`**：指定你要调试的容器（如果目标 Pod 中有多个容器）。

在启动的调试容器中，你可以安装调试工具、查看文件系统，或者进行一些临时配置。

2. 复制并调试一个 Pod

如果你不想直接对正在运行的 Pod 进行调试，而是希望创建一个相同的 Pod 的副本来做调试，可以使用 `kubectl debug` 创建一个调试副本。这样可以避免对生产 Pod 的干扰。

例如，创建 `nginx-pod` 的调试副本并修改命令：

```bash
kubectl debug nginx-pod --copy-to=nginx-pod-debug --container=nginx --image=nginx --set-image=nginx=nginx:1.16 --replace
```

- **`--copy-to=nginx-pod-debug`**：将现有的 `nginx-pod` 复制为 `nginx-pod-debug`。
- **`--container=nginx`**：指定调试的容器。
- **`--set-image`**：可以设置调试副本的镜像，例如用旧版本镜像来复现问题。
- **`--replace`**：替换现有副本，适用于多次调试操作。

3. 调试节点

`kubectl debug` 还可以用来调试 Kubernetes 节点，这个功能在进行节点级别问题（例如网络连接、节点健康状态）诊断时非常有用。

例如，进入节点的调试 shell：

```bash
kubectl debug node/<node-name> -it --image=busybox
```

- **`node/<node-name>`**：指定要调试的节点名称。
- **`-it`**：以交互式模式运行。
- **`--image=busybox`**：使用 `busybox` 镜像进入节点的调试 shell。

这条命令会为你创建一个 DaemonSet，允许你在节点上运行一个调试容器，并进入 shell 中对节点进行操作和排查。

常用选项

- **`-it`**：启动调试容器并进入交互式 shell。
- **`--image=<image>`**：指定用于调试的容器镜像，通常会选择一些轻量级的调试镜像，比如 `busybox` 或 `alpine`。
- **`--copy-to=<name>`**：基于指定的 Pod 创建一个副本，并将副本命名为 `<name>`，这样你可以对副本进行调试而不会影响原 Pod。
- **`--replace`**：如果已经存在同名的目标 Pod，则进行替换操作。
- **`--container=<container>`**：指定要调试的容器（如果 Pod 中有多个容器）。
- **`--set-image=<container-name>=<image>`**：为特定容器设置新镜像，这个选项可以用于调试时测试不同的镜像版本。

示例场景

1. **安装调试工具**：有时候某些调试工具（如 `curl`、`netcat` 等）没有预装在你的应用镜像中，你可以通过 `kubectl debug` 启动调试容器（例如基于 `busybox` 或 `alpine`），这些镜像中包含必要的调试工具，方便你进行排查。

   ```bash
   kubectl debug my-pod -it --image=alpine
   ```

2. **调试网络问题**：如果你需要检查 Pod 与外部服务之间的连接问题，可以进入 Pod 的调试容器，使用一些网络工具（如 `ping`、`curl`）来测试网络连通性。

3. **节点调试**：当某个节点的状态异常，可能需要进入节点进行排查。可以通过 `kubectl debug` 来启动一个基于某个节点的调试容器，进入到节点环境进行详细检查。

注意事项

1. **调试容器的资源共享**：调试容器与目标 Pod 的其他容器共享相同的网络命名空间和挂载卷，这意味着调试容器可以访问原容器的文件系统和网络配置。
  
2. **避免生产环境干扰**：如果你需要调试某个正在处理请求的生产 Pod，最好使用 `--copy-to` 创建一个副本，这样可以在不影响生产流量的情况下进行调试。

3. **删除调试副本**：调试完成后，别忘了删除调试的 Pod 副本，以避免不必要的资源浪费：

   ```bash
   kubectl delete pod nginx-pod-debug
   ```

总结

- `kubectl debug` 是用于调试 Kubernetes 集群中资源的强大工具，特别是 Pod 和节点。
- 可以为现有 Pod 启动一个调试容器、基于现有 Pod 创建调试副本，或者直接在节点中启动调试会话。
- 调试容器可以与目标 Pod 的其他容器共享网络和文件系统，非常适合用于实时排查问题。
- 使用 `--copy-to` 参数可以创建副本，避免对生产资源造成影响。

##### diff
`kubectl diff` 是 Kubernetes 中的一个命令，用于显示本地配置文件与集群中当前资源配置之间的差异。它帮助开发人员和运维人员在应用新的配置之前预览变更，确保了解将要做出的修改，类似于 Git 中的 `git diff` 命令。它在配置变更前提供了一种非常有用的检查手段，以确保变更符合预期，避免意外问题。

```bash
kubectl diff -f <file-path> [options]
```

- **`-f <file-path>`**：指定要与集群中资源进行对比的本地文件，文件可以是 YAML 或 JSON 格式。
- **`[options]`**：其他参数选项，比如命名空间、上下文等。


1. 查看 Deployment 配置的差异

假设你有一个 Deployment 配置文件 `nginx-deployment.yaml`，并且它已应用到集群中。现在你对本地文件进行了一些修改，想要查看这些修改与集群中现有资源之间的差异，可以使用以下命令：

```bash
kubectl diff -f nginx-deployment.yaml
```

输出中会显示本地文件和集群中 Deployment 配置的不同之处，例如添加了新字段、修改了副本数、改变了镜像版本等。

2. 查看有多个资源的差异

如果你有一个 YAML 文件，包含了多个资源（比如一个 Deployment 和一个 Service），你可以用 `kubectl diff` 查看它们与集群中的差异：

```bash
kubectl diff -f multi-resources.yaml
```

Kubernetes 会自动检测该文件中的所有资源，并逐个与集群中已存在的资源进行比较，输出它们的差异。

3. 指定命名空间查看差异

如果你的资源在特定的命名空间中，你可以使用 `-n` 或 `--namespace` 参数指定命名空间，以便正确地查找集群中的资源：

```bash
kubectl diff -f nginx-deployment.yaml -n my-namespace
```

这样，`kubectl` 会在命名空间 `my-namespace` 中查找相应的资源，并进行对比。

4. 与 `apply` 的结合

`kubectl diff` 的输出可以帮助你在执行 `kubectl apply` 之前预览变更。通常，你可以先运行 `kubectl diff` 以确认变更符合预期，然后再运行 `kubectl apply` 进行应用。

```bash
kubectl diff -f nginx-deployment.yaml
# 如果确认变更无误
kubectl apply -f nginx-deployment.yaml
```

这样可以有效避免应用配置变更后出现意外的问题。

示例输出

假设 `nginx-deployment.yaml` 中修改了副本数，从 3 修改为 5，运行 `kubectl diff -f nginx-deployment.yaml` 后，输出可能如下：

```diff
diff -u -N /tmp/LIVE-348221051/apps.v1.Deployment.default.nginx-deployment /tmp/MERGED-662267896/apps.v1.Deployment.default.nginx-deployment
--- /tmp/LIVE-348221051/apps.v1.Deployment.default.nginx-deployment 2024-10-07 08:00:00.000000000 +0000
+++ /tmp/MERGED-662267896/apps.v1.Deployment.default.nginx-deployment 2024-10-07 08:05:00.000000000 +0000
@@ -1,7 +1,7 @@
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: nginx-deployment
 spec:
   replicas: 3
+  replicas: 5
```

从输出可以看到，本地文件中的副本数已经更改为 5，而集群中当前的副本数还是 3。

适用场景

1. **配置变更前的检查**：在应用更改之前，使用 `kubectl diff` 检查变更是否符合预期，避免因为疏忽引入错误配置。
  
2. **团队协作中确认变更**：在多人协作的环境中，`kubectl diff` 可以帮助你检查其他成员对配置的更改，确保了解集群中实际应用的资源与本地修改之间的差异。

3. **自动化部署流程**：在自动化部署管道中，`kubectl diff` 可以用于在应用更改之前进行预检查，确保自动化流程不引入不必要的配置变更。

注意事项

1. **只显示差异，不进行实际更改**：`kubectl diff` 只会显示配置的差异，不会对集群中的资源进行任何实际更改。你需要使用 `kubectl apply` 或其他命令来应用变更。

2. **权限问题**：`kubectl diff` 需要对相应的资源具有读取权限。如果用户权限不足，可能无法获取集群中的现有资源，从而无法进行对比。

3. **变更的完整性**：`kubectl diff` 只能检查当前文件中的内容与集群资源的差异，不能检查其他可能会间接受到影响的资源。

4. **仅支持 Kubernetes API 服务器**：`kubectl diff` 需要连接到 Kubernetes API 服务器进行对比，离线情况下无法使用。

总结

- `kubectl diff` 是 Kubernetes 提供的一个非常有用的工具，用于在应用更改之前显示本地配置文件与集群中资源配置之间的差异。
- 它可以帮助你检查变更内容是否符合预期，从而降低配置错误的风险。
- `kubectl diff` 常用于在进行 `kubectl apply` 之前，以确保变更安全和准确。
- 使用 `-f <file-path>` 指定要比较的文件，使用 `-n <namespace>` 来指定命名空间。

##### kustomize
`kubectl kustomize` 是 Kubernetes 中用于简化配置管理的一种工具，基于 `kustomize`。`kustomize` 提供了一种无需通过模板编写的方式来管理 Kubernetes 应用的配置，是原生支持的 YAML 配置管理工具。它通过声明性地“叠加”各种配置，可以方便地复用和管理 Kubernetes 资源的变更，例如配置覆盖、环境定制等。


在 `kustomize` 中，主要涉及以下几个概念：

1. **Base**：基础的资源清单，例如 Deployment、Service 等。
2. **Overlay**：基于 `Base`，覆盖部分配置，例如不同环境（开发、测试、生产）需要不同的副本数、环境变量等。
3. **Kustomization.yaml**：`kustomize` 使用的配置文件，通过它来定义哪些资源需要管理、如何覆盖配置等。


```bash
kubectl kustomize <kustomization-directory>
```

- **`<kustomization-directory>`**：包含 `kustomization.yaml` 文件的目录，`kustomize` 将根据该文件合成资源配置并输出。

通常我们使用 `kubectl apply` 结合 `kustomize` 来应用这些合成的配置，例如：

```bash
kubectl apply -k <kustomization-directory>
```

`kustomization.yaml` 配置

`kustomization.yaml` 文件是 `kustomize` 的核心，它用于定义如何处理和生成 Kubernetes 资源配置。以下是一个基本的 `kustomization.yaml` 配置示例：

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 指定需要管理的基础资源
resources:
  - deployment.yaml
  - service.yaml

# 增加标签
commonLabels:
  app: nginx

# 修改某些字段的值
patchesStrategicMerge:
  - patch.yaml

# 指定命名空间
namespace: my-namespace
```


假设你有一个 Kubernetes 项目目录结构如下：

```
my-app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── development/
    │   ├── kustomization.yaml
    │   └── patch.yaml
    └── production/
        ├── kustomization.yaml
        └── patch.yaml
```

1. 创建 Base 配置

首先在 `base` 目录中创建基础配置，定义应用的一般性资源。

**`base/deployment.yaml`**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

**`base/kustomization.yaml`**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

这样就定义了一个基础的 Deployment 和 Service 配置，称之为 `Base`。

2. 创建环境覆盖（Overlay）

为了在不同环境中复用基础配置，可以创建环境特定的 `Overlay`。

开发环境的覆盖

在 `overlays/development` 目录下创建 `kustomization.yaml`：

**`overlays/development/kustomization.yaml`**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

patchesStrategicMerge:
  - patch.yaml
```

**`overlays/development/patch.yaml`**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
```

这个 `patch.yaml` 用于覆盖基础配置，将开发环境的副本数改为 1 个。

生产环境的覆盖

类似地，为生产环境创建覆盖文件：

**`overlays/production/kustomization.yaml`**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

patchesStrategicMerge:
  - patch.yaml
```

**`overlays/production/patch.yaml`**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 5
```

这个 `patch.yaml` 用于覆盖基础配置，将生产环境的副本数改为 5 个。

3. 应用不同环境的配置

使用 `kubectl kustomize` 来生成配置内容：

```bash
kubectl kustomize overlays/development
```

这会显示出合成后的 YAML 配置，你可以查看并验证合成的内容。

使用 `kubectl apply` 应用开发环境的配置：

```bash
kubectl apply -k overlays/development
```

对于生产环境，只需要切换目录：

```bash
kubectl apply -k overlays/production
```

常见功能和特性

1. **`commonLabels`**：`commonLabels` 可以用于为所有资源增加相同的标签，例如：

   ```yaml
   commonLabels:
     environment: development
   ```

   这会为所有资源增加一个标签 `environment=development`，方便对资源进行筛选和管理。

2. **`patchesStrategicMerge`**：用于部分修改现有资源，例如调整副本数、添加环境变量等。它会以“合并”的方式修改指定的字段。

3. **`patchesJson6902`**：支持基于 JSON Patch 标准（RFC 6902）的精确补丁，可以进行增、删、改操作，适用于需要精确控制的场景。

4. **`configMapGenerator`** 和 **`secretGenerator`**：可以自动生成 ConfigMap 和 Secret。例如：

   ```yaml
   configMapGenerator:
     - name: my-config
       literals:
         - key1=value1
         - key2=value2
   ```

   这会生成一个名为 `my-config` 的 ConfigMap，并包含指定的键值对。

5. **`namespace`**：可以为所有资源指定命名空间，以便在不同的命名空间中部署相同的配置。

总结

- `kubectl kustomize` 和 `kubectl apply -k` 是 Kubernetes 中用于管理和合成配置的命令，基于 `kustomize` 工具。
- `kustomize` 提供了一种无需模板化的方式管理 Kubernetes 资源，通过声明性的配置覆盖来实现多环境管理和复用。
- **Base** 和 **Overlay** 的概念使得你可以构建基础配置，并在不同环境中通过 Overlay 覆盖某些字段，实现环境特定的部署。
- 使用 `patchesStrategicMerge` 可以对基础配置进行部分覆盖，而无需重写整个配置文件。
  
通过 `kustomize`，你可以方便地管理 Kubernetes 应用的配置，避免重复性工作，提高配置管理的灵活性。如果你有更具体的需求或想了解更多关于 `kustomize` 的功能，随时告诉我！



##### wait
`kubectl wait` 是 Kubernetes 中的一个命令，用于等待某些资源达到特定状态或条件。它通常用于自动化任务或者脚本化部署流程中，确保某些资源（如 Pod、Deployment、Job 等）达到预期的状态之后再执行接下来的步骤。

```bash
kubectl wait <resource-type> <resource-name> --for=<condition> [options]
```

- **`<resource-type>`**：资源类型，例如 `pod`、`deployment`、`job` 等。
- **`<resource-name>`**：资源的名称，可以是一个具体的名称或者使用标签选择器。
- **`--for=<condition>`**：指定等待的条件，例如 `condition=Ready`、`delete`、`complete` 等。

常见的 `--for` 条件

1. **`condition=Ready`**：等待某个资源准备就绪。例如，等待 Pod 就绪。
2. **`delete`**：等待资源被删除，通常用于确保某个资源已经被成功删除。
3. **`condition=complete`**：等待 Job 运行完成。
4. **`condition=available`**：等待 Deployment 可用，确保有足够的副本处于可用状态。

 
 示例

1. 等待 Pod 准备就绪

假设你有一个名为 `nginx-pod` 的 Pod，你希望脚本继续执行之前等待它准备就绪，可以使用以下命令：

```bash
kubectl wait pod/nginx-pod --for=condition=Ready --timeout=60s
```

- **`pod/nginx-pod`**：指定资源类型和资源名称。
- **`--for=condition=Ready`**：等待条件是 Pod 准备好。
- **`--timeout=60s`**：最多等待 60 秒，超时后命令将失败。

2. 等待 Deployment 完全可用

等待名为 `nginx-deployment` 的 Deployment 副本全部可用：

```bash
kubectl wait deployment/nginx-deployment --for=condition=available --timeout=120s
```

- **`--for=condition=available`**：等待 Deployment 副本可用，确保有足够数量的 Pod 在运行并且健康。
- **`--timeout=120s`**：设置超时时间为 120 秒。

3. 等待 Job 完成

等待名为 `backup-job` 的 Job 完成：

```bash
kubectl wait job/backup-job --for=condition=complete --timeout=300s
```

- **`--for=condition=complete`**：等待 Job 完成。
- **`--timeout=300s`**：设置超时时间为 300 秒。

4. 等待资源删除

等待名为 `test-pod` 的 Pod 被删除：

```bash
kubectl delete pod test-pod &
kubectl wait pod/test-pod --for=delete --timeout=30s
```

- **`kubectl delete pod test-pod &`**：后台删除 `test-pod`。
- **`kubectl wait pod/test-pod --for=delete`**：等待该 Pod 被成功删除。
  
这种情况通常用于确认资源已经被完全删除，以便继续创建同名的资源或进行其他清理工作。

5. 使用标签选择器等待多个资源

可以使用标签选择器来等待多个资源的状态。例如，等待所有具有 `app=nginx` 标签的 Pod 准备好：

```bash
kubectl wait pod -l app=nginx --for=condition=Ready --timeout=60s
```

- **`-l app=nginx`**：使用标签选择器选择具有 `app=nginx` 标签的 Pod。
- 这条命令会等待所有匹配的 Pod 都变为 Ready 状态。

选项

- **`--timeout=<duration>`**：指定最大等待时间。例如 `30s`（秒）、`5m`（分钟）。如果超时，命令将返回失败状态。
- **`--namespace=<namespace>`**：指定命名空间。如果资源在特定命名空间中，可以使用这个选项来指定命名空间。
- **`--all`**：针对某种类型的资源（如 Pod），等待该类型所有资源达到目标状态。

使用场景

1. **自动化脚本**：在 CI/CD 流程或者自动化脚本中，使用 `kubectl wait` 确保资源达到预期状态（如 Pod 准备好）再继续执行后续任务，以避免访问尚未准备好的服务。

2. **串行部署**：在进行应用的逐步部署时，使用 `kubectl wait` 确保前一个阶段的资源完全可用后再开始部署下一个阶段。

3. **资源清理**：在删除资源后，使用 `kubectl wait` 确保资源完全被删除，以避免残留影响后续的操作。

`kubectl wait` 与 `kubectl get` 的区别

- **`kubectl wait`**：是一个阻塞命令，直到满足条件或超时。适用于需要自动化执行和等待的场景。
- **`kubectl get`**：是一个非阻塞命令，显示当前的状态，不会主动等待资源状态变化。适合检查资源的当前状态。

总结

- `kubectl wait` 用于等待 Kubernetes 资源达到特定状态，是部署、更新、自动化中的重要工具。
- 支持等待资源的各种条件，例如 `condition=Ready`、`complete`、`available`、`delete` 等。
- 可以结合超时选项 `--timeout` 确保不会无限期等待。
- 在自动化脚本和 CI/CD 管道中，可以用它来确保资源按预期状态进行，减少手动干预。
##### set
`kubectl set` 是 Kubernetes 中用于更新资源配置的命令，可以修改一些特定的属性，例如容器的镜像、环境变量、资源请求和限制等。`kubectl set` 提供了一种快速修改资源的方法，特别适合需要在已有资源上进行简单的更改时使用。


`kubectl set` 主要用于以下几种常见操作：

1. **`set image`**：更新容器的镜像版本。
2. **`set resources`**：为容器设置资源请求和限制。
3. **`set env`**：为容器设置环境变量。
4. **`set selector`**：设置或修改服务的标签选择器。
5. **`set subject`**：为角色绑定或集群角色绑定设置主体（用户、组、服务账号等）。


1. `kubectl set image` - 更新镜像

`kubectl set image` 用于更新 Deployment、DaemonSet、StatefulSet 等资源的容器镜像。

假设你有一个名为 `nginx-deployment` 的 Deployment，当前使用的是 `nginx:1.14.2`，现在想更新到 `nginx:1.16.1`，可以使用以下命令：

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

- **`deployment/nginx-deployment`**：指定要修改的 Deployment。
- **`nginx=nginx:1.16.1`**：指定要更新的容器名称和对应的新镜像。

多个容器可以通过空格分隔进行更新：

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 sidecar=busybox:1.28
```

2. `kubectl set resources` - 设置资源请求和限制

`kubectl set resources` 用于设置容器的 CPU 和内存请求和限制。

例如，给名为 `nginx-deployment` 的 Deployment 中的容器 `nginx` 设置资源限制：

```bash
kubectl set resources deployment nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
```

- **`deployment nginx-deployment`**：指定资源类型和名称。
- **`-c=nginx`**：指定要设置的容器名称。
- **`--limits=cpu=200m,memory=512Mi`**：设置 CPU 和内存的限制。

如果想要同时设置资源请求（requests）和限制（limits），可以这样：

```bash
kubectl set resources deployment nginx-deployment -c=nginx --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi
```

3. `kubectl set env` - 设置环境变量

`kubectl set env` 用于为 Pod、Deployment 等资源设置环境变量。

为名为 `nginx-deployment` 的 Deployment 添加环境变量 `ENV=production`：

```bash
kubectl set env deployment/nginx-deployment ENV=production
```

- **`deployment/nginx-deployment`**：指定资源类型和名称。
- **`ENV=production`**：指定要添加的环境变量。

你也可以从文件或者 ConfigMap 中批量导入环境变量。例如，从文件导入：

```bash
kubectl set env deployment/nginx-deployment --env-file=env.list
```

4. `kubectl set selector` - 设置服务的标签选择器

`kubectl set selector` 用于更新服务的标签选择器。

假设你有一个名为 `my-service` 的 Service，想要将它的标签选择器修改为 `app=nginx`：

```bash
kubectl set selector service/my-service app=nginx
```

5. `kubectl set subject` - 设置角色绑定主体

`kubectl set subject` 用于设置或修改角色绑定（RoleBinding）或集群角色绑定（ClusterRoleBinding）的主体（用户、组、服务账号等）。

例如，给名为 `read-pods` 的角色绑定添加用户 `jane`：

```bash
kubectl set subject rolebinding read-pods --user=jane
```

- **`rolebinding read-pods`**：指定角色绑定名称。
- **`--user=jane`**：添加用户 `jane` 为该角色绑定的主体。

如果想要添加一个服务账号为主体：

```bash
kubectl set subject rolebinding read-pods --serviceaccount=default:my-service-account
```

适用场景

1. **快速修改现有资源配置**：`kubectl set` 可以在资源不变的情况下快速修改特定字段，例如更新镜像、添加环境变量、修改资源限制等，适用于对生产环境进行小幅改动的场景。

2. **滚动更新**：使用 `kubectl set image` 更新镜像后，Deployment 会自动触发滚动更新，可以用这种方式来进行零停机的版本更新。

3. **灵活的资源管理**：通过设置环境变量或资源请求和限制，可以方便地调节应用的运行配置，适应不同的资源需求。

注意事项

1. **滚动更新**：使用 `kubectl set image` 修改镜像后，Deployment 会自动执行滚动更新，以保证服务不中断。

2. **覆盖操作**：`kubectl set` 对配置的修改是直接覆盖的，某些情况下可能会引起不可预见的问题，特别是在更新重要参数时，需要仔细验证。

3. **命令快捷性**：相比于修改 YAML 文件并使用 `kubectl apply`，`kubectl set` 提供了更快捷的命令行方式来修改配置，适合于快速调试和小改动。

示例操作流程

假设你有一个 `nginx` Deployment，当前运行的是镜像 `nginx:1.14.2`，现在想要进行以下修改：

1. **更新镜像到 `nginx:1.16.1`**：

   ```bash
   kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
   ```

2. **增加环境变量 `ENV=production`**：

   ```bash
   kubectl set env deployment/nginx-deployment ENV=production
   ```

3. **设置容器资源请求和限制**：

   ```bash
   kubectl set resources deployment nginx-deployment -c=nginx --requests=cpu=100m,memory=200Mi --limits=cpu=200m,memory=400Mi
   ```

总结

- `kubectl set` 是一个命令行工具，用于快速更新 Kubernetes 资源的配置。
- 主要支持的功能有：更新镜像（`set image`）、设置资源请求和限制（`set resources`）、设置环境变量（`set env`）、设置服务标签选择器（`set selector`）以及修改角色绑定主体（`set subject`）。
- `kubectl set` 非常适合在已有资源上进行小改动和快速更新，避免繁琐的手动 YAML 文件修改操作。


#### 通用命令
##### create

kubectl create 是 Kubernetes 中用于创建新资源的命令。与 kubectl apply 不同的是，kubectl create 通常用于初次创建资源，适合手动创建简单的资源，而 kubectl apply 更适合配置文件的多次应用和更新。kubectl create 既可以通过命令行指定各种参数创建资源，也可以通过 YAML 或 JSON 文件创建。

基本用法

```
kubectl create <resource> [options]
```

resource：要创建的资源类型，例如 pod、deployment、service、configmap 等。
[options]：用于指定资源配置的各种参数。
你还可以通过文件创建资源：

```
kubectl create -f <file-path>
```

常见的 kubectl create 用法示例

1. 创建 Pod

使用命令行直接创建一个 Pod，指定镜像：

```
kubectl create pod nginx --image=nginx:1.14.2
```

这里使用 kubectl create pod 命令创建一个名为 nginx 的 Pod，运行 nginx:1.14.2 镜像。

2. 创建 Deployment

创建一个 Deployment，用于管理多个 Pod：

```
kubectl create deployment nginx-deployment --image=nginx:1.16.1
```

- deployment nginx-deployment：指定 Deployment 名称为 nginx-deployment。
- --image=nginx:1.16.1：指定容器镜像为 nginx:1.16.1。

可以通过以下命令扩展该 Deployment 的副本数，例如设置副本数为 3：

```
kubectl scale deployment/nginx-deployment --replicas=3
```

3. 创建 Service

使用 kubectl create 创建一个 ClusterIP 类型的 Service，暴露端口：

```
kubectl create service clusterip nginx --tcp=80:80
```

- service clusterip nginx：创建一个 ClusterIP 类型的 Service，名称为 nginx。
- --tcp=80:80：将 80 端口对外暴露。

如果你想创建一个 NodePort 类型的 Service，可以这样：

```
kubectl create service nodeport nginx --tcp=80:80 --node-port=30007
```

这里会将 nginx 服务暴露在集群外部，并在节点上使用 30007 端口。

4. 创建 ConfigMap

kubectl create configmap 用于创建 ConfigMap，可以包含应用程序的配置信息。

例如，从键值对创建一个 ConfigMap：

```
kubectl create configmap example-config --from-literal=key1=value1 --from-literal=key2=value2
```

- example-config：ConfigMap 的名称。
- --from-literal=key1=value1：通过键值对设置配置项。

也可以从文件中创建 ConfigMap：

```
kubectl create configmap example-config --from-file=config.txt
```

5. 创建 Secret

kubectl create secret 用于创建 Secret。Secret 用于存储敏感信息，例如密码、令牌等。

创建一个包含用户名和密码的 Secret：

```
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=secret
```

- secret generic db-secret：创建一个 Secret，名称为 db-secret。
- --from-literal=username=admin：指定一个键值对的敏感信息。

也可以从文件中创建 Secret，例如使用证书文件：

```
kubectl create secret tls my-tls-secret --cert=path/to/cert/file --key=path/to/key/file
```

6. 从 YAML 或 JSON 文件创建资源

通过指定文件来创建资源：

```
kubectl create -f deployment.yaml
```

在 deployment.yaml 文件中定义了一个 Deployment 的配置，使用该命令即可创建该 Deployment。

kubectl create 与 kubectl apply 的区别

- kubectl create：用于创建新资源。通常用于初次创建。如果资源已存在，则会提示资源已经存在并返回错误。
- kubectl apply：用于声明式地管理资源。它既可以创建新资源，也可以更新已有资源，非常适合需要多次应用配置文件来管理资源的场景。

其他 kubectl create 子命令

- kubectl create namespace：创建命名空间。

  ```
  kubectl create namespace my-namespace
  ```

- kubectl create job：创建一个 Job。

  ```
  kubectl create job my-job --image=busybox -- echo "Hello, Kubernetes!"
  ```

  这里会创建一个名为 my-job 的 Job，运行一个简单的 echo 命令。

- kubectl create cronjob：创建一个 CronJob。

  ```
  kubectl create cronjob my-cronjob --image=busybox --schedule="*/5 * * * *" -- echo "Hello, every 5 minutes!"
  ```

  这会创建一个名为 my-cronjob 的 CronJob，每 5 分钟执行一次。

使用场景

- 快速创建资源：kubectl create 适用于需要快速创建一个资源的场景，例如临时创建一个 Pod 或 ConfigMap，适合简单的开发和测试用途。
- 脚本化部署：可以通过 kubectl create -f 在脚本中自动创建资源，适用于初次部署时使用。
- 创建 Secret 和 ConfigMap：对于敏感数据的管理，可以使用 kubectl create secret 创建 Secret，或者通过 kubectl create configmap 创建 ConfigMap。

总结

- kubectl create 是 Kubernetes 中用于创建新资源的命令。
- 支持通过命令行直接指定参数创建 Pod、Deployment、Service、ConfigMap、Secret 等资源。
- 也可以通过 -f 参数从 YAML 或 JSON 文件中创建资源。
- 与 kubectl apply 的主要区别是 kubectl create 仅用于初次创建，不能用于更新已有资源。

如果你有具体的场景或者需求，可以告诉我，我可以提供更详细的示例和帮助！




##### get
kubectl get 是 Kubernetes 中最常用的命令之一，用于列出集群中的各种资源及其状态。通过 kubectl get，你可以查看 Pod、Deployment、Service 等资源的详细信息，以帮助你了解集群的状态和资源运行情况。

基本用法

```
kubectl get <resource> [options]
```

resource：资源类型，例如 pods、deployments、services 等。
[options]：用于指定过滤条件或者自定义输出等选项。

示例

1. 获取 Pod 列表

查看所有命名空间中的 Pod 列表：

```
kubectl get pods --all-namespaces
```

只查看当前命名空间中的 Pod 列表：

```
kubectl get pods
```

查看特定 Pod 的详细信息：

```
kubectl get pod nginx-pod
```

查看特定命名空间中的 Pod：

```
kubectl get pods -n my-namespace
```

2. 获取 Deployment 列表

查看所有 Deployment：

```
kubectl get deployments
```

查看特定 Deployment 的详细信息：

```
kubectl get deployment nginx-deployment
```

3. 获取 Service 列表

查看所有 Service：

```
kubectl get services
```

查看特定 Service 的详细信息：

```
kubectl get service my-service
```

4. 使用标签选择器筛选资源

你可以使用标签选择器来筛选资源。例如，查看标签为 app=nginx 的 Pod：

```
kubectl get pods -l app=nginx
```

5. 获取资源的详细输出

可以使用 -o 参数指定不同的输出格式，例如 yaml、json 或 wide：

- -o wide：显示更多详细信息，如 Pod 所在的节点。

  ```
  kubectl get pods -o wide
  ```

- -o yaml：以 YAML 格式输出资源的详细信息。

  ```
  kubectl get deployment nginx-deployment -o yaml
  ```

- -o json：以 JSON 格式输出资源的详细信息。

  ```
  kubectl get service my-service -o json
  ```

- -o name：只显示资源名称。

  ```
  kubectl get pods -o name
  ```

6. 获取特定字段的信息

通过 -o custom-columns 可以自定义输出的字段，例如只查看 Pod 的名称和状态：

```
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

7. 获取所有资源

查看所有资源类型及其实例的列表：

```
kubectl get all
```

该命令将列出所有类型的资源，包括 Pod、Deployment、Service 等。

8. 按时间排序查看 Pod

你可以按创建时间对 Pod 进行排序，查看最早或最新的 Pod：

```
kubectl get pods --sort-by=.metadata.creationTimestamp
```

9. 结合 watch 实时查看资源变化

kubectl get 也可以和 --watch 参数结合使用，实时查看资源的变化情况：

```
kubectl get pods --watch
```

该命令会持续输出 Pod 的状态变化，直到手动停止。

10. 获取特定类型的资源的别名

为了便于使用，Kubernetes 允许资源类型有多个别名。例如：

- Pods：可以使用 pods、pod、po。
- Deployments：可以使用 deployments、deployment、deploy。

常见选项

- -n namespace：指定命名空间，默认是当前命名空间。
- --all-namespaces：列出所有命名空间中的资源。
- -l label-selector：使用标签选择器筛选资源。
- -o output-format：指定输出格式，例如 wide、yaml、json、custom-columns。

使用场景

1. 资源状态监控：通过 kubectl get，你可以快速查看集群中资源的状态，例如 Pod 是否 Running、Deployment 是否达到期望的副本数等。
  
2. 问题排查：结合 -o wide 查看 Pod 所在节点，可以帮助你检查某些节点上的资源分布情况。

3. 脚本和自动化：kubectl get 提供 YAML 和 JSON 格式的输出，可以方便地结合自动化工具或脚本进行进一步处理。

4. 标签管理：使用 -l 标签选择器，可以方便地查看特定应用或者模块的资源情况。

注意事项

1. 资源类型和名称的拼写：资源类型可以使用多种别名，需要注意拼写。例如，deploy、deployment 都可以表示 Deployment。

2. 命名空间的选择：默认情况下，kubectl get 只会列出当前命名空间的资源。需要查看其他命名空间中的资源时，必须使用 -n 或 --all-namespaces。

3. 使用 -o yaml/-o json：当你需要深入了解某个资源的详细信息时，可以使用 -o yaml 或 -o json。这可以帮助你检查具体的配置项或者 debug 集群中的问题。

总结

- kubectl get 是 Kubernetes 中用于查看资源状态和列表的命令。
- 可以查看各种资源类型，例如 Pod、Deployment、Service 等。
- 支持通过 -o 参数选择不同的输出格式，如 yaml、json、wide 等。
- 可以结合标签选择器、命名空间选择、实时监控等功能，方便用户进行集群管理和资源监控。

如果你有具体的场景或想进一步了解某些功能的使用方法，请告诉我，我可以提供更详细的帮助！

##### run
kubectl run 是 Kubernetes 中用于快速启动一个 Pod 的命令，通常用于开发和测试时快速创建容器。kubectl run 可以创建一个简单的 Pod 来运行指定镜像的容器。

基本用法

```
kubectl run <name> --image=<image> [options]
```

name：要创建的 Pod 的名称。
--image=image：要使用的容器镜像。
[options]：用于指定额外的参数。

常见的用法示例

1. 创建一个简单的 Pod

创建一个名为 nginx 的 Pod，使用 nginx 镜像：

```
kubectl run nginx --image=nginx
```

这会在集群中创建一个名为 nginx 的 Pod，并运行 nginx 容器。

2. 创建一个带命令的 Pod

通过 --command 参数来指定 Pod 启动时运行的命令。例如，运行一个 busybox 容器并执行 echo 命令：

```
kubectl run busybox --image=busybox --command -- /bin/sh -c "echo Hello, Kubernetes!"
```

这里会创建一个 busybox Pod，并在启动时运行 echo 命令。

3. 运行交互式 Pod

你可以运行一个交互式 Pod 以进行调试。例如，运行 busybox 并进入 Pod 内的终端：

```
kubectl run -i --tty busybox --image=busybox -- sh
```

- -i：表示交互模式。
- --tty：为容器分配一个 TTY。
- -- sh：启动容器后运行 sh，进入容器的终端。

4. 指定端口

你可以为创建的 Pod 指定容器的端口。例如，运行一个 nginx 容器并暴露 80 端口：

```
kubectl run nginx --image=nginx --port=80
```

请注意，这并不意味着服务将被外部访问，它只是定义了容器中应用监听的端口。

5. 使用环境变量

使用 --env 为 Pod 设置环境变量。例如，创建一个 nginx 容器，并设置环境变量：

```
kubectl run nginx --image=nginx --env="ENV=production"
```

6. 创建 Job 或 CronJob

kubectl run 可以创建一次性任务（Job）或周期性任务（CronJob）。

- 创建一个 Job：

  ```
  kubectl run my-job --image=busybox --restart=OnFailure -- /bin/sh -c "echo Job completed!"
  ```

  使用 --restart=OnFailure 创建一个会重启失败的任务容器。

- 创建一个 CronJob：

  ```
  kubectl run my-cronjob --image=busybox --schedule="*/5 * * * *" -- /bin/sh -c "echo Hello from CronJob"
  ```

  使用 --schedule 创建一个每 5 分钟运行一次的任务。

常见选项

- --image=image：指定要运行的镜像。
- --command：表示后续的命令是要在容器启动时运行的命令。
- --env=key=value：为容器设置环境变量。
- --port=port：指定容器暴露的端口。
- --restart=policy：指定 Pod 的重启策略，如 Never、OnFailure 等。
- --schedule=cron：创建 CronJob 时指定调度计划。
- -i / --tty：用于创建交互式容器，以便用户与容器进行交互。

注意事项

1. 用于临时测试：kubectl run 主要用于创建临时 Pod 进行测试和开发。对于正式环境中的工作负载，推荐使用 kubectl create 或 kubectl apply 结合 YAML 配置文件来声明式管理资源。

2. 不再推荐用于 Deployment 创建：在早期版本中，kubectl run 可以直接创建 Deployment。然而，在新的 Kubernetes 版本中，这种用法已经不推荐使用，建议使用 kubectl create deployment 代替。

3. 重启策略：默认情况下，kubectl run 创建的 Pod 使用 Always 作为重启策略，这意味着如果 Pod 失败，它将会被自动重启。如果想要创建一次性任务，可以将重启策略设置为 Never 或 OnFailure。

使用场景

1. 快速创建 Pod：在开发过程中，可以使用 kubectl run 快速启动一个 Pod 来验证镜像或调试集群中的问题。
  
2. 调试集群问题：使用 kubectl run -i --tty 可以快速创建一个交互式 Pod，用于连接到集群中的网络环境，进行一些测试和排查。

3. 一次性任务：通过 kubectl run 创建 Job 或 CronJob，可以执行一次性任务或定时任务，适用于脚本化处理和自动化任务。

总结

- kubectl run 是 Kubernetes 中用于快速启动 Pod 的命令，适用于测试和调试用途。
- 可以指定镜像、命令、环境变量等来启动容器。
- 适合于临时任务、交互式测试和简单的一次性任务。
- 在新的 Kubernetes 版本中，kubectl run 主要用于创建 Pod，创建 Deployment 建议使用其他方式。

如果你有具体的场景或想了解更多用法，请告诉我，我可以提供更详细的帮助！


##### expose
`kubectl expose` 命令用于将一个已有的 Pod、Deployment、ReplicationController 等资源暴露为一个服务（Service）。这个命令通常用于创建集群内部或外部的服务，使得用户或其他服务能够访问应用程序。

基本用法

```
kubectl expose <resource> <resource-name> [options]
```

- **`<resource>`**：要暴露的资源类型，例如 `pod`、`deployment`、`replicationcontroller` 等。
- **`<resource-name>`**：要暴露的资源名称。
- **`[options]`**：用于指定端口、类型等服务的参数。

常见的用法示例

1. 将 Deployment 暴露为 Service

将名为 `nginx-deployment` 的 Deployment 暴露为 ClusterIP 类型的 Service，监听端口 80：

```
kubectl expose deployment nginx-deployment --port=80
```

这里会自动选择 `nginx-deployment` 中 Pod 的标签来创建一个服务。

2. 指定服务类型

如果希望服务暴露在集群外部，可以使用 `NodePort` 或 `LoadBalancer` 类型的服务：

- 暴露一个 NodePort 类型的 Service：

  ```
  kubectl expose deployment nginx-deployment --type=NodePort --port=80
  ```

  这会为服务分配一个 NodePort，使得可以通过集群节点的 IP 和这个端口来访问服务。

- 暴露一个 LoadBalancer 类型的 Service（适用于云环境）：

  ```
  kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80
  ```

  这会创建一个可以由云提供商的负载均衡器访问的服务。

3. 指定目标端口

你可以为服务指定服务端口（对外暴露）和容器端口（Pod 内的实际端口）：

```
kubectl expose pod nginx --port=8080 --target-port=80
```

- **`--port=8080`**：服务暴露的端口。
- **`--target-port=80`**：Pod 容器中实际监听的端口。

这样，外部访问服务时使用的是 8080 端口，而请求会被转发到 Pod 中的 80 端口。

4. 使用标签选择器创建服务

你也可以通过标签选择器来创建一个服务，使它覆盖某些特定标签的 Pod。例如，将标签为 `app=nginx` 的 Pod 暴露为一个服务：

```
kubectl expose pod -l app=nginx --port=80
```

5. 暴露 ReplicationController

将名为 `my-controller` 的 ReplicationController 暴露为 ClusterIP 类型的 Service：

```
kubectl expose rc my-controller --port=8080 --target-port=80
```

- **`rc`**：表示 ReplicationController。
- **`--port=8080`**：暴露的服务端口。
- **`--target-port=80`**：Pod 的目标端口。

常见选项

- **`--port=<port>`**：指定服务暴露的端口。
- **`--target-port=<port>`**：指定 Pod 中容器的端口。
- **`--type=<type>`**：指定服务类型，可以是 `ClusterIP`、`NodePort`、`LoadBalancer` 等，默认是 `ClusterIP`。
- **`-l <label>`**：通过标签选择器指定需要暴露的资源。
- **`--name=<name>`**：为服务指定一个自定义名称。

使用场景

1. **暴露应用服务**：`kubectl expose` 通常用于将应用暴露为服务，使得它们能够被集群内或集群外的其他组件访问。

2. **开发和测试**：在开发和测试环境中，可以快速使用 `kubectl expose` 创建服务，以便在不编写 YAML 配置文件的情况下，简单地暴露 Pod 或 Deployment。

3. **简化服务创建**：相对于手动编写 Service 的 YAML 文件，`kubectl expose` 提供了一种更加便捷的命令行方式来创建服务。

注意事项

1. **服务类型选择**：使用 `ClusterIP` 服务类型只能在集群内部访问。如果需要外部访问，必须选择 `NodePort` 或 `LoadBalancer`。

2. **端口映射**：`--port` 是服务对外暴露的端口，而 `--target-port` 是容器内部监听的端口，通常两者不同。在不了解具体应用时，需要特别注意端口的配置。

3. **自动标签**：`kubectl expose` 会自动选择被暴露资源的标签来创建服务，这意味着服务会自动选取那些符合标签的 Pod 进行负载均衡。

总结

- `kubectl expose` 用于将已有的 Pod、Deployment、ReplicationController 等资源暴露为服务（Service）。
- 可以选择服务类型（`ClusterIP`、`NodePort`、`LoadBalancer`），以及指定服务端口和容器目标端口。
- 适合于开发、测试及快速暴露服务场景。

##### delete

`kubectl delete` 命令用于删除 Kubernetes 中的资源。通过 `kubectl delete`，你可以删除 Pod、Deployment、Service、ConfigMap 等各种资源，以便对集群中的资源进行管理。

基本用法

```
kubectl delete <resource> <resource-name> [options]
```

- **`<resource>`**：要删除的资源类型，例如 `pod`、`deployment`、`service` 等。
- **`<resource-name>`**：要删除的资源名称。
- **`[options]`**：用于指定命令的额外选项。

常见的用法示例

1. 删除 Pod

删除一个名为 `nginx-pod` 的 Pod：

```
kubectl delete pod nginx-pod
```

删除多个 Pod：

```
kubectl delete pod pod1 pod2 pod3
```

2. 删除 Deployment

删除一个名为 `nginx-deployment` 的 Deployment：

```
kubectl delete deployment nginx-deployment
```

3. 删除 Service

删除一个名为 `my-service` 的 Service：

```
kubectl delete service my-service
```

4. 删除命名空间中的资源

删除特定命名空间下的所有 Pod：

```
kubectl delete pod --all -n my-namespace
```

5. 使用标签选择器删除资源

使用标签选择器来删除匹配某些标签的资源，例如删除标签为 `app=nginx` 的 Pod：

```
kubectl delete pod -l app=nginx
```

6. 从文件中删除资源

你可以通过指定 YAML 或 JSON 文件来删除资源：

```
kubectl delete -f deployment.yaml
```

在 `deployment.yaml` 文件中定义的所有资源都会被删除。

7. 删除所有资源

删除集群中的所有 Pod：

```
kubectl delete pod --all
```

删除所有命名空间中的所有 Pod：

```
kubectl delete pod --all --all-namespaces
```

8. 删除命名空间

删除一个名为 `my-namespace` 的命名空间：

```
kubectl delete namespace my-namespace
```

注意，删除命名空间会同时删除其中的所有资源。

常见选项

- **`-n <namespace>`**：指定命名空间，默认是当前命名空间。
- **`--all`**：删除所有资源，通常结合特定资源类型使用。
- **`-l <label-selector>`**：使用标签选择器来指定要删除的资源。
- **`-f <file>`**：从文件中删除资源。
- **`--grace-period=<seconds>`**：设置优雅关闭的时间，单位为秒。如果设置为 `0`，则立即删除。
- **`--force`**：强制删除资源。

注意事项

1. **删除操作不可逆**：`kubectl delete` 会立即删除指定的资源，因此在执行删除操作之前要谨慎确认。

2. **优雅关闭**：默认情况下，Kubernetes 会在删除资源之前给它们一些时间进行清理（通常为 30 秒）。你可以通过 `--grace-period` 选项来设置这个时间，或者通过 `--force` 强制删除资源而不等待清理。

3. **命名空间的影响**：删除一个命名空间会删除其中的所有资源，因此在操作命名空间时需要特别小心。

使用场景

1. **删除无效或多余的资源**：在开发和测试过程中，有时会创建一些临时的 Pod 或 Deployment，完成后需要通过 `kubectl delete` 来删除它们，以保持集群的干净和资源的高效利用。

2. **集群清理**：通过 `--all` 选项可以快速清理集群中的所有 Pod 或 Deployment，用于重置环境或者在测试之后恢复集群的初始状态。

3. **基于标签的删除**：使用标签选择器可以高效地删除某个特定应用或模块的所有资源，例如删除某个服务所使用的所有 Pod。

总结

- `kubectl delete` 用于删除 Kubernetes 中的各种资源，包括 Pod、Deployment、Service、ConfigMap 等。
- 支持通过标签选择器、文件、命名空间等方式来删除资源。
- 提供优雅关闭和强制删除等选项，帮助用户根据需求安全地删除资源。


#### 进入容器

##### attach
`kubectl attach` 命令用于连接到正在运行的容器，以便实时查看容器的标准输出（stdout）和标准错误输出（stderr），或者与容器中的正在运行的进程交互。它类似于 `kubectl exec`，但 `kubectl attach` 是直接附加到容器的现有进程，而不是启动一个新的命令。

基本用法

```
kubectl attach <pod-name> -c <container-name> [options]
```

- **`<pod-name>`**：要附加的 Pod 的名称。
- **`-c <container-name>`**：指定要附加的容器名称（如果 Pod 中有多个容器）。
- **`[options]`**：用于指定一些附加的选项。

常见的用法示例

1. 附加到 Pod 中的容器

附加到一个名为 `nginx-pod` 的 Pod 中的默认容器：

```
kubectl attach nginx-pod
```

如果 Pod 中只有一个容器，这个命令将附加到该容器上，查看其标准输出。

2. 附加到具有多个容器的 Pod

如果一个 Pod 中包含多个容器，则需要通过 `-c` 指定要附加的容器：

```
kubectl attach nginx-pod -c nginx-container
```

这里 `-c nginx-container` 指定了 Pod 中的具体容器。

3. 附加到交互式进程

你可以使用 `-i` 来与容器中的交互式进程进行交互。例如，如果容器中正在运行一个 `sh` 进程，你可以这样连接：

```
kubectl attach -i nginx-pod
```

- **`-i`**：表示进入交互模式。

注意，这种情况下你必须确保容器中有一个交互式的进程正在运行，比如 shell，否则附加会立即断开。

常见选项

- **`-c <container>`**：指定要附加的容器。如果 Pod 中只有一个容器，则可以省略。
- **`-i`**：进入交互模式，允许与容器中的进程进行交互。
- **`-n <namespace>`**：指定命名空间，默认是当前命名空间。
- **`--tty`**：为容器分配一个 TTY，通常和 `-i` 一起使用以便进行交互。

使用场景

1. **查看日志输出**：`kubectl attach` 可以用来实时查看容器的标准输出和标准错误输出，用于了解容器内部的运行状态。对于那些没有使用 `kubectl logs` 命令的场景，`kubectl attach` 是一个很好的替代方法。

2. **调试容器问题**：当你想要和容器内的某些交互式进程进行交互（例如，查看某些进程的实时状态）时，可以使用 `kubectl attach` 进入容器并进行操作。

注意事项

1. **进程限制**：`kubectl attach` 只能连接到容器内已有的进程，不能启动新的进程。如果你需要启动新的命令或者 shell，应该使用 `kubectl exec`。

2. **交互进程**：如果容器中没有交互式进程（例如，容器内没有运行任何等待输入的进程），使用 `kubectl attach -i` 是没有意义的，因为它无法与没有交互的进程建立连接。

3. **可能中断进程**：在某些情况下，使用 `kubectl attach` 可能会中断容器中正在运行的进程，因此在生产环境中需要谨慎使用，避免影响到正常的业务进程。

4. **用于调试**：通常情况下，`kubectl attach` 更适合用于开发或调试环境，以便实时观察容器内部正在发生的事情。

总结

- `kubectl attach` 用于连接到容器的现有进程，查看其标准输出或进行交互。
- 可以用于查看日志输出、调试容器中运行的进程。
- 适合用于开发和调试场景，但在生产环境中使用时需要谨慎，以避免中断正在运行的进程。

##### auth
`kubectl auth` 是 Kubernetes 中用于检查用户对某些资源或操作的权限的命令。它主要用于帮助用户验证自己或其他主体（如 ServiceAccount）是否具备执行某些操作的权限。

基本用法

```
kubectl auth [command] [options]
```

`kubectl auth` 包含多个子命令，其中最常用的是 `can-i` 子命令，用于检查某个主体是否有权限执行某项操作。

常见的子命令

1. **can-i 子命令**

`kubectl auth can-i` 子命令用于检查某个用户是否有权限在某个资源上执行指定的操作。

基本用法：

```
kubectl auth can-i <verb> <resource> [options]
```

- **`<verb>`**：要检查的操作（例如 `get`、`create`、`delete` 等）。
- **`<resource>`**：要检查的资源（例如 `pods`、`services`、`deployments` 等）。

示例：

1. 检查当前用户是否可以获取 Pods：

```
kubectl auth can-i get pods
```

输出 `yes` 或 `no`，表示当前用户是否具有获取 Pods 的权限。

2. 检查是否可以在特定命名空间创建 Deployment：

```
kubectl auth can-i create deployments -n my-namespace
```

使用 `-n` 指定命名空间。

3. 检查 ServiceAccount 是否有权限删除 Service：

```
kubectl auth can-i delete services --as=system:serviceaccount:my-namespace:my-serviceaccount
```

使用 `--as` 指定某个 ServiceAccount，检查其权限。

4. 检查集群级别的权限：

你可以使用 `--all-namespaces` 来检查集群级别的权限。例如，检查是否可以在所有命名空间删除 Pods：

```
kubectl auth can-i delete pods --all-namespaces
```

常见选项

- **`--as=<user>`**：模拟以指定用户的身份执行检查，用于验证其他用户是否有权限。
- **`-n <namespace>`**：指定要检查的命名空间。
- **`--all-namespaces`**：检查是否具有跨命名空间的权限。
- **`--list`**：列出当前用户或角色对所有资源的权限。例如：

  ```
  kubectl auth can-i --list
  ```

  这会列出当前用户对各种资源的权限，包括哪些操作是允许的。

2. **Reconcile 子命令**

`kubectl auth reconcile` 用于将 RBAC 权限（如 Role 或 ClusterRole）与当前集群中的实际权限进行同步。它通常用于确保 RBAC 配置与 YAML 文件中的配置保持一致。

使用场景

1. **权限检查**：在进行敏感操作（如删除资源、创建服务等）之前，可以使用 `kubectl auth can-i` 检查自己是否有权限，避免权限不足而导致的错误。

2. **调试权限问题**：当某个用户或 ServiceAccount 尝试执行某些操作失败时，可以使用 `kubectl auth can-i` 来检查它是否具有相应的权限，以排查权限问题。

3. **模拟其他用户的权限**：通过 `--as` 选项模拟其他用户的身份，帮助管理员检查某些用户或服务账户的权限，验证权限配置是否正确。

注意事项

1. **权限检查的范围**：默认情况下，`kubectl auth can-i` 会在当前命名空间内进行权限检查。如果想要检查集群级别的权限或者跨命名空间的权限，需要显式地指定命名空间或使用 `--all-namespaces`。

2. **角色绑定的影响**：权限检查与角色绑定（RoleBinding 或 ClusterRoleBinding）密切相关。如果某个用户没有权限，可能需要管理员创建或更新相应的绑定，授予该用户相应的权限。

总结

- `kubectl auth` 命令主要用于检查用户或其他主体对 Kubernetes 资源的权限。
- `can-i` 子命令用于检查是否具有对某个资源执行某项操作的权限，支持模拟其他用户的身份。
- 适用于权限管理和调试，帮助用户快速了解自己或其他主体的权限范围。

如果你有具体的场景或想了解更多用法，请告诉我，我可以提供更详细的帮助！

##### cp
`kubectl cp` 命令用于在本地文件系统和 Kubernetes Pod 中的容器之间复制文件或目录。它类似于 Unix 中的 `cp` 命令，可以用于在本地和集群之间传输文件，方便数据管理和调试。

基本用法

```
kubectl cp <source> <destination> [options]
```

- **`<source>`**：源路径，可以是本地路径，也可以是 Pod 中的路径。
- **`<destination>`**：目标路径，可以是本地路径，也可以是 Pod 中的路径。
- **`[options]`**：用于指定额外选项。

常见的用法示例

1. 从本地复制文件到 Pod

将本地文件 `localfile.txt` 复制到名为 `nginx-pod` 的 Pod 中的 `/tmp` 目录：

```
kubectl cp localfile.txt nginx-pod:/tmp/
```

2. 从 Pod 复制文件到本地

将 Pod 中 `/tmp/podfile.txt` 复制到本地当前目录：

```
kubectl cp nginx-pod:/tmp/podfile.txt .
```

3. 复制整个目录

你也可以复制整个目录。将本地目录 `mydir` 复制到 Pod 中的 `/tmp` 目录：

```
kubectl cp mydir nginx-pod:/tmp/
```

或者将 Pod 中的整个目录 `/tmp/mydir` 复制到本地：

```
kubectl cp nginx-pod:/tmp/mydir ./localdir/
```

4. 指定容器

如果 Pod 中有多个容器，你可以使用 `-c` 指定要操作的容器。例如，复制文件到容器 `my-container` 中：

```
kubectl cp localfile.txt nginx-pod:/tmp/ -c my-container
```

常见选项

- **`-c <container>`**：指定要复制文件到的容器名称（如果 Pod 中有多个容器）。
- **`-n <namespace>`**：指定命名空间，默认是当前命名空间。

使用场景

1. **调试和测试**：`kubectl cp` 常用于在集群中调试和测试时传输文件，例如将配置文件复制到 Pod 中，或者从 Pod 中获取日志文件进行分析。

2. **临时文件管理**：当你需要在容器中处理一些临时文件时，可以通过 `kubectl cp` 方便地将文件传输到 Pod 中，进行处理后再复制回来。

3. **配置文件更新**：在不重新部署的情况下更新容器中的某些配置文件，`kubectl cp` 提供了一种简单的方法。

注意事项

1. **权限要求**：执行 `kubectl cp` 时，用户需要对目标 Pod 拥有足够的权限，包括读取和写入文件的权限。

2. **文件覆盖**：如果目标路径已经存在相同名称的文件，`kubectl cp` 会覆盖该文件。因此在使用时需要小心，以避免误操作。

3. **容器路径**：在指定 Pod 内部的文件路径时，需要注意路径的正确性。例如，使用相对路径时需要根据容器的工作目录设置。

4. **性能限制**：`kubectl cp` 的底层实现依赖于 `tar` 命令，因此在传输大文件时性能可能会受到一定限制。在需要高效传输大量数据时，可能需要使用其他方式。

总结

- `kubectl cp` 用于在本地和 Kubernetes Pod 之间复制文件或目录。
- 支持双向传输，可以从本地复制到 Pod，也可以从 Pod 复制到本地。
- 可通过 `-c` 指定特定容器，通过 `-n` 指定命名空间。
- 适用于调试、测试、文件管理和配置更新等场景。

c##### describe


##### exec
`kubectl exec` 命令用于在 Kubernetes 集群中的某个 Pod 内执行命令或脚本。它是用于调试和管理 Pod 中的容器的常用命令之一，可以帮助你直接访问容器内部，查看系统状态、修改配置或者运行诊断命令。

基本用法

```
kubectl exec <pod-name> -- <command> [args] [options]
```

- **`<pod-name>`**：要执行命令的 Pod 的名称。
- **`<command>`**：要在 Pod 中执行的命令，例如 `/bin/sh`。
- **`[args]`**：传递给命令的参数。
- **`[options]`**：用于指定其他选项，如命名空间或容器。

常见的用法示例

1. 在 Pod 内执行命令

执行命令来查看 Pod 中的文件系统，例如列出 `/tmp` 目录的内容：

```
kubectl exec nginx-pod -- ls /tmp
```

`--` 后面的部分是要执行的命令。在这个例子中，`ls /tmp` 将在名为 `nginx-pod` 的 Pod 内执行。

2. 进入 Pod 内的 shell

如果你想进入 Pod 的交互式 shell，可以使用 `sh` 或 `bash`：

```
kubectl exec -it nginx-pod -- /bin/sh
```

- **`-i`**：进入交互模式，允许你输入命令。
- **`-t`**：分配一个 TTY，便于交互。

如果容器支持 `bash`，你也可以使用 `bash`：

```
kubectl exec -it nginx-pod -- /bin/bash
```

3. 指定容器

如果 Pod 中有多个容器，你需要使用 `-c` 指定容器的名称：

```
kubectl exec nginx-pod -c nginx-container -- ls /tmp
```

这里的 `-c nginx-container` 用于指定 Pod 中的具体容器。

4. 执行复杂命令

你可以执行任何容器中存在的命令，例如查看正在运行的进程：

```
kubectl exec nginx-pod -- ps aux
```

5. 从文件执行命令

你可以通过执行复杂命令来检查或更改容器状态，例如创建一个文件：

```
kubectl exec nginx-pod -- touch /tmp/newfile.txt
```

常见选项

- **`-c <container>`**：指定要在 Pod 中运行命令的容器名称（如果 Pod 中有多个容器）。
- **`-i`**：进入交互模式，允许用户与容器内部进程交互。
- **`-t`**：分配一个 TTY，这样你可以使用类似于命令行的交互环境。
- **`-n <namespace>`**：指定 Pod 所在的命名空间。

使用场景

1. **调试 Pod**：`kubectl exec` 是调试 Pod 的重要工具，你可以直接进入 Pod 内部，查看系统日志、检查网络状态，执行诊断工具（如 `netstat`、`top`、`ping` 等），从而排查和解决问题。

2. **运行一次性命令**：当你需要在 Pod 中运行一次性任务，例如清除缓存文件，修改配置等，使用 `kubectl exec` 可以很方便地完成。

3. **检查应用程序状态**：你可以执行应用程序的命令来检查它的运行状态，例如查看日志文件、数据库状态等。

4. **动态配置**：在 Pod 内部，可能需要动态地修改一些配置文件，或者运行一些脚本来测试新的配置。

注意事项

1. **仅适用于运行中的 Pod**：`kubectl exec` 只能用于状态为 `Running` 的 Pod，如果 Pod 处于其他状态（例如 `Pending` 或 `CrashLoopBackOff`），则无法使用 `kubectl exec`。

2. **命令不可用**：你需要确认容器镜像中包含你要执行的命令，例如某些极简的镜像（如 `alpine`）中可能没有 `/bin/bash`，这种情况下需要用其他可用的 shell。

3. **交互式命令的使用**：如果你需要与 Pod 内部交互，例如使用 shell，需要加上 `-it` 参数，这样会为你分配一个 TTY，以便进行命令交互。

4. **指定容器**：如果 Pod 内有多个容器，必须使用 `-c` 指定具体的容器，否则 Kubernetes 无法知道你要在哪个容器中执行命令。

总结

- `kubectl exec` 是 Kubernetes 中用于在 Pod 内执行命令的命令。
- 可以执行单次命令，进入交互式 shell 或修改容器内部配置。
- 可以结合 `-it` 进入交互模式，或者使用 `-c` 指定容器。
- 适用于调试、运行一次性任务、检查应用状态等场景。

##### logs

`kubectl logs` 命令用于查看 Kubernetes 集群中 Pod 的日志输出。Pod 的日志是调试和监控应用的重要工具，能够帮助你了解应用的运行状态、错误信息以及其他输出内容。

基本用法

```
kubectl logs <pod-name> [options]
```

- **`<pod-name>`**：要查看日志的 Pod 的名称。
- **`[options]`**：用于指定额外的参数，例如命名空间、容器名称等。

常见的用法示例

1. 查看单个 Pod 的日志

查看名为 `nginx-pod` 的 Pod 的日志：

```
kubectl logs nginx-pod
```

这会显示 `nginx-pod` 中默认容器的所有日志输出。

2. 查看具有多个容器的 Pod 的日志

如果 Pod 中有多个容器，需要通过 `-c` 参数指定要查看日志的容器：

```
kubectl logs nginx-pod -c nginx-container
```

这里的 `-c nginx-container` 用于指定 Pod 中的具体容器。

3. 查看最近的日志

你可以通过 `--tail` 参数来指定查看日志的行数。例如，只查看最近的 10 行日志：

```
kubectl logs nginx-pod --tail=10
```

4. 实时查看日志

你可以通过 `-f` 参数来实时查看日志输出，类似于 Linux 中的 `tail -f`：

```
kubectl logs -f nginx-pod
```

当 Pod 产生新日志时，输出会自动更新，非常适合用于调试和监控应用。

5. 查看所有容器的日志

如果一个 Pod 中有多个容器，并且你希望查看所有容器的日志，可以使用 `--all-containers` 参数：

```
kubectl logs nginx-pod --all-containers=true
```

6. 查看特定时间范围内的日志

通过 `--since` 或 `--since-time` 参数可以查看特定时间范围内的日志：

- 查看最近 5 分钟的日志：

  ```
  kubectl logs nginx-pod --since=5m
  ```

- 查看某个特定时间之后的日志（时间格式为 RFC3339）：

  ```
  kubectl logs nginx-pod --since-time="2024-09-25T14:00:00Z"
  ```

7. 查看特定命名空间中的日志

如果 Pod 位于特定命名空间，可以通过 `-n` 参数指定命名空间：

```
kubectl logs nginx-pod -n my-namespace
```

8. 查看以前的日志

如果 Pod 被重新启动过，可以通过 `--previous` 参数查看它在上一次运行时的日志：

```
kubectl logs nginx-pod --previous
```

这对于分析应用由于某些原因崩溃后重新启动的情况非常有用。

常见选项

- **`-c <container>`**：指定容器名称，适用于 Pod 中有多个容器的情况。
- **`-n <namespace>`**：指定 Pod 所在的命名空间。
- **`-f`**：实时查看日志输出（类似于 `tail -f`）。
- **`--tail=<lines>`**：查看日志的最后几行。
- **`--since=<duration>`**：查看指定时间段内的日志，例如 `10s`、`5m`、`1h` 等。
- **`--since-time=<timestamp>`**：查看从指定时间开始的日志。
- **`--all-containers=true`**：查看 Pod 中所有容器的日志。
- **`--previous`**：查看上一个容器实例的日志，适用于崩溃或重启的情况。

使用场景

1. **调试应用**：`kubectl logs` 是调试应用的重要工具，可以帮助开发者查看应用的标准输出和标准错误日志，分析问题和错误。

2. **监控应用状态**：可以实时查看应用的日志输出，了解应用的运行状态和处理请求的情况。

3. **分析应用崩溃原因**：在 Pod 崩溃后重新启动的情况下，通过 `--previous` 参数查看上次运行时的日志，可以帮助分析导致 Pod 崩溃的原因。

4. **实时查看日志输出**：通过 `-f` 参数实时查看应用日志，适合于开发调试和问题追踪。

注意事项

1. **日志存储时间有限**：Kubernetes 默认只保留容器的当前日志，如果容器被重新创建或日志轮换，旧日志会丢失。为了长期存储日志，可以使用集中化日志管理工具，如 Fluentd、ELK Stack 等。

2. **多容器 Pod**：对于包含多个容器的 Pod，需要使用 `-c` 参数指定容器名称，否则 Kubernetes 无法确定要查看哪个容器的日志。

3. **实时日志限制**：当实时查看日志时，如果网络连接中断或者 Pod 被重新调度，实时日志可能会中断，需要重新执行 `kubectl logs -f` 命令。

总结

- `kubectl logs` 用于查看 Kubernetes 中 Pod 的日志输出。
- 可以查看单个 Pod、多个容器、实时日志、历史日志等。
- 支持通过命名空间、时间范围等多种方式过滤日志。
- 适用于调试、监控和分析应用状态。

##### port-forward
`kubectl port-forward` 命令用于将本地计算机上的端口转发到 Kubernetes 集群中的 Pod 或 Service，从而使得你可以在本地访问集群内部的资源。它常用于调试和开发场景，特别是当你想从本地访问集群内部的服务，而不想创建负载均衡器或 NodePort 类型的 Service 时。

基本用法

```
kubectl port-forward <resource> <local-port>:<pod-port> [options]
```

- **`<resource>`**：可以是 Pod 或 Service 的名称。
- **`<local-port>`**：本地计算机上的端口号。
- **`<pod-port>`**：Pod 内部或者服务的端口号。
- **`[options]`**：指定其他的参数。

常见的用法示例

1. 将本地端口转发到 Pod 中的端口

将本地的 `8080` 端口转发到名为 `nginx-pod` 的 Pod 的 `80` 端口：

```
kubectl port-forward nginx-pod 8080:80
```

这样，你可以在浏览器中通过 `http://localhost:8080` 访问 `nginx-pod` 的 `80` 端口。

2. 将本地端口转发到 Service

你也可以将本地端口转发到 Service 中，而不是直接转发到 Pod。例如，将本地的 `9090` 端口转发到名为 `nginx-service` 的 Service 的 `80` 端口：

```
kubectl port-forward service/nginx-service 9090:80
```

通过 `http://localhost:9090`，你可以访问该 Service 暴露的端口。

3. 在特定命名空间中使用

如果 Pod 或 Service 位于特定命名空间中，你可以使用 `-n` 来指定命名空间。例如，将本地端口 `3000` 转发到 `my-namespace` 中名为 `my-app` 的 Pod 的 `8080` 端口：

```
kubectl port-forward -n my-namespace my-app 3000:8080
```

4. 多端口转发

你可以同时转发多个端口。例如，将本地的 `7000` 和 `8000` 端口分别转发到 Pod 的 `7000` 和 `8000` 端口：

```
kubectl port-forward nginx-pod 7000:7000 8000:8000
```

常见选项

- **`-n <namespace>`**：指定 Pod 或 Service 所在的命名空间。
- **`--address=<address>`**：默认情况下，端口转发只绑定到 `localhost`。你可以使用 `--address` 指定绑定的地址，例如 `0.0.0.0`，允许来自其他机器的连接：

  ```
  kubectl port-forward nginx-pod 8080:80 --address=0.0.0.0
  ```

  这样，本地网络中的其他计算机也可以通过 `http://<your-local-ip>:8080` 访问这个端口。

- **`--pod-running-timeout=<duration>`**：等待 Pod 处于 `Running` 状态的时间，默认是 1 分钟。例如，设置为 2 分钟：

  ```
  kubectl port-forward nginx-pod 8080:80 --pod-running-timeout=2m
  ```

使用场景

1. **本地调试服务**：`kubectl port-forward` 常用于从本地访问集群中的服务，特别是在开发和调试阶段。你可以在本地通过浏览器或工具直接访问集群中的服务，调试应用程序的接口和功能。

2. **无需暴露服务**：当你不希望将服务暴露给集群外部（如通过 `NodePort` 或 `LoadBalancer`），而只希望临时访问它时，`kubectl port-forward` 是一种安全的选择。

3. **数据库访问**：开发者经常使用 `kubectl port-forward` 来访问集群内的数据库，例如 MySQL、MongoDB 等。通过端口转发，可以用本地的数据库客户端连接到集群内的数据库进行查询和调试。例如，访问集群内的 MongoDB：

   ```
   kubectl port-forward mongo-pod 27017:27017
   ```

   这样，你可以使用本地的 MongoDB 客户端连接 `localhost:27017`。

4. **安全地访问服务**：`kubectl port-forward` 提供了一种无需暴露服务的方式，让你可以安全地访问集群中的应用，而不用担心潜在的安全风险。

注意事项

1. **端口冲突**：确保本地端口没有被占用，否则会导致端口转发失败。

2. **仅限本地访问**：默认情况下，端口转发只允许本地访问。如果需要其他机器访问，需要使用 `--address` 参数绑定到 `0.0.0.0` 或具体的 IP 地址。

3. **性能问题**：端口转发的性能受限于 `kubectl` 的连接速度和可靠性，通常不适用于高负载的生产场景，只推荐在开发和调试环境中使用。

4. **连接稳定性**：如果 Kubernetes API Server 出现不稳定或者网络连接中断，端口转发会被终止，因此要确保网络连接稳定。

总结

- `kubectl port-forward` 用于将本地端口转发到集群中 Pod 或 Service 的端口，方便在本地访问集群中的资源。
- 支持指定端口、命名空间和绑定地址等选项。
- 适用于开发和调试场景，尤其是在无需暴露服务的情况下安全地访问集群中的资源。

##### proxy
`kubectl proxy` 命令用于启动一个代理服务器，代理 Kubernetes API Server。它的主要作用是使你能够通过本地的 `kubectl` 访问 Kubernetes API，无需进行复杂的身份认证。通过 `kubectl proxy`，你可以在本地访问 Kubernetes API 或者一些服务的 UI 页面，例如 Dashboard。

基本用法

```
kubectl proxy [options]
```

当运行 `kubectl proxy` 时，它会在本地启动一个代理服务器，你可以通过本地的 HTTP 请求来访问 Kubernetes API 服务器。

常见的用法示例

1. 启动代理

默认启动一个监听在本地 `127.0.0.1:8001` 的代理服务器：

```
kubectl proxy
```

此时可以通过浏览器访问 `http://127.0.0.1:8001/` 来与 Kubernetes API Server 交互。

2. 绑定到特定的地址

你可以使用 `--address` 参数来指定代理绑定的地址，例如绑定到所有可用的网络接口（`0.0.0.0`），允许其他设备访问这个代理：

```
kubectl proxy --address=0.0.0.0 --port=8080
```

这样本地网络中的其他设备也可以通过 `http://<your-ip>:8080/` 访问代理。

3. 指定监听端口

你可以通过 `--port` 指定代理监听的端口，例如监听在 `9090` 端口：

```
kubectl proxy --port=9090
```

此时可以通过 `http://127.0.0.1:9090/` 访问代理。

4. 指定 API 前缀

可以通过 `--api-prefix` 参数修改代理暴露的 API 前缀，例如只代理特定的 API 前缀：

```
kubectl proxy --api-prefix=/custom/
```

此时，代理的 API 访问路径将变为 `http://127.0.0.1:8001/custom/`。

5. 禁用过滤器

默认情况下，`kubectl proxy` 只允许访问安全的 API 路径。你可以使用 `--disable-filter` 参数来禁用这种安全过滤器，允许访问所有 API：

```
kubectl proxy --disable-filter=true
```

常见选项

- **`--address=<address>`**：指定代理绑定的地址，默认是 `127.0.0.1`，只能本地访问。如果设置为 `0.0.0.0`，则可以通过网络中的其他设备访问代理。
- **`--port=<port>`**：指定代理监听的端口，默认是 `8001`。
- **`--accept-hosts=<regex>`**：指定可以访问代理的主机名的正则表达式。
- **`--accept-paths=<regex>`**：指定可以访问的 API 路径的正则表达式。
- **`--api-prefix=<prefix>`**：指定 API 访问的前缀，默认是 `/`.
- **`--disable-filter=true`**：禁用默认的安全过滤器，允许访问所有 API。

使用场景

1. **访问 Kubernetes API**：`kubectl proxy` 主要用于从本地访问 Kubernetes API，例如调试和开发时，你可以通过代理来测试对 API 的请求。

2. **访问服务 UI**：可以通过代理访问 Kubernetes 集群中的某些服务的 Web 界面，例如 Kubernetes Dashboard。在不暴露 Service 的情况下，使用 `kubectl proxy` 来安全访问 UI。

3. **编写自定义工具**：开发自定义工具时，`kubectl proxy` 可以用来代理 API Server，使工具无需处理复杂的身份认证问题，通过简单的 HTTP 请求就可以与 API Server 交互。

注意事项

1. **安全性**：`kubectl proxy` 默认只允许本地访问，以确保安全性。如果绑定到 `0.0.0.0`，则需要小心，避免将 API Server 暴露给不安全的网络。

2. **API 限制**：`kubectl proxy` 默认只允许访问一部分安全的 API。如果你需要更全面的访问权限，可以使用 `--disable-filter`，但这样做可能带来安全风险。

3. **临时用途**：`kubectl proxy` 更适合临时使用，特别是在开发和调试阶段。如果需要长期使用，应考虑通过 `Ingress` 或 `Service` 来暴露服务，而不是使用代理。

4. **长连接**：`kubectl proxy` 依赖于本地的长连接，因此在本地代理终止时，代理访问也会终止。

总结

- `kubectl proxy` 用于启动一个本地代理来访问 Kubernetes API Server。
- 通过代理，可以在本地安全地访问 Kubernetes API 或服务的 UI 页面。
- 支持自定义地址、端口、API 前缀等选项。
- 适用于开发、调试和临时访问集群资源的场景。

##### top
`kubectl top` 命令用于查看 Kubernetes 集群中资源（如 Pod 和 Node）的资源使用情况（CPU 和内存）。它类似于 Linux 系统中的 `top` 命令，能够帮助你实时监控集群中资源的使用，从而有效地管理和调试集群。

基本用法

```
kubectl top [command] [options]
```

常见的子命令是 `kubectl top nodes` 和 `kubectl top pods`，它们用于查看节点和 Pod 的资源使用情况。

常见的用法示例

1. 查看节点的资源使用情况

使用 `kubectl top nodes` 查看集群中各个节点的 CPU 和内存使用情况：

```
kubectl top nodes
```

输出内容包括节点的名称、CPU 使用量、CPU 可用百分比、内存使用量和内存可用百分比。示例输出如下：

```
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node1           500m         25%    1Gi             50%
node2           600m         30%    1.5Gi           75%
```

2. 查看 Pod 的资源使用情况

使用 `kubectl top pods` 查看各个 Pod 的 CPU 和内存使用情况：

```
kubectl top pods
```

这会显示当前命名空间中每个 Pod 的资源使用情况，包括 Pod 名称、CPU 使用量和内存使用量。

3. 查看特定命名空间的 Pod 资源使用情况

使用 `-n` 参数指定命名空间，查看该命名空间中的所有 Pod 的资源使用情况：

```
kubectl top pods -n my-namespace
```

4. 查看特定 Pod 的资源使用情况

你也可以通过 `kubectl top pods` 并指定 Pod 名称来查看特定 Pod 的资源使用：

```
kubectl top pod my-pod
```

5. 查看资源使用情况的详细信息

可以使用 `--containers` 参数查看 Pod 中各个容器的资源使用情况：

```
kubectl top pod my-pod --containers
```

这样你可以看到 Pod 内的每个容器的 CPU 和内存使用情况。

常见选项

- **`-n <namespace>`**：指定要查看的命名空间，默认是当前命名空间。
- **`--containers`**：显示 Pod 中各个容器的资源使用情况。
- **`--sort-by=<resource>`**：按资源排序，可以按 `cpu` 或 `memory` 排序。例如：

  ```
  kubectl top pods --sort-by=cpu
  ```

  这会按照 CPU 使用量排序显示 Pod。

使用场景

1. **监控集群健康状态**：通过 `kubectl top nodes` 可以快速查看集群中每个节点的资源使用情况，帮助你了解集群的健康状况以及各个节点的负载情况。

2. **管理应用的资源使用**：通过 `kubectl top pods` 查看各个 Pod 的 CPU 和内存使用量，帮助你发现那些消耗大量资源的 Pod，从而进行适当的资源分配或优化应用。

3. **调试性能问题**：当集群中某些应用出现性能问题时，可以使用 `kubectl top` 来查看这些应用的资源使用情况，帮助排查是否存在资源瓶颈，例如 CPU 或内存不足。

4. **按需扩缩容**：根据 Pod 的资源使用情况，可以判断是否需要扩展或缩减某些 Deployment 的副本数量，以优化资源利用率。

注意事项

1. **需要 Metrics Server**：`kubectl top` 命令依赖于 Metrics Server 或其他指标收集工具来获取资源使用数据。因此，在使用 `kubectl top` 之前，集群中必须部署 Metrics Server。

2. **资源单位**：`kubectl top` 中的 CPU 使用量以 `millicores` 为单位（例如，`500m` 表示 0.5 个 CPU），而内存使用量以字节（`Mi`、`Gi`）为单位。

3. **权限**：你需要具备足够的权限来访问节点或 Pod 的资源使用数据。通常需要具备 `view` 或更高的角色权限。

总结

- `kubectl top` 用于查看 Kubernetes 集群中节点和 Pod 的资源使用情况，包括 CPU 和内存。
- 支持查看特定命名空间、特定 Pod，或者查看各容器的资源使用。
- 适用于监控集群状态、调试性能问题、管理资源使用等场景。
- 依赖 Metrics Server，因此需要在集群中安装 Metrics Server 才能使用。

#### 集群管理


###### api-versions 
`kubectl api-versions` 命令用于列出 Kubernetes API 服务器中可用的 API 组和版本。它是了解集群中支持的 API 资源以及对应的版本信息的有用工具，特别是在你想确定某个资源是否可用或想了解它使用的 API 版本时。

基本用法

```
kubectl api-versions
```

执行此命令后，会列出集群中支持的所有 API 组和它们的版本。

常见的用法示例

1. 列出所有可用的 API 版本

运行以下命令来列出集群中可用的 API 组和版本：

```
kubectl api-versions
```

输出内容通常类似于：

```
admissionregistration.k8s.io/v1
apps/v1
authentication.k8s.io/v1
authorization.k8s.io/v1
autoscaling/v1
batch/v1
certificates.k8s.io/v1
networking.k8s.io/v1
policy/v1
rbac.authorization.k8s.io/v1
...
```

每行输出包含一个 API 组名和对应的版本号。

2. 查看某个特定 API 组是否可用

你可以使用 `kubectl api-versions` 列出的信息，来确定某个特定的 API 组和版本是否可用。例如，如果你想使用 `HorizontalPodAutoscaler`，你可以检查是否有 `autoscaling/v1` 或 `autoscaling/v2` 版本。

3. 结合 `kubectl api-resources` 使用

当你想了解具体的资源及其详细版本信息时，可以结合 `kubectl api-resources` 使用。例如，先用 `kubectl api-versions` 确定有哪些 API 组和版本可用，然后使用 `kubectl api-resources` 了解具体资源及其 API 组。

常见选项

- **`-v=<verbosity>`**：设置命令的详细信息级别，以帮助调试或获取更多的输出信息。例如，使用 `-v=8` 可以获取请求的详细信息。

使用场景

1. **了解集群 API 版本支持情况**：在 Kubernetes 集群中可能同时存在多个 API 组和不同的版本，通过 `kubectl api-versions` 可以快速了解集群中支持的 API 版本，以便编写兼容的资源定义文件。

2. **验证集群兼容性**：当你使用 Helm charts、CRDs（自定义资源定义）或应用清单时，可以通过 `kubectl api-versions` 检查当前集群是否支持它们使用的 API 版本，避免不兼容问题。

3. **升级集群时使用**：在 Kubernetes 版本升级过程中，某些 API 版本可能会被弃用（deprecated）或移除。你可以使用 `kubectl api-versions` 检查集群当前支持的 API 版本，确保所使用的资源配置文件与集群兼容。

注意事项

1. **输出格式**：`kubectl api-versions` 的输出为每个 API 组的可用版本，因此可能包含多个版本，如 `v1`, `v1beta1` 等，需要根据应用情况选择合适的版本。

2. **与资源 API 结合使用**：列出的 API 版本中，仅显示集群当前支持的版本。某些较新的或自定义的 API 可能不包含在内，因此使用 CRDs 或其他自定义资源时，需确保相应 API 已被注册到集群中。

总结

- `kubectl api-versions` 列出集群中所有可用的 API 组和版本。
- 有助于了解集群的 API 支持情况，特别是对于兼容性和 API 可用性验证。
- 在编写资源清单文件、验证兼容性或进行集群升级时非常有用。



###### certificate 

`kubectl certificate` 命令用于管理 Kubernetes 中的证书签名请求 (CSR, Certificate Signing Requests)。它可以批准或拒绝待处理的证书请求，是管理集群安全通信的重要工具之一。

在 Kubernetes 中，证书签名请求通常用于让集群内的组件或用户获取合法的 SSL 证书，以便安全地与 API Server 或其他服务通信。

基本用法

```
kubectl certificate [approve|deny] <csr-name> [options]
```

- **`approve`**：批准一个证书签名请求。
- **`deny`**：拒绝一个证书签名请求。
- **`<csr-name>`**：证书签名请求的名称。

常见的用法示例

1. 查看所有证书签名请求

使用 `kubectl get csr` 可以查看集群中所有的证书签名请求：

```
kubectl get csr
```

这会列出所有的证书签名请求，包括其状态（如 `Pending`、`Approved`、`Denied`）。

2. 批准证书签名请求

批准名为 `my-csr` 的证书签名请求：

```
kubectl certificate approve my-csr
```

这样，CSR 的状态会变为 `Approved`，并且将会向请求者发放证书。

3. 拒绝证书签名请求

拒绝名为 `my-csr` 的证书签名请求：

```
kubectl certificate deny my-csr
```

这会将 CSR 的状态设置为 `Denied`，表示不允许此请求者获取证书。

4. 查看证书签名请求的详细信息

要查看某个 CSR 的详细信息，可以使用 `kubectl describe`：

```
kubectl describe csr my-csr
```

这会显示证书请求的详细信息，例如请求者信息、证书的详细内容等。

常见选项

- **`-n <namespace>`**：指定命名空间，不过 CSR 通常是集群范围的资源，因此通常不需要指定命名空间。
- **`--kubeconfig`**：指定要使用的 kubeconfig 文件，用于连接到特定集群进行操作。

使用场景

1. **自动化证书管理**：在 Kubernetes 中，各种组件之间通常需要通过 TLS 证书进行安全通信。`kubectl certificate` 可以用于批准 CSR，帮助自动化管理集群中的证书签名过程。

2. **自定义组件请求证书**：当自定义组件需要与 Kubernetes API Server 安全通信时，可以使用 CSR 请求证书，并使用 `kubectl certificate approve` 批准请求，以便让组件获得合法的 TLS 证书。

3. **集群组件的证书续期**：集群中可能会有一些组件需要续期证书，可以通过创建 CSR 并使用 `kubectl certificate` 来批准，达到续期的目的。

注意事项

1. **权限要求**：批准或拒绝证书请求需要管理员权限，通常需要拥有 `certificates.k8s.io` API 组的相应权限。

2. **安全性**：对证书签名请求的批准操作要谨慎，确保请求者是受信任的，并且请求是合法的。批准不受信任的请求可能会导致集群安全问题。

3. **证书有效期**：批准的证书有一定的有效期，到期后需要重新生成并批准新的证书请求。

总结

- `kubectl certificate` 用于管理 Kubernetes 中的证书签名请求，包括批准和拒绝请求。
- 使用 `approve` 和 `deny` 子命令对证书请求进行处理。
- 在自动化证书管理、安全通信和证书续期等场景中非常有用。
- 批准证书请求需要具备足够的管理员权限，操作时需要特别注意安全性。


###### cluster-info 
`kubectl cluster-info` 命令用于查看 Kubernetes 集群的基本信息和关键组件的地址。它是快速了解集群状态和找到关键服务（如 API Server、DNS、控制器等）信息的有用工具，特别是在调试和管理集群时。

基本用法

```
kubectl cluster-info
```

此命令会返回关于集群核心组件的详细信息，包括 API Server 的地址、DNS 组件的地址等。例如：

```
Kubernetes control plane is running at https://<api-server-ip>:<port>
CoreDNS is running at https://<api-server-ip>:<port>/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

常见的用法示例

1. 查看集群的基本信息

运行 `kubectl cluster-info` 来查看集群的控制平面和核心服务的地址：

```
kubectl cluster-info
```

你会得到集群控制平面组件（例如 Kubernetes API Server）和其他重要组件（例如 DNS 服务）的地址。

2. 打开 Kubernetes Dashboard

在早期的 Kubernetes 版本中，如果集群中部署了 Dashboard，你可以使用 `kubectl cluster-info` 来获取 Dashboard 的地址，并在浏览器中访问它。

```
kubectl cluster-info
```

输出中可能包含类似这样的信息：

```
Kubernetes Dashboard is running at https://<api-server-ip>/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

你可以将该地址粘贴到浏览器中来访问 Dashboard。

3. 查看特定集群的信息

如果你管理多个 Kubernetes 集群，可以使用 `--context` 参数指定集群上下文，来查看特定集群的信息：

```
kubectl cluster-info --context=my-cluster
```

这会显示名为 `my-cluster` 的集群的控制平面和核心组件的信息。

常见选项

- **`--context=<context>`**：指定集群上下文，以便查看特定集群的信息。
- **`--kubeconfig=<file>`**：指定 kubeconfig 文件，适用于多集群环境下使用不同的配置文件。

使用场景

1. **快速定位集群核心组件**：`kubectl cluster-info` 是了解集群运行状态的快捷方式，可以帮助你快速找到集群的控制平面（如 API Server）和其他关键服务的地址，以便进一步调试和管理。

2. **排查集群状态**：当你遇到无法访问集群或其他连接问题时，可以使用 `kubectl cluster-info` 来检查控制平面和服务的状态，确保 API Server 等组件正常工作。

3. **确认集群环境**：在管理多个 Kubernetes 集群时，使用 `kubectl cluster-info` 可以确认当前连接的是哪个集群，并查看该集群的基本组件信息。

注意事项

1. **依赖于集群访问权限**：`kubectl cluster-info` 依赖于你对集群的访问权限。如果你没有足够的权限，可能无法获取集群的相关信息。

2. **只能查看基本信息**：`kubectl cluster-info` 只能显示集群的一些基本组件的信息，例如 API Server 和 DNS 的地址。对于更详细的状态和监控信息，可能需要使用其他工具，例如 `kubectl get`、`kubectl describe` 或 Prometheus/Grafana 等。

总结

- `kubectl cluster-info` 用于查看 Kubernetes 集群的基本信息，特别是控制平面和关键组件的地址。
- 支持通过 `--context` 和 `--kubeconfig` 查看特定集群的信息。
- 适用于调试、确认集群状态和查找集群关键服务的地址。
###### cordon
`kubectl cordon` 命令用于将 Kubernetes 集群中的节点标记为不可调度状态。这样可以防止新的 Pod 被调度到该节点上，但不会影响已经运行的 Pod。这在进行节点维护或升级时非常有用。

基本用法

```
kubectl cordon <node-name> [options]
```

- **`<node-name>`**：要标记为不可调度的节点名称。

常见的用法示例

1. 标记节点为不可调度

将节点 `node1` 标记为不可调度状态：

```
kubectl cordon node1
```

输出类似于：

```
node/node1 cordoned
```

此时，`node1` 节点将被标记为不可调度，新创建的 Pod 将不会被调度到该节点上，但已在该节点上运行的 Pod 不会受到影响。

2. 查看节点状态

可以使用 `kubectl get nodes` 来查看节点的调度状态。在节点状态列中，如果节点被标记为不可调度，状态会显示 `SchedulingDisabled`：

```
kubectl get nodes
```

输出示例：

```
NAME    STATUS                     ROLES    AGE   VERSION
node1   Ready,SchedulingDisabled   <role>   ...   ...
node2   Ready                      <role>   ...   ...
```

3. 使用 `kubectl describe` 查看节点详细信息

要查看节点的详细状态信息，可以使用 `kubectl describe` 命令。查看节点 `node1` 的状态：

```
kubectl describe node node1
```

在输出中，你会看到类似 `Taints: node.kubernetes.io/unschedulable:NoSchedule` 的信息，表示该节点已被标记为不可调度。

常见选项

- **`--context=<context>`**：指定集群上下文，用于管理多个集群。
- **`--kubeconfig=<file>`**：指定 kubeconfig 文件，适用于多集群环境下使用不同的配置文件。

使用场景

1. **节点维护**：当你需要对某个节点进行维护（例如系统更新、硬件更换等）时，可以使用 `kubectl cordon` 将该节点标记为不可调度，确保不会有新的 Pod 被调度到该节点上，但现有的 Pod 会继续运行。

2. **节点逐步升级**：在需要对集群节点逐步进行升级（例如 Kubernetes 版本更新）时，首先使用 `kubectl cordon` 标记节点为不可调度，确保不会有新的工作负载调度到该节点，然后进行节点的升级操作。

3. **资源管理**：如果某个节点的资源不足或负载过高，你可以使用 `kubectl cordon` 来暂时停止向该节点调度新的 Pod，等负载恢复正常后再重新开启调度。

注意事项

1. **不会驱逐现有的 Pod**：`kubectl cordon` 只会阻止新的 Pod 调度到该节点上，但现有的 Pod 仍然会继续运行。如果你需要将现有的 Pod 迁移到其他节点，你可以使用 `kubectl drain`。

2. **恢复节点调度**：如果需要恢复该节点的调度状态，可以使用 `kubectl uncordon` 命令来允许调度新 Pod 到节点上：

   ```
   kubectl uncordon node1
   ```

3. **适用于无状态应用的迁移**：对于无状态应用，可以使用 `kubectl drain` 结合 `kubectl cordon` 来进行节点上的 Pod 迁移和节点维护。但对于有状态应用，可能需要提前做好数据备份等准备。

总结

- `kubectl cordon` 用于将节点标记为不可调度，防止新的 Pod 被调度到该节点。
- 适用于节点维护、逐步升级和资源管理场景。
- 使用 `kubectl uncordon` 可以恢复节点的调度状态。
- 只影响新调度的 Pod，现有 Pod 不受影响。

###### drain
`kubectl drain` 命令用于将 Kubernetes 集群中的某个节点上的所有 Pod 驱逐出去，从而准备对该节点进行维护或升级等操作。它会使得节点上现有的 Pod 迁移到其他节点，同时也会将该节点标记为不可调度状态。这是 Kubernetes 集群中执行节点维护的常用方法。

基本用法

```
kubectl drain <node-name> [options]
```

- **`<node-name>`**：需要驱逐其上所有 Pod 的节点名称。

`kubectl drain` 命令会：
1. 标记节点为不可调度（`kubectl cordon` 的效果）。
2. 驱逐节点上所有可驱逐的 Pod，直到节点上没有运行中的 Pod 为止。

常见的用法示例

1. 驱逐节点上的所有 Pod

驱逐节点 `node1` 上的所有 Pod：

```
kubectl drain node1
```

这会将 `node1` 上所有可驱逐的 Pod 驱逐出去，通常 Pod 会被重新调度到集群中其他可用的节点上。如果某个 Pod 具有不可驱逐的特性（例如 `DaemonSet`），`kubectl drain` 会给出错误提示。

2. 忽略 `DaemonSet` 的 Pod

默认情况下，`kubectl drain` 会因为 `DaemonSet` 的 Pod 而失败，因为这些 Pod 是不可驱逐的。你可以通过 `--ignore-daemonsets` 选项来忽略 `DaemonSet` 的 Pod：

```
kubectl drain node1 --ignore-daemonsets
```

这会继续驱逐节点上的其他 Pod，而不管 `DaemonSet` 的 Pod。

3. 忽略无法删除的本地存储 Pod

如果 Pod 使用了本地存储，默认情况下 `kubectl drain` 也会失败。你可以通过 `--delete-emptydir-data` 选项忽略这些 Pod：

```
kubectl drain node1 --ignore-daemonsets --delete-emptydir-data
```

这会强制删除使用 `emptyDir` 卷的 Pod。

4. 强制驱逐 Pod

在某些情况下，Pod 可能无法正常终止（例如 Pod 持续重启）。你可以使用 `--force` 参数来强制驱逐这些 Pod：

```
kubectl drain node1 --force --ignore-daemonsets
```

需要注意，强制驱逐可能会导致某些 Pod 没有机会进行优雅关闭，因此可能会丢失数据或中断服务。

常见选项

- **`--ignore-daemonsets`**：忽略属于 `DaemonSet` 的 Pod。由于 `DaemonSet` Pod 通常是每个节点必需的，因此无法被驱逐。
- **`--delete-emptydir-data`**：允许删除使用 `emptyDir` 存储的 Pod。默认情况下，这些 Pod 会阻止驱逐，以避免丢失本地存储的数据。
- **`--force`**：强制删除无法优雅终止的 Pod。这可能会中断服务，因此应谨慎使用。
- **`--grace-period=<seconds>`**：指定 Pod 删除的优雅终止时间（以秒为单位）。默认情况下，系统会使用 Pod 的 `terminationGracePeriodSeconds` 设置。

使用场景

1. **节点维护或升级**：`kubectl drain` 常用于节点维护场景，例如系统升级或硬件更换。它确保在维护过程中不会有新的 Pod 被调度到该节点，同时将现有 Pod 安全地迁移到其他节点。

2. **节点缩容**：当你需要从集群中移除一个节点时，可以先使用 `kubectl drain` 驱逐节点上的所有 Pod，然后将节点下线或从集群中删除。

3. **负载均衡和资源优化**：在节点资源不足或负载过高的情况下，可以使用 `kubectl drain` 将节点上的 Pod 迁移到其他负载较低的节点，以优化集群资源使用。

注意事项

1. **不可驱逐的 Pod**：`DaemonSet` 和某些使用本地存储（如 `emptyDir`）的 Pod 是默认不可驱逐的。你可以使用相应的选项（如 `--ignore-daemonsets` 和 `--delete-emptydir-data`）来处理这些情况。

2. **数据丢失风险**：对于有状态的应用，特别是使用本地存储的 Pod，`kubectl drain` 可能会导致数据丢失。因此，在驱逐这些 Pod 时需要小心，确保数据已经备份或在其他节点上有副本。

3. **强制驱逐的风险**：使用 `--force` 强制驱逐 Pod 可能会导致服务中断或数据丢失。因此，尽量在非生产环境或在确认没有严重影响的情况下使用。

4. **驱逐过程中的时间**：如果节点上有大量的 Pod，或者 Pod 需要较长的优雅终止时间，那么驱逐操作可能会花费较长时间。因此，建议在驱逐前考虑节点上的 Pod 数量和每个 Pod 的终止时间。

总结

- `kubectl drain` 用于驱逐节点上的所有 Pod，以便对节点进行维护、升级或从集群中移除。
- 在执行驱逐时，节点会被标记为不可调度 (`kubectl cordon`)。
- 可以使用 `--ignore-daemonsets`、`--delete-emptydir-data`、`--force` 等选项来处理特殊情况。
- 适用于节点维护、集群缩容和资源优化场景。


###### drain 和 cordon 区别

drain 和 cordon 都是用于管理 Kubernetes 集群中的节点，确保节点在维护或升级时停止接收新的 Pod，但它们的功能和适用场景有所不同。

主要区别

1. 基本功能
   - cordon：将节点标记为不可调度状态，防止新创建的 Pod 被调度到该节点上。现有的 Pod 不受影响，继续在该节点上运行。
   - drain：将节点标记为不可调度状态，并驱逐节点上所有的 Pod，使得节点空闲，便于进行维护、升级等操作。驱逐后的 Pod 会被重新调度到其他可用节点上。

2. 适用场景
   - cordon：当需要暂时停止将新 Pod 调度到节点时使用，适用于准备对节点进行维护的场景，确保不会有新的工作负载被调度到该节点上，然后逐步进行维护。
   - drain：用于将节点上的所有 Pod 驱逐出去，为节点腾空，方便对节点进行维护、升级或移除，适用于需要对节点进行停机维护或缩容的场景。

3. 对现有 Pod 的影响
   - cordon：不会对现有 Pod 造成任何影响，只是停止新的 Pod 调度到该节点。
   - drain：会驱逐节点上的所有 Pod，并将其重新调度到其他节点。

4. 节点状态
   - cordon：节点状态变为不可调度 (SchedulingDisabled)，但节点上仍然有正在运行的 Pod。
   - drain：节点状态变为不可调度 (SchedulingDisabled)，并且该节点上的所有 Pod 被驱逐，节点上不会有运行的 Pod。

使用场景比较

- cordon：适用于需要暂时停止调度新的 Pod 到节点，但不影响现有 Pod 的运行。例如，准备对节点进行维护时，可以先使用 cordon，等到负载逐渐减少后，再使用 drain。
- drain：适用于腾空节点，以便进行维护、升级或移除节点。通过驱逐所有 Pod 确保节点上没有负载。

示例比较

- cordon
  ```
  kubectl cordon node1
  ```
  将 `node1` 标记为不可调度状态，新的 Pod 无法调度到 `node1`，但现有 Pod 会继续运行。

- drain
  ```
  kubectl drain node1
  ```
  将 `node1` 标记为不可调度，并驱逐所有 Pod。执行完成后，`node1` 上不会再有运行的 Pod。

注意事项

1. 不可驱逐的 Pod
   - DaemonSet 和一些使用本地存储的 Pod 默认不可被驱逐。使用 `kubectl drain` 时可以通过 `--ignore-daemonsets` 或 `--delete-emptydir-data` 参数来处理这些情况。

2. 强制驱逐
   - 使用 `--force` 选项可以强制驱逐无法正常终止的 Pod，但这可能会导致服务中断或数据丢失，应该谨慎使用。

3. 恢复节点调度
   - 可以使用 `kubectl uncordon` 恢复节点的调度状态，使其重新接收新的 Pod：
     ```
     kubectl uncordon node1
     ```

总结

- cordon：用于阻止新 Pod 调度到节点，但不影响节点上现有的 Pod 运行。
- drain：用于将节点上的所有 Pod 驱逐出去，并将节点标记为不可调度，以便对节点进行维护或升级。
- cordon 操作较快，仅标记节点不可调度；drain 则需要迁移 Pod，可能会耗时较长。



###### taint

 `kubectl taint` 命令用于为 Kubernetes 集群中的节点设置污点（Taint），以控制哪些 Pod 可以被调度到该节点。污点的作用是限制节点的工作负载类型，通过设置污点，只有带有特定容忍度（Toleration）的 Pod 才可以调度到该节点。通常用于隔离特定类型的应用，或者避免在资源有限的节点上调度一般工作负载。

基本用法

```
kubectl taint nodes <node-name> <key>=<value>:<effect>
```

- **`<node-name>`**：需要设置污点的节点名称。
- **`<key>`**：污点的键。
- **`<value>`**：污点的值。
- **`<effect>`**：污点的效果，有以下几种可能的取值：
  - **`NoSchedule`**：不允许没有相应容忍度的 Pod 调度到此节点上。
  - **`PreferNoSchedule`**：尽量避免没有相应容忍度的 Pod 调度到此节点上，但不是强制性的。
  - **`NoExecute`**：不仅会阻止没有相应容忍度的 Pod 被调度到该节点，还会驱逐已经在该节点上运行的没有相应容忍度的 Pod。

常见的用法示例

1. 添加污点

将节点 `node1` 设置为带有 `key1=value1` 且效果为 `NoSchedule` 的污点：

```
kubectl taint nodes node1 key1=value1:NoSchedule
```

这意味着没有相应容忍度的 Pod 将无法调度到 `node1` 上。

2. 删除污点

要删除节点上的污点，可以使用 `-` 符号来删除：

```
kubectl taint nodes node1 key1:NoSchedule-
```

这会删除 `node1` 上 `key1` 这个污点，允许所有 Pod 调度到该节点。

3. 设置多个污点

为节点设置多个污点，可以多次使用 `kubectl taint` 命令，或者在同一命令中使用多个键值对：

```
kubectl taint nodes node1 key1=value1:NoSchedule key2=value2:PreferNoSchedule
```

这会在 `node1` 上设置两个不同的污点，分别限制 Pod 调度行为。

使用场景

1. **隔离特定工作负载**：在集群中，一些节点可能具有特殊的硬件（如 GPU），需要将这些节点仅用于特定的工作负载，可以通过设置污点来限制不符合要求的 Pod 被调度到这些节点。

2. **维护和升级**：在节点维护或升级之前，可以使用污点和容忍度组合来控制哪些 Pod 可以继续留在节点上，而哪些需要被迁移走。

3. **节点资源管理**：当某些节点的资源有限时，可以通过设置污点来确保这些节点不会接收新的工作负载，保证现有的服务稳定运行。

注意事项

1. **结合容忍度使用**：污点需要和 Pod 的容忍度（Toleration）配合使用。只有具备相应容忍度的 Pod 才能被调度到带有污点的节点。例如，要让某个 Pod 可以调度到带有 `key1=value1:NoSchedule` 污点的节点上，需要在 Pod 配置中添加对应的 `tolerations`。

2. **操作对现有 Pod 的影响**：当为节点添加 `NoExecute` 类型的污点时，所有没有对应容忍度的 Pod 会被驱逐。因此在使用 `NoExecute` 污点时要格外小心，避免影响现有的服务。

3. **优雅控制 Pod 调度**：污点和容忍度提供了更精细的调度控制手段，可以根据集群的需求灵活配置节点的可调度性，满足不同应用对资源和隔离的需求。

总结

- `kubectl taint` 用于为节点添加污点，限制哪些 Pod 可以调度到该节点。
- 通过 `key=value:effect` 的格式指定污点，其中 `effect` 可以是 `NoSchedule`、`PreferNoSchedule`、`NoExecute`。
- 删除污点可以使用 `-` 符号，例如 `kubectl taint nodes node1 key1:NoSchedule-`。
- 污点需要结合 Pod 的容忍度使用，以实现对节点的精细调度控制。

###### uncordon
就是cordon的解除冻结的方法
`kubectl uncordon` 命令用于将之前通过 `kubectl cordon` 标记为不可调度的节点恢复为可调度状态。也就是说，执行 `kubectl uncordon` 后，新的 Pod 可以再次调度到该节点上。

基本用法

```
kubectl uncordon <node-name>
```

- **`<node-name>`**：需要恢复可调度状态的节点名称。

常见的用法示例

1. 恢复节点为可调度状态

假设之前将节点 `node1` 标记为不可调度，运行以下命令可以将其恢复为可调度：

```
kubectl uncordon node1
```

输出类似于：

```
node/node1 uncordoned
```

此时，`node1` 节点会恢复为可调度，新的 Pod 可以再次被调度到该节点。

2. 查看节点状态

你可以使用以下命令查看节点的状态：

```
kubectl get nodes
```

输出示例：

```
NAME    STATUS   ROLES    AGE   VERSION
node1   Ready    <role>   ...   ...
node2   Ready    <role>   ...   ...
```

在 `kubectl cordon` 后节点的状态为 `SchedulingDisabled`，而在 `kubectl uncordon` 后，节点的状态将会变回 `Ready`。

使用场景

1. **节点维护完成后恢复调度**：当对节点的维护（例如硬件更换、系统更新等）完成后，可以使用 `kubectl uncordon` 恢复节点的调度状态，使新创建的 Pod 可以被调度到该节点上。

2. **逐步增加集群可用资源**：在集群中逐步增加资源或重新启用节点时，可以通过 `kubectl uncordon` 恢复节点的调度能力，以让新工作负载被调度到这些节点上。

注意事项

1. **确保节点正常运行**：在运行 `kubectl uncordon` 之前，确保节点的状态为 `Ready`，并且节点的资源和服务都可以正常运行。如果节点状态异常，可能无法调度新 Pod 到该节点上。

2. **结合 drain 使用**：通常 `cordon` 和 `drain` 结合使用，`cordon` 用于防止新的 Pod 调度到节点，`drain` 用于将现有 Pod 驱逐并迁移到其他节点。而 `uncordon` 则是用于节点维护完成后恢复节点调度。

总结

- `kubectl uncordon` 用于将节点恢复为可调度状态，使新创建的 Pod 可以再次调度到该节点。
- 适用于节点维护完成后，恢复节点的调度能力。
- 结合 `cordon` 和 `drain` 命令，可以更好地管理节点的调度和维护。


#### kubectl本身的配置

###### alpha
新功能尝试, 不建议使用.
###### api-resources
`kubectl api-resources` 命令用于列出 Kubernetes 集群中支持的 API 资源及其详细信息。它帮助你了解集群中可用的 API 资源类型，例如 Pod、Service、Deployment 等，并可以查看这些资源的短名称、API 组和资源种类等信息。

基本用法

```
kubectl api-resources [options]
```

常见的选项

- **`--namespaced`**：仅列出具有命名空间的资源。
- **`--verbs=<list>`**：显示特定操作（例如 `get`、`list`、`create`）支持的资源。
- **`--api-group=<group>`**：列出属于特定 API 组的资源。
- **`-o`**：输出格式，支持 `wide`，提供更多的列信息。

常见的用法示例

1. **列出所有 API 资源**

列出当前 Kubernetes 集群中所有可用的 API 资源：

```
kubectl api-resources
```

输出示例：

```
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                          true         Binding
configmaps            cm                                                   true         ConfigMap
endpoints              ep                                                    true         Endpoints
events                    ev                                                    true         Event
pods                        po                                                    true         Pod
services                  svc                                                   true         Service
...
```

- **`NAME`**：资源的名称。
- **`SHORTNAMES`**：资源的简称，例如 `po` 是 `pods` 的简称。
- **`APIGROUP`**：资源所属的 API 组（如果有）。
- **`NAMESPACED`**：是否命名空间范围内（`true` 表示是，`false` 表示是集群级别的资源）。
- **`KIND`**：资源类型的对象名。

2. **仅列出具有命名空间的资源**

如果只想查看那些在命名空间中管理的资源，可以使用 `--namespaced` 选项：

```
kubectl api-resources --namespaced=true
```

这会列出所有属于命名空间范围的资源类型，例如 Pod、Service 等。

3. **列出集群范围的资源**

要查看集群范围的资源，例如节点（Node）等，可以使用 `--namespaced=false` 选项：

```
kubectl api-resources --namespaced=false
```

4. **根据 API 组筛选资源**

如果你只想查看属于特定 API 组的资源，可以使用 `--api-group` 选项。例如，查看属于 `apps` 组的资源：

```
kubectl api-resources --api-group=apps
```

这将返回所有属于 `apps` API 组的资源类型，例如 Deployment、StatefulSet 等。

5. **查看支持特定操作的资源**

你可以使用 `--verbs` 选项查看支持某些操作的资源。例如，查看所有支持 `list` 操作的资源：

```
kubectl api-resources --verbs=list
```

6. **以更宽的格式输出**

如果想要获得更多的输出信息，可以使用 `-o wide` 选项：

```
kubectl api-resources -o wide
```

这将显示更多的列信息，例如 API 版本等。

使用场景

1. **了解集群支持的资源**：通过 `kubectl api-resources`，你可以了解集群中支持哪些资源类型，方便在管理和编写 Kubernetes 配置文件时更好地使用正确的资源。

2. **查找资源的简写**：一些 Kubernetes 资源有简写形式，`kubectl api-resources` 可以帮助你找到这些简写形式，以便更快速地输入命令。

3. **验证 API 组**：不同资源可能属于不同的 API 组，例如 `apps`、`batch` 等，使用该命令可以帮助你确认资源属于哪个 API 组。

总结

- `kubectl api-resources` 用于列出集群中所有支持的 API 资源类型。
- 可以使用选项过滤输出，例如按命名空间、API 组或操作类型进行筛选。
- 适合用于查找资源类型的详细信息，了解集群中支持的资源种类，以及验证资源的 API 组。

###### ★completion 自动补全

`kubectl completion` 命令用于生成用于命令行自动补全的脚本，支持 Bash、Zsh、Fish 和 PowerShell 等多种 Shell。通过使用 `kubectl completion`，你可以为你的 Shell 配置命令补全功能，这样在输入 Kubernetes 命令时可以自动提示可用选项、资源名称等，极大提升命令行操作的效率。

基本用法

```
kubectl completion <shell>
```

- **`<shell>`**：指定你想为哪个 Shell 生成补全脚本，例如 `bash`、`zsh`、`fish` 或 `powershell`。

常见的用法示例

1. **Bash 自动补全**

为 Bash 生成自动补全脚本，并使其立即生效：

```
source <(kubectl completion bash)
```

如果你想让自动补全在每次打开终端时都可用，可以将上面的命令添加到 `~/.bashrc` 文件中：

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

2. **Zsh 自动补全**

为 Zsh 生成自动补全脚本，可以将其添加到你的 `~/.zshrc` 文件中：

```
echo "source <(kubectl completion zsh)" >> ~/.zshrc
```

然后重新加载 `~/.zshrc`：

```
source ~/.zshrc
```

3. **Fish 自动补全**

如果你使用 Fish shell，可以使用以下命令为其生成自动补全脚本：

```
kubectl completion fish | source
```

将其添加到 Fish 配置文件中，使其每次终端启动时生效：

```
kubectl completion fish > ~/.config/fish/completions/kubectl.fish
```

4. **PowerShell 自动补全**

如果你使用 PowerShell，可以使用以下命令启用补全：

```
kubectl completion powershell | Out-String | Invoke-Expression
```

将其添加到你的 PowerShell 配置文件中（例如 `$PROFILE`），以便每次启动 PowerShell 时自动启用补全。

使用场景

1. **提高命令输入效率**：通过补全脚本，可以快速补全 Kubernetes 命令和资源名称，减少输入错误，并节省时间。

2. **快速查找命令选项**：自动补全功能可以帮助你查看特定命令的所有可用选项，避免需要频繁查阅文档。

3. **降低学习成本**：对于新接触 Kubernetes 的用户，自动补全可以帮助他们熟悉 Kubernetes 的命令结构和可用选项。

注意事项

1. **需要合适的 Shell 环境**：确保你安装并使用支持补全功能的 Shell，例如 Bash 4.1+、Zsh、Fish 或 PowerShell。

2. **持久化配置**：使用 `source <(kubectl completion <shell>)` 命令可以即时启用补全功能，但为了持久化配置，需要将该命令写入相应的 Shell 配置文件中（例如 `~/.bashrc`、`~/.zshrc` 等）。

总结

- `kubectl completion` 用于为不同的 Shell 生成命令自动补全脚本。
- 支持多种 Shell，包括 Bash、Zsh、Fish 和 PowerShell。
- 可以通过将补全命令写入配置文件来持久化自动补全功能，提高命令行的操作效率。
###### config
`kubectl config` 命令用于管理 Kubernetes 的配置文件（通常是 `~/.kube/config`），包括设置集群、用户认证、上下文等。通过 `kubectl config`，你可以方便地管理不同集群和用户之间的连接，切换当前上下文，并查看或修改配置。

基本用法

```
kubectl config <subcommand> [options]
```

常见的子命令

1. **`kubectl config view`**：查看当前的配置文件内容。

```
kubectl config view
```
这个命令会显示当前 `kubeconfig` 文件的完整内容，包括集群、上下文和用户信息。

2. **`kubectl config use-context`**：切换当前的上下文。

```
kubectl config use-context <context-name>
```
将当前上下文切换到 `<context-name>`，从而连接到不同的 Kubernetes 集群或者使用不同的用户身份。

3. **`kubectl config get-contexts`**：列出所有可用的上下文。

```
kubectl config get-contexts
```
输出会列出所有已配置的上下文，包括当前使用的上下文。

4. **`kubectl config current-context`**：显示当前的上下文。

```
kubectl config current-context
```
这个命令会返回当前正在使用的上下文名称。

5. **`kubectl config set-context`**：设置或修改一个上下文。

```
kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name> [--namespace=<namespace>]
```
为 `<context-name>` 创建或修改一个上下文，可以指定关联的集群和用户，以及默认的命名空间。

6. **`kubectl config unset`**：删除配置中的某项内容。

```
kubectl config unset users.<user-name>
```
可以用来删除指定的用户、集群或者上下文。

7. **`kubectl config set-cluster`**：添加或修改一个集群的信息。

```
kubectl config set-cluster <cluster-name> --server=<server-address>
```
为 `<cluster-name>` 设置 API 服务器地址等信息。

8. **`kubectl config set-credentials`**：添加或修改一个用户的认证信息。

```
kubectl config set-credentials <user-name> --username=<username> --password=<password>
```
用于为 `<user-name>` 设置认证信息，包括用户名、密码、证书路径等。

使用场景

1. **管理多个集群**：当你有多个 Kubernetes 集群时，可以使用 `kubectl config` 来管理不同的集群配置，通过切换上下文快速连接到不同的集群。

2. **多用户身份管理**：对于需要以不同身份访问同一集群的情况，可以使用 `kubectl config` 配置多个用户，并通过上下文切换来管理不同的用户访问。

3. **简化集群访问**：`kubectl config` 可以帮助你轻松查看和设置集群的访问配置，无需手动修改 `kubeconfig` 文件。

示例

1. **列出所有上下文**：
```
kubectl config get-contexts
```

2. **切换到另一个上下文**：
```
kubectl config use-context dev-cluster
```

3. **查看当前使用的上下文**：
```
kubectl config current-context
```

4. **为新集群创建上下文**：
```
kubectl config set-context new-context --cluster=new-cluster --user=admin-user
```

5. **删除用户认证信息**：
```
kubectl config unset users.admin-user
```

总结

- `kubectl config` 用于管理 Kubernetes 的 `kubeconfig` 文件，可以查看、修改、添加和删除集群、用户和上下文。
- 常用命令包括 `view`、`use-context`、`get-contexts`、`set-context` 和 `set-credentials` 等。
- 通过 `kubectl config` 可以方便地管理多个集群和用户，简化 Kubernetes 的访问控制。

###### explain

`kubectl explain` 命令用于查看 Kubernetes 资源的文档和字段描述。它可以帮助你理解 Kubernetes 资源的结构、各字段的作用及其数据类型，特别是在编写 YAML 文件或者使用 `kubectl` 操作 Kubernetes 资源时非常有用。

基本用法

```
kubectl explain <resource> [options]
```

- **`<resource>`**：Kubernetes 资源的名称，例如 Pod、Deployment、Service 等。
- 可以使用点符号进一步查看资源的具体字段，例如 `kubectl explain deployment.spec`.

常见的用法示例

1. **查看资源的基本结构**

查看 Pod 资源的基本结构和描述信息：

```
kubectl explain pod
```

输出中包含资源的定义、描述及各个字段的介绍。

2. **查看资源字段的详细信息**

如果你想查看特定字段的信息，可以使用点符号。以下命令查看 `Deployment` 的 `spec` 字段：

```
kubectl explain deployment.spec
```

输出中会包含 `spec` 字段的详细描述及其子字段的说明。

3. **查看嵌套字段的信息**

可以查看更深层次的字段，例如查看 `spec` 下的 `replicas` 字段：

```
kubectl explain deployment.spec.replicas
```

这会显示 `replicas` 字段的用途及其类型。

4. **列出所有可用资源类型**

你可以使用 `kubectl api-resources` 来查看所有可用的资源类型，然后使用 `kubectl explain` 来获取这些资源的具体信息。

5. **查看资源的简写**

当你不确定资源的简写形式时，可以通过 `kubectl explain` 来获取。例如，查看 `rc`（ReplicationController）的全称和描述：

```
kubectl explain rc
```

使用场景

1. **编写和修改资源清单文件**：在编写 Kubernetes YAML 文件时，`kubectl explain` 可以帮助你理解每个字段的用途以及它们应该使用的值。

2. **学习和理解 Kubernetes 资源**：`kubectl explain` 是学习 Kubernetes 资源及其字段的有效工具，帮助理解 Kubernetes API 及其结构。

3. **排查配置问题**：当你遇到不理解的字段或者配置错误时，使用 `kubectl explain` 可以快速找到相关的说明。

示例

1. **查看 Pod 资源的基本信息**：
```
kubectl explain pod
```

2. **查看 Deployment 的 spec 字段**：
```
kubectl explain deployment.spec
```

3. **查看 Service 的 spec.type 字段**：
```
kubectl explain service.spec.type
```

注意事项

1. **字段路径使用点符号**：在查看资源的子字段时，需要使用点符号。例如，查看 Deployment 的 `spec.replicas`，使用 `kubectl explain deployment.spec.replicas`。

2. **集群版本影响输出**：`kubectl explain` 的输出基于 Kubernetes 集群的 API 版本，不同版本可能会有字段上的变化。

总结

- `kubectl explain` 用于查看 Kubernetes 资源及其字段的文档和描述。
- 可以查看资源的基本结构，也可以深入查看具体字段的详细说明。
- 在编写 YAML 文件、学习 Kubernetes 资源及排查配置问题时非常有用。


###### ★options
`kubectl options` 命令用于列出所有 `kubectl` 命令的全局选项。这些选项适用于所有的 `kubectl` 子命令，例如 `get`、`apply` 等，可以帮助你在使用 `kubectl` 命令时进行通用的配置和控制。

一般都是在集群初始化时, 常用.

基本用法

```
kubectl options
```

执行上述命令后，你将看到一份包含所有全局选项的列表。这些选项在执行任何 `kubectl` 子命令时都可以使用。

常见的全局选项

1. **`--kubeconfig`**：指定 kubeconfig 文件的路径，默认情况下，`kubectl` 会使用 `~/.kube/config` 文件，但你可以通过这个选项指定其他的配置文件。

```
kubectl get pods --kubeconfig /path/to/your/kubeconfig
```

2. **`--context`**：指定使用的上下文，便于在多个集群或用户配置之间切换。

```
kubectl get nodes --context=my-cluster
```

3. **`--namespace`**：指定要操作的命名空间。如果没有指定，`kubectl` 将使用当前上下文中的默认命名空间。

```
kubectl get pods --namespace=dev
```

4. **`--output` 或 `-o`**：指定输出格式，支持 `json`、`yaml`、`wide`、`name` 等格式。

```
kubectl get pods -o yaml
```

5. **`--token`**：用于设置 Bearer Token 以便和 API 服务器进行身份认证。

```
kubectl get pods --token=<your-token>
```

6. **`--server`**：指定 API 服务器的地址，覆盖 kubeconfig 文件中的服务器地址。

```
kubectl get nodes --server=https://my-k8s-api-server:6443
```

7. **`--insecure-skip-tls-verify`**：跳过对 API 服务器证书的验证，适用于测试环境，但不推荐在生产环境中使用。

```
kubectl get nodes --insecure-skip-tls-verify=true
```

8. **`--user`**：指定操作所使用的用户身份，可以覆盖 kubeconfig 文件中的用户设置。

```
kubectl get pods --user=admin
```

9. **`--request-timeout`**：指定请求的超时时间，默认值为 0，表示永不超时。

```
kubectl get pods --request-timeout='30s'
```

10. **`--v=<level>`**：设置详细日志输出的级别，数值越高，输出的详细程度越高，适合调试使用。

```
kubectl get pods --v=6
```

使用场景

1. **在多个集群和用户间切换**：通过选项 `--kubeconfig`、`--context` 和 `--user`，你可以方便地在不同的集群和用户之间切换，而不需要修改默认的 kubeconfig 文件。

2. **设置输出格式**：使用 `--output` 选项可以获取不同格式的输出，例如 YAML、JSON，以便用于脚本或配置文件。

3. **调试和测试**：通过 `--insecure-skip-tls-verify` 可以跳过 TLS 验证，使用 `--v` 选项可以获得详细的日志输出，便于调试。

4. **指定命名空间**：通过 `--namespace` 可以避免反复输入命名空间的名称，提高效率。

总结

- `kubectl options` 用于列出所有 `kubectl` 命令的全局选项，这些选项适用于任何 `kubectl` 子命令。
- 通过这些选项可以控制 `kubectl` 的行为，例如指定 kubeconfig 文件、上下文、命名空间、输出格式等。
- 全局选项在管理多个集群、调试、测试以及获取特定格式输出时非常有用。

###### ★plugin

`kubectl plugin` 命令用于扩展和管理 Kubernetes 的插件。插件是自定义的扩展命令，可以通过 `kubectl` 命令行工具加载和运行。它们通常用于实现 Kubernetes 的功能扩展，适合处理特定需求或增加自定义的操作功能。

基本用法

```
kubectl plugin <subcommand> [options]
```

Kubernetes 插件的开发和使用非常灵活，只需在 `PATH` 中包含可执行文件，该文件的名称格式为 `kubectl-<plugin-name>`，即可以通过 `kubectl <plugin-name>` 形式调用。

常见的使用场景和命令示例

1. **查看可用插件**

```
kubectl plugin list
```

该命令会列出当前系统中所有可用的插件。`kubectl` 会扫描环境变量 `PATH` 中的可执行文件，寻找以 `kubectl-` 开头的插件，并将它们显示出来。

2. **安装插件**

`kubectl` 本身不提供安装插件的功能，但你可以通过下载或开发自定义脚本来实现插件功能。通常插件可以是任何可执行脚本或程序（例如 Shell 脚本、Python 脚本等）。

假设你创建了一个 Shell 脚本，并命名为 `kubectl-example`：

```bash
#!/bin/bash
echo "This is a custom kubectl plugin example."
```

将其放置在系统的 `PATH` 路径中并赋予执行权限：

```
chmod +x kubectl-example
```

然后你可以通过以下方式调用它：

```
kubectl example
```

3. **开发自定义插件**

开发插件时，只需要确保命名规则遵循 `kubectl-<plugin-name>`，并且将插件可执行文件放置在系统的 `PATH` 中。例如，可以创建一个名为 `kubectl-hello` 的 Python 脚本：

```python
#!/usr/bin/env python3
print("Hello from kubectl plugin!")
```

将其保存为 `kubectl-hello`，并赋予可执行权限。然后可以通过以下方式调用：

```
kubectl hello
```

使用场景

1. **功能扩展**：`kubectl plugin` 允许你将一些常用的工作流或复杂的命令封装为插件，这样就可以通过 `kubectl` 命令轻松调用，方便快捷。

2. **团队协作**：可以为团队开发一些自定义的 Kubernetes 插件，帮助成员在开发和运维中提高效率。例如，编写一个用于特定环境的插件来管理资源。

3. **自动化操作**：通过插件，可以自动化一些常见的 Kubernetes 操作，例如批量更新、日志收集、监控配置等，减少重复性工作。

注意事项

1. **插件的位置**：插件必须是系统中某个路径下的可执行文件，并且该路径必须在环境变量 `PATH` 中。

2. **命名规则**：插件必须以 `kubectl-` 开头，这样才能被 `kubectl plugin list` 检测到并列出。

3. **权限**：确保插件文件具有可执行权限，否则无法正常运行。

总结

- `kubectl plugin` 用于扩展 Kubernetes 的功能，可以通过插件增加自定义的命令。
- 通过 `kubectl plugin list` 可以查看系统中所有可用的插件。
- 插件可以是任何脚本或可执行程序，只要命名符合 `kubectl-<plugin-name>` 规则并位于 `PATH` 路径中。

###### version
`kubectl version` 命令用于显示 Kubernetes 集群的客户端和服务器版本信息。这个命令对于检查 Kubernetes 版本兼容性、排查问题以及查看集群和客户端版本是否一致等情况非常有用。

基本用法

```
kubectl version [options]
```

常见的选项

1. **`--client`**：仅显示客户端版本信息。

```
kubectl version --client
```

这会返回本地安装的 `kubectl` 客户端版本，不会连接到 Kubernetes API 服务器。

2. **`--short`**：以简短格式显示版本信息。

```
kubectl version --short
```

这会以简短的格式输出客户端和服务器的版本信息，例如：

```
Client Version: v1.24.0
Server Version: v1.23.1
```

3. **`--output=json`** 或 **`-o json`**：以 JSON 格式显示版本信息。

```
kubectl version -o json
```

这会以 JSON 格式输出详细的版本信息，方便用于脚本化和自动化工具。

4. **`--output=yaml`**：以 YAML 格式显示版本信息。

```
kubectl version -o yaml
```

这会以 YAML 格式输出版本信息，类似于 JSON 输出，适合人类阅读或自动化任务。

使用场景

1. **检查版本兼容性**：在 Kubernetes 集群升级过程中，使用 `kubectl version` 可以查看服务器和客户端的版本是否一致，避免因为版本不兼容导致的功能异常。

2. **诊断问题**：在提交问题或请求帮助时，通常需要提供 Kubernetes 的版本信息，这有助于快速定位和解决版本相关的错误。

3. **集群管理**：当你管理多个 Kubernetes 集群时，使用 `kubectl version` 确保连接的是正确版本的集群，避免误操作。

示例

1. **查看客户端和服务器的版本信息**：

```
kubectl version
```

输出示例：

```
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", ...}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.1", ...}
```

2. **仅显示客户端版本信息**：

```
kubectl version --client
```

输出示例：

```
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", ...}
```

3. **简短格式显示版本**：

```
kubectl version --short
```

输出示例：

```
Client Version: v1.24.0
Server Version: v1.23.1
```

4. **以 JSON 格式显示版本信息**：

```
kubectl version -o json
```

输出示例：

```json
{
  "clientVersion": {
    "major": "1",
    "minor": "24",
    "gitVersion": "v1.24.0",
    ...
  },
  "serverVersion": {
    "major": "1",
    "minor": "23",
    "gitVersion": "v1.23.1",
    ...
  }
}
```

总结

- `kubectl version` 用于查看 Kubernetes 客户端和服务器版本信息。
- 常用选项有 `--client`、`--short`、`-o json` 等，可以以不同格式查看版本信息。
- 在管理 Kubernetes 集群、检查版本兼容性以及排查问题时非常有用。

#### 案例

##### 水平扩展
![](assets/Pasted%20image%2020241016212337.png)
![](assets/Pasted%20image%2020241016212319.png)

##### 获取配置文件
![](assets/Pasted%20image%2020241016212451.png)

##### 缩容
![](assets/Pasted%20image%2020241016212538.png)

##### 访问控制
![](assets/Pasted%20image%2020241016215019.png)
### 常用运维方式
![](assets/Pasted%20image%2020241016160021.png)

#### pod深入
Docker Desktop for Windows 上的 Kubernetes 环境主要适用于开发和轻量测试，默认内置的 CNI 网络功能比较有限，难以支持如 Calico 等插件.

k8s的cri容器镜像源, 是跟 container images mirror 保持一直的. 所以我们这里要修改一下镜像源.
![](assets/Pasted%20image%2020241019060132.png)


![](pod/nginx-demo.yaml)






如果是完全版本的cni插件,  应该会自动路由 
![](assets/Pasted%20image%2020241019023420.png)







##### 探针




##### 生命周期









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


