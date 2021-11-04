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

# 注释掉 swap 行
sudo cat /etc/fstab

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
查看默认网卡和IP：
```shell
ip route show

ip addr
```
示例：
```shell
kubeadm init \
--apiserver-advertise-address=192.168.199.129 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.3 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=10.244.0.0/16 \
--ignore-preflight-errors=all
```
初始哈失败：
```shell
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
```

```shell
# 方案一
vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"
# 方案二
vi /usr/lib/systemd/system/docker.service
# 在这一行
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# 添加
--exec-opt native.cgroupdriver=systemd

# 随后先删除初始化导致的错误文件夹
rm -rf /etc/kubernetes/manifests
# 然后重启docker
systemctl daemon-reload
systemctl restart docker
```
重新安装准备：
```shell
sudo kubeadm reset

rm -rf $HOME/.kube/config/
rm -rf /etc/cni/net.d
rm -rf /etc/kubernetes/

```
安装成功：
```shell
[root@k8s-master k8s]# kubeadm init \
> --apiserver-advertise-address=192.168.199.129 \
> --image-repository registry.aliyuncs.com/google_containers \
> --kubernetes-version v1.22.3 \
> --service-cidr=10.96.0.0/16 \
> --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.22.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.199.129]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.199.129 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.199.129 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 10.004729 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 70w5nh.u38kykr4057gt369
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.199.129:6443 --token 70w5nh.u38kykr4057gt369 \
	--discovery-token-ca-cert-hash sha256:ad6428f299fa5a190e79349a7faa20e6644d3b79e93db33c42d079fb17bb790f
```
根据提示信息完成相应的操作，包括pod network，join信息，权限设置。


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

下面我们结合自己应用安装，安裝应用命令如下：
```shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# 卸载命令
kubectl delete -f "delete"
```
效果如下：
```shell
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

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

在加入结点之前，可以使用命令查看当前节点状态。
```shell
# 使用该命令查找不到，需要名称空间
kubectl get pods
No resources found in default namespace.

# 查看名称空间
kubectl get ns
NAME              STATUS   AGE
default           Active   74m
kube-node-lease   Active   74m
kube-public       Active   74m
kube-system       Active   74m

# 直接获取所有
kubectl get pods --all-namespaces
kube-system   coredns-7f6cbbb7b8-4fv72             0/1     ContainerCreating   0             77m
kube-system   coredns-7f6cbbb7b8-rdb26             0/1     ContainerCreating   0             77m
kube-system   etcd-k8s-master                      1/1     Running             0             77m
kube-system   kube-apiserver-k8s-master            1/1     Running             0             77m
kube-system   kube-controller-manager-k8s-master   1/1     Running             0             77m
kube-system   kube-proxy-h8msq                     1/1     Running             0             77m
kube-system   kube-scheduler-k8s-master            1/1     Running             0             77m
kube-system   weave-net-tg5q8                      2/2     Running             1 (17m ago)   20m
```
加入其它结点命令（内容为kubeadm init 输出）：
```shell
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```
示例：
```shell
# 查看节点状态
kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
k8s-master   Ready    control-plane,master   81m   v1.22.3
# 在结点机器运行
kubeadm join 192.168.199.129:6443 --token 70w5nh.u38kykr4057gt369 \
	--discovery-token-ca-cert-hash sha256:ad6428f299fa5a190e79349a7faa20e6644d3b79e93db33c42d079fb17bb790f
```
> 添加的结点名称记得更改，如果你是使用的虚拟机直接复制创建，hostname 会控制节点一样，使用下面命令更改：  
>     ```hostnamectl set-hostname k8s-node1```

加入节点成功输出如下：
```shell
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

