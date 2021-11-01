# 下载 Kubernetes
Kubernetes 是一门技术的总称，涉及不同的操作平台，不同的使用环境。不同的需求也有不同的使用方式。下面就包括学习环境，开发环境，生产环境等需要的技术支持。
## 学习环境
如果您正在学习Kubernetes，请使用Kubernetes社区支持的工具，或者使用生态系统中的工具在本地机器上建立Kubernetes集群。看到 [安装工具](https://kubernetes.io/docs/tasks/tools/) 。

## 生产环境
在评估一个生产环境的解决方案时，请考虑操作Kubernetes集群(或抽象)的哪些方面您希望自己管理，哪些方面更愿意交给提供商。

对于您自己管理的集群，官方支持的部署Kubernetes的工具是[kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/) 。

## 核心Kubernetes组件
在 [CHANGELOG](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG) 文件中找到下载Kubernetes组件(及其校验和)的链接。

或者，使用 [downloadkubernetes.com](https://www.downloadkubernetes.com/) 来根据版本和架构进行过滤。


## kubectl

Kubernetes命令行工具kubectl允许您对Kubernetes集群运行命令。

kubectl可以实现应用部署、集群资源巡检和管理、日志查看等功能。有关包括kubectl操作的完整列表的更多信息，请参阅kubectl参考文档。

kubectl可在各种Linux平台、macOS和Windows上安装。在下面找到您喜欢的操作系统。

* [在Linux上安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux)
* [在macOS上安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos)
* [在Windows上安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows)


Kubernetes是为其控制平面在Linux上运行而设计的。在集群中，您可以在Linux或其他操作系统(包括Windows)上运行应用程序。
* [学习使用Windows节点设置集群](https://kubernetes.io/docs/setup/production-environment/windows/)
## 参考
* [官网地址](https://kubernetes.io/)
* [Kubernetes文档](https://kubernetes.io/docs/home/)
* [使用Minikube创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)
* [交互式教程-创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)
* [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
* [交互式教程-部署应用程序](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/)
* [交互式教程-探索你的应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-interactive/)
* [Kubernetes 中文网](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/)
* [Kubectl 文档](https://kubernetes.io/docs/reference/kubectl/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
