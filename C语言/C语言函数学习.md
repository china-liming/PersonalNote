[toc]
## fflush(stdout)
参考：https://www.jianshu.com/p/fb86daebf528
使用 printf 或 cout 打印内容时，输出永远不会直接写入“屏幕”。而是，被发送到 stdout。 （stdout 就像一个缓冲区）
默认情况下，发送到 stdout 的输出然后再发送到屏幕（我们可以根据需要将其重定向到其他文件/流）。同样，stdin 默认映射到键盘，但可以重定向到任何其他文件/流。

现在，默认情况下，stdout 是行缓冲的。这意味着，发送到 stdout 的输出不会被立即发送到屏幕以供显示（或重定向文件/流），直到它在其中获得换行符。因此，如果要覆盖默认缓冲行为，则可以使用 fflush 清除缓冲区（立即将所有内容发送到屏幕/文件/流）。
默认情况下，发送到 stdout 的输出然后再发送到屏幕（我们可以根据需要将其重定向到其他文件/流）。同样，stdin 默认映射到键盘，但可以重定向到任何其他文件/流。

现在，默认情况下，stdout 是行缓冲的。这意味着，发送到 stdout 的输出不会被立即发送到屏幕以供显示（或重定向文件/流），直到它在其中获得换行符。因此，如果要覆盖默认缓冲行为，则可以使用 fflush 清除缓冲区（立即将所有内容发送到屏幕/文件/流）。
例子1：

```c
#include <stdio.h>

void delay(unsigned long long ctr)
{
    while (ctr > 0)
        --ctr;
}

int main()
{
    printf("Hello, ");
    delay(1000000000ULL);

    printf("world!\n");
    delay(1000000000ULL);

    printf("Welcome.\n");
    return 0;
}
```
运行结果：
过一段时间后输出 hello world! 再过一段时间输出 Welcome.

例子2：
```c
#include <stdio.h>

void delay(unsigned long long ctr)
{
    while (ctr > 0)
        --ctr;
}

int main()
{
    printf("Hello, ");
    fflush(stdout);
    delay(1000000000ULL);

    printf("world!\n");
    delay(1000000000ULL);

    printf("Welcome.\n");
    return 0;
}
```
运行结果：
立即输出 Hello, 过一段时间输出 world!，又过一段时间输出 Welcome.
就单拿输出到屏幕上来说，printf 这样的函数不是直接打印到屏幕上的，而是先放在一个缓冲区中（stdout）中。如果收到了一个换行符，就会把这个缓冲区的内容打印到屏幕上，并清空。而 fflush 的作用就是直接把缓冲区的内容打印到屏幕上，并清空缓冲区。不必等换行符。

## assert()函数用法总结
assert宏的原型定义在<assert.h>中，其作用是如果它的条件返回错误，则终止程序执行，原型定义：
```c
#include <assert.h>
void assert( int expression );
```
assert的作用是现计算表达式 expression ，如果其值为假（即为0），那么它先向stderr打印一条出错信息，然后通过调用 abort 来终止程序运行。

##  calloc()
### 声明：	
```c
void* calloc (size_t num, size_t size);
```
- calloc() 函数用来动态地分配 num 个长度为 size 字节内存空间并初始化为 0，分配成功返回指向该内存的地址，失败则返回 NULL。
- 函数的返回值类型是 void *，所以在使用 calloc() 时通常需要进行强制类型转换。
- calloc() 与 malloc() 的一个重要区别是：calloc() 在动态分配完内存后，自动初始化该内存空间为零，而malloc() 不初始化，里边数据是未知的垃圾数据。

原文链接：https://blog.csdn.net/u011762313/article/details/51586099

## signal()
### 描述
C 库函数 void (\*signal(int sig, void (\*func)(int)))(int)设置一个函数来处理信号，即带有 sig参数的信号处理程序。
### 声明
`void ( *signal( int sig, void (* handler)( int ))) ( int );`