在master结点使用 kubectl get nodes 查看情况：
```shell
kubectl get nodes
NAME         STATUS     ROLES                  AGE     VERSION
k8s-master   Ready      control-plane,master   99m     v1.22.3
k8s-node1    NotReady   <none>                 26s     v1.22.3
k8s-node2    NotReady   <none>                 2m11s   v1.22.3
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

查看详细信息：
```shell
kubectl get pod -n kube-system -o wide
NAME                                 READY   STATUS                  RESTARTS      AGE    IP                NODE         NOMINATED NODE   READINESS GATES
coredns-7f6cbbb7b8-4fv72             0/1     ContainerCreating       0             119m   <none>            k8s-master   <none>           <none>
coredns-7f6cbbb7b8-rdb26             0/1     ContainerCreating       0             119m   <none>            k8s-master   <none>           <none>
etcd-k8s-master                      1/1     Running                 0             119m   192.168.199.129   k8s-master   <none>           <none>
kube-apiserver-k8s-master            1/1     Running                 0             119m   192.168.199.129   k8s-master   <none>           <none>
kube-controller-manager-k8s-master   1/1     Running                 0             119m   192.168.199.129   k8s-master   <none>           <none>
kube-proxy-h8msq                     1/1     Running                 0             119m   192.168.199.129   k8s-master   <none>           <none>
kube-proxy-l5szl                     1/1     Running                 1 (12m ago)   22m    192.168.199.131   k8s-node2    <none>           <none>
kube-proxy-sr65d                     1/1     Running                 0             20m    192.168.199.130   k8s-node1    <none>           <none>
kube-scheduler-k8s-master            1/1     Running                 0             119m   192.168.199.129   k8s-master   <none>           <none>
weave-net-7bzz8                      0/2     Init:ImagePullBackOff   0             20m    192.168.199.130   k8s-node1    <none>           <none>
weave-net-jdfwf                      2/2     Running                 1 (16m ago)   22m    192.168.199.131   k8s-node2    <none>           <none>
weave-net-tg5q8                      2/2     Running                 1 (59m ago)   62m    192.168.199.129   k8s-master   <none>           <none>
```
看状态中，有的服务在运行，有的服务存在初始化，有的在创建，等他们创建完成，并没有发生错误，一个简单的集群就搭建好了。剩下的就可以使用集群。
master节点安装的镜像如下：
```shell
docker images
REPOSITORY                                           TAG       IMAGE ID       CREATED        SIZE
registry.aliyuncs.com/google_containers/kube-apiserver            v1.22.3         53224b502ea4   7 days ago     128MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.22.3         0aa9c7e31d30   7 days ago     52.7MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.22.3         05c905cef780   7 days ago     122MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.22.3         6120bd723dce   7 days ago     104MB
registry.aliyuncs.com/google_containers/etcd                      3.5.0-0         004811815584   4 months ago   295MB
registry.aliyuncs.com/google_containers/coredns                   v1.8.4          8d147537fb7d   5 months ago   47.6MB
registry.aliyuncs.com/google_containers/pause                     3.5             ed210e3e4a5b   7 months ago   683kB
weaveworks/weave-npc                                              2.8.1           7f92d556d4ff   9 months ago   39.3MB
weaveworks/weave-kube                                             2.8.1           df29c0a4002c   9 months ago   89MB
quay.io/coreos/flannel                                            v0.11.0-amd64   ff281650a721   2 years ago    52.6MB
```
node节点安装的镜像如下：
```shell
REPOSITORY                                           TAG       IMAGE ID       CREATED        SIZE
registry.aliyuncs.com/google_containers/kube-proxy   v1.22.3   6120bd723dce   7 days ago     104MB
registry.aliyuncs.com/google_containers/pause        3.5       ed210e3e4a5b   7 months ago   683kB
weaveworks/weave-npc                                 2.8.1     7f92d556d4ff   9 months ago   39.3MB
weaveworks/weave-kube                                2.8.1     df29c0a4002c   9 months ago   89MB

```
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
```shell
kubectl config delete-cluster
```
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
本页面介绍了如何自定义 `kubeadm` 部署的组件。 你可以使用 `ClusteConfiguration` 结构中定义的参数，或者在每个节点上应用补丁来定制控制平面组件。 你可以使用 `KubeletConfiguration` 和 `KubeProxyConfiguration` 结构分别定制 `kubelet` 和 `kube-proxy` 组件。

所有这些选项都可以通过 kubeadm 配置 API 实现。 有关配置中的每个字段的详细信息，你可以导航到我们的 API 参考页面 。

> 说明：  
> kubeadm 目前不支持对 CoreDNS 部署进行定制。 你必须手动更新 kube-system/coredns ConfigMap 并在更新后重新创建 CoreDNS Pods。 或者，你可以跳过默认的 CoreDNS 部署并部署你自己的 CoreDNS 变种。 有关更多详细信息，请参阅在 kubeadm 中使用 init phases.

kubeadm ClusterConfiguration 对象为用户提供了一种方法， 用以覆盖传递给控制平面组件（如 APIServer、ControllerManager、Scheduler 和 Etcd）的默认参数。 各组件配置使用如下字段定义：
* apiServer
* controllerManager
* scheduler
* etcd
  

这些结构包含一个通用的 `extraArgs` 字段，该字段由 `key: value` 组成。 要覆盖控制平面组件的参数：
* 将适当的字段 `extraArgs` 添加到配置中。
* 向字段 `extraArgs` 添加要覆盖的参数值。
* 用 `--config <YOUR CONFIG YAML>` 运行 `kubeadm init`。

> 说明：  
> 你可以通过运行 `kubeadm config print init-defaults` 并将输出保存到你所选的文件中， 以默认值形式生成 `ClusterConfiguration` 对象。

> 说明：  
> ClusterConfiguration 对象目前在 kubeadm 集群中是全局的。 这意味着你添加的任何标志都将应用于同一组件在不同节点上的所有实例。 要在不同节点上为每个组件应用单独的配置，您可以使用补丁。

> 说明：  
> 当前不支持重复的参数（keys）或多次传递相同的参数 --foo。 要解决此问题，你必须使用补丁。
#### APIServer 参数
有关详细信息，请参阅 [kube-apiserver 参考文档](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) 。

使用示例：

```properties
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
apiServer:
  extraArgs:
    anonymous-auth: "false"
    enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
    audit-log-path: /home/johndoe/audit.log
```
#### ControllerManager 参数
有关详细信息，请参阅 [kube-controller-manager 参考文档](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) 。

使用示例：

```properties
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
controllerManager:
  extraArgs:
    cluster-signing-key-file: /home/johndoe/keys/ca.key
    deployment-controller-sync-period: "50"
```
Scheduler 参数:

有关详细信息，请参阅 [kube-scheduler 参考文档](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) 。

使用示例：

```properties
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
scheduler:
  extraArgs:
    config: /etc/kubernetes/scheduler-config.yaml
  extraVolumes:
    - name: schedulerconfig
      hostPath: /home/johndoe/schedconfig.yaml
      mountPath: /etc/kubernetes/scheduler-config.yaml
      readOnly: true
      pathType: "File"
