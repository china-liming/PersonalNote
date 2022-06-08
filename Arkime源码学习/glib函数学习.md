@[toc]
比较好的glib笔记
[glib书籍](https://blog.csdn.net/rambo_jay/article/details/6340214)
## g_thread_new ()

```c
GThread *
g_thread_new (const gchar *name,
              GThreadFunc func,
              gpointer data);
```
### 描述：
`
The `name` can be useful for discriminating threads in a debugger. It is not used for other purposes and does not have to be unique. Some systems restrict the length of `name` to 16 bytes.
该名称可用于在调试器中区分线程。它不用于其他目的，也不必是唯一的。有些系统将名称的长度限制为16字节。

If the thread can not be created the program aborts. See `g_thread_try_new()` if you want to attempt to deal with failures.
如果无法创建线程，程序将中止。如果要尝试处理失败，请参阅g_thread_try_new（）。
If you are using threads to offload (potentially many) short-lived tasks, `GThreadPool` may be more appropriate than manually spawning and tracking multiple `GThreads`.
如果您使用线程来卸载（可能有很多）短期任务，那么GThreadPool可能比手动生成和跟踪多个GThread更合适。

To free the struct returned by this function, use g_thread_unref(). Note that `g_thread_join()` implicitly unrefs the `GThread` as well.
要释放此函数返回的结构，请使用g_thread_unref（）。请注意，g_thread_join（）也隐式地解除了GThread的锁定。

New threads by default inherit their scheduler policy (POSIX) or thread priority (Windows) of the thread creating the new thread.
默认情况下，新线程继承创建新线程的线程的调度程序策略（POSIX）或线程优先级（Windows）。

This behaviour changed in GLib 2.64: before threads on Windows were not inheriting the thread priority but were spawned with the default priority. Starting with GLib 2.64 the behaviour is now consistent between Windows and POSIX and all threads inherit their parent thread’s priority.
这种行为在GLib 2.64中发生了改变：Windows上的线程没有继承线程优先级，而是以默认优先级生成。从GLib 2.64开始，Windows和POSIX之间的行为现在是一致的，所有线程都继承其父线程的优先级。
`

此函数创建一个新线程。新线程通过使用参数数据调用func开始。该线程将一直运行，直到func返回，或者直到从新线程调用g_thread_exit()。func的返回值成为线程的返回值，可以通过**g_thread_join()**获得。

### 参数：

name ：新线程名称（可选）
func：执行新线程的函数
data：新线程函数的参数
### 返回：
一个新的GThread

## g_thread_unref
void g_thread_unref (GThread* thread)

Description

Decrease the reference count on thread, possibly freeing all resources associated with it.

Note that each thread holds a reference to its GThread while it is running, so it is safe to drop your own reference to it if you don’t need it anymore.
### 描述
减少线程上的引用计数，可能会释放与之关联的所有资源。
请注意，每个线程在运行时都持有对其GThread的引用，因此，如果不再需要它，可以安全地删除对它的引用。

## GLib基本类型
### Description 描述
定义一些普通使用类型，可分为4组：
1.不属于标准C的新类型：gboolean, gsize, gssize.
2.可以在任何平台使用的整数：gint8, guint8, gint16, guint16, gint32, guint32, gint64, guint64.
3.与标准C相似但更好用的类型：gpointer, gconstpointer, guchar, guint, gushort, gulong.
4.与标准C基本一致的类型：gchar, gint, gshort, glong, gfloat, gdouble.

### Details 详细说明
```
#### gboolean
typedef gint   gboolean;
标准的boolean类型。它只储存两个值：TRUE和FALSE

#### gpointer
typedef void* gpointer;
无类型的指针。Gpointer比 void*更好看和更好用。

#### gconstpointer
typedef const void *gconstpointer;
一个常数的无类型指针。指针的值不能被更改。
典型的使用子函数原型上，指出指针所指向的数据是不能被函数更改的。

#### gchar
typedef char   gchar;
和标准C的char类型一致。

#### guchar
typedef unsigned char   guchar;
和标准C的 unsigned char一致。

#### gint
typedef int    gint;
类似标准C的int类型。其值能被设置G_MININT 到 G_MAXINT范围。

#### guint
typedef unsigned int    guint;
类似标准C的unsigned int类型。其值能被设置0到 G_MAXUINT的范围。

#### gshort
typedef short  gshort;
类似标准C的short类型。其值能被设置G_MINSHORT 到 G_MAXSHORT范围。

#### gushort
typedef unsigned short  gushort;
类似标准C的unsigned short类型。其值能被设置0到 G_MAXUSHORT的范围。

#### glong
typedef long   glong;
类似标准C的long类型。其值能被设置G_MINLONG 到 G_MAXLONG范围。

#### gulong
typedef unsigned long   gulong;
类似标准C的unsigned long类型。其值能被设置0到 G_MAXULONG的范围。

#### gint8
typedef signed char gint8;
在任何平台上都保证是一个有符号8位整数。
其取值返回为 -128 到 127

#### guint8
typedef unsigned char guint8;
在任何平台上都保证是一个无符号8位整数。
其取值返回为 0 到 255

#### gint16
typedef signed short gint16;
在任何平台上都保证是一个有符号16位整数。
其取值返回为 -32,768 到 32,767

#### guint16
typedef unsigned short guint16;
在任何平台上都保证是一个无符号16位整数。
其取值返回为 0 到 65,535

#### gint32
typedef signed int gint32;
在任何平台上都保证是一个有符号32位整数。
其取值返回为 -2,147,483,648 到 2,147,483,647

#### guint32
typedef unsigned int guint32;
在任何平台上都保证是一个无符号32位整数。
其取值返回为 0 到 4,294,967,295

#### gint64
G_GNUC_EXTENSION typedef signed long long gint64;
在任何平台上都保证是一个有符号64位整数。
其取值返回为 -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807

#### guint64 ()
GLIB_VAR            guint64                             ();
在任何平台上都保证是一个无符号32位整数。
其取值返回为 0 到 18,446,744,073,709,551,615

#### G_GINT64_CONSTANT()
#define G_GINT64_CONSTANT(val)  (G_GNUC_EXTENSION (val##LL))
This macro is used to insert 64-bit integer literals into the source code.
（不太懂）
val :  a literal integer value, e.g. 0x1d636b02300a7aa7U.

#### G_GUINT64_CONSTANT()
#define G_GUINT64_CONSTANT(val)  (G_GNUC_EXTENSION (val##ULL))
This macro is used to insert 64-bit unsigned integer literals into the source code.
（不太懂）
val :  a literal integer value, e.g. 0x1d636b02300a7aa7U.
Since 2.10

#### gfloat
typedef float   gfloat;
类似标准C的float类型。其值可以设置-G_MAXFLOAT 到 G_MAXFLOAT范围

#### gdouble
typedef double  gdouble;
类似标准C的double类型。其值可以设置-G_MAXDOUBLE 到G_MAXDOUBLE范围

#### gsize
typedef unsigned int gsize;
一个无符号整形，用来储存sizeof的结果，与C99的size_t类型类似。
这个类型能足够存储下一个指针的数据。它常常在32为平台上是32位的，在64位平台上是64位的。

#### gssize
typedef signed int gssize;
一个有符号gsize，与大多数平台的ssize_t类型类似。

#### goffset
typedef gint64 goffset;
一个有符号的整形，用来作为文件偏移。类似于C99的off64_t
Since: 2.14
```


## GKeyFile读取配置文件
### 实例
参考：https://blog.csdn.net/hubbybob1/article/details/48293127
```
struct GKeyFile 
{
  /* No available fields */
}

GKeyFile * config;
```
在Windows系统中，也存在这类文件，通常后缀名是ini。在GTK的世界中，称这类文件为Key File（因为这个文件包含很多的字段（key）？）。这两类文件看上去差不多，但是还是有一些区别的：

首先就是注释，init文件把“;”视作注释开始，而Key File，显然是用“#”
Key File所有的配置项都在配置段中，即任何配置项之前肯定有类似”[配置段]“的东西。
还有就是众所周知的，Windows下的ini文件通常不是UTF8编码，而Linux下，显然推荐这么干
另外就是配置项和配置段大小写，Linux下的key file是区分的
在Keyfile中允许数据类型为逻辑型的配置项，取值为true或者false。而ini文件里大概只有用整形的配置项目与之对应了
```c
//简单介绍完了，我们在用一个简单的例子：
#include
#include
int main(int argc, char **argv)
{
GKeyFile * config;
gchar *str;
config = g_key_file_new();
g_key_file_load_from_file(config, argv[1], 0, NULL);
str=g_key_file_get_string(config,"options","HoldPkg",NULL);
printf("HoldPkg of options section is "%s"n", str);
g_key_file_free(config);
return 0;
}
```
使用下面的命令编译：
```bash
gcc pkg-config --cflags --libs glib-2.0 dummy.c
```

这里的pkg-config –cflags –libs glib-2.0用于自动查找调用glib所需的头文件和库文件路径，并且按照CFLAGS所需的格式输出。
我们用的范例配置文件（援引自pacman的配置文件）如下：
```
[options]
HoldPkg = pacman glibc
```
运行效果怎么样呢？
```
athurg@AT.nts-intl.com tmp]$./a.out /etc/pacman.conf
HoldPkg of options section is "pacman glibc"
```
范例程序很简单，但是五脏俱全，要使用glib来解析配置文件，大概有下面的几个流程：
首先用g_key_file_new()建立一个GKeyFile缓冲区
然后用g_key_file_load_from_file()来初始化填充这个缓冲区
接着，你就可以用g_key_file_get_数据类型()来获取数据了
或者，使用g_key_file_set_数据类型()来更新设置数据
而且，使用g_key_file_remove_key()来删除设置项
那么增加设置项呢？当设置数据时，如果没有该数据项，默认就会增加
当然，在此之前，如果不放心的话，还可以通过g_key_file_has_group()、g_key_file_has_key()来判断数据段、数据项是否存在
末了，你可能想把更新后的配置写回去，这个有点怪，需要用g_key_file_to_data()将缓冲区里的配置数据转成字符串，然后将这个字符串写入到配置文件即可。这个函数同时也会返回字符串的长度，供写入时使用
最后，一个良好的习惯——调用g_key_file_free()来释放缓冲区
数据类型包括字符串、整形、长整型、浮点……，以及这类数据组成的数组。

### g_key_file_new
#### 定义：
```c
GKeyFile* g_key_file_new (void)
```
#### 描述：
```
Creates a new empty GKeyFile object. Use g_key_file_load_from_file(), g_key_file_load_from_data(), g_key_file_load_from_dirs() or g_key_file_load_from_data_dirs() to read an existing key file.
Available since:	2.6
创建一个新的空GKeyFile对象。使用g_key_file_load_from_file()、
g_key_file_load_from_data()、g_key_file_load_from_dirs()或g_key_file_load_from_data_dirs()
读取现有的密钥文件。
可用自:2.6
```
#### 返回
Returns:	GKeyFile
An empty GKeyFile.

### g_key_file_load_from_file
```c
gboolean 
g_key_file_load_from_file (GKeyFile* key_file,const gchar* file,GKeyFileFlags flags,GError** error)
```

#### 描述
Loads a key file into an empty GKeyFile structure.
If the OS returns an error when opening or reading the file, a G_FILE_ERROR is returned. If there is a problem parsing the file, a G_KEY_FILE_ERROR is returned.
This function will never return a G_KEY_FILE_ERROR_NOT_FOUND error. If the file is not found, G_FILE_ERROR_NOENT is returned.
Available since:	2.6

将密钥文件加载到空的GKeyFile结构中。
如果操作系统在打开或读取文件时返回错误，则会返回G_file_错误。如果解析文件时出现问题，将返回G_KEY_file_错误。
此函数永远不会返回G_KEY_FILE_ERROR_NOT_FOUND错误。如果找不到该文件，则返回G_file_ERROR_NOENT。
有效期：2.6

#### 参数

Parameters
file	const gchar*
        The path of a filename to load, in the GLib filename encoding.
 	    The data is owned by the caller of the function.
 	    The value is a file system path, using the OS encoding.
flags	GKeyFileFlags
        Flags from GKeyFileFlags.
        ==G_KEY_FILE_NONE==: No flags for the key file will be set.
		==G_KEY_FILE_KEEP_COMMENTS==: All of the comments in the key file should be read so that they can remain in the same position when the file is saved. If you do not set this flag, all comments will be lost.
		应该读取密钥文件中的所有注释，以便在保存文件时它们可以保持在同一位置。如果你没有设置这个标志，所有的评论将会丢失。
		==G_KEY_FILE_KEEP_TRANSLATIONS==: All of the translations in the key file should be read so that they will not be lost when you save the file.
		应该读取密钥文件中的所有翻译，以便在保存文件时不会丢失它们。

error	GError **
 	    The return location for a GError*, or NULL.

#### 返回
Return value
Returns:	gboolean
 	
TRUE if a key file could be loaded, FALSE otherwise.


### g_key_file_has_key
#### Declaration
```c
gboolean
g_key_file_has_key (
  GKeyFile* key_file,
  const gchar* group_name,
  const gchar* key,
  GError** error
)
```

#### Description
Looks whether the key file has the key key in the group group_name.

Note that this function does not follow the rules for GError strictly; the return value both carries meaning and signals an error. To use this function, you must pass a GError pointer in error, and check whether it is not NULL to see if an error occurred.

Language bindings should use g_key_file_get_value() to test whether or not a key exists.

Available since:	2.6
This method is not directly available to language bindings.
[−]
#### Parameters
group_name	const gchar*
A group name.
 	The data is owned by the caller of the function.
 	The value is a NUL terminated UTF-8 string.

key	const gchar*
A key name.
 	The data is owned by the caller of the function.
 	The value is a NUL terminated UTF-8 string.

error	GError **
 	The return location for a GError*, or NULL.
[−]
#### Return
Returns:	gboolean
 	
TRUE if key is a part of group_name, FALSE otherwise.



### g_key_file_get_string_list
```
gchar**
g_key_file_get_string_list (
  GKeyFile* key_file,
  const gchar* group_name,
  const gchar* key,
  gsize* length,
  GError** error
)
```
### strdup
```
gchar*
g_strdup (
  const gchar* str
)
```

#### Description
Duplicates a string. If `str` is `NULL` it returns `NULL`. The returned string should be freed with `g_free()` when no longer needed.
复制一个字符串。如果'str'为'NULL'，则返回'NULL'。当不再需要时，应使用'g_free（）'释放返回的字符串。


## GMainLoop
### 描述
文档描述：
The `GMainLoop` struct is an opaque data type representing the main event loop of a GLib or GTK+ application.
GMainLoop结构是一种不透明的数据类型，表示GLib或GTK+应用程序的主事件循环。
参考描述：
[链接](https://blog.csdn.net/arag2009/article/details/17095361)
main loop使用模式大致如下：
```
loop = g_main_loop_new (NULL, TRUE);
g_main_loop_run (loop);
```
g_main_loop_new创建一个main loop对象，一个main loop对象只能被一个线程使用，但一个线程可以有多个main loop对象。
在GTK+应用中，一个线程使用多个main loop的主要用途是实现模态对话框，它在gtk_dialog_run函数里创建一个新的main loop，通过该main loop分发消息，直到对话框关闭为止。 
g_main_loop_run则是进入主循环，它会一直阻塞在这里，直到让它退出为止。有事件时，它就处理事件，没事件时就睡眠。
g_main_loop_quit则是用于退出主循环，相当于Win32下的PostQuitMessage函数。 
Glib main loop的最大特点就是支持多事件源，使用非常方便。来自用户的键盘和鼠标事件、来自系统的定时事件和socket事件等等，还支持一个称为idle的事件源，其主要用途是实现异步事件。

我们需要了解event loop的这三个基本结构：GMainLoop, GMainContext和GSource。  
它们之间的关系是这样的：  
GMainLoop -> GMainContext -> {GSource1, GSource2, GSource3......}  
每个GmainLoop都包含一个GMainContext成员，而这个GMainContext成员可以装各种各样的GSource，GSource则是具体的各种Event处理逻辑了。在这里，可以把GMainContext理解为GSource的容器。（不过它的用处不只是装GSource）  
创建GMainLoop使用函数g_main_loop_new, 它的第一个参数就是需要关联的GMainContext，如果这个值为空，程序会分配一个默认的Context给GMainLoop。  
把GSource加到GMainContext呢，则使用函数g_source_attach。
### 构建
[g_main_loop_new](##g_main_loop_new)
### 内部方法
[g_main_loop_get_context](##g_main_loop_get_context)
[g_main_loop_is_running](##g_main_loop_is_running)
[g_main_loop_quit](##g_main_loop_quit)
[g_main_loop_ref](##g_main_loop_ref)
[g_main_loop_run](##g_main_loop_run)
[g_main_loop_unref](##g_main_loop_unref)

### g_main_loop_new()
#### 定义
```
GMainLoop * g_main_loop_new (GMainContext* context,gboolean is_running)

```
#### 描述
Creates a new `GMainLoop` structure.
#### 参数
参数|类型
-------- | -----
context|	GMainContext <br/>A GMainContext (if NULL, the default context will be used).<br/>The argument can be NULL.<br/>The data is owned by the caller of the function.
is_running	|gboolean <br/>Set to TRUE to indicate that the loop is running. <br/>This is not very important since calling g_main_loop_run() will set this to TRUE anyway.
#### 返回
GMainLoop
A new `GMainLoop`.

### g_main_loop_get_context
#### 描述
Returns the GMainContext of loop.

### g_main_loop_is_running
#### 描述
Checks to see if the main loop is currently being run via g_main_loop_run().

### g_main_loop_quit
#### 描述
Stops a GMainLoop from running. Any calls to g_main_loop_run() for the loop will return.

### g_main_loop_ref
#### 描述
Increases the reference count on a GMainLoop object by one.

### g_main_loop_run
#### 描述
Runs a main loop until g_main_loop_quit() is called on the loop. If this is called for the thread of the loop’s GMainContext, it will process events from the loop, otherwise it will simply wait.

### g_main_loop_unref
#### 描述
Decreases the reference count on a GMainLoop object by one. If the result is zero, free the loop and free all associated memory.




## GOptionContext-命令行解析
### 使用
参考：[glib命令行解析库简单使用](https://blog.csdn.net/ciahi/article/details/6076786)
[用glib标准化程序的命令行解析](https://blog.csdn.net/magod/article/details/6086561)

简单来说，就是定义GOptionEntry结构，这个结构里面包含了命令项名字、类型以及简单介绍
然后创建GOptionContext，把定义的GOptionEntry结构放到GOptionContext中，调用g_option_context_parse就可以将命令选项都解出来
默认情况下，-h和--help可以查看程序的帮助，这个帮助信息是使用的GOptionEntry中定义的信息，还有一些辅助函数用来添加一些其它信息，或对这些信息的格式进行设置。
```c
static  gint repeats = 2;  
static  gint max_size = 8;  
static  gboolean verbose = FALSE;  
static  gboolean beep = FALSE;  
static  gboolean op_rand = FALSE;  
static  gchar arg_data[32] = { "arg data" };  
  
static  GOptionEntry entries[] =  
{  
    {"long name" ,  's' /*short-name*/ , 0 /*flags*/ , G_OPTION_ARG_STRING /*arg*/ ,  
        arg_data,  
        "description" ,  "arg_description" },  
  
    {"repeats" ,  'r' , 0, G_OPTION_ARG_INT,  
        &repeats, "Average over N repetitions" ,  "N" },  
  
    {"max-size" ,  'm' , 0, G_OPTION_ARG_INT,  
        &max_size, "Test up to 2^M items" ,  "M" },  
  
    {"verbose" ,  'v' , 0, G_OPTION_ARG_NONE,  
        &verbose, "Be verbose" , NULL},  
  
    {"beep" ,  'b' , 0, G_OPTION_ARG_NONE,  
        &beep, "Beep when done" , NULL},  
  
    {"rand" , 0, 0, G_OPTION_ARG_NONE,  
        &op_rand, "Randomize the data" , NULL},  
  
    {NULL}  
};  
  
int main (int  argc,  char  *argv[])  
{  
    GError *error = NULL;  
    GOptionContext *context = NULL;  
  
    context = g_option_context_new("- test tree model performance" );  
    g_option_context_add_main_entries(context, entries, NULL);  
    g_option_context_set_summary(context, "my-add-summary" );  
//  g_option_context_add_group(context, gtk_get_option_group(TRUE));   
    if  (!g_option_context_parse(context, &argc, &argv, &error)) {  
        g_print("option parsing failed: %s/n" , error->message);  
        exit (1);  
    }  
  
    g_option_context_free(context);  
  
    g_print("Now value is: repeats=%d, max_size=%d, /n" ,  
            repeats, max_size);  
  
    return  0;  
}  
```
从上面的例子可以看出，程序的Commandline parse主要工作就变成了
GOptionEntry 的定义：
```c
typedef   struct  {  
  const  gchar *long_name;     // 参数的完整名   
  gchar        short_name;   // 简写名   
  gint         flags;        // 参数选项标准 不关心直接赋0   
  
  GOptionArg   arg;          // 参数类型，int,string...   
  gpointer     arg_data;     // 默认参数值, 解析出来的数据，所要存储的位置
    
  const  gchar *description;   // 参数意义说明   --help可以查看到
  const  gchar *arg_description;   // 参数占位符说明   
} GOptionEntry;  
```
运行结果
```bash
[yunm@yunmiao src]$ ./test --help  
Usage:  
  test [OPTION...] - test tree model performance  
  
my-add-summary  
  
Help Options:  
  -h, --help                          Show help options  
  
Application Options:  
  -s, --long  name=arg_description     description  
  -r, --repeats=N                     Average over N repetitions  
  -m, --max-size=M                    Test up to 2^M items  
  -v, --verbose                       Be verbose  
  -b, --beep                          Beep when done  
  --rand                              Randomize the data  
  
[yunm@yunmiao src]$ ./test -r 10 -m 2  
Now value is: repeats=10, max_size=2,   
[yunm@yunmiao src]$ ./test   
Now value is: repeats=2, max_size=8,   
[yunm@yunmiao src]$ ./test -r 111111111111111111111  
option parsing failed: Integer value '111111111111111111111'   for  -r out of range  

```

### g_option_context_new
```c
GOptionContext* g_option_context_new (
  const gchar* parameter_string
)
```

#### 参数
```

parameter_string	const gchar*
	
A string which is displayed in the first line of --help output, after the usage summary programname [OPTION...]
```

#### 返回
```
GOptionContext
A newly created GOptionContext, which must be freed with g_option_context_free() after use.
```

### g_option_context_add_main_entries
```c
void g_option_context_add_main_entries (
  GOptionContext* context,
  const GOptionEntry* entries,
  const gchar* translation_domain
)
```

#### 描述
```c
A convenience function which creates a main group if it doesn’t exist, adds the entries to it and sets the translation domain.
一个方便的函数，如果主组不存在，则创建该主组，向其添加条目并设置转换域。
```

#### 参数
```c

entries
	An array of GOptionEntry
 	
	A NULL-terminated array of GOptionEntrys.
	一个以 null 结尾的 GOptionEntrys 数组。
translation_domain	const gchar*
 	
	A translation domain to use for translating the --help output for the options in entries with gettext(), or NULL.
	用于翻译 gettext ()或 NULL 条目中的选项的 -- help 输出的翻译域。

```

### g_option_context_parse
```c
gboolean
g_option_context_parse (
  GOptionContext* context,
  gint* argc,
  gchar*** argv,
  GError** error
)
```
### 描述
```
Parses the command line arguments, recognizing options which have been added to context. A side-effect of calling this function is that g_set_prgname() will be called.

If the parsing is successful, any parsed arguments are removed from the array and argc and argv are updated accordingly. A ‘—’ option is stripped from argv unless there are unparsed options before and after it, or some of the options after it start with ‘-‘. In case of an error, argc and argv are left unmodified.

If automatic --help support is enabled (see g_option_context_set_help_enabled()), and the argv array contains one of the recognized help options, this function will produce help output to stdout and call exit (0).

Note that function depends on the [current locale][setlocale] for automatic character set conversion of string and filename arguments.
```

## g_timeout_add_seconds
### 定义
```c
guint
g_timeout_add_seconds (
  guint interval,
  GSourceFunc function,
  gpointer data
)
```
### 描述
```
Sets a function to be called at regular intervals with the default priority, G_PRIORITY_DEFAULT.
设置一个函数，以默认优先级 g_ priority _ default 定期调用该函数。
The function is called repeatedly until it returns G_SOURCE_REMOVE or FALSE, at which point the timeout is automatically destroyed and the function will not be called again.
函数被反复调用，直到返回 g_source_remove 或 FALSE，此时超时将被自动销毁，不会再次调用函数。
This internally creates a main loop source using g_timeout_source_new_seconds() and attaches it to the main loop context using g_source_attach(). You can do these steps manually if you need greater control. Also see g_timeout_add_seconds_full().
这在内部使用 g_timeout_source_new_seconds()创建一个主循环源，并使用 g_source_attach()将其附加到主循环上下文。如果您需要更大的控制，可以手动执行这些步骤。也可以参见 g_timeout_add_seconds_full() 。

It is safe to call this function from any thread.
从任何线程调用此函数都是安全的。

Note that the first call of the timer may not be precise for timeouts of one second. If you need finer precision and have such a timeout, you may want to use g_timeout_add() instead.
请注意，计时器的第一次调用对于一秒钟的超时可能不精确。如果您需要更精确的精度并且有这样的超时，您可能希望使用 g _ timeout _ add ()来代替。

See [memory management of sources][mainloop-memory-management] for details on how to handle the return value and memory management of data.
有关如何处理数据的返回值和内存管理的详细信息，请参阅[源内存管理][ mainloop-memory-management ]。

The interval given is in terms of monotonic time, not wall clock time. See g_get_monotonic_time().
给出的时间间隔是单调时间，而不是壁钟时间。

Available since:	2.14
This function is not directly available to language bindings
此函数不能直接用于语言绑定
The implementation of this function is provided by g_timeout_add_seconds_full() in language bindings
这个函数的实现是由语言绑定中的 g _ timeout _ add _ seconds _ full ()提供的

```


### 参数
- interval	guint
	The time between calls to the function, in seconds.
- function	GSourceFunc
	Function to call.
- data	gpointer
	Data to pass to function.
 	The argument can be NULL.
	
## g_timeout_add
### 定义
```
guint
g_timeout_add (
  guint interval,
  GSourceFunc function,
  gpointer data
)

```

### 描述
```
Sets a function to be called at regular intervals, with the default priority, G_PRIORITY_DEFAULT.
设置定期调用的函数，默认优先级为 g _ priority _ default。
The given function is called repeatedly until it returns G_SOURCE_REMOVE or FALSE, at which point the timeout is automatically destroyed and the function will not be called again. The first call to the function will be at the end of the first interval.
反复调用给定的函数，直到返回 g _ source _ remove 或 FALSE，此时超时将被自动销毁，不再调用该函数。对函数的第一次调用将在第一个间隔结束时进行。

Note that timeout functions may be delayed, due to the processing of other event sources. Thus they should not be relied on for precise timing. After each call to the timeout function, the time of the next timeout is recalculated based on the current time and the given interval (it does not try to ‘catch up’ time lost in delays).
请注意，由于其他事件源的处理，超时函数可能会被延迟。因此，不应依赖它们来确定精确的时间。每次调用超时函数后，都会根据当前时间和给定的间隔重新计算下一次超时的时间(它不会尝试“追赶”延迟中损失的时间)。

See [memory management of sources][mainloop-memory-management] for details on how to handle the return value and memory management of data.
有关如何处理数据的返回值和内存管理的详细信息，请参阅[源内存管理][ mainloop-memory-management ]。

If you want to have a timer in the “seconds” range and do not care about the exact time of the first call of the timer, use the g_timeout_add_seconds() function; this function allows for more optimizations and more efficient system power usage.
如果您希望将计时器设置在“秒”范围内，并且不关心计时器第一次调用的确切时间，请使用 g _ timeout _ add _ seconds ()函数; 这个函数允许更优化和更有效地使用系统电源。

This internally creates a main loop source using g_timeout_source_new() and attaches it to the global GMainContext using g_source_attach(), so the callback will be invoked in whichever thread is running that main context. You can do these steps manually if you need greater control or to use a custom main context.
这将在内部使用 g _ timeout _ source _ new ()创建一个主循环源，并使用 g _ source _ attach ()将其附加到全局 GMainContext，因此无论在哪个运行该主上下文的线程中都将调用该回调。如果需要更大的控制或使用自定义主上下文，可以手动执行这些步骤。

It is safe to call this function from any thread.
从任何线程调用此函数都是安全的。

The interval given is in terms of monotonic time, not wall clock time. See g_get_monotonic_time().
给出的时间间隔是单调时间，而不是壁钟时间。

This function is not directly available to language bindings
The implementation of this function is provided by g_timeout_add_full() in language bindings
此函数不能直接用于语言绑定
这个函数的实现是由语言绑定中的 g _ timeout _ add _ full ()提供的
```

### 参数

- interval	guint
	The time between calls to the function, in milliseconds (1/1000ths of a second)
- function	GSourceFunc
	Function to call.
- data	gpointer
	Data to pass to function.
	The argument can be NULL.


### 返回
```

```

### 使用
```c
//https://blog.csdn.net/arag2009/article/details/17095361
#include<glib.h> 
GMainLoop* loop;
 
gint counter = 10;
gboolean callback(gpointer arg)
{
    g_print(".");
    if(--counter ==0){
        g_print("/n");
        //退出循环
        g_main_loop_quit(loop);
        //注销定时器
        return FALSE;
    }
    //定时器继续运行
    return TRUE;
}
 
int main(int argc, char* argv[])
{
    if(g_thread_supported() == 0)
        g_thread_init(NULL);
    g_print("g_main_loop_new/n");
    loop = g_main_loop_new(NULL, FALSE);
    //增加一个定时器，100毫秒运行一次callback
    g_timeout_add(100,callback,NULL);
    g_print("g_main_loop_run/n");
    g_main_loop_run(loop);
    g_print("g_main_loop_unref/n");
    g_main_loop_unref(loop);
    return 0;
}
```


##  GHashTable
### g_hash_table_lookup
```c
gpointer
g_hash_table_lookup (
  GHashTable* hash_table,
  gconstpointer key
)
```
#### 描述
Looks up a key in a GHashTable. Note that this function cannot distinguish between a key that is not present and one which is present and has the value NULL. If you need this distinction, use g_hash_table_lookup_extended().
在 GHashTable 里查找key。请注意，此函数不能区分不存在的键和存在的值为 NULL 的键。如果需要这种区分，可以使用 g _ hash _ table _ lookup _ extended () 。

## g_strsplit 
```
gchar**
g_strsplit (
  const gchar* string,
  const gchar* delimiter,
  gint max_tokens
)
```
### 描述
Splits a string into a maximum of max_tokens pieces, using the given delimiter. If max_tokens is reached, the remainder of string is appended to the last token.
使用给定的分隔符将字符串拆分为最多 max_token 个部分。如果达到 max_token，则字符串的其余部分将追加到最后一个令牌。
### 参数
- max_tokens	gint	
The maximum number of pieces to split string into. If this is less than 1, the string is split completely.
将字符串拆分为的最大块数。如果小于1，则字符串将被完全拆分

### 返回
Returns:	An array of utf8
A newly-allocated NULL-terminated array of strings. Use g_strfreev() to free it.
 	The array is NULL-terminated.




## g_strjoinv
```
gchar*
g_strjoinv (
  const gchar* separator,
  gchar** str_array
)
```
### 描述
Joins a number of strings together to form one long string, with the optional separator inserted between each of them. The returned string should be freed with g_free().
将多个字符串连接在一起，形成一个长字符串，并在每个字符串之间插入可选的分隔符。返回的字符串应该用 g _ free ()释放。
If str_array has no items, the return value will be an empty string. If str_array contains a single item, separator will not appear in the resulting string.
如果 str_array 没有条目，返回值将是一个空字符串。如果 str_array 包含单个项，则分隔符不会出现在结果字符串中。
### 参数
separator	const gchar*	
A string to insert between each of the strings, or NULL.
 	The argument can be NULL.

str_array	gchar**	
A NULL-terminated array of strings to join.
 	The data is owned by the caller of the function.
 	The value is a NUL terminated UTF-8 string.
### 返回
gchar*
A newly-allocated string containing all of the strings joined together, with separator between them.


## g_strchomp
```c
gchar*
g_strchomp (
  gchar* string
)
```
### 描述
Removes trailing whitespace from a string.
从字符串中移除尾随空格。
This function doesn’t allocate or reallocate any memory; it modifies string in place. Therefore, it cannot be used on statically allocated strings.
这个函数不分配或重新分配任何内存，而是适当地修改字符串。因此，它不能用于静态分配的字符串。
The pointer to string is returned to allow the nesting of functions.
返回字符串指针以允许函数嵌套。
Also see g_strchug() and g_strstrip().
还可以参见 g _ strchug ()和 g _ strstrip ()。

## g_strfreev
### 描述
Frees a NULL-terminated array of strings, as well as each string it contains.
If str_array is NULL, this function simply returns.
释放以 null 结尾的字符串数组及其包含的每个字符串。
如果 str_array 为 NULL，则此函数简单地返回。
### 声明
```c
void
g_strfreev (
  gchar** str_array
)
```
### 参数
str_array	gchar**
A NULL-terminated array of strings to free.
 	The argument can be NULL.