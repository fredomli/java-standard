# Web 界面 (Dashboard)

Dashboard 是基于网页的 Kubernetes 用户界面。 你可以使用 Dashboard 将容器应用部署到 Kubernetes 集群中，也可以对容器应用排错，还能管理集群资源。 你可以使用 Dashboard 获取运行在集群中的应用的概览信息，也可以创建或者修改 Kubernetes 资源 （如 Deployment，Job，DaemonSet 等等）。 例如，你可以对 Deployment 实现弹性伸缩、发起滚动升级、重启 Pod 或者使用向导创建新的应用。

Dashboard 同时展示了 Kubernetes 集群中的资源状态信息和所有报错信息。

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/kubernetes-ui-dashboard.png)

## 部署 Dashboard UI

默认情况下不会部署 Dashboard。可以通过以下命令部署：

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

被墙了所以这个地址不能直接下载，所以提供一份配置文件：
```yaml
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Configuration to deploy release version of the Dashboard UI compatible with
# Kubernetes 1.8.
#
# Example usage: kubectl create -f <this_file>

# ------------------- Dashboard Secret ------------------- #

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create 'kubernetes-dashboard2-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create"]
  # Allow Dashboard to create 'kubernetes-dashboard2-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard2-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
        - --auto-generate-certificates
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8443
      nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard
```

## 访问 Dashboard UI

使用这个命令查看我们安装的port:
```shell
kubectl get pods -n kube-system
# 或
kubectl get svc,pod --all-namespaces | grep dashboard

kubectl -n kube-system get svc

NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   25h
kubernetes-dashboard   NodePort    10.96.83.20   <none>        80:32301/TCP             5m28s
```
> 访问 `localhost:32301` 这个地址。localhost改为你本机的IP

为了保护你的集群数据，默认情况下，Dashboard 会使用最少的 RBAC 配置进行部署。 当前，Dashboard 仅支持使用 Bearer 令牌登录。 要为此样本演示创建令牌，你可以按照 创建示例用户 上的指南进行操作。