```

#### Etcd 参数

使用示例：

```properties
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
etcd:
  local:
    extraArgs:
      election-timeout: 1000
```

#### 使用补丁定制控制平面

Kubeadm 允许将包含补丁文件的目录传递给各个节点上的 InitConfiguration 和 JoinConfiguration。 这些补丁可被用作控制平面组件清单写入磁盘之前的最后一个自定义步骤。

可以使用 --config <你的 YAML 格式控制文件> 将配置文件传递给 kubeadm init：

```properties
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
nodeRegistration:
  patches:
    directory: /home/user/somedir
```

> 说明：  
> 对于 kubeadm init，你可以传递一个包含 ClusterConfiguration 和 InitConfiguration 的文件，以 --- 分隔。

你可以使用 --config <你的 YAML 格式配置文件> 将配置文件传递给 kubeadm join：

### 高可用拓扑选项

本页面介绍了配置高可用（HA） Kubernetes 集群拓扑的两个选项。

您可以设置 HA 集群：
* 使用堆叠（stacked）控制平面节点，其中 etcd 节点与控制平面节点共存
* 使用外部 etcd 节点，其中 etcd 在与控制平面不同的节点上运行

在设置 HA 集群之前，您应该仔细考虑每种拓扑的优缺点。

#### 堆叠（Stacked） etcd 拓扑
堆叠（Stacked） HA 集群是一种这样的拓扑，其中 etcd 分布式数据存储集群堆叠在 kubeadm 管理的控制平面节点上，作为控制平面的一个组件运行。

每个控制平面节点运行 `kube-apiserver`，`kube-scheduler` 和 `kube-controller-manager` 实例。

`kube-apiserver` 使用负载均衡器暴露给工作节点。

每个控制平面节点创建一个本地 etcd 成员（member），这个 etcd 成员只与该节点的 kube-apiserver 通信。这同样适用于本地 `kube-controller-manager` 和 `kube-scheduler` 实例。

这种拓扑将控制平面和 etcd 成员耦合在同一节点上。相对使用外部 etcd 集群，设置起来更简单，而且更易于副本管理。

然而，堆叠集群存在耦合失败的风险。如果一个节点发生故障，则 etcd 成员和控制平面实例都将丢失，并且冗余会受到影响。您可以通过添加更多控制平面节点来降低此风险。

因此，您应该为 HA 集群运行至少三个堆叠的控制平面节点。

这是 kubeadm 中的默认拓扑。当使用 `kubeadm init` 和 `kubeadm join --control-plane` 时，在控制平面节点上会自动创建本地 etcd 成员。

![pc5](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/kubeadm-ha-topology-stacked-etcd.svg)

#### 外部 etcd 拓扑

具有外部 etcd 的 HA 集群是一种这样的拓扑，其中 etcd 分布式数据存储集群在独立于控制平面节点的其他节点上运行。

就像堆叠的 etcd 拓扑一样，外部 etcd 拓扑中的每个控制平面节点都运行 `kube-apiserver`，`kube-scheduler` 和 `kube-controller-manager` 实例。同样， `kube-apiserver` 使用负载均衡器暴露给工作节点。但是，etcd 成员在不同的主机上运行，每个 etcd 主机与每个控制平面节点的 `kube-apiserver` 通信。

这种拓扑结构解耦了控制平面和 etcd 成员。因此，它提供了一种 HA 设置，其中失去控制平面实例或者 etcd 成员的影响较小，并且不会像堆叠的 HA 拓扑那样影响集群冗余。

但是，此拓扑需要两倍于堆叠 HA 拓扑的主机数量。

具有此拓扑的 HA 集群至少需要三个用于控制平面节点的主机和三个用于 etcd 节点的主机。

![pc6](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/kubeadm-ha-topology-external-etcd.svg)

### 利用 kubeadm 创建高可用集群
本文讲述了使用 kubeadm 设置一个高可用的 Kubernetes 集群的两种不同方式：

* 使用具有堆叠的控制平面节点。这种方法所需基础设施较少。etcd 成员和控制平面节点位于同一位置。
* 使用外部集群。这种方法所需基础设施较多。控制平面的节点和 etcd 成员是分开的。
  
在下一步之前，你应该仔细考虑哪种方法更好的满足你的应用程序和环境的需求。 这是[对比文档](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/ha-topology/) 讲述了每种方法的优缺点。

这篇文档没有讲述在云提供商上运行集群的问题。在云环境中，此处记录的方法不适用于类型为 LoadBalancer 的服务对象，或者具有动态的 PersistentVolumes。

#### 准备开始  
对于这两种方法，你都需要以下基础设施：
* 配置满足 kubeadm 的最低要求 的三台机器作为控制面节点
* 配置满足 kubeadm 的最低要求 的三台机器作为工作节点
* 在集群中，确保所有计算机之间存在全网络连接（公网或私网）
* 在所有机器上具有 sudo 权限
* 从某台设备通过 SSH 访问系统中所有节点的能力
* 所有机器上已经安装 kubeadm 和 kubelet，kubectl 是可选的。
  
仅对于外部 etcd 集群来说，你还需要：
* 给 etcd 成员使用的另外三台机器

#### 这两种方法的第一步
##### 为 kube-apiserver 创建负载均衡器
> *说明：*  
> 使用负载均衡器需要许多配置。你的集群搭建可能需要不同的配置。 下面的例子只是其中的一方面配置。

##### 1. 创建一个名为 kube-apiserver 的负载均衡器解析 DNS。 

* 在云环境中，应该将控制平面节点放置在 TCP 后面转发负载平衡。该负载均衡器将流量分配给目标列表中所有运行状况良好的控制平面节点。 API 服务器的健康检查是在 kube-apiserver 的监听端口（默认值 :6443） 上进行的一个 TCP 检查。
* 不建议在云环境中直接使用 IP 地址。
* 负载均衡器必须能够在 API 服务器端口上与所有控制平面节点通信。 它还必须允许其监听端口的入站流量。
* 确保负载均衡器的地址始终匹配 kubeadm 的 ControlPlaneEndpoint 地址。

##### 2. 添加第一个控制平面节点到负载均衡器并测试连接：
```shell
nc -v LOAD_BALANCER_IP PORT
```
* 由于 apiserver 尚未运行，预期会出现一个连接拒绝错误。 然而超时意味着负载均衡器不能和控制平面节点通信。 如果发生超时，请重新配置负载均衡器与控制平面节点进行通信。

##### 3. 将其余控制平面节点添加到负载均衡器目标组。

#### 使用堆控制平面和 etcd 节点
##### 1. 控制平面节点的第一步
初始化控制平面：
```shell
sudo kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" --upload-certs
```
* 你可以使用 `--kubernetes-version` 标志来设置要使用的 `Kubernetes` 版本。建议将 `kubeadm`、`kebelet`、`kubectl` 和 `Kubernetes` 的版本匹配。
* 这个 `--control-plane-endpoint` 标志应该被设置成负载均衡器的地址或 DNS 和端口。
* 这个 `--upload-certs` 标志用来将在所有控制平面实例之间的共享证书上传到集群。 如果正好相反，你更喜欢手动地通过控制平面节点或者使用自动化 工具复制证书，请删除此标志并参考如下部分证书分配手册。

> 说明：  
> 标志 `kubeadm init`、`--config` 和 `--certificate-key` 不能混合使用，
> 因此如果你要使用
> [kubeadm 配置](/docs/reference/config-api/kubeadm-config.v1beta3/)，你必须在相应的配置文件
> （位于 `InitConfiguration` 和 `JoinConfiguration: controlPlane`）添加 `certificateKey` 字段。

> 说明：  
> 一些 CNI 网络插件如 Calico 需要 CIDR 例如 `192.168.0.0/16` 和一些像 Weave 没有。参考
> [CNI 网络文档](/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)。
> 通过传递 `--pod-network-cidr` 标志添加 pod CIDR，或者你可以使用 kubeadm
> 配置文件，在 `ClusterConfiguration` 的 `networking` 对象下设置 `podSubnet` 字段。

输出类似于：
```
You can now join any number of control-plane node by running the following command on each as a root:
kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use kubeadm init phase upload-certs to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:
  kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```
将此输出复制到文本文件。 稍后你将需要它来将控制平面节点和工作节点加入集群。

当 `--upload-certs` 与 `kubeadm init` 一起使用时，主控制平面的证书 被加密并上传到 `kubeadm-certs` Secret 中。

要重新上传证书并生成新的解密密钥，请在已加入集群节点的控制平面上使用以下命令：

```shell
sudo kubeadm init phase upload-certs --upload-certs
```
你还可以在 `init` 期间指定自定义的 `--certificate-key`，以后可以由 `join` 使用。 要生成这样的密钥，可以使用以下命令：

```shell
kubeadm certs certificate-key
```
> 说明：`kubeadm-certs` 密钥和解密密钥会在两个小时后失效。

> 注意：正如命令输出中所述，证书密钥可访问群集敏感数据。请妥善保管！

应用你所选择的 CNI 插件： 请遵循以下指示 安装 CNI 提供程序。如果适用，请确保配置与 kubeadm 配置文件中指定的 Pod CIDR 相对应。

在此示例中，我们使用 Weave Net：

```shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