[原文](https://blog.csdn.net/lulugui/article/details/6290747)
对于 信号处理函数 位于 <signal.h> 中.  
`void ( *signal( int sig, void (* handler)( int ))) ( int );`
这个函数的声明很是吓人, 一看就难弄懂. 下面是解释用法.  
一步一步解释:  
int (\*p)();
这是一个函数指针, p所指向的函数是一个不带任何参数, 并且返回值为int的一个函数.  
int (\*fun())();  
这个式子与上面式子的区别在于用fun()代替了p,而fun()是一个函数,所以说就可以看成是fun()这个函数执行之后,它的返回值是一个函数指针,这个函数指针(其实就是上面的p)所指向的函数是一个不带任何参数,并且返回值为int的一个函数.
所以说对于 void (\*signal(int signo, void (\*handler)(int)))(int);就可以看成是==signal()函数(它自己是带两个参数,一个为整型,一个为函数指针的函数), 而这个signal()函数的返回值也为一个函数指针,这个函数指针指向一个带一个整型参数,并且返回值为void的一个函数.==
而你在写信号处理函数时对于信号处理的函数也是void sig_fun(int signo);这种类型,恰好与上面signal()函数所返回的函数指针所指向的函数是一样的.

也可以用 typedef 这样理解：
```c
// Type of a signal handler
typedef void (*__sighandler_t)(int);
__sighandler_t signal(int __sig, __sighandler_t __handler);
```

### 参数
-   **sig** -- 在信号处理程序中作为变量使用的信号码。下面是一些重要的标准信号常量：

宏    | 信号
-------- | -----
SIGABRT  | (Signal Abort) 程序异常终止。
SIGFPE  |(Signal Floating-Point Exception) 算术运算出错，如除数为 0 或溢出（不一定是浮点运算）。
SIGILL  | (Signal Illegal Instruction) 非法函数映象，如非法指令，通常是由于代码中的某个变体或者尝试执行数据导致的。
SIGINT |(Signal Interrupt) 中断信号，如 ctrl-C，通常由用户生成。
SIGSEGV|(Signal Segmentation Violation) 非法访问存储器，如访问不存在的内存单元。
SIGTERM|(Signal Terminate) 发送给本程序的终止请求信号。
SIGHUP|SIGHUP，hong up ，挂断。本信号在用户终端连接(正常或非正常)结束时发出, 通常是在终端的控制进程结束时, 通知同一session内的各个作业, 这时它们与控制终端不再关联。
 Linux终端下信号列表:
```
SIGABRT 	由调用abort函数产生，进程非正常退出
SIGALRM 	用alarm函数设置的timer超时或setitimer函数设置的interval timer超时
SIGBUS 	        某种特定的硬件异常，通常由内存访问引起
SIGCANCEL 	由Solaris Thread Library内部使用，通常不会使用
SIGCHLD 	进程Terminate或Stop时候，SIGCHLD会发送给它的父进程。缺省情况下该Signal会被忽略
SIGCONT 	当被stop的进程恢复运行的时候，自动发送
SIGEMT 	        和实现相关的硬件异常
SIGFPE 	        数学相关的异常，如被0除，浮点溢出，等等
SIGFREEZE 	Solaris专用，Hiberate或者Suspended时候发送
SIGHUP 	        发送给具有Terminal的Controlling Process，当terminal被disconnect时候发送
SIGILL 	        非法指令异常
SIGINFO 	BSD signal。由Status Key产生，通常是CTRL+T。发送给所有Foreground Group的进程
SIGINT 	        由Interrupt Key产生，通常是CTRL+C或者DELETE。发送给所有ForeGround Group进程
SIGIO 	        异步IO事件
SIGIOT 	        实现相关的硬件异常，一般对应SIGABRT
SIGKILL 	无法处理和忽略。中止某个进程
SIGLWP 	        由Solaris Thread Libray内部使用
SIGPIPE 	在reader中止之后写Pipe的时候发送
SIGPOLL 	当某个事件发送给Pollable Device的时候发送
SIGPROF 	Setitimer指定的Profiling Interval Timer所产生
SIGPWR 	        和系统相关。和UPS相关。
SIGQUIT 	输入Quit Key的时候（CTRL+\）发送给所有Foreground Group的进程
SIGSEGV 	非法内存访问
SIGSTKFLT 	Linux专用，数学协处理器的栈异常
SIGSTOP 	中止进程。无法处理和忽略。
SIGSYS 	        非法系统调用
SIGTERM 	请求中止进程，kill命令缺省发送
SIGTHAW 	Solaris专用，从Suspend恢复时候发送
SIGTRAP 	实现相关的硬件异常。一般是调试异常
SIGTSTP 	Suspend Key，一般是Ctrl+Z。发送给所有Foreground Group的进程
SIGTTIN 	当Background Group的进程尝试读取Terminal的时候发送
SIGTTOU 	当Background Group的进程尝试写Terminal的时候发送
SIGURG 	        当out-of-band data接收的时候可能发送
SIGUSR1 	用户自定义signal 1
SIGUSR2 	用户自定义signal 2
SIGVTALRM 	setitimer函数设置的Virtual Interval Timer超时的时候
SIGWAITING 	Solaris Thread Library内部实现专用
SIGWINCH 	当Terminal的窗口大小改变的时候，发送给Foreground Group的所有进程
SIGXCPU 	当CPU时间限制超时的时候
SIGXFSZ 	进程超过文件大小限制
SIGXRES 	Solaris专用，进程超过资源限制的时候发送
```

-   **func** -- 一个指向函数的指针。它可以是一个由程序定义的函数，也可以是下面预定义函数之一：

函数    | 说明
-------- | -----
SIG_DFL|默认的信号处理程序。
SIG_IGN|忽视信号。

### 返回值
该函数返回信号处理程序之前的值，当发生错误时返回 SIG_ERR。


## time相关
几个时间概念：
- 1：Coordinated Universal Time(UTC)：
协调世界时，又称世界标准时间，也即格林威治标准时间(Greenwich Mean Time,GMT)，中国内地的时间与UTC得时差为+8，也即UTC+8，美国为UTC-5。
- 2：Calendar Time：
日历时间，是用"从一个标准时间点到此时的时间经过的秒数"来表示的时间。标准时间点对不同编译器可能会不同，但对一个编译系统来说，标准时间是不变的。一般是表示距离UTC时间 1970-01-01 00:00:00的秒数。
- 3：epoch：
时间点。在标准c/c++中是一个整数，用此时的时间和标准时间点相差的秒数（即日历时间）来表示。
- 4：clock tick：
时钟计时单元（而不叫做时钟滴答次数），一个时钟计时单元的时间长短是由cpu控制的，一个clock tick不是cpu的一个时钟周期，而是c/c++的一个基本计时单位。

time()提供了秒级的精确度 。用time()函数结合其他函数（如：localtime、gmtime、asctime、ctime）可以获得当前系统时间或是标准时间。

如果需要更高的时间精确度，就需要struct timespec 和 struct timeval来处理：

### 结构体
```c
//为了增强程序的可移植性，便有了size_t，它是为了方便系统之间的移植而定义的，不同的系统上，定义size_t可能不一样。
typedef unsigned int size_t;

typedef long  clock_t;

typedef long  time_t;    /* 时间值time_t 为长整型的别名*/

#define CLOCKS_PER_SEC ((clock_t)1000)/*它用来表示一秒钟会有多少个时钟计时单元，*/

struct tm {
  int tm_sec;   // 秒，正常范围从 0 到 59，但允许至 61
  int tm_min;   // 分，范围从 0 到 59
  int tm_hour;  // 小时，范围从 0 到 23
  int tm_mday;  // 一月中的第几天，范围从 1 到 31
  int tm_mon;   // 月，范围从 0 到 11
  int tm_year;  // 自 1900 年起的年数
  int tm_wday;  // 一周中的第几天，范围从 0 到 6，从星期日算起
  int tm_yday;  // 一年中的第几天，范围从 0 到 365，从 1 月 1 日算起
  int tm_isdst; // 夏令时
};

struct timespec {
	time_t tv_sec; // seconds
	long tv_nsec; // and nanoseconds 纳秒
};


struct timeval {
	time_t tv_sec; // seconds
	long tv_usec; // microseconds 微秒	
};
struct timezone{
	int tz_minuteswest; //miniutes west of Greenwich
	int tz_dsttime; //type of DST correction
};

```


### time()
描述
函数功能: 得到当前日历时间或者设置日历时间。

声明：
`time_t time(time_t \*timer)`

参数: 
- timer=NULL时，得到当前日历时间（从1970-01-01 00:00:00到现在的秒数）。
- timer=时间数值时，用于设置日历时间，time_t是一个unsigned long类型。
- 如果 timer不为空，则返回值也存储在变量 timer中。

返回: 
当前日历时间
实例
```c
#include <stdio.h>
#include <time.h>

int main ()
{
  time_t seconds;

  seconds = time(NULL);
  printf("自 1970-01-01 起的小时数 = %ld\n", seconds/3600);

  return(0);
}
//运行结果：自 1970-01-01 起的小时数 = 459172
```


### strftime()
C语言时间格式化
#### 定义
```c
#include <time.h>
size_t strftime(char *str, size_t count, const char *format, const struct tm *tm);
```
#### 参数
- str, 表示返回的时间字符串
- count, 要写入的字节的最大数量
- format, 格式字符串由零个或多个转换符和普通字符(除%)
- tm, 输入时间

#### return
如果包含终止的空字符在内的结果字符的总数不大于count，则函数strftime返回字符数，这些字符被放到str指向的数组中但不包含终止的空字符。否则，函数返回零，且数组的内容不确定。
#### 格式说明

格式符号	|说明
-------- | -----
%a	|区域设置的缩写星期名
%A	|区域设置的完整星期名
%b	|区域设置的缩写月份名
%B	|区域设置的完整月份名
%c	|区域设置的适当的日期和时间表示
%d	|表示为一个十进制数(01-31)的当月的第几天
%H	|表示为一个十进制数(00-23)的小时时间(24小时制)
%I	|表示为一个十进制数(01-12)的小时时间(12小时制)
%j	|表示为一个十进制数(001-366)的当年的第几天
%m	|表示为一个十进制数(01-12)的月份
%M	|表示为一个十进制数(00-59)的分钟数
%p	|区域设置的、与12小时制相关的AM/PM指示符等价的东西
%S	|表示为一个十进制数(00-61)的秒数
%U	|表示为一个十进制数(00-53)的当年的第几周(第一个星期日是第一个星期的第一天)
%w	|表示为一个十进制数(0-6)的星期几，星期日表示0
%W	|表示为一个十进制数(00-53)的当年的第几周(第一个星期一是第一个星期的第一天)
%x	|区域设置的适当的日期表示
%X	|区域设置的适当的时间表示
%y	|表示为一个十进制(00-99)的不带世纪的年份
%Y	|表示为一个十进制的带世纪的年份
%Z	|时区名字或者它的缩写取代，如果不能确定时区，则不能被字符
\%\%	|表示\%


#### 实例
```c
//原文链接：https://blog.csdn.net/yybmec/article/details/48953635
#include <stdio.h>
#include <time.h>

int main()
{
    time_t time_T;
    time_T = time(NULL);

    struct tm *tmTime;
    // tm对象格式的时间
    tmTime = localtime(&time_T);

    printf("Now Time is: %d:%d:%d\n", (*tmTime).tm_hour, (*tmTime).tm_min, (*tmTime).tm_sec);

    char* format = "%Y-%m-%d %H:%M:%S";
    char strTime[100];
    strftime(strTime, sizeof(strTime), format, tmTime);

    printf("Time is :%s\n", strTime);

    return 0;
}

// 代码结果
// Now Time is: 20:46:1                                                                                                                                    
// Time is :2015-10-07 20:46:01

```


### localtime
#### 描述
使用 timer 的值来填充 tm 结构。timer 的值被分解为 tm 结构，并用本地时区表示。
#### 定义
```c
struct tm *localtime(const time_t *timer)
```
#### 参数
- timer -- 这是指向表示日历时间的 time_t 值的指针。

#### return
该函数返回指向 tm 结构的指针，该结构带有被填充的时间信息。
实例
```c
#include <stdio.h>
#include <time.h>
 
int main ()
{
   time_t rawtime;
   struct tm *info;
   char buffer[80];
 
   time( &rawtime );
 
   info = localtime( &rawtime );
   printf("当前的本地时间和日期：%s", asctime(info));
 
   return(0);
}
//运行结果：
//当前的本地时间和日期：Thu Aug 23 09:12:05 2012
```

### asctime 
#### 描述
返回一个指向字符串的指针，它代表了结构 struct timeptr 的日期和时间。
#### 声明
`char *asctime(const struct tm *timeptr)`
#### 参数
timeptr 是指向 tm 结构的指针
#### 返回
该函数返回一个 C 字符串，包含了可读格式的日期和时间信息
#### 实例
```c
#include <stdio.h>
#include <time.h>
 
int main(void)
{
    time_t timeValue = 0;
    struct tm *p = NULL;
    char *str = NULL;
 
    time(&timeValue);
    p = localtime(&timeValue);
    str = asctime(p);
    printf("现在的时间字符串是 = %s\n",str);
 
    return 0;
}
 
//运行结果：现在的时间字符串是 = Fri May 20 12:25:39 2022
```

### ctime
#### 描述
C 库函数 char *ctime(const time_t *timer) 返回一个表示当地时间的字符串，当地时间是基于参数 timer。
返回的字符串格式如下： Www Mmm dd hh:mm:ss yyyy 其中，Www 表示星期几，Mmm 是以字母表示的月份，dd 表示一月中的第几天，hh:mm:ss 表示时间，yyyy 表示年份。
#### 声明
下面是 ctime() 函数的声明。
`char *ctime(const time_t *timer)`
#### 参数
timer \-- 这是指向==time_t==对象的指针，该对象包含了一个日历时间。
#### 返回值
该函数返回一个 C 字符串，该字符串包含了可读格式的日期和时间信息。
#### 实例
```c
#include <stdio.h>
#include <time.h>
 
int main ()
{
   time_t curtime;
 
   time(&curtime);
 
   printf("当前时间 = %s", ctime(&curtime));
 
  	 return(0);
}
//运行结果：当前时间 = Fri May 20 12:19:33 2022
```
对比分析：asctime()和ctimed都是将时间转换为日期的字符串，ctime()转换的是时间的秒数，asctime()转换的是tm结构体中的时间，即两个函数的参数不一样。

### gmtime
#### 描述
得到以结构tm表示的时间信息，并用格林威治标准时间表示。
此函数返回的时间日期未经时区转换，而是UTC时间（格林尼治时间，北京时间晚8个小时，所以）。
#### 声明
`struct tm *gmtime(time_t *timer)`
#### 参数
timer用函数time()得到的时间信息
#### 返回
 以结构tm表示的时间信息指针
#### 实例
```c
#include <stdio.h>
#include <time.h>

#define BST (+1)
#define CCT (+8)

int main ()
{

   time_t rawtime;
   struct tm *info;

   time(&rawtime);
   /* 获取 GMT 时间 */
   info = gmtime(&rawtime );

   printf("当前的世界时钟：\n");
   printf("伦敦：%2d:%02d\n", (info->tm_hour+BST)%24, info->tm_min);
   printf("中国：%2d:%02d\n", (info->tm_hour+CCT)%24, info->tm_min);

   return(0);
}
/* 运行结果
当前的世界时钟：
伦敦： 8:19
中国：15:19
*/
```

### mktime
#### 描述
把 timeptr 所指向的结构转换为一个依据本地时区的 time_t 值。
mktime()用来将参数timeptr所指的tm结构数据转换成从公元1970年1月1日0时0分0 秒算起至今的UTC时间所经过的秒数。
#### 声明
`time_t mktime(struct tm *timeptr)`
#### 参数
struct tm

#### 返回
该函数返回一个 time_t 值，该值对应于以参数传递的日历时间。如果发生错误，则返回 -1 值。
#### 实例
```c
#include <stdio.h>
#include <time.h>
 
int main(void)
{
    time_t timeValue = 0;
    struct tm *p = NULL;
 
    time(&timeValue);
    printf("转换之前的时间秒数是 = %ld\n",timeValue);
    p = localtime(&timeValue);
 
    timeValue = 0;
    timeValue = mktime(p);
    printf("转换之后的时间秒数是 = %ld\n",timeValue);
 
    return 0;
}
/*
转换之前的时间秒数是 = 1653031628
转换之后的时间秒数是 = 1653031628
*/
```

### difftime
#### 描述
得到两次机器时间差，单位为秒
#### 声明
`double difftime(time_t time2, time_t time1)`
#### 参数
time1,time2分别表示两个不同的机器时间，该参数应使用time函数获得
#### 返回
时间差，单位为秒
#### 实例
```c
#include <time.h>  
#include <stdio.h>  
int main()  
{  
    time_t first,second;  
    time(&first);  
    sleep(1);  
    time(&second);
    printf("The difference is: %f seconds",difftime(second,first));  

    return 0;  
}  
//result: The difference is: 1.000000 seconds
```

### clock函数
#### 描述
用来计算程序或程序的某一段的执行时间。
clock()是程序从启动到函数调用占用CPU的时间。这个函数返回从“开启这个程序进程”到“程序中调用clock()函数”时之间的CPU时钟计时单元（clock tick）数。
#### 声明
`clock_t clock(void)`
#### 参数
void
#### 返回
返回clock函数执行起（一般为程序的开头），处理器时钟所使用的时间。
#### 实例
```c
//https://blog.csdn.net/shomy_liu/article/details/45110949
#include<stdio.h>
#include<time.h>

int main()
{
    int i = 100000000;
	clock_t start,finish; //定义开始，结束变量
	start = clock();//初始化
	while( i-- );
	finish = clock();//初始化结束时间
	double duration = (double)(finish - start) / CLOCKS_PER_SEC;//转换浮点型
	printf( "%lf seconds\n", duration );
    return 0;
}
/*
result:
0.149000 seconds
*/
```

### clock_gettime
参考：https://blog.csdn.net/weixin_44880138/article/details/102605681
```c
struct timespec {
	time_t tv_sec; // seconds
	long tv_nsec; // and nanoseconds 纳秒
};
```
struct timespec有两个成员，一个是秒，一个是纳秒, 所以最高精确度是纳秒。
一般由函数`int clock_gettime(clockid_t, struct timespec *)`获取特定时钟的时间。
#### 描述
https://linux.die.net/man/3/clock_gettime

The function clock_getres() finds the resolution (precision) of the specified clock clk_id, and, if res is non-NULL, stores it in the struct timespec pointed to by res. The resolution of clocks depends on the implementation and cannot be configured by a particular process. If the time value pointed to by the argument tp of clock_settime() is not a multiple of res, then it is truncated to a multiple of res.
The functions clock_gettime() and clock_settime() retrieve and set the time of the specified clock clk_id.

函数 clock _ getres ()查找指定时钟 clk _ id 的分辨率(精度) ，如果 res 为非 null，则将其存储在 res 指向的结构 timespec 中。时钟的解析取决于实现，不能由特定进程配置。如果时钟 _ settime ()的参数 tp 指向的时间值不是 res 的倍数，则将其截断为 res 的倍数。

函数 clock _ gettime ()和 clock _ settime ()检索并设置指定时钟 clk _ id 的时间。
#### 声明
`int clock_gettime(clockid_t clk_id,struct timespec *tp);`
#### 参数
- clk_id : 检索和设置的clk_id指定的时钟时间。常用如下4种时钟：
```c
CLOCK_REALTIME 系统当前时间，从1970年1.1日算起
CLOCK_MONOTONIC 系统的启动时间，不能被设置
CLOCK_PROCESS_CPUTIME_ID 本进程运行时间
CLOCK_THREAD_CPUTIME_ID 本线程运行时间
```

#### 返回
 return 0 for success, or -1 for failure (in which case errno is set appropriately)
 返回0表示成功,-1表示失败(在这种情况下恰当地设置了 errno)
#### 实例
```c
#include <time.h>
#include <stdio.h>
#include <unistd.h>
 
int main(int argc, char **argv)
{
    struct timespec time1 = {0, 0}; 
    struct timespec time2 = {0, 0};
	
    float temp;
 
    clock_gettime(CLOCK_REALTIME, &time1);      
    usleep(1000);
    clock_gettime(CLOCK_REALTIME, &time2);   
    temp = (time2.tv_nsec - time1.tv_nsec) / 1000000;
    printf("time = %f ms\n", temp);
    return 0;
}
//结果：time = 8.000000 ms
```

### gettimeofday
```c
struct timeval {
	time_t tv_sec; // seconds
	long tv_usec; // microseconds 微秒	
};
struct timezone{
	int tz_minuteswest; //miniutes west of Greenwich
	int tz_dsttime; //type of DST correction
};
```
struct timeval有两个成员，一个是秒，一个是微秒, 所以最高精确度是微秒。
一般由函数`int gettimeofday(struct timeval *tv, struct timezone *tz)`获取系统的时间
#### 描述
gettimeofday()会把目前的时间用tv 结构体返回，当地时区的信息则放到tz所指的结构中。
#### 声明
`int gettimeofday(struct  timeval*tv,struct  timezone *tz )`
#### 参数
struct  timeval
struct  timezone
在gettimeofday()函数中tv或者tz都可以为空。如果为空则就不返回其对应的结构体。
#### 返回
函数执行成功后返回0，失败后返回-1，错误代码存于errno中。
#### 实例
通过这个函数即可 准确获取多线程序运行时间。
```c
#include <stdio.h>
#include <sys/time.h>
#include <stdlib.h>
int main()
{
    float time_use=0;
    struct timeval start;
    struct timeval end;

    gettimeofday(&start,NULL); //gettimeofday(&start,&tz);结果一样
    printf("start.tv_sec:%ld\n",start.tv_sec);
    printf("start.tv_usec:%ld\n",start.tv_usec);

    int i = 100000000;
    while(i--);

    gettimeofday(&end,NULL);
    printf("end.tv_sec:%ld\n",end.tv_sec);
    printf("end.tv_usec:%ld\n",end.tv_usec);

    time_use=(end.tv_sec-start.tv_sec)*1000000+(end.tv_usec-start.tv_usec);//微秒
    printf("time_use is %.10f\n",time_use);
}
/*
start.tv_sec:1653034868
start.tv_usec:750288
end.tv_sec:1653034868
end.tv_usec:903878
time_use is 153590.0000000000
*/
```




## toupper
### 描述
C 库函数 int toupper(int c) 把小写字母转换为大写字母。
### 声明
`int toupper(int c);`

### 参数
c -- 这是要被转换为大写的字母。
### 返回
如果 c 有相对应的大写字母，则该函数返回 c 的大写字母，否则 c 保持不变。返回值是一个可被隐式转换为 char 类型的 int 值。
### 实例
```c
#include <stdio.h>
#include <ctype.h>

int main()
{
   int i = 0;
   char c;
   char str[] = "runoob";
   
   while(str[i])
   {
      putchar (toupper(str[i]));
      i++;
   }
   
  return(0);
}
//result: RUNOOB
```
## atol
### 描述
C 库函数 long int atol(const char *str) 把参数 str 所指向的字符串转换为一个长整数（类型为 long int 型）。

### 声明
下面是 atol() 函数的声明。
`long int atol(const char *str)`
### 参数
str -- 要转换为长整数的字符串。
### 返回值
该函数返回转换后的长整数，如果没有执行有效的转换，则返回零。
实例
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
   long val;
   char str[20];
   
   strcpy(str, "98993489");
   val = atol(str);
   printf("字符串值 = %s, 长整型值 = %ld\n", str, val);

   strcpy(str, "runoob.com");
   val = atol(str);
   printf("字符串值 = %s, 长整型值 = %ld\n", str, val);

   return(0);
}
```
运行结果：
```
字符串值 = 98993489, 长整型值 = 98993489
字符串值 = runoob.com, 长整型值 = 0
```
## isspace
C 库函数 int isspace(int c) 检查所传的字符是否是空白字符。

标准的空白字符包括：
```
' '     (0x20)    space (SPC) 空格符
'\t'    (0x09)    horizontal tab (TAB) 水平制表符    
'\n'    (0x0a)    newline (LF) 换行符
'\v'    (0x0b)    vertical tab (VT) 垂直制表符
'\f'    (0x0c)    feed (FF) 换页符
'\r'    (0x0d)    carriage return (CR) 回车符
```
### 声明
下面是 isspace() 函数的声明。
`int isspace(int c);`

### 参数
c -- 这是要检查的字符。
### 返回值
如果 c 是一个空白字符，则该函数返回非零值（true），否则返回 0（false）。
## snprintf
描述
C 库函数` int snprintf(char *str, size_t size, const char *format, ...)` 设将可变参数(...)按照 format 格式化成字符串，并将字符串复制到 str 中，size 为要写入的字符的最大数目，超过 size 会被截断。
声明
`int snprintf ( char * str, size_t size, const char * format, ... );`
参数
str -- 目标字符串。
size -- 拷贝字节数(Bytes)。
format -- 格式化成字符串。
... -- 可变参数。

应用：writer-simple.c --388

## fopen()函数：
作用: 在C语言中fopen()函数用于打开指定路径的文件，获取指向该文件的指针。

```c
FILE * fopen(const char * path,const char * mode);
    -- path: 文件路径，如："F:\Visual Stdio 2012\test.txt"
    -- mode: 文件打开方式，例如：
             "r" 以只读方式打开文件，该文件必须存在。
             "w" 打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。
            "w+" 打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。
             "a" 以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留）
             "a+" 以附加方式打开可读写的文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾后，即文件原先的内容会被保留。（原来的EOF符不保留）
            "wb" 只写打开或新建一个二进制文件，只允许写数据。
            "wb+" 读写打开或建立一个二进制文件，允许读和写。
             "ab" 追加打开一个二进制文件，并在文件末尾写数据。
             "ab+"读写打开一个二进制文件，允许读，或在文件末追加数据。   
    --返回值: 文件顺利打开后，指向该流的文件指针就会被返回。如果文件打开失败则返回NULL，并把错误代码存在errno中。        
