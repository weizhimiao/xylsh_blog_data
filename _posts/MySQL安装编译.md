---
title: MySQL编译安装
date: 2016-09-25 20:30:00
ctime: 2016-09-25 20:30:00
utime: 2016-09-25 20:30:00
modif_times: 0
tags:
- 
categories:
- MySQL
---

![MySQL](http://n.sinaimg.cn/games/3ece443e/20160925/mysql_log.png)
<!-- more -->

## 环境
OS：CentOS 7.2 64

MySQL：[mysql-5.7.15](http://dev.mysql.com/downloads/mysql/)

## 编译环境准备
```
# yum -y install make gcc-c++ cmake bison-devel ncurses-devel
```
> make，Linux下非常重要的编译工具，最主要也是最基本的功能就是通过makefile文件来描述源程序之间的相互关系并自动维护编译工作。
> gcc-c++，C++ 编译器（gcc，C编译器）
> cmake，一个跨平台的编译自动配置工具，（作用生成makefile 文件）
> bison-devel 一个语法分析器生成器
> ncurses-devel，Ncurses介绍摘要:Ncurses是一个能提供功能键定义(快捷键),屏幕绘制以及基于文本终端.

### ncurses
Ncurses 提供字符终端处理库，包括面板和菜单。它提供了一套控制光标，建立窗口，改变前景背景颜色以及处理鼠标操作的函数。使用户在字符终端下编写应用程序时绕过了那些恼人的底层机制。简而言之，他是一个可以使应用程序直接控制终端屏幕显示的函数库。
1、yum安装
```
yum -y install ncurses-devel
```
注：如果报错，包找不到，是*通配符没有识别，给文件名加双引号  “ncurses*”

2、源代码编译:
```
下载解压
cd ncurses-5.9
./configure --with-shared --without-debug --without-ada --enable-overwrite
make
make install
```
* 若不安装ncurses编译MySQL时会报错
* --without-ada 参数为设定不编译为ada绑定，因进入chroot环境不能使用ada ；--enable-overwrite参数为定义把头文件安装到/tools/include下而不是/tools/include/ncurses目录
* --with-shared 生成共享库

### 安装cmake和bison
mysql在5.5以后，不再使用./configure工具，进行编译安装。而使用cmake工具替代了./configure工具。cmake的具体用法参考文档cmake说明。

bison是一个自由软件，用于自动生成语法分析器程序，可用于所有常见的操作系统
```
yum -y install cmake
yum -y install bison
```

## 编译

解压源码包
```
tar -zxvf mysql-5.7.15.tar.gz
```
进入源码包目录
```
cd mysql-5.7.15
```
编译
```
cmake  \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_BOOST=/usr/local/boost
```
Boost，是为C++语言标准库提供扩展的一些C++程序库的总称，，由Boost社区组织开发、维护。
最后一行配置，是配置boost库的，如果没有boost包，编译会报错。
如果之前没有安装过，可以单独安装，也可在mysql安装的时候直接下载安装，这样的话，最后一行配置修改如下：
```
-DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/boost #直接下载并安装
```
```
make && make install
```

## 配置

### 创建MySQL运行用户和用户组
查看mysql用户及用户组
```
cat /etc/passwd     查看用户列表
cat /etc/group      查看用户组列表
```
如果没有就创建
```
groupadd mysql
useradd -g mysql mysql
```
修改/usr/local/mysql权限
```
chown -R mysql:mysql /usr/local/mysql
```
初始化配置
```
cd /usr/local/mysql
cp support-files/my-default.cnf /etc/my.cnf
```
注：在启动MySQL服务时，会按照一定次序搜索my.cnf，先在/etc目录下找，找不到则会在安装目录下面找，在本例中就是 /usr/local/mysql/my.cnf，这是新版MySQL的配置文件的默认位置。

### 初始化数据库并生成初始密码
```
/usr/local/mysql/bin/mysqld --initialize --user=mysql
```
会生成一个初始密码
```
# A temporary password is generated for root@localhost: -qeFRRlHV0jf
```
密码：-qeFRRlHV0jf

### 设置环境变量（使得mysql服务可以全局访问）
```
vi ~/.bash_profile
```
在修改PATH=$PATH:$HOME/bin为：
```
PATH=$PATH:$HOME/bin:/usr/local/mysql/bin:/usr/local/mysql/lib
```
重新加载环境变量
```
[root@root ~]# source /root/.bash_profile    
```
## 管理
### 启动mysql

方法一：
```
# 将mysql的启动服务添加到系统服务中
# cp support-files/mysql.server /etc/init.d/mysql
# service mysql start
```
方法二：
```
/usr/local/mysql/bin/mysqld_safe --user=mysql &
```

### 设置开机自启动
方法一：

通过chkconfig实现。

方法二：直接修改 rc.local 文件
```
echo "/usr/local/mysql/bin/mysqld_safe --user=mysql &" >> /etc/rc.local
```

### 修改密码
登录并修改初始密码（不修改密码不让你操作，就是这么任性）
```
# mysql -uroot -hlocalhost -p
Enter password:-qeFRRlHV0jf（初始密码）
# mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('xxxxxxx');
```

重新登录
```
#  mysql> exit
#  mysql -u root -p
Enter password:
```
能够登录进去，则说明MySQL安装成功。over~