输入以下内容，并查看控制平面组件的 Pods 启动：

```shell
kubectl get pod -n kube-system -w
```

##### 2. 其余控制平面节点的步骤

> 说明：从 kubeadm 1.15 版本开始，你可以并行加入多个控制平面节点。 在此版本之前，你必须在第一个节点初始化后才能依序的增加新的控制平面节点。

对于每个其他控制平面节点，你应该：  
执行先前由第一个节点上的 kubeadm init 输出提供给你的 join 命令。 它看起来应该像这样：

```shell
sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07
```

这个 `--control-plane` 命令通知 `kubeadm join` 创建一个新的控制平面。  
`--certificate-key ...` 将导致从集群中的 `kubeadm-certs` Secret 下载 控制平面证书并使用给定的密钥进行解密。
#### 外部 etcd 节点
使用外部 etcd 节点设置集群类似于用于堆叠 etcd 的过程， 不同之处在于你应该首先设置 etcd，并在 kubeadm 配置文件中传递 etcd 信息。
##### 设置 ectd 集群
* 按照 这些指示 去设置 etcd 集群。
* 根据这里的描述配置 SSH。
* 将以下文件从集群中的任何 etcd 节点复制到第一个控制平面节点：
```shell
export CONTROL_PLANE="ubuntu@10.0.0.7"
scp /etc/kubernetes/pki/etcd/ca.crt "${CONTROL_PLANE}":
scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${CONTROL_PLANE}":
scp /etc/kubernetes/pki/apiserver-etcd-client.key "${CONTROL_PLANE}":
```
> 用第一台控制平面机的 user@host 替换 CONTROL_PLANE 的值。