```
## fwrite()函数：
作用：在C语言中fwrite()函数常用语将一块内存区域中的数据写入到本地文本。
函数原型：

```c
size_t fwrite(const void* buffer, size_t size, size_t count, FILE* stream);
    -- buffer:指向数据块的指针
    -- size:每个数据的大小，单位为Byte(例如：sizeof(int)就是4)
    -- count:数据个数
    -- stream:文件指针
```
注意：返回值随着调用格式的不同而不同：

```c
(1) 调用格式：fwrite(buf,sizeof(buf),1,fp);
成功写入返回值为1(即count)
(2)调用格式：fwrite(buf,1,sizeof(buf),fp);
成功写入则返回实际写入的数据个数(单位为Byte)
(3)写完数据后要调用fclose()关闭流，不关闭流的情况下，每次读或写数据后，文件指针都会指向下一个待写或者读数据位置的指针。
```
## fread函数
函数原型：

```c
size_t   fread(   void   *buffer,   size_t   size,   size_t   count,   FILE   *stream   ) 
  buffer   是读取的数据存放的内存的指针（可以是数组，也可以是新开辟的空间，buffer就是一个索引）   
    size       是每次读取的字节数  
  count     是读取次数  
  strean   是要读取的文件的指针  
```
例如   从文件fp里读取100个字节   可用以下语句  
```c
  fread(buffer,100,1,fp)  
  fread(buffer,50,2,fp)  
  fread(buffer,1,100,fp)   
