[toc]
# GUN简介
参考：https://www.php.cn/linux-414104.html

&emsp;&emsp;GNU是基于Unix开发设计，并且是与Unix兼容的操作系统，该项目由Richard Stallman在1983年创建，目标是生成非专有软件。因此，用户可以直接下载，修改和重新分发GNU软件，GNU是GNU's Not Unix! 的缩写.

&emsp;&emsp;GNU是一类Unix操作系统,它是由多个应用程序、系统库、开发工具乃至游戏构成的程序集合。GNU的开发始于1984年1月，称为GNU工程，GNU的许多程序都是在GNU工程下发布，我们称之为GNU软件包。

&emsp;&emsp;GNU具有类似Unix的设计，但它可以作为免费软件使用，并且不包含任何Unix代码，GNU由一系列软件应用程序组成，并且和开发人员工具以及一个分配资源并以及硬件或内核通信的程序组成，GNU可以与其他内核一起使用，并且通常与Linux内核一起使用。GNU/Linux组合是GNU/Linux操作系统，GNU系统的主要组件包括：
- GNU编译器集合
- GNU C库
- GNU Emacs文本编辑器
- GNOME桌面环境

GNU程序可以移植到许多其他操作系统，包括不同的平台，如Mac OS X和Microsoft Windows，GNU有时安装在Unix系统上，作为专有实用程序的替代品。我们建议安装这些GNU版本（更确切地说是，GNU/Linux发行版），因为它完全是自由软件。

参考链接：https://www.zhihu.com/question/319783573/answer/656033035

&emsp;&emsp;Unix 系统被发明之后，大家用的很爽。但是后来开始收费和商业闭源了。一个叫 RMS 的大叔觉得很不爽，于是发起 GNU 计划，模仿 Unix 的界面和使用方式，从头做一个开源的版本。然后他自己做了编辑器 Emacs 和编译器 GCC。
&emsp;&emsp;GNU 是一个计划或者叫运动。在这个旗帜下成立了 FSF，起草了 GPL 等。
&emsp;&emsp;接下来大家纷纷在GNU计划下做了很多的工作和项目，基本实现了当初的计划。包括核心的gcc和glibc。但是GNU系统缺少操作系统内核。原定的内核叫 HURD，一直完不成。同时BSD（一种UNIX发行版）陷入版权纠，x86平台开发暂停。然后一个叫 Linus 的同学为了在 PC 上运行 Unix，在 Minix 的启发下，开发了Linux。注意，Linux只是一个系统内核，系统启动之后使用的仍然是 gcc 和 bash 等软件。Linus 在发布 Linux的时候选择了 GPL，因此符合 GNU 的宗旨。
&emsp;&emsp;最后，大家突然发现，这玩意不正好是 GNU 计划缺的么。于是合在一起打包发布叫 GNU / Linux。然后大家念着念着省掉了前面部分，变成了 Linux 系统。实际上 Debian，RedHat 等 Linux 发行版中内核只占了很小一部分容量。
# Linux C语言编译步骤详解
参考：https://www.cnblogs.com/zhjblogs/p/13055081.html
https://www.cnblogs.com/zhjblogs/p/13646399.html
https://blog.csdn.net/Wmll1234567/article/details/109681262

