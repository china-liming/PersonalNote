[toc]
## 第一天
centos7设置开机默认使用root账户登陆
使用root账户进入系统：

```bash
vi /etc/gdm/custom.conf
```

在[daemon]下写入：

```bash
AutomaticLoginEnable=True
AutomaticLogin=root
```

## 第二天
Centos7中 Visual Studio Code 安装与卸载

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
yum check-update
# yum安装VSCode
sudo yum install code
# 打开VSCode
code
```
root 用户打开应用时应在属性命令行里加入

```bash
--no-sandbox
```

### 设置桌面图标

安装完毕之后，点击左上角 Place-> computers -> 进入 /usr/share/applications 文件夹 -> 找到 Visual Studio Code -> 复制到桌面 -> 在桌面双击启动并 Trust and Launch，就大功告成了.
### 卸载
卸载命令：

```bash
sudo yum remove code
```
###  "sh -c" 命令
它可以让 bash 将一个字串作为完整的命令来执行，这样就可以将 sudo 的影响范围扩展到整条命令。具体用法如下：

```bash
$ sudo /bin/sh -c 'echo "hahah" >> test.asc'
```

## Linux 下软件安装方法
### 最常用的yum安装
解释: yum就是类似于360软件管家，腾讯软件管家这种专门管理软件的管理器，像手机Appstore, 谷歌商店这种。
yum就是yellow dog Updater, Modified, 简单理解就是“黄狗软件管理”(为什么叫黄狗， 可能是当时的开发团队比较喜欢吧，哈哈哈)
注意: yum只是管理器，它所管理的安装包就是rpm包，千万不要昏掉。
就像在appstore安装软件一样，它能帮我们一键安装，但是它下载一键安装的软件还是是APK程序，
并不是它自己就是安装软件，它是管理程序的管理器。
工作模式: yum安装可以直接从服务器下载安装，实现- -键操作(不用去纠结哪里下载,安装在那个目录，需要哪些组件等)
方法:
1.配置yum源(也叫仓库)
2.更换国内的源(因为官方的速度慢，而且软件少)
3.更新源(防止软件太旧了)
4.运行yum安装软件
yum安装：
```c
# yum install 包名
```
yum卸载：

```c
# yum -y remove 包名
```

### rpm包安装方式步骤：
rpm就是"Redhat Package Manager”, 就是红帽安装包管理。rpm包，就是编译后打包好一个完整安装包。
工作模式:下载好rpm包后，使用rpm命令进行安装。若安装报错需要运行库，需要安装运行库依赖库。

```bash
安装: rpm -i包名
卸载: rpm -e包名
升级: rpm -u包名
查找: rpm -qa | grep包名
特点:最基础的安装方法,必须掌握，可以自定义相关的设置，缺点是要自行安装运行库依赖库
```
步骤：
```c
1、找到相应的软件包，比如soft.version.rpm，下载到本机某个目录；
2、打开一个终端，su -成root用户；
3、cd soft.version.rpm所在的目录；
4、输入rpm -ivh soft.version.rpm
```
### 源码包安装
解释: 一般大公司的软件会使用。使用软件官方的源码进行安装，相比rpm跟yum更倾向区别在包上,最纯净无修改的官方源码安装包。
工作模式:适用于一套或者大型软件的安装,例如MYSQL, php等,而且适用于对开发或者软件运行有要求的环境。且用户对LINUX或者软件有一定技术基础。
方法:
1.先安装依赖运行库
2.下载源码包
3.编译安装
命令：

```c
1、先下载到本地目录：wget http://www.zlib.net/zlib-1.2.12.tar.gz
2、cd源码所在目录
3、. / configure [opts]
4、make
5、make install 
```
特点:有技术基础或者大型软件适用，对技术要求稍微高一点点, 适应于开发者环境，不过兼容性好，文档齐全，技术人员首选

#### tar.gz源代码包安装方式：
```c
1、找到相应的软件包，比如soft.tar.gz，下载到本机某个目录；
2、打开一个终端，su -成root用户；
3、cd soft.tar.gz所在的目录；
4、tar -xzvf soft.tar.gz //一般会生成一个soft目录
5、cd soft
6、./configure
7、make
8、make install
```
#### tar.bz2源代码包安装方式：
```c
1、找到相应的软件包，比如soft.tar.bz2，下载到本机某个目录；
2、打开一个终端，su -成root用户；
3、cd soft.tar.bz2所在的目录；
4、tar -xjvf soft.tar.bz2 //一般会生成一个soft目录
5、cd soft
6、./configure
7、make
8、make install
```
- 查看configure 脚本的使用帮助及其选项，可以执行命令：./configure --help 查看使用方法。

如果直接执行：./configure，那么默认安装路径是/usr/local，对应的头文件、可执行文件和库文件分别对应的目录是：'/usr/local/include'、'/usr/local/bin'，'/usr/local/lib'。

- 我本人设置了自定义安装路径，执行命令如下：
```bash
./configure --prefix=/usr/local/glib
```
<Tips> ==要卸载glib库==，命令为：

```bash
make uninstall
```

## Centos 安装zlib
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
```bash
yum install -y zlib zlib-devel
```
或者源码安装：

```c
wget http://www.zlib.net/zlib-1.2.12.tar.gz
./configure --prefix=/usr/local/zlib
make
make install
```

2. 添加lib库自动搜索路径并使之生效

echo “/data/install/zlib/lib” >> /etc/ld.so.conf

ldconfig

## 安装glib
windows下安装地址：
https://download.gnome.org/binaries/win64/glib/
```bash
wget http://ftp.acc.umu.se/pub/GNOME/sources/glib/2.45/glib-2.45.2.tar.xz
tar -xvf glib/2.45/glib-2.45.2.tar.xz
cd glib-2.45.2
./configure --prefix=/usr/local/glib
make
make install
```
安装完后再编译一下,居然还说我没有头文件？原来是我们使用的头文件用了<>，会在环境变量设置的头文件目录查找头文件，目录是/usr/include 目录这样cp一下就好了:

```bash
cp -r /usr/local/glib/include/glib-2.0/* /usr/include/  
cp /usr/local/glib/lib/glib-2.0/include/glibconfig.h /usr/include/
```
## 安装软件时提示颁发的证书已经过期。
参考：https://www.icode9.com/content-4-1253936.html

```c
sudo yum install -y ca-certificates
```

## wget下载慢的问题
原文：
https://blog.csdn.net/Sara_cloud/article/details/111166031
wget相比于mwget下载速度较慢，mwget是一个多线程的下载应用，可以提高下载速度。

mwget安装步骤：

```bash
wget http://jaist.dl.sourceforge.net/project/kmphpfm/mwget/0.1/mwget_0.1.0.orig.tar.bz2
```

```bash
yum install bzip2 gcc-c++ openssl-devel intltool -y

bzip2 -d mwget_0.1.0.orig.tar.bz2

tar -xvf mwget_0.1.0.orig.tar
 
cd mwget_0.1.0.orig

./configure 

make

make install

```
## python3 安装
原文链接：
https://blog.csdn.net/i_likechard/article/details/119994177
https://blog.csdn.net/carooo/article/details/111991974

```bash
mwget https://www.python.org/ftp/python/3.8.9/Python-3.8.9.tgz
```
3.下载python3编译的依赖包

```bash
yum install -y gcc patch libffi-devel python-devel zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

```bash
tar -zxvf Python-3.8.9.tgz
cd Python-3.8.9
./configure --prefix=/usr/local/python38
make && make install
```
### 配置环境变量
方法1：
```bash
vim /etc/profile.d/python.sh
export PYTHON_HOME=/usr/local/python38
export PATH=$PYTHON_HOME/bin:$PATH
source /etc/profile.d/python.sh
```

方法2：创建软链接
```bash
ln -s /usr/local/python38/bin/python3 /usr/bin/python3
ln -s /usr/local/python36/bin/pip3 /usr/bin/pip3
```
另外：此法安装后，使用yum会报错。可按照下面的方法进行修改

修改yum配置文件(vi /usr/bin/yum)。
把文件头部的#!/usr/bin/python改成#!/usr/bin/python2.7。

修改/usr/libexec/urlgrabber-ext-down文件，将python同样指向python2.7

因为yum是基于Python编写的，而Python3和Python2有部分语法是不同的

至此，python3安装完成。

## 软链接

```bash
#创建软链接
ln -s 链接源 链接名
# 删除软连接
rm -rf 软连接地址
```



## 环境变量
参考：https://blog.csdn.net/weixin_38088485/article/details/89508432
### /etc/profile
此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.并从/etc/profile.d目录的配置文件中搜集shell的设置.（login shell读）
### /etc/bashrc
为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.
### ~/.bash_profile
每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件.（login shell读）
### ~/.bashrc
该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取.（non-login shell会读）
![在这里插入图片描述](https://img-blog.csdnimg.cn/c6a23de7521a4bf3b52866f3543f71a0.png)
/etc/profile中设定的变量(全局)的可以作用于任何用户,而~/.bashrc等中设定的变量(局部)只能继承/etc/profile中的变量,他们是"父子"关系.
一般来说，修改.bashrc文件,这种方法更为安全，它可以把使用这些环境变量的权限控制到用户级别,这里是针对某一个特定的用户，如果需要给某个用户权限使用这些环境变量，只需要修改其个人用户主目录下的.bashrc文件就可以了。

### source 读入环境配置文件的指令
由于 /etc/profile 与 ~/.bash_profile 都是在取得 login shell 的时候才会读取的配置文件，所 以， 如果你将自己的偏好设置写入上述的文件后，通常都是得登出再登陆后，该设置才会生 效。那么，能不能直接读取配置文件而不登出登陆呢？ 可以的！那就得要利用 source 这个指 令了！

```bash
[dmtsai@study ~]$ source 配置文件文件名 
范例：将主文件夹的 ~/.bashrc 的设置读入目前的 bash 环境中 
[dmtsai@study ~]$ source ~/.bashrc &lt;==下面这两个指令是一样的！
[dmtsai@study ~]$ . ~/.bashrc
```
利用 source 或小数点 （.） 都可以将配置文件的内容读进来目前的 shell 环境中！ 举例来 说，我修改了 ~/.bashrc ，那么不需要登出，立即以 source ~/.bashrc 就可以将刚刚最新设置 的内容读进来目前的环境中！很不错吧！还有，包括 ~/bash_profile 以及 /etc/profile 的设置 中， 很多时候也都是利用到这个 source （或小数点） 的功能喔！

## 安装pip
### python2安装pip
1、安装epel-release

```bash
[root@localhost ~]# yum -y install epel-release

```
2、安装python-pip

```bash
[root@localhost ~]# yum -y install python-pip

```
### python3安装pip

```bash
wget  https://bootstrap.pypa.io/pip/3.6/get-pip.py
python3 get-pip.py
ln /usr/local/python36/bin/pip3 /usr/bin/pip3
```
## 安装node npm
https://www.php.cn/centos/488471.html
如何解决centos npm命令找不到的问题？

centos下安装npm 解决 centos下安装 node 报： command not found

安装
```bash
# wget http://nodejs.org/dist/v0.10.25/node-v0.10.25.tar.gz

# yum install gcc openssl-devel gcc-c++ compat-gcc-34 compat-gcc-34-c++

# tar -xf node-v0.10.25.tar.gz# cd node-v0.10.25

# ./configure --prefix=/usr/local/node

# make && make install

# ln -s /usr/local/node/bin/* /usr/sbin/
```
安装完成之后，npm也被自动安装完成。

测试

```bash
# node -v
# npm -v
```
## 解决Centos虚拟机复制文件失败问题
参考：
https://blog.csdn.net/qq_40500571/article/details/109357428

## sudo pip3 找不到命令
原文：https://blog.csdn.net/xuelang532777032/article/details/113865382

报错信息：
sudo: 在加载插件“sudoers_policy”时在 /etc/sudo.conf 第 19 行出错
sudo: /usr/libexec/sudo/sudoers.so 必须只对其所有者可写
sudo: 致命错误，无法加载插件
输入以下命令
```bash
su root
chmod 644 usr/libexec/sudo/sudoers.so
chown -R root /usr/libexec/sudo
 
```
这种方法没解决问题，sudo + 命令还是不能执行。。。
将pip3加上软链接， 可以执行。

## meson创建软链接
参考：https://blog.csdn.net/andylauren/article/details/106194941
提示meson找不到命令，安装时提示已经存在。
于是创建软链接

```bash
ln -s /usr/local/python38/bin/meson /usr/bin
```
## centos无法建立ssl连接
参考：https://www.cnblogs.com/lixinsi/p/15188196.html
在centos下使用wget安装mysql5.7时，提示无法建立ssl连接

查阅资料，在命令wget后加上 --no-check-certificate也还是无法建立SSL连接。 后来，觉得可能是由于是新装的centos，还未安装ssl服务，所以开始安装ssl和apache。

安装apache
#yum install httpd

安装SSL模块
#yum install mod_ssl

重启apache
#service httpd restart

更新下载wget
#yum install wget

## 安装ninja
参考：https://www.cnblogs.com/bjarnescottlee/p/13872893.html
https://zhuanlan.zhihu.com/p/321882707

## nghttp2 安装
参考：https://kirk91.github.io/posts/28c07e17/

## 找不到ifconfig
　centos7中运行ifconfig提示-bash: ifconfig: command not found

　　查看/sbin/下是否有ifconfig，若没有通过如下命令安装

```bash
sudo yum install net-tools
```


