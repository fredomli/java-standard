# 使用Gitee和PicGo搭建图床  

## 1.简介
图床是指一种存储图片的服务器，在Web上有很多类似图床的应用，比如提供图片存储服务的站点，各种的图片网站（类似于壁纸站点，图片引擎）都可以看做是一个图床。  

这里我们自己搭建一个图床服务，图床包括提供处理服务和存储服务两部分，这里的PicGo和Gitee就是这两部分的解决方案。  

* [Gitee.com](https://gitee.com/) (码云) 是 OSCHINA.NET 推出的代码托管平台,支持 Git 和 SVN,提供免费的私有仓库托管。
* [PicGo](https://github.com/Molunerfinn/PicGo/releases) 一个简单漂亮的图片工具，由`vue-clie-electron-builder`构建。  

> 所谓图床工具，就是自动把本地图片转换成链接的一款工具，网络上有很多图床工具，就目前使用种类而言，PicGo 算得上一款比较优秀的图床工具。它是一款用 Electron-vue 开发的软件，可以支持微博，七牛云，腾讯云COS，又拍云，GitHub，阿里云OSS，SM.MS，imgur 等8种常用图床，功能强大，简单易用

## 2.注册Gitee账号
[Gitee 注册地址](https://gitee.com/signup)  

![gitee 注册](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/%5D8JLD65~EGW%7B6HIW35%5BKFUL.png)  
## 3.创建仓库
创建一个仓库用来存储图片。在Gitee主页面（右上角加号处），选择创建创库。 
![创建创库](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/gitee-create-des.png)
## 4.下载&安装PicGo
去Github上下载[PicGo](https://github.com/Molunerfinn/PicGo/releases) ，选择适合自己版本和操作系统的PicGo进行下载。  
这里windows系统选择 `*.exe` 版本的直接安装。

![picgo download](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/picgo-download.png)

安装完成后就可以看到如下界面。  

![主页页面](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/picgo-home-page.png)


## 5.在PicGo配置Gitee仓库信息
在图床设置中我们配置需要的信息。配置完成后在`上传区`通过拖拽的方式直接上传图片，然后在相册中就可以看见上传后的图片。  

![picgo setting](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/picgo-setting.png)


*这样我们就可以将图片保存在自己的创库中，也不用担心有一天自己的图库因为三方问题而遗失。*  

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
