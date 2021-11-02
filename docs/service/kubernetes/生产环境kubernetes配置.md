# Kubernetes配置

生产质量的 Kubernetes 集群需要规划和准备。 如果你的 Kubernetes 集群是用来运行关键负载的，该集群必须被配置为弹性的（Resilient）。 本页面阐述你在安装生产就绪的集群或将现有集群升级为生产用途时可以遵循的步骤。 如果你已经熟悉生产环境安装，因此只关注一些链接，则可以跳到接下来节。

## 生产环境考量
通常，一个生产用 Kubernetes 集群环境与个人学习、开发或测试环境所使用的 Kubernetes 相比有更多的需求。生产环境可能需要被很多用户安全地访问，需要 提供一致的可用性，以及能够与需求变化相适配的资源。

在你决定在何处运行你的生产用 Kubernetes 环境（在本地或者在云端），以及 你希望承担或交由他人承担的管理工作量时，需要考察以下因素如何影响你对 Kubernetes 集群的需求：

* 可用性：一个单机的 Kubernetes 学习环境 具有单点失效特点。创建高可用的集群则意味着需要考虑：

    * 将控制面与工作节点分开
    * 在多个节点上提供控制面组件的副本
    * 为针对集群的 API 服务器 的流量提供负载均衡
    * 随着负载的合理需要，提供足够的可用的（或者能够迅速变为可用的）工作节点
  
* 规模：如果你预期你的生产用 Kubernetes 环境要承受固定量的请求， 你可能可以针对所需要的容量来一次性完成安装。 不过，如果你预期服务请求会随着时间增长，或者因为类似季节或者特殊事件的 原因而发生剧烈变化，你就需要规划如何处理请求上升时对控制面和工作节点 的压力，或者如何缩减集群规模以减少未使用资源的消耗。
* 安全性与访问管理：在你自己的学习环境 Kubernetes 集群上，你拥有完全的管理员特权。 但是针对运行着重要工作负载的共享集群，用户账户不止一两个时，就需要更细粒度 的方案来确定谁或者哪些主体可以访问集群资源。 你可以使用基于角色的访问控制（RBAC） 和其他安全机制来确保用户和负载能够访问到所需要的资源，同时确保工作负载及集群 自身仍然是安全的。 你可以通过管理策略和 容器资源来 针对用户和工作负载所可访问的资源设置约束，

## 生产用集群安装
在生产质量的 Kubernetes 集群中，控制面用不同的方式来管理集群和可以 分布到多个计算机上的服务。每个工作节点则代表的是一个可配置来运行 Kubernetes Pods 的实体。

### 生产用控制面
最简单的 Kubernetes 集群中，整个控制面和工作节点服务都运行在同一台机器上。你可以通过添加工作节点来提升环境能力，正如 Kubernetes 组件示意图所示。 如果只需要集群在很短的一段时间内可用，或者可以在某些事物出现严重问题时直接丢弃， 这种配置可能符合你的需要。

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/components-of-kubernetes.svg)

如果你需要一个更为持久的、高可用的集群，那么你就需要考虑扩展控制面的方式。 根据设计，运行在一台机器上的单机控制面服务不是高可用的。 如果保持集群处于运行状态并且需要确保在出现问题时能够被修复这点很重要， 可以考虑以下步骤：