##### 设置第一个控制平面节点
用以下内容创建一个名为 kubeadm-config.yaml 的文件：
```properties
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
etcd:
    external:
        endpoints:
        - https://ETCD_0_IP:2379
        - https://ETCD_1_IP:2379
        - https://ETCD_2_IP:2379
        caFile: /etc/kubernetes/pki/etcd/ca.crt
        certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
        keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
```

```text
这里的内部（stacked） etcd 和外部 etcd 之前的区别在于设置外部 etcd
需要一个 `etcd` 的 `external` 对象下带有 etcd 端点的配置文件。
如果是内部 etcd，是自动管理的。
```
在你的集群中，将配置模板中的以下变量替换为适当值：

* LOAD_BALANCER_DNS
* LOAD_BALANCER_PORT
* ETCD_0_IP
* ETCD_1_IP
* ETCD_2_IP

以下的步骤与设置内置 etcd 的集群是相似的：

* 在节点上运行 sudo kubeadm init --config kubeadm-config.yaml --upload-certs 命令。
* 记下输出的 join 命令，这些命令将在以后使用。
* 应用你选择的 CNI 插件。以下示例适用于 Weave Net：
```text
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
其他控制平面节点的步骤
* 确保第一个控制平面节点已完全初始化。
* 使用保存到文本文件的 join 命令将每个控制平面节点连接在一起。 建议一次加入一个控制平面节点。
* 不要忘记默认情况下，`--certificate-key` 中的解密秘钥会在两个小时后过期。

#### 列举控制平面之后的常见任务
安装工作节点, 你可以使用之前存储的 `kubeadm init` 命令的输出将工作节点加入集群中：

```shell
sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

#### 手动证书分发
##### 手动证书分发
如果你选择不将 `kubeadm init` 与 `--upload-certs` 命令一起使用， 则意味着你将必须手动将证书从主控制平面节点复制到 将要加入的控制平面节点上。

有许多方法可以实现这种操作。在下面的例子中我们使用 ssh 和 scp：

如果要在单独的一台计算机控制所有节点，则需要 SSH。

1. 在你的主设备上启用 ssh-agent，要求该设备能访问系统中的所有其他节点： 
    ```shell
    eval $(ssh-agent)
    ```
2. 将 SSH 身份添加到会话中：
    ```shell
    ssh-add ~/.ssh/path_to_private_key
    ```
3. 检查节点间的 SSH 以确保连接是正常运行的

    SSH 到任何节点时，请确保添加 -A 标志：
    ```shell
    ssh -A 10.0.0.7
    ```
    当在任何节点上使用 sudo 时，请确保保持环境变量设置，以便 SSH 转发能够正常工作：
    ```shell
    sudo -E -s
    ```
4. 在所有节点上配置 SSH 之后，你应该在运行过 kubeadm init 命令的第一个 控制平面节点上运行以下脚本。 该脚本会将证书从第一个控制平面节点复制到另一个控制平面节点：
   在以下示例中，用其他控制平面节点的 IP 地址替换 CONTROL_PLANE_IPS。
   ```shell
    USER=ubuntu # 可定制
    CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
    for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
    done
    ```
    注意：
    只需要复制上面列表中的证书。kubeadm 将负责生成其余证书以及加入控制平面实例所需的 SAN。 如果你错误地复制了所有证书，由于缺少所需的 SAN，创建其他节点可能会失败。

5. 然后，在每个即将加入集群的控制平面节点上，你必须先运行以下脚本，然后 再运行 kubeadm join。 该脚本会将先前复制的证书从主目录移动到 `/etc/kubernetes/pki`：

    ```shell
    USER=ubuntu # 可定制
    mkdir -p /etc/kubernetes/pki/etcd
    mv /home/${USER}/ca.crt /etc/kubernetes/pki/
    mv /home/${USER}/ca.key /etc/kubernetes/pki/
    mv /home/${USER}/sa.pub /etc/kubernetes/pki/
    mv /home/${USER}/sa.key /etc/kubernetes/pki/
    mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
    mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
    mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
    mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
    ```

### 使用 kubeadm 创建一个高可用 etcd 集群

> 说明：  
> 在本指南中，当 kubeadm 用作为外部 etcd 节点管理工具，请注意 kubeadm 不计划支持此类节点的证书更换或升级。对于长期规划是使用 etcdadm 增强工具来管理这方面。

默认情况下，kubeadm 运行单成员的 etcd 集群，该集群由控制面节点上的 kubelet 以静态 Pod 的方式进行管理。由于 etcd 集群只包含一个成员且不能在任一成员不可用时保持运行，所以这不是一种高可用设置。本任务，将告诉你如何在使用 kubeadm 创建一个 kubernetes 集群时创建一个外部 etcd：有三个成员的高可用 etcd 集群。

#### 准备开始
* 三个可以通过 `2379` 和 `2380` 端口相互通信的主机。本文档使用这些作为默认端口。不过，它们可以通过 kubeadm 的配置文件进行自定义。
* 每个主机必须 安装有 docker、kubelet 和 kubeadm。
* 一些可以用来在主机间复制文件的基础设施。例如 ssh 和 scp 就可以满足需求。

#### 建立集群
一般来说，是在一个节点上生成所有证书并且只分发这些必要的文件到其它节点上。
> kubeadm 包含生成下述证书所需的所有必要的密码学工具；在这个例子中，不需要其他加密工具。