```
注意：返回值随着调用格式的不同而不同：
(1) 调用格式：fread(buf,sizeof(buf),1,fp);
读取成功时：当读取的数据量正好是sizeof(buf)个Byte时，返回值为1(即count)
                       否则返回值为0(读取数据量小于sizeof(buf))
(2)调用格式：fread(buf,1,sizeof(buf),fp);
读取成功返回值为实际读回的数据个数(单位为Byte)

## ftell函数
### 作用： ftell() 返回当前文件位置，也就是说返回FILE指针当前位置。
#### 头文件：#include <stdio.h>
```c
long ftell(FILE *stream);
```
函数 ftell() 用于得到文件位置指针当前位置相对于文件首的偏移字节数。在随机方式存取文件时，由于文件位置频繁的前后移动，程序不容易确定文件的当前位置。使用fseek函数后再调用函数ftell()就能非常容易地确定文件的当前位置。

```c
ftell(fp);
```

利用函数 ftell() 也能方便地知道一个文件的长。如以下语句序列：

```c
 fseek(fp, 0L,SEEK_END); 
 len =ftell(fp)+1; 
```

 首先将文件的当前位置移到文件的末尾，然后调用函数ftell()获得当前位置相对于文件首的位移，该位移值等于文件所含字节数。


## fseek函数的基本使用
### 作用：重定位流（数据流/文件）的内部位置指针。
#### 头文件：#include <stdio.h>
```c
函数原型：int fseek(FILE *stream, long offset, int fromwhere);
```

参数：

```c
stream：指向打开的文件指针。