* 选择部署工具：你可以使用类似 kubeadm、kops 和 kubespray 这类工具来部署控制面。参阅使用部署工具安装 Kubernetes 以了解使用这类部署方法来完成生产就绪部署的技巧。 存在不同的容器运行时 可供你的部署采用。
  * [部署工具](https://kubernetes.io/zh/docs/setup/production-environment/tools/)
  
* 管理证书：控制面服务之间的安全通信是通过证书来完成的。证书是在部署期间 自动生成的，或者你也可以使用你自己的证书机构来生成它们。 参阅 PKI 证书和需求了解细节。
* 为 API 服务器配置负载均衡：配置负载均衡器来将外部的 API 请求散布给运行在 不同节点上的 API 服务实例。参阅 创建外部负载均衡器 了解细节。
* 分离并备份 etcd 服务：etcd 服务可以运行于其他控制面服务所在的机器上， 也可以运行在不同的机器上以获得更好的安全性和可用性。 因为 etcd 存储着集群的配置数据，应该经常性地对 etcd 数据库进行备份， 以确保在需要的时候你可以修复该数据库。与配置和使用 etcd 相关的细节可参阅 etcd FAQ。 更多的细节可参阅为 Kubernetes 运维 etcd 集群 和使用 kubeadm 配置高可用的 etcd 集群。
* 创建多控制面系统：为了实现高可用性，控制面不应被限制在一台机器上。 如果控制面服务是使用某 init 服务（例如 systemd）来运行的，每个服务应该 至少运行在三台机器上。不过，将控制面作为服务运行在 Kubernetes Pods 中可以确保你所请求的个数的服务始终保持可用。 调度器应该是可容错的，但不是高可用的。 某些部署工具会安装 Raft 票选算法来对 Kubernetes 服务执行领导者选举。如果主节点消失，另一个服务会被选中并接手相应服务。
* 跨多个可用区：如果保持你的集群一直可用这点非常重要，可以考虑创建一个跨 多个数据中心的集群；在云环境中，这些数据中心被视为可用区。 若干个可用区在一起可构成地理区域。 通过将集群分散到同一区域中的多个可用区内，即使某个可用区不可用，整个集群 能够继续工作的机会也大大增加。 更多的细节可参阅跨多个可用区运行。
* 管理演进中的特性：如果你计划长时间保留你的集群，就需要执行一些维护其 健康和安全的任务。例如，如果你采用 kubeadm 安装的集群，则有一些可以帮助你完成 证书管理 和升级 kubeadm 集群 的指令。 参见管理集群了解一个 Kubernetes 管理任务的较长列表。

要了解运行控制面服务时可使用的选项，可参阅 `kube-apiserver`、 `kube-controller-manager` 和 `kube-scheduler` 组件参考页面。 如要了解高可用控制面的例子，可参阅 `高可用拓扑结构选项`、 使用 `kubeadm 创建高可用集群` 以及为 `Kubernetes 运维 etcd 集群`。 关于制定 etcd 备份计划，可参阅 `对 etcd 集群执行备份`。


### 生产用工作节点
生产质量的工作负载需要是弹性的；它们所依赖的其他组件（例如 CoreDNS）也需要是弹性的。 无论你是自行管理控制面还是让云供应商来管理，你都需要考虑如何管理工作节点 （有时也简称为节点）。
* 配置节点：节点可以是物理机或者虚拟机。如果你希望自行创建和管理节点， 你可以安装一个受支持的操作系统，之后添加并运行合适的 节点服务。 考虑：
  * 在安装节点时要通过配置适当的内存、CPU 和磁盘速度、存储容量来满足 你的负载的需求。
  * 是否通用的计算机系统即足够，还是你有负载需要使用 GPU 处理器、Windows 节点 或者 VM 隔离。
* 验证节点：参阅验证节点配置 以了解如何确保节点满足加入到 Kubernetes 集群的需求。

* 添加节点到集群中：如果你自行管理你的集群，你可以通过安装配置你的机器， 之后或者手动加入集群，或者让它们自动注册到集群的 API 服务器。参阅 节点节，了解如何配置 Kubernetes 以便以这些方式来添加节点。
* 向集群中添加 Windows 节点：Kubernetes 提供对 Windows 工作节点的支持； 这使得你可以运行实现于 Windows 容器内的工作负载。参阅 Kubernetes 中的 Windows 了解进一步的详细信息。
* 扩缩节点：制定一个扩充集群容量的规划，你的集群最终会需要这一能力。 参阅大规模集群考察事项 以确定你所需要的节点数；这一规模是基于你要运行的 Pod 和容器个数来确定的。 如果你自行管理集群节点，这可能意味着要购买和安装你自己的物理设备。
* 节点自动扩缩容：大多数云供应商支持 集群自动扩缩器（Cluster Autoscaler） 以便替换不健康的节点、根据需求来增加或缩减节点个数。参阅 常见问题 了解自动扩缩器的工作方式，并参阅 Deployment 了解不同云供应商是如何实现集群自动扩缩器的。 对于本地集群，有一些虚拟化平台可以通过脚本来控制按需启动新节点。
* 安装节点健康检查：对于重要的工作负载，你会希望确保节点以及在节点上 运行的 Pod 处于健康状态。通过使用 Node Problem Detector， 你可以确保你的节点是健康的。
### 生产级用户环境
在生产环境中，情况可能不再是你或者一小组人在访问集群，而是几十 上百人需要访问集群。在学习环境或者平台原型环境中，你可能具有一个 可以执行任何操作的管理账号。在生产环境中，你可需要对不同名字空间 具有不同访问权限级别的很多账号。

建立一个生产级别的集群意味着你需要决定如何有选择地允许其他用户访问集群。 具体而言，你需要选择验证尝试访问集群的人的身份标识（身份认证），并确定 他们是否被许可执行他们所请求的操作（鉴权）：

* 认证（Authentication）：API 服务器可以使用客户端证书、持有者令牌、身份 认证代理或者 HTTP 基本认证机制来完成身份认证操作。 你可以选择你要使用的认证方法。通过使用插件，API 服务器可以充分利用你所在 组织的现有身份认证方法，例如 LDAP 或者 Kerberos。 关于认证 Kubernetes 用户身份的不同方法的描述，可参阅 身份认证。
* 鉴权（Authorization）：当你准备为一般用户执行权限判定时，你可能会需要 在 RBAC 和 ABAC 鉴权机制之间做出选择。参阅 鉴权概述，了解 对用户账户（以及访问你的集群的服务账户）执行鉴权的不同模式。
  * 基于角色的访问控制（RBAC）： 让你通过为通过身份认证的用户授权特定的许可集合来控制集群访问。 访问许可可以针对某特定名字空间（Role）或者针对整个集群（CLusterRole）。 通过使用 RoleBinding 和 ClusterRoleBinding 对象，这些访问许可可以被 关联到特定的用户身上。
  * 基于属性的访问控制（ABAC）： 让你能够基于集群中资源的属性来创建访问控制策略，基于对应的属性来决定 允许还是拒绝访问。策略文件的每一行都给出版本属性（apiVersion 和 kind） 以及一个规约属性的映射，用来匹配主体（用户或组）、资源属性、非资源属性 （/version 或 /apis）和只读属性。 参阅示例以了解细节。

作为在你的生产用 Kubernetes 集群中安装身份认证和鉴权机制的负责人， 要考虑的事情如下：

* 设置鉴权模式：当 Kubernetes API 服务器 （kube-apiserver） 启动时，所支持的鉴权模式必须使用 --authorization-mode 标志配置。 例如，kube-apiserver.yaml（位于 /etc/kubernetes/manifests 下）中对应的 标志可以设置为 Node,RBAC。这样就会针对已完成身份认证的请求执行 Node 和 RBAC 鉴权。
* 创建用户证书和角色绑定（RBAC）：如果你在使用 RBAC 鉴权，用户可以创建 由集群 CA 签名的 CertificateSigningRequest（CSR）。接下来你就可以将 Role 和 ClusterRole 绑定到每个用户身上。 参阅证书签名请求 了解细节。
* 创建组合属性的策略（ABAC）：如果你在使用 ABAC 鉴权，你可以设置属性组合 以构造策略对所选用户或用户组执行鉴权，判定他们是否可访问特定的资源 （例如 Pod）、名字空间或者 apiGroup。进一步的详细信息可参阅 示例。
* 考虑准入控制器：针对指向 API 服务器的请求的其他鉴权形式还包括 Webhook 令牌认证。 Webhook 和其他特殊的鉴权类型需要通过向 API 服务器添加 准入控制器 来启用。

### 为负载资源设置约束
生产环境负载的需求可能对 Kubernetes 的控制面内外造成压力。 在针对你的集群的负载执行配置时，要考虑以下条目：
* 设置名字空间限制：为每个名字空间的内存和 CPU 设置配额。 参阅管理内存、CPU 和 API 资源 以了解细节。你也可以设置 层次化名字空间 来继承这类约束。
* 为 DNS 请求做准备：如果你希望工作负载能够完成大规模扩展，你的 DNS 服务 也必须能够扩大规模。参阅 自动扩缩集群中 DNS 服务。
* 创建额外的服务账户：用户账户决定用户可以在集群上执行的操作，服务账号则定义的 是在特定名字空间中 Pod 的访问权限。 默认情况下，Pod 使用所在名字空间中的 default 服务账号。

### 接下来
* 决定你是想自行构造自己的生产用 Kubernetes 还是从某可用的 云服务外包厂商 或 Kubernetes 合作伙伴获得集群。
* 如果你决定自行构造集群，则需要规划如何处理 证书 并为类似 etcd 和 API 服务器 这些功能组件配置高可用能力。
* 选择使用 kubeadm、 kops 或 Kubespray 作为部署方法。
* 通过决定身份认证和 鉴权方法来配置用户管理。
* 通过配置资源限制、 DNS 自动扩缩 和服务账号 来为应用负载作准备。

## 自行构造自己的生产用 Kubernetes
### 使用部署工具安装 Kubernetes
1. [使用 kubeadm 引导集群](https://github.com/fredomli/java-standard/blob/main/docs/service/kubernetes/https://github.com/fredomli/java-standard/blob/main/docs/service/kubernetes/生产环境kubernetes配置.md###使用kubeadm引导集群) 
2. 使用 Kops 安装 Kubernetes
3. 使用 Kubespray 安装 Kubernetes

### 使用 kubeadm 引导集群
本页面显示如何安装 kubeadm 工具箱。 有关在执行此安装过程后如何使用 kubeadm 创建集群的信息，请参见 使用 kubeadm 创建集群 页面。
### 安装 kubeadm
#### 准备开始
* 一台兼容的 Linux 主机。Kubernetes 项目为基于 Debian 和 Red Hat 的 Linux 发行版以及一些不提供包管理器的发行版提供通用的指令
* 每台机器 2 GB 或更多的 RAM （如果少于这个数字将会影响你应用的运行内存)
* 2 CPU 核或更多
* 集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)
* 节点之中不可以有重复的主机名、MAC 地址或 product_uuid。
* 开启机器上的某些端口。
* 禁用交换分区。为了保证 kubelet 正常工作，你 必须 禁用交换分区。

