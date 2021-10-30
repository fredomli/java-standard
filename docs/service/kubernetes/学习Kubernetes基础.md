# Learn Kubernetes Basics

## Kubernetes 基础知识
本教程提供了Kubernetes集群编排系统的基础知识。每个模块包含一些关于Kubernetes主要特性和概念的背景信息，并包括一个交互式在线教程。这些交互式教程让您可以自己管理一个简单的集群及其容器化应用程序。

使用交互式教程，您可以学习:
* 在集群上部署容器化的应用程序。
* 规模的部署。
* 使用新软件版本更新容器化应用程序。
* 调试容器化应用程序。

教程使用Katacoda在您的浏览器中运行一个虚拟终端，该终端运行Minikube，一个小型的Kubernetes本地部署，可以在任何地方运行。不需要安装任何软件或配置任何东西;每个交互式教程直接运行在您的web浏览器本身。

## Kubernetes 能为你做什么
在现代web服务中，用户希望应用程序24/7可用，而开发人员希望每天多次部署这些应用程序的新版本。容器化有助于将软件打包以满足这些目标，使应用程序能够在不停机的情况下发布和更新。Kubernetes可以帮助您确保这些容器化的应用程序在您希望的时间和地点运行，并帮助它们找到工作所需的资源和工具。Kubernetes是一个可用于生产的开放源码平台，它使用谷歌在容器编排方面积累的经验，并结合社区的最佳想法而设计。

## Kubernetes基础模块
1. 创建Kubernetes集群
2. 部署应用程序
3. 探索你的应用
4. 公开应用程序
5. 扩大应用的规模
6. 更新应用程序

## 创建Kubernetes集群

### 使用 Minikube 创建集群
* 了解什么是Kubernetes集群。
* 了解Minikube是什么。
* 使用在线终端启动Kubernetes集群。

#### Kubernetes 集群

Kubernetes协调一个高度可用的计算机集群，这些计算机作为一个单元连接起来工作。

Kubernetes中的抽象允许您将容器化的应用程序部署到集群中，而无需将它们专门绑定到单独的机器上。要使用这种新的部署模型，应用程序需要以一种将其与单个主机分离的方式进行打包:它们需要被容器化。

容器化的应用程序比过去的部署模型更加灵活和可用，在过去的部署模型中，应用程序作为包直接安装到特定的机器上，与主机深度集成。

Kubernetes以一种更有效的方式自动化了应用程序容器在集群中的分布和调度。Kubernetes是一个开源平台，可以投入生产。

Kubernetes集群包含两种类型的资源:
* `控制平面用`于协调集群
* `节点`是运行应用程序的工人

> 英文：  
> The Control Plane coordinates the cluster   
> Nodes are the workers that run applications

集群图：
![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/module_01_cluster.svg)

控制平面负责管理集群。Control Plane协调集群中的所有活动，例如调度应用程序、维护应用程序所需的状态、扩展应用程序和推出新的更新。

节点是Kubernetes集群中作为工作机器的虚拟机或物理计算机。每个节点都有一个Kubelet，它是管理节点和与Kubernetes控制平面通信的代理。该节点还应该有处理容器操作的工具，如containerd或Docker。处理生产流量的Kubernetes集群至少应该有三个节点。

当您在Kubernetes上部署应用程序时，您告诉控制面启动应用程序容器。控制平面安排容器在集群的节点上运行。节点使用Kubernetes API与控制平面通信，控制平面公开该API。最终用户还可以直接使用Kubernetes API与集群进行交互。


Kubernetes集群可以部署在物理机上，也可以部署在虚拟机上。要开始Kubernetes的开发，可以使用Minikube。Minikube是一个轻量级的Kubernetes实现，它在您的本地机器上创建一个VM，并部署一个仅包含一个节点的简单集群。Minikube适用于Linux、macOS和Windows系统。Minikube CLI为使用集群提供了基本的引导操作，包括启动、停止、状态和删除。然而，在本教程中，您将使用预先安装了Minikube的在线终端。

现在您知道了Kubernetes是什么，让我们进入在线教程并启动我们的第一个集群!

### 交互式教程-创建集群
使用官网的 [交互式创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/) 

这个交互式场景的目标是使用minikube部署一个本地开发Kubernetes集群

在线终端是一个预配置的Linux环境，可以用作常规控制台(可以输入命令)。单击后面跟着ENTER符号的代码块将在终端中执行该命令。

## 部署应用程序

### 使用kubectl创建部署

* 了解应用程序部署。
* 使用kubectl在Kubernetes上部署您的第一个应用程序。

#### Kubernetes部署
一旦有了正在运行的Kubernetes集群，就可以在其上部署容器化的应用程序。为此，您需要创建一个Kubernetes Deployment配置。部署将指示Kubernetes如何创建和更新应用程序的实例。一旦创建了部署，Kubernetes控制平面就会安排部署中包含的应用程序实例在集群中的单个节点上运行。

一旦创建了应用程序实例，Kubernetes部署控制器就会持续监控这些实例。如果承载实例的Node宕机或被删除，部署控制器将用集群中另一个Node上的实例替换该实例。这提供了一种自修复机制来处理机器故障或维护。

在预编制环境中，安装脚本通常用于启动应用程序，但它们不允许从机器故障中恢复。Kubernetes deployment通过创建应用程序实例并让它们跨节点运行，为应用程序管理提供了一种完全不同的方法。

#### 在Kubernetes上部署您的第一个应用程序

您可以使用Kubernetes命令行界面Kubectl创建和管理部署。Kubectl使用Kubernetes API与集群进行交互。在本模块中，您将学习创建在Kubernetes集群上运行应用程序的部署所需的最常见的Kubectl命令。

在创建部署时，需要为应用程序指定容器映像和希望运行的副本数量。您可以稍后通过更新部署来更改该信息;训练营的第5和第6单元讨论了如何扩展和更新部署。

在你的第一次部署中，你将使用一个打包在Docker容器中的hello-node应用程序，它使用NGINX回显所有的请求。(如果您还没有尝试创建Hello -node应用程序并使用容器来部署它，您可以先按照Hello Minikube教程中的说明来做)。

现在你知道了什么是部署，让我们进入在线教程并部署我们的第一个应用程序!

### 交互式教程-部署应用程序
* [交互式教程-部署应用程序](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/)


## 参考
* [官网地址](https://kubernetes.io/)
* [Kubernetes文档](https://kubernetes.io/docs/home/)  
* [使用Minikube创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)
* [交互式教程-创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)
  
* [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)





*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