将 kubelet 配置为 etcd 的服务管理器。
> 你必须在要运行 etcd 的所有主机上执行此操作。

由于 etcd 是首先创建的，因此你必须通过创建具有更高优先级的新文件来覆盖 kubeadm 提供的 kubelet 单元文件。

```shell
cat << EOF > /etc/systemd/system/kubelet.service.d/20-etcd-service-manager.conf
[Service]
ExecStart=
# 将下面的 "systemd" 替换为你的容器运行时所使用的 cgroup 驱动。
# kubelet 的默认值为 "cgroupfs"。
ExecStart=/usr/bin/kubelet --address=127.0.0.1 --pod-manifest-path=/etc/kubernetes/manifests --cgroup-driver=systemd
Restart=always
EOF

systemctl daemon-reload
systemctl restart kubelet
```
检查 kubelet 的状态以确保其处于运行状态：

```shell
systemctl status kubelet
```

为 kubeadm 创建配置文件。

```shell
# 使用 IP 或可解析的主机名替换 HOST0、HOST1 和 HOST2
export HOST0=10.0.0.6
export HOST1=10.0.0.7
export HOST2=10.0.0.8

# 创建临时目录来存储将被分发到其它主机上的文件
mkdir -p /tmp/${HOST0}/ /tmp/${HOST1}/ /tmp/${HOST2}/

ETCDHOSTS=(${HOST0} ${HOST1} ${HOST2})
NAMES=("infra0" "infra1" "infra2")

for i in "${!ETCDHOSTS[@]}"; do
HOST=${ETCDHOSTS[$i]}
NAME=${NAMES[$i]}
cat << EOF > /tmp/${HOST}/kubeadmcfg.yaml
apiVersion: "kubeadm.k8s.io/v1beta3"
kind: ClusterConfiguration
etcd:
    local:
        serverCertSANs:
        - "${HOST}"
        peerCertSANs:
        - "${HOST}"
        extraArgs:
            initial-cluster: infra0=https://${ETCDHOSTS[0]}:2380,infra1=https://${ETCDHOSTS[1]}:2380,infra2=https://${ETCDHOSTS[2]}:2380
            initial-cluster-state: new
            name: ${NAME}
            listen-peer-urls: https://${HOST}:2380
            listen-client-urls: https://${HOST}:2379
            advertise-client-urls: https://${HOST}:2379
            initial-advertise-peer-urls: https://${HOST}:2380
EOF
done
```
生成证书颁发机构：

如果你已经拥有 CA，那么唯一的操作是复制 CA 的 crt 和 key 文件到，`etc/kubernetes/pki/etcd/ca.crt` 和 `/etc/kubernetes/pki/etcd/ca.key`。复制完这些文件后继续下一步，“为每个成员创建证书”。

如果你还没有 CA，则在 `$HOST0`（你为 kubeadm 生成配置文件的位置）上运行此命令。

```shell
kubeadm init phase certs etcd-ca
```

这一操作创建如下两个文件：

```
/etc/kubernetes/pki/etcd/ca.crt

/etc/kubernetes/pki/etcd/ca.key
```
为每个成员创建证书：
```shell
kubeadm init phase certs etcd-server --config=/tmp/${HOST2}/kubeadmcfg.yaml
kubeadm init phase certs etcd-peer --config=/tmp/${HOST2}/kubeadmcfg.yaml
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
cp -R /etc/kubernetes/pki /tmp/${HOST2}/
# 清理不可重复使用的证书
find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

kubeadm init phase certs etcd-server --config=/tmp/${HOST1}/kubeadmcfg.yaml
kubeadm init phase certs etcd-peer --config=/tmp/${HOST1}/kubeadmcfg.yaml
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
cp -R /etc/kubernetes/pki /tmp/${HOST1}/
find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

kubeadm init phase certs etcd-server --config=/tmp/${HOST0}/kubeadmcfg.yaml
kubeadm init phase certs etcd-peer --config=/tmp/${HOST0}/kubeadmcfg.yaml
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
# 不需要移动 certs 因为它们是给 HOST0 使用的

# 清理不应从此主机复制的证书
find /tmp/${HOST2} -name ca.key -type f -delete
find /tmp/${HOST1} -name ca.key -type f -delete
```
复制证书和 kubeadm 配置：

证书已生成，现在必须将它们移动到对应的主机。
```shell
USER=ubuntu
HOST=${HOST1}
scp -r /tmp/${HOST}/* ${USER}@${HOST}:
ssh ${USER}@${HOST}
USER@HOST $ sudo -Es
root@HOST $ chown -R root:root pki
root@HOST $ mv pki /etc/kubernetes/
```

确保已经所有预期的文件都存在

$HOST0 所需文件的完整列表如下：
```text
/tmp/${HOST0}
└── kubeadmcfg.yaml
---
/etc/kubernetes/pki
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
└── etcd
    ├── ca.crt
    ├── ca.key
    ├── healthcheck-client.crt
    ├── healthcheck-client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key
```
在 $HOST1 上：

```text
$HOME
└── kubeadmcfg.yaml
---
/etc/kubernetes/pki
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
└── etcd
    ├── ca.crt
    ├── healthcheck-client.crt
    ├── healthcheck-client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key
```
在 $HOST2 上：
```text
$HOME
└── kubeadmcfg.yaml
---
/etc/kubernetes/pki
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
└── etcd
    ├── ca.crt
    ├── healthcheck-client.crt
    ├── healthcheck-client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key
```
创建静态 Pod 清单