参考：
* [VM-虚拟机安装](https://github.com/fredomli/java-standard/blob/main/docs/vm/虚拟机安装.md)
* [Centos镜像下载](https://github.com/fredomli/java-standard/blob/main/docs/vm/Centos系统镜像下载.md)
* [使用VMware和Centos7创建虚拟机](https://github.com/fredomli/java-standard/blob/main/docs/vm/使用VMware和Centos7创建虚拟机.md)
* [XShell工具非商业版本](https://github.com/fredomli/java-standard/blob/main/docs/utils/shell/XShell工具学生版.md)
* [Linux命令大全](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/Linux%E5%91%BD%E4%BB%A4%E5%A4%A7%E5%85%A8.pdf)

#### 确保每个节点上 MAC 地址和 product_uuid 的唯一性
* 你可以使用命令 kip lin 或 ifconfig -a 来获取网络接口的 MAC 地址
* 可以使用 sudo cat /sys/class/dmi/id/product_uuid 命令对 product_uuid 校验
  
```shell
[root@localhost ~]# ifconfig -a
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:86:dd:a6:39  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.199.129  netmask 255.255.255.0  broadcast 192.168.199.255
        inet6 fe80::bcda:ff1c:1d39:874d  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:ac:41:3b  txqueuelen 1000  (Ethernet)
        RX packets 87003  bytes 128802512 (122.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18802  bytes 1308704 (1.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 18  bytes 1488 (1.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18  bytes 1488 (1.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        
[root@localhost ~]# cat /sys/class/dmi/id/product_uuid
2FC14D56-9F55-90CA-F2B9-89D8C7AC413B
```
一般来讲，硬件设备会拥有唯一的地址，但是有些虚拟机的地址可能会重复。 Kubernetes 使用这些值来唯一确定集群中的节点。 如果这些值在每个节点上不唯一，可能会导致安装 失败。

#### 检查网络适配器
如果你有一个以上的网络适配器，同时你的 Kubernetes 组件通过默认路由不可达，我们建议你预先添加 IP 路由规则，这样 Kubernetes 集群就可以通过对应的适配器完成连接。

#### 允许 iptables 检查桥接流量
确保 `br_netfilter` 模块被加载。这一操作可以通过运行 `lsmod | grep br_netfilter` 来完成。若要显式加载该模块，可执行 `sudo modprobe br_netfilter`。

为了让你的 Linux 节点上的 iptables 能够正确地查看桥接流量，你需要确保在你的 sysctl 配置中将 `net.bridge.bridge-nf-call-iptables` 设置为 1。例如：

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
#### 检查所需端口

1. 控制平面节点:

|协议|方向|端口范围|作用|使用者|
|---|---|---|---|---|
|TCP	|入站	|6443	|Kubernetes API 服务器	|所有组件|
|TCP	|入站	|2379-2380|etcd 服务器客户端 API| kube-apiserver, etcd|
|TCP	|入站	|10250	|Kubelet API	|kubelet 自身、控制平面组件|
|TCP	|入站	|10251	|kube-scheduler	|kube-scheduler 自身|
|TCP	|入站	|10252	|kube-controller-manager	|kube-controller-manager 自身|

查看端口使用情况：
```shell
netstat -ntpl
```

2. 工作节点:  

|协议|方向|端口范围|作用|使用者|
|---|---|---|---|---|
|TCP|	入站	|10250	|Kubelet API	|kubelet 自身、控制平面组件|
|TCP|	入站	|30000-32767|	NodePort 服务	|所有组件|

> NodePort 服务 的默认端口范围。

使用 * 标记的任意端口号都可以被覆盖，所以你需要保证所定制的端口是开放的。

虽然控制平面节点已经包含了 etcd 的端口，你也可以使用自定义的外部 etcd 集群，或是指定自定义端口。

你使用的 Pod 网络插件 (见下) 也可能需要某些特定端口开启。由于各个 Pod 网络插件都有所不同， 请参阅他们各自文档中对端口的要求。

#### 安装 runtime
为了在 Pod 中运行容器，Kubernetes 使用 容器运行时（Container Runtime）。比如我们熟悉的Docker技术。



> 在Linux结点中：
>
> 默认情况下，Kubernetes 使用 容器运行时接口（Container Runtime Interface，CRI） 来与你所选择的容器运行时交互。
>
> 如果你不指定运行时，则 kubeadm 会自动尝试检测到系统上已经安装的运行时， 方法是扫描一组众所周知的 Unix 域套接字。 下面的表格列举了一些容器运行时及其对应的套接字路径：

|运行时	|域套接字|
|---|---|
|Docker	|/var/run/dockershim.sock|
|containerd	|/run/containerd/containerd.sock|
|CRI-O	|/var/run/crio/crio.sock|

如果同时检测到 Docker 和 containerd，则优先选择 Docker。 这是必然的，因为 Docker 18.09 附带了 containerd 并且两者都是可以检测到的， 即使你仅安装了 Docker。 如果检测到其他两个或多个运行时，kubeadm 输出错误信息并退出。
kubelet 通过内置的 dockershim CRI 实现与 Docker 集成。
#### 安装 kubeadm、kubelet 和 kubectl

你需要在每台机器上安装以下的软件包：

* kubeadm：用来初始化集群的指令。
* kubelet：在集群中的每个节点上用来启动 Pod 和容器等。
* kubectl：用来与集群通信的命令行工具。

kubeadm 不能 帮你安装或者管理 kubelet 或 kubectl，所以你需要 确保它们与通过 kubeadm 安装的控制平面的版本相匹配。 如果不这样做，则存在发生版本偏差的风险，可能会导致一些预料之外的错误和问题。 然而，控制平面与 kubelet 间的相差一个次要版本不一致是支持的，但 kubelet 的版本不可以超过 API 服务器的版本。 例如，1.7.0 版本的 kubelet 可以完全兼容 1.8.0 版本的 API 服务器，反之则不可以。

[安装配置kubectl](https://kubernetes.io/zh/docs/tasks/tools/)

> 警告：  
> 这些指南不包括系统升级时使用的所有 Kubernetes 程序包。这是因为 kubeadm 和 Kubernetes 有特殊的升级注意事项。

关于版本偏差的更多信息，请参阅以下文档：
* [Kubernetes 版本与版本间的偏差策略](https://kubernetes.io/zh/docs/setup/release/version-skew-policy/)
* [Kubeadm 特定的版本偏差策略](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

##### 环境配置
修改主机名，以下配置需要在所有的主机上执行，这里是在master结点执行。
```shell
# 查看 hostname
hostname
# 修改 hostname
hostnamectl set-hostname k8s-master
```
编辑`/etc/hosts` 文件，添加域名解析：
```shell
cat <<EOF>>/etc/hosts
192.163.199.129 k8s-master
192.168.199.130 k8s-node-1
192.168.199.131 k8s-node-2
EOF
```
```shell
cat /etc/hosts
```
> 这里IP地址根据自己物理机或者虚拟机的真实IP而定。

关闭防火墙,swap,和selinux。
```shell
sudo systemctl stop firewalld
sudo systemctl disable firewalld

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
sudo cat /etc/selinux/config

sudo swapoff -a
sudo sed -i 's/.*swap.*/#&/' /etc/fstab

```
配置国内yum源：
```shell
# 备份
mkdir /etc/yum.repos.d/bak && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
 
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
yum clean all 
```
配置国内kubernetes源：
```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
 
[kubernetes]
 
name=Kubernetes
 
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
 
enabled=1
 
gpgcheck=1
 
repo_gpgcheck=1
 
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
配置Docker源：
```shell
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```
注意：
* 通过运行命令 setenforce 0 和 sed ... 将 SELinux 设置为 permissive 模式 可以有效地将其禁用。 这是允许容器访问主机文件系统所必需的，而这些操作时为了例如 Pod 网络工作正常。
* 你必须这么做，直到 kubelet 做出对 SELinux 的支持进行升级为止。
* 如果你知道如何配置 SELinux 则可以将其保持启用状态，但可能需要设定 kubeadm 不支持的部分配置。

##### 安装命令
```shell

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

kubelet 现在每隔几秒就会重启，因为它陷入了一个等待 kubeadm 指令的死循环。

> 到这里 Kubeadm 安装成功，下面我们就使用 Kubeadm 创建集群。

### 对 kubeadm 进行故障排查
### 使用 kubeadm 创建集群
使用 `kubeadm`，你能创建一个符合最佳实践的最小化 Kubernetes 集群。事实上，你可以使用 kubeadm 配置一个通过 Kubernetes 一致性测试 的集群。 kubeadm 还支持其他集群生命周期功能， 例如 启动引导令牌 和集群升级。
kubeadm 工具很棒，如果你需要：
* 一个尝试 Kubernetes 的简单方法。
* 一个现有用户可以自动设置集群并测试其应用程序的途径。
* 其他具有更大范围的生态系统和/或安装工具中的构建模块。
  
你可以在各种机器上安装和使用 kubeadm：笔记本电脑， 一组云服务器，Raspberry Pi 等。无论是部署到云还是本地， 你都可以将 kubeadm 集成到预配置系统中，例如 Ansible 或 Terraform。
#### 准备开始
* 一台或多台运行兼容 deb/rpm 的 Linux 操作系统的计算机；例如：Ubuntu 或 CentOS。
* 每台机器 2 GB 以上的内存，内存不足时应用会受限制。
* 用作控制平面节点的计算机上至少有2个 CPU。
* 集群中所有计算机之间具有完全的网络连接。你可以使用公共网络或专用网络。

你还需要使用可以在新集群中部署特定 Kubernetes 版本对应的 kubeadm。

Kubernetes 版本及版本倾斜支持策略 适用于 kubeadm 以及整个 Kubernetes。 查阅该策略以了解支持哪些版本的 Kubernetes 和 kubeadm。 该页面是为 Kubernetes v1.22 编写的。

kubeadm 工具的整体功能状态为一般可用性（GA）。一些子功能仍在积极开发中。 随着工具的发展，创建集群的实现可能会略有变化，但总体实现应相当稳定。
#### 目标
* 安装单个控制平面的 Kubernetes 集群
* 在集群上安装 Pod 网络，以便你的 Pod 可以相互连通

#### 操作指南
> 如果你已经安装了`kubeadm`，执行 `apt-get update` && `apt-get upgrade` 或 `yum update` 以获取 kubeadm 的最新版本。
>  
> 升级时，`kubelet` 每隔几秒钟重新启动一次， 在 crashloop 状态中等待 kubeadm 发布指令。crashloop 状态是正常现象。 初始化控制平面后，kubelet 将正常运行。

#### 准备所需的容器镜像
这个步骤是可选的，只适用于你希望 `kubeadm init` 和 `kubeadm join` 不去下载存放在 `k8s.gcr.io` 上的默认的容器镜像的情况。

当你在离线的节点上创建一个集群的时候，Kubeadm 有一些命令可以帮助你预拉取所需的镜像。 阅读[离线运行 kubeadm](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/docs/reference/setup-tools/kubeadm/kubeadm-init#custom-images) 获取更多的详情。

Kubeadm 允许你给所需要的镜像指定一个自定义的镜像仓库。 阅读使用[自定义镜像](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/docs/reference/setup-tools/kubeadm/kubeadm-init#custom-images) 获取更多的详情。

#### 初始化控制平面节点
控制平面节点是运行控制平面组件的机器， 包括 etcd （集群数据库） 和 API Server （命令行工具 kubectl 与之通信）。
1. （推荐）如果计划将单个控制平面 kubeadm 集群升级成高可用， 你应该指定 --control-plane-endpoint 为所有控制平面节点设置共享端点。 端点可以是负载均衡器的 DNS 名称或 IP 地址。
2. 选择一个 Pod 网络插件，并验证是否需要为 kubeadm init 传递参数。 根据你选择的第三方网络插件，你可能需要设置 --pod-network-cidr 的值。 请参阅 安装Pod网络附加组件。
3. （可选）从版本1.14开始，kubeadm 尝试使用一系列众所周知的域套接字路径来检测 Linux 上的容器运行时。 要使用不同的容器运行时， 或者如果在预配置的节点上安装了多个容器，请为 kubeadm init 指定 --cri-socket 参数。 请参阅安装运行时。
4. （可选）除非另有说明，否则 kubeadm 使用与默认网关关联的网络接口来设置此控制平面节点 API server 的广播地址。 要使用其他网络接口，请为 kubeadm init 设置 --apiserver-advertise-address=<ip-address> 参数。 要部署使用 IPv6 地址的 Kubernetes 集群， 必须指定一个 IPv6 地址，例如 --apiserver-advertise-address=fd00::101

要初始化控制平面节点，请运行：
```shell
kubeadm init <args>
```
#### 关于 apiserver-advertise-address 和 ControlPlaneEndpoint 的注意事项

`--apiserver-advertise-address` 可用于为控制平面节点的 API server 设置广播地址，`--control-plane-endpoint` 可用于为所有控制平面节点设置共享端点。

`--control-plane-endpoint` 允许 IP 地址和可以映射到 IP 地址的 DNS 名称。 请与你的网络管理员联系，以评估有关此类映射的可能解决方案。

这是一个示例映射：
```shell
192.168.0.102 cluster-endpoint
```
其中 `192.168.0.102` 是此节点的 IP 地址，`cluster-endpoint` 是映射到该 IP 的自定义 DNS 名称。 这将允许你将 `--control-plane-endpoint=cluster-endpoint` 传递给 `kubeadm init`,并将相同的 DNS 名称传递给 `kubeadm join`。 稍后你可以修改 `cluster-endpoint` 以指向高可用性方案中的负载均衡器的地址。kubeadm 不支持将没有 `--control-plane-endpoint` 参数的单个控制平面集群转换为高可用性集群。


#### Kubernetes 常用命令
* kubeadm init 用于搭建控制平面节点
* kubeadm join 用于搭建工作节点并将其加入到集群中
* kubeadm upgrade 用于升级 Kubernetes 集群到新版本
* kubeadm config 如果你使用了 v1.7.x 或更低版本的 kubeadm 版本初始化你的集群，则使用 kubeadm upgrade 来配置你的集群
* kubeadm token 用于管理 kubeadm join 使用的令牌
* kubeadm reset 用于恢复通过 kubeadm init 或者 kubeadm join 命令对节点进行的任何变更
* kubeadm certs 用于管理 Kubernetes 证书
* kubeadm kubeconfig 用于管理 kubeconfig 文件
* kubeadm version 用于打印 kubeadm 的版本信息
* kubeadm alpha 用于预览一组可用于收集社区反馈的特性

要再次运行 kubeadm init，你必须首先[卸载集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#tear-down) 。

如果将具有不同架构的节点加入集群， 请确保已部署的 DaemonSet 对这种体系结构具有容器镜像支持。

`kubeadm init` 首先运行一系列预检查以确保机器 准备运行 Kubernetes。这些预检查会显示警告并在错误时退出。然后 `kubeadm init` 下载并安装集群控制平面组件。这可能会需要几分钟。 完成之后你应该看到：

```shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
要使非 root 用户可以运行 `kubectl`，请运行以下命令， 它们也是 `kubeadm init` 输出的一部分：

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
或者，如果你是 `root` 用户，则可以运行：
```shell
export KUBECONFIG=/etc/kubernetes/admin.conf
```
> kubeadm 对 admin.conf 中的证书进行签名时，将其配置为 Subject: O = system:masters, CN = kubernetes-admin。 system:masters 是一个例外的、超级用户组，可以绕过鉴权层（例如 RBAC）。 不要将 admin.conf 文件与任何人共享，应该使用 kubeadm kubeconfig user 命令为其他用户生成 kubeconfig 文件，完成对他们的定制授权。

记录 `kubeadm init` 输出的 `kubeadm join` 命令。 你需要此命令将节点[加入集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes) 。

令牌用于控制平面节点和加入节点之间的相互身份验证。 这里包含的令牌是密钥。确保它的安全， 因为拥有此令牌的任何人都可以将经过身份验证的节点添加到你的集群中。 可以使用 `kubeadm token` 命令列出，创建和删除这些令牌。 请参阅 `kubeadm` 参考指南。

#### 安装 Pod 网络附加组件
你可以使用以下命令在控制平面节点或具有 kubeconfig 凭据的节点上安装 Pod 网络附加组件：

```shell
kubectl apply -f <add-on.yaml>
```
每个集群只能安装一个 Pod 网络。

安装 Pod 网络后，您可以通过在 `kubectl get pods --all-namespaces` 输出中检查 CoreDNS Pod 是否 Running 来确认其是否正常运行。 一旦 `CoreDNS Pod` 启用并运行，你就可以继续加入节点。

如果您的网络无法正常工作或 CoreDNS 不在“运行中”状态，请查看 kubeadm 的 故障排除指南。

#### 控制平面节点隔离
默认情况下，出于安全原因，你的集群不会在控制平面节点上调度 Pod。 如果你希望能够在控制平面节点上调度 Pod， 例如用于开发的单机 Kubernetes 集群，请运行：

```shell
kubectl taint nodes --all node-role.kubernetes.io/master-
```
输出看起来像：
```shell
node "test-01" untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```
这将从任何拥有 `node-role.kubernetes.io/master` taint 标记的节点中移除该标记， 包括控制平面节点，这意味着调度程序将能够在任何地方调度 Pods。

#### 加入节点
节点是你的工作负载（容器和 Pod 等）运行的地方。要将新节点添加到集群，请对每台计算机执行以下操
* SSH 到机器
* 成为 root （例如 sudo su -）
* 运行 kubeadm init 输出的命令。例如：

```shell
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```
如果没有令牌，可以通过在控制平面节点上运行以下命令来获取令牌：

```shell
kubeadm token list
```
```
TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                   signing          token generated by     bootstrappers:
                                                                    'kubeadm init'.        kubeadm:
                                                                                           default-node-token
```
默认情况下，令牌会在24小时后过期。如果要在当前令牌过期后将节点加入集群， 则可以通过在控制平面节点上运行以下命令来创建新令牌：

```shell
kubeadm token create
```

输出类似于以下内容：
```shell
5didvk.d09sbcov8ph2amjw
```
如果你没有 `--discovery-token-ca-cert-hash` 的值，则可以通过在控制平面节点上执行以下命令链来获取它：

````shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
````
输出类似于以下内容：
```shell
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```

> 说明： 要为 <control-plane-host>:<control-plane-port> 指定 IPv6 元组，必须将 IPv6 地址括在方括号中，例如：[fd00::101]:2073

```shell
[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to control-plane and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on control-plane to see this machine join.
```
几秒钟后，当你在控制平面节点上执行 kubectl get nodes，你会注意到该节点出现在输出中。

#### （可选）从控制平面节点以外的计算机控制集群
为了使 kubectl 在其他计算机（例如笔记本电脑）上与你的集群通信， 你需要将管理员 kubeconfig 文件从控制平面节点复制到工作站，如下所示：

```shell
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```
> 上面的示例假定为 root 用户启用了SSH访问。如果不是这种情况， 你可以使用 scp 将 admin.conf 文件复制给其他允许访问的用户。

> admin.conf 文件为用户提供了对集群的超级用户特权。 该文件应谨慎使用。对于普通用户，建议生成一个你为其授予特权的唯一证书。 你可以使用 kubeadm alpha kubeconfig user --client-name <CN> 命令执行此操作。 该命令会将 KubeConfig 文件打印到 STDOUT，你应该将其保存到文件并分发给用户。 之后，使用 kubectl create (cluster)rolebinding 授予特权。

#### （可选）将API服务器代理到本地主机
如果要从集群外部连接到 API 服务器，则可以使用 kubectl proxy：
```shell
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```

你现在可以在本地访问API服务器 http://localhost:8001/api/v1

#### 清理
如果你在集群中使用了一次性服务器进行测试，则可以关闭这些服务器，而无需进一步清理。你可以使用 `kubectl config delete-cluster` 删除对集群的本地引用。

但是，如果要更干净地取消配置群集， 则应首先清空节点并确保该节点为空， 然后取消配置该节点。

#### 删除节点
使用适当的凭证与控制平面节点通信，运行：
```shell
kubectl drain <node name> --delete-emptydir-data --force --ignore-daemonsets
```
在删除节点之前，请重置 kubeadm 安装的状态：
```shell
kubeadm reset
```
重置过程不会重置或清除 iptables 规则或 IPVS 表。如果你希望重置 iptables，则必须手动进行：

```shell
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```
如果要重置 IPVS 表，则必须运行以下命令：
```shell
ipvsadm -C
```
现在删除节点：
```shell
kubectl delete node <node name>
```
如果你想重新开始，只需运行 `kubeadm init` 或 `kubeadm join` 并加上适当的参数。
#### 清理控制平面
你可以在控制平面主机上使用 `kubeadm reset` 来触发尽力而为的清理。

有关此子命令及其选项的更多信息，请参见[kubeadm reset参考文档](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-reset/) 。
#### 下一步
* 使用 Sonobuoy 验证集群是否正常运行。
* 有关使用 kubeadm 升级集群的详细信息，请参阅升级 kubeadm 集群。
* 在 kubeadm 参考文档中了解有关高级 kubeadm 用法的信息。
* 了解有关 Kubernetes 概念和 kubectl 的更多信息。
* 有关 Pod 网络附加组件的更多列表，请参见集群网络页面。
* 请参阅附加组件列表以探索其他附加组件， 包括用于 Kubernetes 集群的日志记录，监视，网络策略，可视化和控制的工具。
* 配置集群如何处理集群事件的日志以及 在 Pods 中运行的应用程序。 有关所涉及内容的概述，请参见日志架构。

### 使用 kubeadm API 定制组件
### 高可用拓扑选项
### 利用 kubeadm 创建高可用集群
### 使用 kubeadm 创建一个高可用 etcd 集群
### 使用 kubeadm 配置集群中的每个 kubelet

## 参考
* [官网地址](https://kubernetes.io/)
* [Kubernetes文档](https://kubernetes.io/docs/home/)
* [使用Minikube创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)
* [交互式教程-创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)
* [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
* [交互式教程-部署应用程序](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/)
* [交互式教程-探索你的应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-interactive/)
* [Kubernetes 中文网](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/)
``
*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