offset：以基准点为起始点的偏移量。

fromwhere：基准点。
```

返回值：
```c
成功，返回0；失败返回-1。
```
其中基准点包括这三个枚举：
```c
SEEK_SET：文件头。

SEEK_CUR：当前位置。

SEEK_END：文件件尾。
```
接下来将用一个小例子来说明使用

```c
// 这是想要的函数
FILE* fp;
int test_data = 100;
fp = fopen("test.bin", "r+"); // 打开文件
if(fp == NULL)
	exit(0);
fseek(fp, 10, SEEK_SET); //将指针移到开始后的10字节位置
fwrite(&test_data, sizeof(test_data), 1, fp);//写数据
fclose(fp);

```






## inet_pton()、inet_ntop()相关函数详解
链接：https://blog.csdn.net/zyy617532750/article/details/58595700

<font color=red> **1.把ip地址转化为用于网络传输的二进制数值**</font>

```c
int inet_aton(const char *cp, struct in_addr *inp);
```

inet_aton() 转换网络主机地址ip(如192.168.1.10)为二进制数值，并存储在struct in_addr结构中，即第二个参数\*inp,函数返回非0表示cp主机有地有效，返回0表示主机地址无效。（这个转换完后不能用于网络传输，还需要调用htons或htonl函数才能将主机字节顺序转化为网络字节顺序）

```c
in_addr_t inet_addr(const char *cp);
```

inet_addr函数转换网络主机地址（如192.168.1.10)为网络字节序二进制值，如果参数char \*cp无效，函数返回-1(INADDR_NONE),这个函数在处理地址为255.255.255.255时也返回－1,255.255.255.255是一个有效的地址，不过inet_addr无法处理;


<font color=red> **2.将网络传输的二进制数值转化为成点分十进制的ip地址**</font>

```c
char *inet_ntoa(struct in_addr in);
```

inet_ntoa 函数转换网络字节排序的地址为标准的ASCII以点分开的地址,该函数返回指向点分开的字符串地址（如192.168.1.10)的指针，该字符串的空间为静态分配的，这意味着在第二次调用该函数时，上一次调用将会被重写（复盖），所以如果需要保存该串最后复制出来自己管理！
我们如何输出一个点分十进制的IP呢？我们来看看下面的程序：

```c
#include <stdio.h>   
#include <sys/socket.h>   
#include <netinet/in.h>   
#include <arpa/inet.h>   
#include <string.h>   
int main()   
{   
	struct in_addr addr1,addr2;   
	ulong l1,l2;   
	l1= inet_addr("192.168.0.74");   
	l2 = inet_addr("211.100.21.179");   
	memcpy(&addr1, &l1, 4);   
	memcpy(&addr2, &l2, 4);   
	printf("%s : %s\n", inet_ntoa(addr1), inet_ntoa(addr2)); //注意这一句的运行结果   
	printf("%s\n", inet_ntoa(addr1));   
	printf("%s\n", inet_ntoa(addr2));  
	return 0;   
}   
```
实际运行结果如下：　
```c
192.168.0.74 : 192.168.0.74          //从这里可以看出,printf里的inet_ntoa只运行了一次。　　