既然证书和配置已经就绪，是时候去创建清单了。 在每台主机上运行 kubeadm 命令来生成 etcd 使用的静态清单。

```shell
root@HOST0 $ kubeadm init phase etcd local --config=/tmp/${HOST0}/kubeadmcfg.yaml
root@HOST1 $ kubeadm init phase etcd local --config=/tmp/${HOST1}/kubeadmcfg.yaml
root@HOST2 $ kubeadm init phase etcd local --config=/tmp/${HOST2}/kubeadmcfg.yaml
```

可选：检查群集运行状况

```shell
docker run --rm -it \
--net host \
-v /etc/kubernetes:/etc/kubernetes k8s.gcr.io/etcd:${ETCD_TAG} etcdctl \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--endpoints https://${HOST0}:2379 endpoint health --cluster
...
https://[HOST0 IP]:2379 is healthy: successfully committed proposal: took = 16.283339ms
https://[HOST1 IP]:2379 is healthy: successfully committed proposal: took = 19.44402ms
https://[HOST2 IP]:2379 is healthy: successfully committed proposal: took = 35.926451ms
```
将 ${ETCD_TAG} 设置为你的 etcd 镜像的版本标签，例如 3.4.3-0。 要查看 kubeadm 使用的 etcd 镜像和标签，请执行 kubeadm config images list --kubernetes-version ${K8S_VERSION}，例如，其中的 ${K8S_VERSION} 可以是 v1.17.0。

将 ${HOST0} 设置为要测试的主机的 IP 地址。

### 使用 kubeadm 配置集群中的每个 kubelet

kubeadm CLI 工具的生命周期与 kubelet 解耦；kubelet 是一个守护程序，在 Kubernetes 集群中的每个节点上运行。 当 Kubernetes 初始化或升级时，kubeadm CLI 工具由用户执行，而 kubelet 始终在后台运行。

由于kubelet是守护程序，因此需要通过某种初始化系统或服务管理器进行维护。 当使用 DEB 或 RPM 安装 kubelet 时，配置系统去管理 kubelet。 你可以改用其他服务管理器，但需要手动地配置。

