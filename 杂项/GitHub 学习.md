
## FastGithub
参考：https://zhuanlan.zhihu.com/p/431933179
地址：GitHub搜索：FastGitHub
fastgithub是使用dotnet开发的一款github加速器，

提供域名的纯净IP解析；
提供IP测速并选择最快的IP；
提供域名的tls连接自定义配置；
google的CDN资源替换，解决大量国外网站无法加载js和css的问题；
这是一个本机工具，无任何中转的远程服务器，但也能让你的网络产生很大的改善：

## git学习
参考：https://zhuanlan.zhihu.com/p/193140870
### 用户名密码
因为Git是分布式版本控制系统，所以需要填写用户名和邮箱作为一个标识，用户和邮箱为你github注册的账号和邮箱
```bash
git config --global user.name "liming-design"
git config --global user.email "1731002129@qq.com"

```

git config –global 参数，有了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然你也可以对某个仓库指定的不同的用户名和邮箱。


### 生成ssh key
众所周知ssh key是加密传输。

加密传输的算法有好多，git使用rsa，rsa要解决的一个核心问题是，如何使用一对特定的数字，使其中一个数字可以用来加密，而另外一个数字可以用来解密。这两个数字就是你在使用git和github的时候所遇到的public key也就是公钥以及private key私钥。

其中，公钥就是那个用来加密的数字，这也就是为什么你在本机生成了公钥之后，要上传到github的原因。从github发回来的，用那公钥加密过的数据，可以用你本地的私钥来还原。

如果你的key丢失了，不管是公钥还是私钥，丢失一个都不能用了，解决方法也很简单，重新再生成一次，然后在http://github.com里再设置一次就行。

首先检查是否已生成密钥 cd ~/.ssh，ls如果有2个文件，则密钥已经生成，id_rsa.pub就是公钥
如果没有生成，那么通过`$ ssh-keygen -t rsa -C “xxxxxx@163.com”`来生成。

### 为Github账户设置SSH key
切换到github，展开个人头像的小三角，点击settings，然后打开SSH keys菜单， 点击Add SSH key新增密钥，填上标题，跟仓库保持一致吧，好区分。

接着将id_rsa.pub文件中key粘贴到此，最后Add key生成密钥吧。
如此，github账号的SSH keys配置完成。
### 上传本地项目到github
1 创建一个本地项目

```
git init //把这个目录变成Git可以管理的仓库
git add README.md //文件添加到仓库
git add . //不但可以跟单一文件，还可以跟通配符，更可以跟目录。一个点就把当前目录下所有未追踪的文件全部add了 
git commit -m "first commit" //把文件提交到仓库
git remote add origin git@github.com:wangjiax9/practice.git //关联远程仓库
git push -u origin master //把本地库的所有内容推送到远程库上
```

2 建立本地仓库

首先，进入到note项目目录。
然后执行指令：`git init`
初始化成功后会发现项目里多了一个隐藏文件夹.git
这个目录是Git用来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。

接着，将所有文件添加到仓库
执行指令：`git add .`

然后，把文件提交到仓库，双引号内是提交注释。
执行指令：`git commit -m "提交文件"`
==注意：-m "" 双引号中的信息一定要加，否则会报错！！==
如此本地仓库建立好了。

3 关联github仓库
到github note 仓库复制仓库地址，在ssh目录下：
`git@github.com:liming-design/PersonalNote.git`
然后执行指令：`git remote add origin git@github.com:liming-design/PersonalNote.git`

4 上传本地代码
执行指令：`git push -u origin master`

到此，本地代码已经推送到github仓库了

### 修改remote origin
方法一：

`git remote set-url origin git@192.168.1.18:mStar/OTT-dual/K3S/supernova`

方法二：

```
git remote rm origin 
git remote add origin git@192.168.1.18:mStar/OTT-dual/K3S/supernova
```