192.168.0.74　　

211.100.21.179　
```
inet_ntoa返回一个char *,而这个char *的空间是在inet_ntoa里面静态分配的，所以inet_ntoa后面的调用会覆盖上一次的调用。第一句printf的结果只能说明在printf里面的可变参数的求值是从右到左的，仅此而已。


<font color=red> **3.新型网路地址转化函数inet_pton和inet_ntop**</font>

这两个函数是随IPv6出现的函数，对于IPv4地址和IPv6地址都适用，函数中p和n分别代表表达（presentation)和数值（numeric)。地址的表达格式通常是ASCII字符串，数值格式则是存放到套接字地址结构的二进制值。

```c
#include <arpe/inet.h>
int inet_pton(int family, const char *strptr, void *addrptr);     //将点分十进制的ip地址转化为用于网络传输的数值格式
        返回值：若成功则为1，若输入不是有效的表达式则为0，若出错则为-1
 将点分十进制串转换成网络字节序二进制值，此函数对IPv4地址和IPv6地址都能处理。
 第一个参数可以是AF_INET或AF_INET6：
 第二个参数是一个指向点分十进制串的指针：
 第三个参数是一个指向转换后的网络字节序的二进制值的指针。
 
const char * inet_ntop(int family, const void *addrptr, char *strptr, size_t len);     //将数值格式转化为点分十进制的ip地址格式
        返回值：若成功则为指向结构的指针，若出错则为NULL
 和inet_pton函数正好相反，inet_ntop函数是将网络字节序二进制值转换成点分十进制串。
 第一个参数可以是AF_INET或AF_INET6：
 第二个参数是一个指向网络字节序的二进制值的指针；
 第三个参数是一个指向转换后的点分十进制串的指针；
 第四个参数是目标的大小，以免函数溢出其调用者的缓冲区。

