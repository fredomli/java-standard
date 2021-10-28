# Centos7 安装 MySQL
1. 安装之前，我们需要检测系统是否自带MySQL。
```shell
rpm -qa | grep mysql
```

如果你的系统有安装可以选择进行卸载：
```shell
rpm -e mysql  # 普通删除模式
rpm -e --nodeps mysql  # 强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除
```
2. 检查Linux是否安装了mariadb数据库，mariadb数据库是mysql的分支

```shell
yum list installed| grep mariadb
```
如果Linux中安装了mariadb数据库，先卸载掉，因为CentOS 7.x 内部集成了mariadb，而安装mysql的话会和mariadb的文件冲突，所以需要先卸载掉mariadb ，执行命令：
```shell
yum -y remove mariadb-libs.x86_64
```

### 下载安装包
在MySQL官网下载自己需要的安装文件，这里我们下载压缩文件，再将压缩文件上传到系统中，通过解压手动配置。

* MySQL官网地址：[https://www.mysql.com/](https://www.mysql.com/)
* MySQL下载地址: [https://dev.mysql.com/downloads/](https://dev.mysql.com/downloads/)

下载页面如下图所示：

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/mysql=instal-and-download-page3.png)

### 解压文件
将从官网下载的MySQL压缩文件上传到系统的目录下，使用jar命令进行解压，如下所示：

```shell
tar -zxvf mysql-5.7.36-linux-glibc2.12-x86_64.tar.gz
```
将解压后的mysql-5.7.36-linux-glibc2.12-x86_64改名为mysql-5.7.36 或者 mysql，看个人习惯。

```shell
# 重命名
mv mysql-5.7.36-linux-glibc2.12-x86_64 mysql-5.7.36
```

创建一个数据目录，从来保存数据库数据。目录位置建议和数据库安装目录在同一个文件下。
```shell
mkdir -p mysqldb
```
这里mysql安装文件存放地址为：
```shell
/usr/local/mysql/

# 安装目录为
/usr/local/mysql/mysql-5.7.36

# 数据保存目录（datadir）为
/usr/local/mysql/mysqldb
```

### 添加mysql用户及用户组
```shell
groupadd mysql

useradd -r -g mysql mysql  # (-g: 是指定用户所在组)
```
为数据目录赋予权限：
```shell
chown -R mysql:mysql /usr/local/mysql/mysqldb
```
### 配置my.cnf
```shell
vi /etc/my.cnf
```
内容如下：
```shell
[mysqld]
bind-address=0.0.0.0
port=3306
basedir=/usr/local/mysql/mysql-5.7.36
datadir=/usr/local/mysql/mysqldb
socket=/tmp/mysql.sock
log-error=/usr/local/mysql/mysqldb/error.log
pid-file=/usr/local/mysql/mysqldb/mysql.pid
# character config
character_set_server =utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
```

### 初始化
进入 `/usr/local/mysql/mysql-5.7.36/bin`进行数据库初始化。

```shell
cd /usr/local/mysql/mysql-5.7.36/bin
```
初始化命令如下：
```shell
./mysqld \
--defaults-file=/etc/my.cnf \
--initialize \
--user=mysql \
--datadir=/usr/local/mysql/mysqldb \
--basedir=/usr/local/mysql/mysql-5.7.36
```

### 查看密码
```shell
cat /usr/local/mysql/mysqldb/error.log
# >pMg5gZWzYos
```

![mysql-install-password](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/mysql-install-default-password.png)

### 启动MySQL服务
先将mysql.server放置到/etc/init.d/mysql中，加入系统服务的意思。
```shell
cp /usr/local/mysql/mysql-5.7.36/support-files/mysql.server /etc/init.d/mysqld
```

启动mysql：
```shell
# 启动命令 可以不启动，先加入系统服务之后使用systemctl来启动
service mysql start

# 加入系统服务，使用 systemctl 来管理
chkconfig --add mysqld

# 设置为开机自启
chkconfig mysqld on

# 以后启动服务就可以像这样
systemctl start mysqld

# 查看状态
ps -ef | grep mysql
```
安装成功如下所示：

![mysql-install-success](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/mysql-install-success.png)

### 修改密码
```sql
# 登录MySQL 登录密码为初始化时出现的密码
mysql -u root -p

# 修改密码
SET PASSWORD = PASSWORD('123456');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;
```
exit; 退出系统，重新登录验证密码。

### 防火墙开发3306端口
```shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

### 配置MySQL可以远程访问
```sql
use mysql;
update user set host="%" where user="root";
flush privileges;
```

### 将mysql客户端连接到/etc/bin
```shell
ln -s /usr/local/mysql/mysql-5.7.36/bin/mysql /usr/bin
```

这样每次访问就不需要到安装目录下的bin目录中运行mysql服务。

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
