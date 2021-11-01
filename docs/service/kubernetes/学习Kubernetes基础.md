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

#### 交互式过程
1. 集群启动并运行，我们已经为你安装了minikube。运行minikube version命令检查是否正确安装:

```shell
minikube version
```
效果如下：
```text
minikube version: v1.18.0
commit: ec61815d60f66a6e4f6353030a40b12362557caa-dirty
```
我们可以看到minikube已经就位了。

2. 启动集群，使用minikube Start命令:
```shell
minikube start
```
```text
* minikube v1.18.0 on Ubuntu 18.04 (kvm/amd64)
* Using the none driver based on existing profile

X The requested memory allocation of 2200MiB does not leave room for system overhead (total system memory: 2460MiB). You may face stability issues.
* Suggestion: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

* Starting control plane node minikube in cluster minikube
* Running on localhost (CPUs=2, Memory=2460MB, Disk=194868MB) ...
* OS release is Ubuntu 18.04.5 LTS
* Preparing Kubernetes v1.20.2 on Docker 19.03.13 ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring local host environment ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v4
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
太棒了!现在在您的在线终端中有一个正在运行的Kubernetes集群。Minikube为您启动了一个虚拟机，并且Kubernetes集群现在正在该虚拟机中运行。


3. 集群的版本

为了在这个训练营期间与Kubernetes进行交互，我们将使用命令行界面kubectl。我们将在下一个模块中详细解释kubectl，但现在，我们只看一些集群信息。执行kubectl version命令检查kubectl是否已安装:
```shell
kubectl version
```
```text
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.4", GitCommit:"e87da0bd6e03ec3fea7933c4b5263d151aafd07c", GitTreeState:"clean", BuildDate:"2021-02-18T16:12:00Z", GoVersion:"go1.15.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.2", GitCommit:"faecb196815e248d3ecfb03c680a4507229c2a56", GitTreeState:"clean", BuildDate:"2021-01-13T13:20:00Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```
kubectl已经配置好了，我们可以看到客户端和服务器的版本。客户端版本是kubectl版本;服务器版本是安装在主服务器上的Kubernetes版本。您还可以查看有关构建的详细信息。

4. 集群详情
让我们查看集群的详细信息。我们将通过运行kubectl cluster-info来实现:
```shell
kubectl cluster-info
```
```text
Kubernetes control plane is running at https://172.17.0.96:8443
KubeDNS is running at https://172.17.0.96:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

在本教程中，我们将重点关注用于部署和探索应用程序的命令行。需要查看集群中的节点，使用kubectl get nodes命令。

```shell
kubectl get nodes
```
```text
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   4m37s   v1.20.2
```

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
[交互式教程-部署应用程序](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/)

#### 交互式过程
1. kubectl基础知识   

和minikube一样，kubectl也安装在在线终端中。在终端中输入kubectl查看其用法。kubectl命令的常用格式为:kubectl action resource。这将对指定的资源(如节点、容器)执行指定的操作(如创建、描述)。您可以在命令后使用——help来获得关于可能的额外信息
   
`kubectl get nodes——help`

通过运行kubectl version命令，检查kubectl是否配置为与您的集群对话:
```shell
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.4", GitCommit:"e87da0bd6e03ec3fea7933c4b5263d151aafd07c", GitTreeState:"clean", BuildDate:"2021-02-18T16:12:00Z", GoVersion:"go1.15.8", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```
好的，kubectl已经安装，您可以看到客户端和服务器版本。

需要查看集群中的节点，使用kubectl get nodes命令。
```shell
$ kubectl get nodes
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   2m56s   v1.20.2
```
这里我们看到了可用的节点(在我们的例子中是1)。Kubernetes将根据Node的可用资源选择部署应用程序的位置。
2. 部署我们的应用程序
   
让我们使用kubectl create deployment命令在Kubernetes上部署第一个应用程序。我们需要提供部署名称和应用程序图像位置(包括Docker中心外托管的图像的完整存储库url)。