```
（1）这两个函数的family参数既可以是AF_INET（ipv4）也可以是AF_INET6（ipv6）。如果，以不被支持的地址族作为family参数，这两个函数都返回一个错误，并将errno置为EAFNOSUPPORT.
（2）第一个函数尝试转换由strptr指针所指向的字符串，并通过addrptr指针存放二进制结果，若成功则返回值为1，否则如果所指定的family而言输入字符串不是有效的表达式格式，那么返回值为0.

（3）inet_ntop进行相反的转换，从数值格式（addrptr）转换到表达式（strptr)。inet_ntop函数的strptr参数不可以是一个空指针。调用者必须为目标存储单元分配内存并指定其大小，调用成功时，这个指针就是该函数的返回值。len参数是目标存储单元的大小，以免该函数溢出其调用者的缓冲区。如果len太小，不足以容纳表达式结果，那么返回一个空指针，并置为errno为ENOSPC。

```c
inet_pton(AF_INET, ip, &foo.sin_addr);   //  代替 foo.sin_addr.addr=inet_addr(ip);
 
 
char str[INET_ADDRSTRLEN];
char *ptr = inet_ntop(AF_INET,&foo.sin_addr, str, sizeof(str));      // 代替 ptr = inet_ntoa(foo.sin_addr)
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
int main (void)
{
	char IPdotdec[20]; //存放点分十进制IP地址
	struct in_addr s; // IPv4地址结构体
	// 输入IP地址
	printf("Please input IP address: ");
	scanf("%s", IPdotdec);
	// 转换
	inet_pton(AF_INET, IPdotdec, (void *)&s);
	printf("inet_pton: 0x%x\n", s.s_addr); // 注意得到的字节序
	// 反转换
	inet_ntop(AF_INET, (void *)&s, IPdotdec, 16);
	printf("inet_ntop: %s\n", IPdotdec);
}