![image](https://img2020.cnblogs.com/blog/1959382/202006/1959382-20200606160429297-1158393851.png)

gcc/g++在执行编译工作的时候，分为以下四个过程：

1.预处理，生成.i的文件
2.将预处理后的文件转换成汇编语言，生成.s文件
3.汇编变为目标代码(机器代码)生成.o的文件
4.连接目标代码,生成可执行程序
下面用个小例子说明这四个过程：
```c
 //建个main.cpp
 //This is the test code
 #include <iostream>
  using namespace std;  
 #define pi 3.14
 static int t = 1; 

 int main()
 {
  　　cout<<"Hello World: The t+pi is "<<t+pi<<endl;
  　　return 0;
  } 
 ```
## （1）预处理阶段
预处理阶段。预处理器（cpp）根据以字符#开头的命令，修改原始的C程序。比如hello.c中第一行的#include<stdio.h>命令告诉预处理器读取系统头文件stdio.h的内容，并把它直接插入程序文本中，结果就得到了另一个C程序，通常是以.i作为文件扩展名。
```bash
g++ -E main.cpp > main.i 
```
预处理后的文件 linux下以.i为后缀名，这个过程只激活预处理，不生成文件,因此你需要把它重定向到一个输出文件里 。

这一步的功能：
宏的替换，还有注释的消除，还有找到相关的库文件，将#include文件的全部内容插入。若用<>括起文件则在系统的INCLUDE目录中寻找文件，若用" "括起文件则在当前目录中寻找文件。关于更多宏的内容请[移步](https://blog.csdn.net/Wmll1234567/article/details/109681262)

用编辑器打开main.i会发现有很多很多代码，你只需要看最后部分就会发现，预处理做了宏的替换，还有注释的消除，可以理解为无关代码的清除。
![image](https://images2015.cnblogs.com/blog/966989/201606/966989-20160603202057055-1394044460.png)
## (2)将预处理后的文件转换成汇编语言,生成.s文件
编译阶段。编译器（ccl）将文本文件hello.i翻译成文本文件hello.s，它包含一个汇编语言程序。汇编语言程序中的每条语句都以一种标准的文本格式确切的描述了一条低级机器语言指令。
```bash
g++ -S main.cpp
```
这一步的功能：生成main.s文件，.s文件表示是汇编文件，用编辑器打开就都是汇编指令。
下面是main.s文件的一部分：
![image](https://images2015.cnblogs.com/blog/966989/201606/966989-20160603202238461-1629663226.png)

## (3)汇编变为目标代码(机器代码)生成.o的文件
汇编阶段。汇编器（as）将hello.s翻译成机器语言指令，把这些指令打包成一种可重定位目标程序的格式，并将结果保存在目标文件hello.o中。hello.o文件是一个二进制文件，它的字节编码是机器语言指令而不是字符，如果我们在文本文件中打开hello.o文件，看到的将是一堆乱码。
```bash
g++ -c main.cpp 
```
这一步的功能：
.o是gcc生成的目标文件，用编辑器打开就都是二进制机器码。
下面是main.o文件的一部分:
![image](https://images2015.cnblogs.com/blog/966989/201606/966989-20160603202442321-1476028639.png)
## (4)连接目标代码，生成可执行程序
链接阶段。链接器（ld）负责处理合并目标代码，生成一个可执行目标文件，可以被加载到内存中，由系统执行。
```c
g++ main.o -o main //生成的可执行程序名为main ，如果执行命令 g++ main.o  这样默认生成a.out，也就是main与a.out是一个只是名字不同而已
```

# gcc 编译过程详解
## gcc 工作流程

参考：https://blog.csdn.net/Wmll1234567/article/details/109852877

![image](https://img-blog.csdnimg.cn/20201120175154902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dtbGwxMjM0NTY3,size_16,color_FFFFFF,t_70)

## gcc常用参数
```c
gcc最基本的用法是：gcc [options] [filenames]
其中，options就是编译器所需要的参数，filenames给出相关的文件名称
```

| gcc/g++指令选项 | 功 能|
| --- | --- |
| -E（大写） | 预处理指定的源文件，不进行编译。 |
| -S（大写） | 编译指定的源文件，但是不进行汇编。 |
|-C  (大写)|	阻止 GCC 删除源文件和头文件中的注释|
|-c|	编译、汇编指定的源文件，但是不进行链接。|
|-o|	指定生成文件的文件名。如果不使用-o选项，那么将采用默认的输出文件。例如默认情况下，生成的可执行文件的名字默认为 a.out|
|-g|可执行程序包含调试信息|
-l|	手动添加链接库,选项用于指明所需静态链接库的名称
-L|	为 GCC 增加另一个搜索链接库的目录,选项用于向 GCC 编译器指明静态链接库的存储位置
-ansi	|对于 C 语言程序来说，其等价于 -std=c90；对于 C++ 程序来说，其等价于 -std=c++98。
-std=	|手动指令编程语言所遵循的标准，例如 c89、c90、c++98、c++11 等。
-rdynamic	|它将指示连接器把所有符号（而不仅仅只是程序已使用到的外部符号）都添加到动态符号表（即.dynsym表）里，以便那些通过 dlopen() 或 backtrace() （这一系列函数使用.dynsym表内符号）这样的函数使用
-static	|选项强制 GCC 编译器使用静态链接库
-shared	|生成动态链接库
-fpic/-fPIC	|生成位置无关代码，令 GCC 编译器生成动态链接库（多个目标文件的压缩包）时，表示各目标文件中函数、类等功能模块的地址使用相对地址，而非绝对地址。这样，无论将来链接库被加载到内存的什么位置，都可以正常使用
-Wall	|使用它能够使GCC产生尽可能多的警告信息
-Werror	|将警告信息当成编码错误来对待,GCC会在所有产生警告的地方停止编译，迫使程序员对自己的代码进行修改
-Wl,option	|表示后面的参数将传给link程序ld（链接器）（例如：-Wl,rpath=     |优先到rpath指定的目录去寻找依赖库）
|-std	|指定GCC 编译程序时所使用的编译标准，例如-std=c89、-std=c11、-std=gnu90 等。注意：对于在 for 循环中声明变量i的做法，是违反C89 标准的|

## linux上大型C/C++程序编译
参考：https://blog.csdn.net/wongnoubo/article/details/79696052
- 一步到位的编译命令
    gcc test.c -o test