```shell
kubectl create deployment \
kubernetes-bootcamp \
--image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

太棒了!您刚刚通过创建部署部署了第一个应用程序。这为你做了一些事情:
* 搜索可以运行应用程序实例的合适节点(我们只有1个可用节点)
* 调度应用程序在该节点上运行
* 将集群配置为在需要时在新Node上重新调度实例

要列出您的部署，请使用get deployment命令:

```shell
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           2m41s
```
我们看到有一个部署运行着你的应用的一个实例。这个实例运行在你的节点的Docker容器中。

3. 查看我们的程序
   
kubernetes体内运行的豆荚是在一个私人的独立网络上运行的。默认情况下，它们在同一kubernetes集群中的其他pod和服务中是可见的，但在该网络之外是不可见的。当我们使用kubectl时，我们通过API端点与应用程序进行交互。

我们将在模块4中讨论如何在kubernetes集群之外公开应用程序的其他选项。

kubectl命令可以创建一个代理，将通信转发到集群范围内的专用网络。代理可以通过按control-C终止，并且在运行时不会显示任何输

我们将打开第二个终端窗口来运行代理。
```shell
echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; 
kubectl proxy
```

现在，我们的主机(在线终端)和Kubernetes集群之间已经建立了连接。代理允许从这些终端直接访问API。

您可以通过代理端点看到所有这些api。例如，我们可以使用curl命令直接通过API查询版本

> 注意:检查终端顶部。代理在一个新选项卡(终端2)中运行，最近的命令在原始选项卡(终端1)中执行。代理仍然在第二个选项卡中运行，这允许我们的curl命令使用localhost:8001工作。

如果端口8001不可访问，请确保上面启动的kubectl代理正在运行。

API服务器将根据荚名自动为每个荚创建端点，该荚名也可以通过代理访问。

首先，我们需要获取Pod名称，并将其存储在环境变量POD_NAME中:

```shell
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
```
你可以通过运行API访问Pod:

```shell
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/
```
为了可以在不使用代理的情况下访问新部署，需要一个服务，这将在下一个模块中解释

## 探索你的应用
### 查看豆荚和节点
##### 目标
* 了解Kubernetes豆荚。
* 了解Kubernetes节点。
* 解决部署的应用程序。

#### Kubernetes豆荚
在模块2中创建部署时，Kubernetes创建了一个Pod来承载应用程序实例。Pod是Kubernetes的一个抽象，它表示一组一个或多个应用程序容器(如Docker)，以及这些容器的一些共享资源。这些资源包括:
* 共享存储，如卷
* 网络中，作为唯一的集群IP地址
* 有关如何运行每个容器的信息，例如容器映像版本或要使用的特定端口


Pod为特定于应用程序的“逻辑主机”建模，并可以包含相对紧密耦合的不同应用程序容器。例如，Pod可能既包括你的Node.js应用程序的容器，也包括为Node.js web服务器发布的数据提供feed的不同容器。Pod中的容器共享IP地址和端口空间，总是在同一节点上的共享上下文中运行。

豆荚是Kubernetes平台上的原子单位。当我们在Kubernetes上创建部署时，该部署会创建pod，其中包含容器(而不是直接创建容器)。每个Pod都绑定到调度它的节点上，并一直在那里直到终止(根据重启策略)或删除。在节点故障的情况下，将在集群中的其他可用节点上调度相同的pod。

豆荚概述：


#### 节点
Pod总是在Node上运行。Node是Kubernetes中的工作机器，根据集群的不同，它可以是虚拟机，也可以是物理机。每个节点由控制平面管理。一个Node可以有多个pod, Kubernetes控制平面自动处理集群中各个Node之间的pod调度。控制平面的自动调度考虑了每个节点上的可用资源。
s
![pc2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/module_03_pods.svg)

每个Kubernetes节点至少运行:

* Kubelet，负责Kubernetes控制平面和节点之间通信的进程;它管理在机器上运行的Pods和容器。
* 一个容器运行时(如Docker)，负责从注册表中取出容器映像，解包容器，并运行应用程序。

节点的概述
![pc3](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/module_03_nodes%20(1).svg)

#### 故障排除与kubectl
在模块2中，您使用了Kubectl命令行界面。在模块3中，您将继续使用它来获取有关已部署应用程序及其环境的信息。最常见的操作可以通过以下kubectl命令来完成:

您可以使用这些命令查看应用程序的部署时间、当前状态、运行位置以及配置。

现在我们对集群组件和命令行有了更多的了解，让我们来研究一下我们的应用程序。

### 交互式教程-探索您的应用程序
[交互式教程-探索你的应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-interactive/)

## 公开你的应用
### 使用服务暴露你的应用

目标：
* 了解Kubernetes中的服务
* 了解标签和标签选择器对象如何与服务相关
* 使用服务在Kubernetes集群外公开应用程序 
  
#### Kubernetes服务概述
Kubernetes荚是会销毁的。豆荚实际上有一个生命周期。当一个工作节点死亡时，在该节点上运行的Pods也会丢失。然后，ReplicaSet可以通过创建新的pod来保持应用程序运行，从而动态地将集群驱动回所需的状态。
作为另一个例子，考虑一个带有3个副本的图像处理后端。这些复制品是可以交换的;前端系统不应该关心后端副本，甚至不应该关心Pod是否丢失并重新创建。也就是说，Kubernetes集群中的每个Pod都有一个唯一的IP地址，即使是同一节点上的Pod也一样，因此需要一种方法来自动协调Pod之间的变化，以便您的应用程序继续运行。

Kubernetes中的Service是一个抽象，它定义了一组pod的逻辑集合和访问它们的策略。服务支持依赖pod之间的松散耦合。像所有Kubernetes对象一样，服务是使用YAML(首选)或JSON定义的。服务的目标Pods集合通常由LabelSelector决定(请参阅下面的说明，为什么您可能希望在规范中不包含选择器)。

尽管每个Pod都有一个唯一的IP地址，但如果没有服务，这些IP不会暴露在集群外部。服务允许您的应用程序接收流量。通过在ServiceSpec中指定一个类型，可以以不同的方式公开服务:

* ClusterIP(默认)-在集群的一个内部IP上暴露服务。这种类型使得服务只能从集群内部访问。
* NodePort -使用NAT在集群中每个选定节点的同一个端口上公开服务。使用<NodeIP>:<NodePort>使一个服务可以从集群外部访问。ClusterIP的超集。
* LoadBalancer -在当前云(如果支持)中创建一个外部负载均衡器，并为服务分配一个固定的外部IP。NodePort的超集。
* ExternalName -通过返回带有其值的CNAME记录，将服务映射到ExternalName字段的内容(例如foo.bar.example.com)。没有建立任何形式的代理。这种类型需要v1.7或更高的kube-dns，或CoreDNS版本0.0.8或更高。

### 交互式教程-开放您的应用程序


## 运行应用的多个实例

### 运行应用的多个实例
目标：
* 使用kubectl扩展应用程序。
 
#### 扩展一个应用程序
在前面的模块中，我们创建了一个Deployment，然后通过服务公开它。部署只创建了一个Pod来运行我们的应用程序。当流量增加时，我们将需要扩展应用程序以跟上用户需求。

扩展是通过更改部署中的副本数量来实现的


向外扩展部署将确保创建新的Pods，并将其调度到具有可用资源的节点。扩展将使pod的数量增加到所需的新状态。Kubernetes还支持pod的自动缩放，但这超出了本教程的范围。扩展到零也是可能的，它将终止指定部署的所有pod。

运行应用程序的多个实例需要一种将流量分配给所有实例的方法。服务有一个集成的负载平衡器，它将网络流量分发到公开部署的所有pod。服务将使用端点持续监视正在运行的pod，以确保流量只发送到可用的pod。

一旦有多个应用程序实例运行，就可以在不停机的情况下执行Rolling更新。我们将在下一个模块中讨论它。现在，让我们进入在线终端并扩展应用程序。

### 交互式教程-扩展你的应用
[交互式教程-扩展你的应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-interactive/)

## 更新应用程序
### 执行滚动更新
#### 更新应用程序
用户希望应用程序随时可用，而开发人员希望每天部署几次新版本。在Kubernetes中，这是通过滚动更新完成的。滚动更新通过增量更新Pods实例，使deployment的更新无需停机。新的Pods将被安排在具有可用资源的节点上。

在前一个模块中，我们扩展了应用程序以运行多个实例。这是执行更新而不影响应用程序可用性的要求。默认情况下，在更新期间可以不可用的Pods的最大数量和可以创建的新Pods的最大数量是1。这两个选项都可以配置为数字或百分比(pod)。在Kubernetes中，更新是版本化的，任何部署更新都可以恢复到以前的(稳定的)版本。

与应用程序伸缩类似，如果一个部署公开，在更新期间，服务将只将流量负载平衡到可用的pod。可用的Pod是应用程序用户可用的实例。

滚动更新允许以下操作:
* 将应用程序从一个环境提升到另一个环境(通过容器映像更新)
* 回滚到以前的版本
* 在零停机的情况下持续集成和持续交付应用程序

在下面的交互式教程中，我们将把应用程序更新到新版本，并执行回滚。

### 交互式教程-更新您的应用程序
[交互式教程-更新您的应用程序](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-interactive/)



## 参考
* [官网地址](https://kubernetes.io/)
* [Kubernetes文档](https://kubernetes.io/docs/home/)  
* [使用Minikube创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)
* [交互式教程-创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)
* [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
* [交互式教程-部署应用程序](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/)
* [交互式教程-探索你的应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-interactive/)




*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
