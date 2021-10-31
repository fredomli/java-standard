# Github访问缓慢解决方法

## 查询可以访问的地址
Github不能访问的原因可能很多，但是解决方法本质上就一种，找一个能够访问的服务器访问即可。使用下面网站查询自己能够访问的地址即可。
```
https://websites.ipaddress.com/
```
导航到[https://websites.ipaddress.com/](https://websites.ipaddress.com/) ,
如下图所示输入自己需要查询的地址，我们这里输入`github.com`:

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/hosts-update-method-home-page.png)

效果如下：

![pc2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/hosts-find-page.png)

## 修改我们的hosts文件

如下所示找到hosts文件，以管理员身份打开，并将我们查找到的IP地址添加进去。

![pc3](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/hosts-admin-pudate-github-address.png)

添加内容如下：
```
140.82.114.3 github.com
```

更新DNS缓存：
```shell
ipconfig /flushdns
```

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
