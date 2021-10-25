# 使用VMware和Centos7创建虚拟机

## 安装虚拟机
[虚拟机安装](https://github.com/fredomli/java-standard/blob/main/docs/vm/虚拟机安装.md)

## 下载镜像
根据自己的需要下载不同版本不同类型的虚拟机。这里选择使用免费的Centos7版本虚拟机。

[Centos镜像下载](https://github.com/fredomli/java-standard/blob/main/docs/vm/Centos系统镜像下载.md)

## 创建虚拟机

#### 打开虚拟机软件，如下所示：

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/open-vm.png)

#### 创建虚拟机，如下所示：

![pc2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-pc1.png)
![pc3](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-pc2.png)

这里创建虚拟机有两种类型一种是推荐类型，一种是自定义类型。推荐类型是一种标准的配置，比较友好，如果没有特别的需求，资源足够的情况下就可以选择这种方式。配置参数后期是可以修改的，所以选择哪一种都可以，根据自己的需求来就可以。

这里我们选择使用自定义的方式。

#### 选择虚拟机硬件兼容性。不同的版本之间具有不同的兼容性。选择接近虚拟机安装版本的就行。
![pc4](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/dev-service.png)


#### 选择需要安装的客户端操作系统，比如Centos7,三种方式选择自己适合的就行。第三种方式可以在虚拟机创建完成后在虚拟机设置中导入。这里选择稍后安装。

![pc5](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/option-system.png)

#### 选择操作系统类型
![pc6](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/option-system-type.png)

#### 虚拟机命名和安装位置选择
![pc7](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/name-and-location-option.png)

#### 给虚拟机分配内存，根据自己本地资源情况选择，内存大可以多分配，内存小就少分配。
![pc8](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/mv-memory-get.png)

#### 选择虚拟机的网络类型，在虚拟机的操作系统中我们需要上网，这就需要进行网络配置。这里选择`NAT`类型。
![pc9](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-net-type.png)

> 注意：
> 桥接类型表示和当前本地网络连接在一起，同用一个网关，都在一个局域网之下。虚拟机和本地网络是同级的，除IP不同其他配置都是相同的，网络在同一个网段下。
> 填个坑：校园网情况下配置成功也不能访问网络，但是本地主机和虚拟机之间可以ping通。因为虚拟机的IP也需要认证，就和你本地上网一样。
> 
> NAT类型表示直接使用的本地的网络,相当于在本地网络开个热点，虚拟机使用的这个热点网络。
> 
> 第三种主机模式直接将本地网络配置给虚拟机。
#### IO（输入输出）选择推荐类型即可。
![pc10](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-io-type.png)

#### 磁盘类型，选择推荐类型即可。
![pc11](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-scsi-type.png)

#### 分配磁盘容量，根据本地情况分配。
![pc12](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-disk-size.png)

#### 对分配的磁盘文件命名，并选择存储位置。
![pc13](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-name.png)

#### 到这里基本完成，可以在当前界面对设备进行管理，添加或者删除设备。根据自己情况选择。
![pc14](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-complete.png)

#### 添加系统镜像文件
![pc15](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-add-os.png)

![pc16](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/create-vm-add-os1.png)


### 启动虚拟机
首次启动需要配置一些参数，有语言类型，时区，设置用户名密码，配置界面。根据操作指引一步步完成即可。

这里没有选择使用界面，直接基本配置完成后，通过用户名和密码登录。

直接使用Shell工具连接虚拟机，通过Shell工具来操作系统。

参阅：
> * [虚拟机安装](https://github.com/fredomli/java-standard/blob/main/docs/vm/虚拟机安装.md)
> * [Centos镜像下载](https://github.com/fredomli/java-standard/blob/main/docs/vm/Centos系统镜像下载.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