集群中涉及的所有 kubelet 的一些配置细节都必须相同， 而其他配置方面则需要基于每个 kubelet 进行设置，以适应给定机器的不同特性（例如操作系统、存储和网络）。 你可以手动地管理 kubelet 的配置，但是 kubeadm 现在提供一种 `KubeletConfiguration` API 类型 用于[集中管理 kubelet 的配置](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/kubelet-integration/#configure-kubelets-using-kubeadm) 。

#### Kubelet 配置模式
以下各节讲述了通过使用 kubeadm 简化 kubelet 配置模式，而不是在每个节点上手动地管理 kubelet 配置。

##### 将集群级配置传播到每个 kubelet 中

你可以通过使用 `kubeadm init` 和 `kubeadm join` 命令为 kubelet 提供默认值。 有趣的示例包括使用其他 CRI 运行时或通过服务器设置不同的默认子网。

如果你想使用子网 `10.96.0.0/12` 作为services的默认网段，你可以给 kubeadm 传递 `--service-cidr` 参数：

```shell
kubeadm init --service-cidr 10.96.0.0/12
```

现在，可以从该子网分配服务的虚拟 IP。 你还需要通过 kubelet 使用 `--cluster-dns` 标志设置 DNS 地址。在集群中的每个管理器和节点上的 kubelet 的设置需要相同。 kubelet 提供了一个版本化的结构化 API 对象，该对象可以配置 kubelet 中的大多数参数，并将此配置推送到集群中正在运行的每个 kubelet 上。 此对象被称为 KubeletConfiguration。 KubeletConfiguration 允许用户指定标志，例如用骆峰值代表集群的 DNS IP 地址，如下所示：

##### 提供指定实例的详细配置信息
由于硬件、操作系统、网络或者其他主机特定参数的差异。某些主机需要特定的 kubelet 配置。 以下列表提供了一些示例。

* 由 kubelet 配置标志 --resolv-conf 指定的 DNS 解析文件的路径在操作系统之间可能有所不同， 它取决于你是否使用 systemd-resolved。 如果此路径错误，则在其 kubelet 配置错误的节点上 DNS 解析也将失败。

* 除非你使用云驱动，否则默认情况下 Node API 对象的 .metadata.name 会被设置为计算机的主机名。 如果你需要指定一个与机器的主机名不同的节点名称，你可以使用 --hostname-override 标志覆盖默认值。

* 当前，kubelet 无法自动检测 CRI 运行时使用的 cgroup 驱动程序， 但是值 --cgroup-driver 必须与 CRI 运行时使用的 cgroup 驱动程序匹配，以确保 kubelet 的健康运行状况。

* 取决于你的集群所使用的 CRI 运行时，你可能需要为 kubelet 指定不同的标志。 例如，当使用 Docker 时，你需要指定如 --network-plugin=cni 这类标志；但是如果你使用的是外部运行时， 则需要指定 --container-runtime=remote 并使用 --container-runtime-endpoint=<path> 指定 CRI 端点。

你可以在服务管理器（例如 systemd）中设定某个 kubelet 的配置来指定这些参数。

#### 使用 kubeadm 配置 kubelet

如果自定义的 `KubeletConfiguration` API 对象使用像 `kubeadm ... --config some-config-file.yaml` 这样的配置文件进行传递，则可以配置 kubeadm 启动的 kubelet。

通过调用 `kubeadm config print init-defaults --component-configs KubeletConfiguration`， 你可以看到此结构中的所有默认值。

##### 当使用 kubeadm init时的工作流程
当调用 kubeadm init 时，kubelet 配置被编组到磁盘上的 `/var/lib/kubelet/config.yaml` 中， 并且上传到集群中的 ConfigMap。 ConfigMap 名为 kubelet-config-1.X，其中 X 是你正在初始化的 kubernetes 版本的次版本。 在集群中所有 kubelet 的基准集群范围内配置，将 kubelet 配置文件写入`/etc/kubernetes/kubelet.conf` 中。此配置文件指向允许 kubelet 与 API 服务器通信的客户端证书。 这解决了将集群级配置传播到每个 kubelet 的需求。

```shell
cat /var/lib/kubelet/config.yaml

cat /etc/kubernetes/kubelet.conf
```
该文档 提供特定实例的配置详细信息 是第二种解决模式， kubeadm 将环境文件写入 `/var/lib/kubelet/kubeadm-flags.env`，其中包含了一个标志列表， 当 kubelet 启动时，该标志列表会传递给 kubelet 标志在文件中的显示方式如下：

```shell
KUBELET_KUBEADM_ARGS="--network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.5"
```

除了启动 kubelet 时使用该标志外，该文件还包含动态参数，例如 cgroup 驱动程序以及是否使用其他 CRI 运行时 socket（`--cri-socket`）。

将这两个文件编组到磁盘后，如果使用 systemd，则 kubeadm 尝试运行以下两个命令：

```shell
systemctl daemon-reload && systemctl restart kubelet
```
如果重新加载和重新启动成功，则正常的 kubeadm init 工作流程将继续。

##### 当使用 kubeadm join时的工作流程
当运行 `kubeadm join` 时，kubeadm 使用 Bootstrap Token 证书执行 TLS 引导，该引导会获取一份证书，该证书需要下载 `kubelet-config-1.X` ConfigMap 并把它写入 `/var/lib/kubelet/config.yaml` 中。 动态环境文件的生成方式恰好与 kubeadm init 相同。

接下来，kubeadm 运行以下两个命令将新配置加载到 kubelet 中：

```shell
systemctl daemon-reload && systemctl restart kubelet
```
在 kubelet 加载新配置后，kubeadm 将写入 `/etc/kubernetes/bootstrap-kubelet.conf` KubeConfig 文件中， 该文件包含 CA 证书和引导程序令牌。 kubelet 使用这些证书执行 TLS 引导程序并获取唯一的凭据，该凭据被存储在 `/etc/kubernetes/kubelet.conf` 中。 当此文件被写入后，kubelet 就完成了执行 TLS 引导程序。
```shell
cat /etc/kubernetes/bootstrap-kubelet.conf
```
#### kubelet 的 systemd 文件

`kubeadm` 中附带了有关系统如何运行 `kubelet` 的 `systemd` 配置文件。 请注意 kubeadm CLI 命令不会修改此文件。

通过 `kubeadm` DEB 或者 RPM 包 安装的配置文件被写入 `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 并由系统使用。 它对原来的 RPM 版本 `kubelet.service` 或者 DEB 版本 `kubelet.service` 作了增强：

```shell
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
--kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# 这是 "kubeadm init" 和 "kubeadm join" 运行时生成的文件，动态地填充 KUBELET_KUBEADM_ARGS 变量
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# 这是一个文件，用户在不得已下可以将其用作替代 kubelet args。
# 用户最好使用 .NodeRegistration.KubeletExtraArgs 对象在配置文件中替代。
# KUBELET_EXTRA_ARGS 应该从此文件中获取。
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```
该文件为 kubelet 指定由 kubeadm 管理的所有文件的默认位置。

* 用于 TLS 引导程序的 KubeConfig 文件为 /etc/kubernetes/bootstrap-kubelet.conf， 但仅当 /etc/kubernetes/kubelet.conf 不存在时才能使用。
* 具有唯一 kubelet 标识的 KubeConfig 文件为 /etc/kubernetes/kubelet.conf。
* 包含 kubelet 的组件配置的文件为 /var/lib/kubelet/config.yaml。
* 包含的动态环境的文件 KUBELET_KUBEADM_ARGS 是来源于 /var/lib/kubelet/kubeadm-flags.env。
* 包含用户指定标志替代的文件 KUBELET_EXTRA_ARGS 是来源于 /etc/default/kubelet（对于 DEB），或者 /etc/sysconfig/kubelet（对于 RPM）。 KUBELET_EXTRA_ARGS 在标志链中排在最后，并且在设置冲突时具有最高优先级。

##### Kubernetes 可执行文件和软件包内容

Kubernetes 版本对应的 DEB 和 RPM 软件包是：

|Package name|	Description|
|---|---|
|kubeadm	|给 kubelet 安装 /usr/bin/kubeadm CLI 工具和 kubelet 的 systemd 文件。|
|kubelet	|安装 kubelet 可执行文件到 /usr/bin 路径，安装 CNI 可执行文件到 /opt/cni/bin 路径。|
|kubectl	|安装 /usr/bin/kubectl 可执行文件。|
|cri-tools|	从 cri-tools git 仓库中安装 /usr/bin/crictl 可执行文件。|

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
