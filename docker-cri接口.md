#### k8s已弃用docker的直接支持,但是可以选择安装插件的方式继续使用k8s-docker.

https://github.com/Mirantis/cri-dockerd


k8s官方推荐容器
Kubernetes 支持容器运行时，例如containerd、CRI-O

docker可以装个插件cri-dockerd，然后用k8s去做集群
docker swarm是docker自己的集群