```
## htonl()、ntohl()、htons()、ntohs()

```c
htonl()--"Host to Network Long"
ntohl()--"Network to Host Long"
htons()--"Host to Network Short"
ntohs()--"Network to Host Short" 
```
网络传输模式
     网络传输通讯过程中，采用大端在前的方式进行数据发送，对于采用小端模式进行数据存储的系统，要把多字节的数据通过网络传输出去，则需要将数据的字节序进行转换，再进行发送。接收的过程亦然。
 在Intel机器下，执行以下程序
 
```c
int main()
{
    printf("%d n",htons(16));
    return 0;
}
```
得到的结果是4096，初一看感觉很怪。
 解释如下，数字16的16进制表示为0x0010，数字4096的16进制表示为0x1000。 由于Intel机器是小尾端，存储数字16时实际顺序为1000，==存储4096时实际顺序为0010。==因此在发送网络包时为了报文中数据为==0010==，需要 经过htons进行字节转换。如果用IBM等大尾端机器，则没有这种字节顺序转换，但为了程序的可移植性，也最好用这个函数。
 - htons和htonl都是把主机字节序转换成网络字节序。那什么时候用htons，什么时候用htonl？？听网上说一个是16位一个是32位，但是如何去判断？假设servaddr.sin_port = htons(5555);用htonl可以吗？根据什么可以判断？
 - 根据要转换的值是否超过16位来决定，5555转换为2进制为1 0101 1011 0011 ，为13位，所以一般用htons,当然用htonl也可以；
但是如果要转换的数 转换成2进制超过16位，则只能用htonl，此时如果用htons，16位以上的数舍去，造成数据值偏差。