赋予权限，创建一个`dashboard-adminuser.yaml`，具有`ServiceAccount` and `ClusterRoleBinding`两项配置。如下所示：
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
```
使用`kubectl apply -f dashboard-adminuser.yaml`运行。
```shell
$ kubectl apply -f dashboard-adminuser.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```
### 获取token
现在我们需要找到可以用于登录的令牌。执行下面的命令:
```shell
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```
它应该打印如下内容:
```shell
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
```

[comment]: <> (eyJhbGciOiJSUzI1NiIsImtpZCI6Ik5LeHFKa0NPQ2ZvYS04ODR5UEZCT2FBNXBGYkMwWDZZRnVQZUprbmlRVTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLW12NjRoIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwNjMzNDY0NS1iNDE5LTQ5Y2EtODY3ZC04ZmUxZTkwNmY3ZmIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.c3ilPjlab_1LkAGBRIrlzG_c-SY9To0U6nwMz5gc-k5FwWRDclH6nEG36K9u7mD7-bF77K_4jcNYONe0jz_Fn3JR0iijTwaR-HbKb3dB7hzWP4XDdaEItqUkY5p66ZP-H_4r-HwJtws9jDpwoKt7ngAE9Q56oL1lPBb8YFW1-BqhnxaXcHbhBco6jKHZ31s1Grqxb9N6ohbyiQxQviqLf5GFuIe5vR5kJkPhB3OcgSY0BbsT5dPYgxk5tjHq2mizUteOP0UjtDGp56PMEGYBTC6H8RWK6_eewqLG-mzNDqgOpyEPVlaxVYN6sng1mgcwchh6BLJA5k0t6tpgrMuxvg)

### 清理
删除admin `ServiceAccount`和`ClusterRoleBinding`。
```shell
kubectl -n kubernetes-dashboard delete serviceaccount admin-user
kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
```
要了解更多关于如何在Kubernetes中授予/拒绝权限的信息，请阅读官方认证和授权文档。
## 命令行代理
你可以使用 kubectl 命令行工具访问 Dashboard，命令如下：
```shell
kubectl proxy
```
kubectl 会使得 Dashboard 可以通过 http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/ 访问。

UI 只能 通过执行这条命令的机器进行访问。更多选项参见 `kubectl proxy --help`。

## 欢迎界面
当访问空集群的 Dashboard 时，你会看到欢迎界面。 页面包含一个指向此文档的链接，以及一个用于部署第一个应用程序的按钮。 此外，你可以看到在默认情况下有哪些默认系统应用运行在 kube-system 名字空间 中，比如 Dashboard 自己。

![pc2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/kubernetes-ui-dashboard-zerostate.png)

## 部署容器化应用
通过一个简单的部署向导，你可以使用 Dashboard 将容器化应用作为一个 Deployment 和可选的 Service 进行创建和部署。可以手工指定应用的详细配置，或者上传一个包含应用配置的 YAML 或 JSON 文件。

点击任何页面右上角的 CREATE 按钮以开始。

### 指定应用的详细配置
部署向导需要你提供以下信息：
* 应用名称（必填）：应用的名称。内容为应用名称的 标签 会被添加到任何将被部署的 Deployment 和 Service。
* 在选定的 Kubernetes 名字空间 中， 应用名称必须唯一。必须由小写字母开头，以数字或者小写字母结尾， 并且只含有小写字母、数字和中划线（-）。小于等于24个字符。开头和结尾的空格会被忽略。
* 容器镜像（必填）：公共镜像仓库上的 Docker 容器镜像 或者私有镜像仓库 （通常是 Google Container Registry 或者 Docker Hub）的 URL。容器镜像参数说明必须以冒号结尾。
* Pod 的数量（必填）：你希望应用程序部署的 Pod 的数量。值必须为正整数。系统会创建一个 Deployment 以保证集群中运行期望的 Pod 数量。
* 服务（可选）：对于部分应用（比如前端），你可能想对外暴露一个 Service ，这个 Service 可能用的是集群之外的公网 IP 地址（外部 Service）。

        > 对于外部服务，你可能需要开放一个或多个端口才行。
        
        其它只能对集群内部可见的 Service 称为内部 Service。
        
        不管哪种 Service 类型，如果你选择创建一个 Service，而且容器在一个端口上开启了监听（入向的）， 那么你需要定义两个端口。创建的 Service 会把（入向的）端口映射到容器可见的目标端口。 该 Service 会把流量路由到你部署的 Pod。支持 TCP 协议和 UDP 协议。 这个 Service 的内部 DNS 解析名就是之前你定义的应用名称的值。
        
        如果需要，你可以打开 Advanced Options 部分，这里你可以定义更多设置：


* 描述：这里你输入的文本会作为一个 注解 添加到 Deployment，并显示在应用的详细信息中。
* 标签：应用默认使用的 标签 是应用名称和版本。 你可以为 Deployment、Service（如果有）定义额外的标签，比如 release（版本）、 environment（环境）、tier（层级）、partition（分区） 和 release track（版本跟踪）。

    ```shell
    release=1.0
    tier=frontend
    environment=pod
    track=stable
    ```
* 名字空间：Kubernetes 支持多个虚拟集群依附于同一个物理集群。 这些虚拟集群被称为 名字空间， 可以让你将资源划分为逻辑命名的组。

    Dashboard 通过下拉菜单提供所有可用的名字空间，并允许你创建新的名字空间。 名字空间的名称最长可以包含 63 个字母或数字和中横线（-），但是不能包含大写字母。
    
    名字空间的名称不能只包含数字。如果名字被设置成一个数字，比如 10，pod 就
    
    在名字空间创建成功的情况下，默认会使用新创建的名字空间。如果创建失败，那么第一个名字空间会被选中。


* 镜像拉取 Secret：如果要使用私有的 Docker 容器镜像，需要拉取 Secret 凭证。

    Dashboard 通过下拉菜单提供所有可用的 Secret，并允许你创建新的 Secret。 Secret 名称必须遵循 DNS 域名语法，比如 new.image-pull.secret。 Secret 的内容必须是 base64 编码的，并且在一个 .dockercfg 文件中声明。Secret 名称最大可以包含 253 个字符。
    
    在镜像拉取 Secret 创建成功的情况下，默认会使用新创建的 Secret。 如果创建失败，则不会使用任何 Secret。

* CPU 需求（核数）和内存需求（MiB）：你可以为容器定义最小的 资源限制。 默认情况下，Pod 没有 CPU 和内存限制。
* 运行命令和运行命令参数：默认情况下，你的容器会运行 Docker 镜像的默认 入口命令。 你可以使用 command 选项覆盖默认值。
* 以特权模式运行：这个设置决定了在 特权容器 中运行的进程是否像主机中使用 root 运行的进程一样。 特权容器可以使用诸如操纵网络堆栈和访问设备的功能。
* 环境变量：Kubernetes 通过 环境变量 暴露 Service。你可以构建环境变量，或者将环境变量的值作为参数传递给你的命令。 它们可以被应用用于查找 Service。值可以通过 $(VAR_NAME) 语法关联其他变量。

### 上传 YAML 或者 JSON 文件
Kubernetes 支持声明式配置。所有的配置都存储在遵循 Kubernetes API 规范的 YAML 或者 JSON 配置文件中。

作为一种替代在部署向导中指定应用详情的方式，你可以在 YAML 或者 JSON 文件中定义应用，并且使用 Dashboard 上传文件：

## 使用 Dashboard

以下各节描述了 Kubernetes Dashboard UI 视图；包括它们提供的内容，以及怎么使用它们。

### 导航
当在集群中定义 Kubernetes 对象时，Dashboard 会在初始视图中显示它们。 默认情况下只会显示 默认 名字空间中的对象，可以通过更改导航栏菜单中的名字空间筛选器进行改变。

Dashboard 展示大部分 Kubernetes 对象，并将它们分组放在几个菜单类别中。

### 管理概述
集群和名字空间管理的视图, Dashboard 会列出节点、名字空间和持久卷，并且有它们的详细视图。 节点列表视图包含从所有节点聚合的 CPU 和内存使用的度量值。 详细信息视图显示了一个节点的度量值，它的规格、状态、分配的资源、事件和这个节点上运行的 Pod。

### 负载
显示选中的名字空间中所有运行的应用。 视图按照负载类型（如 Deployment、ReplicaSet、StatefulSet 等）罗列应用，并且每种负载都可以单独查看。 列表总结了关于负载的可执行信息，比如一个 ReplicaSet 的准备状态的 Pod 数量，或者目前一个 Pod 的内存使用量。

工作负载的详情视图展示了对象的状态、详细信息和相互关系。 例如，ReplicaSet 所控制的 Pod，或者 Deployment 关联的 新 ReplicaSet 和 Pod 水平扩展控制器。

### 服务
展示允许暴露给外网服务和允许集群内部发现的 Kubernetes 资源。 因此，Service 和 Ingress 视图展示他们关联的 Pod、给集群连接使用的内部端点和给外部用户使用的外部端点。

### 存储
存储视图展示持久卷申领（PVC）资源，这些资源被应用程序用来存储数据。

### ConfigMap 和 Secret
展示的所有 Kubernetes 资源是在集群中运行的应用程序的实时配置。 通过这个视图可以编辑和管理配置对象，并显示那些默认隐藏的 secret。

### 日志查看器

Pod 列表和详细信息页面可以链接到 Dashboard 内置的日志查看器。查看器可以钻取属于同一个 Pod 的不同容器的日志。

![pc3](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/kubernetes-ui-dashboard-logs-view.png)

## 参考
* [官网地址](https://kubernetes.io/)
* [Kubernetes文档](https://kubernetes.io/docs/home/)
* [使用Minikube创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)
* [交互式教程-创建集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)
* [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
* [交互式教程-部署应用程序](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/)
* [交互式教程-探索你的应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-interactive/)
* [Kubernetes 中文网](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/)
* [Kubernetes基础学习](https://github.com/fredomli/java-standard/blob/main/docs/service/kubernetes/学习Kubernetes基础.md)
* [生产环境kubernetes配置](https://github.com/fredomli/java-standard/blob/main/docs/service/kubernetes/生产环境kubernetes配置.md)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
