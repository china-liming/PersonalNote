[toc]
## 类型定义 （typedef) 
摘自 《C程序设计语言》6.7节

C语言提供了一个称为typedef的功能，它用来建立新的数据类型名，例如，声明

```c
typedef int Length;
```

将Length定义为与int具有同等意义的名字。类型Length可用于类型声明、类型转换等，它和类型int完全相同，例如:

```c
Length len, maxlen;
Length *lengths [ ] ;
```

类似地，声明

```c
typedef char* string;
```

将string 定义为与 char *或字符指针同义，此后，便可以在类型声明和类型转换中使用string，例如:

```c
String p, lineptr[MAXLINES], alloc ( int) ;
int strcmp ( string , string );
p = (string) malloc (100) ;
```

==注意，typedef中声明的类型在变量名的位置出现，而不是紧接在关键字typedef之后。typedef在语法上类似于存储类extern、static 等==。我们在这里以大写字母作为typedef定义的类型名的首字母，以示区别。

这里举一个更复杂的例子:用typedef定义本章前面介绍的树节点。如下所示:

```c
typedefstruct tnode *Treeptr;
typedefstruct tnode {/ * the tree node: */
char *word;/* points to the text */
int count ;/*number of occurrences */
struct tnode *left;/*left child * /
struct tnode *right;/*right child */
} Treenode;
```

上述类型定义创建了两个新类型关键字:Treenode (一个结构）和Treeptr (一个指向该结构的指针)。这样，函数talloc可相应地修改为:

```c
Treeptr talloc (void)
{
	return (Treeptr) malloc (sizeof ( Treenode ) ) ;
}
```

这里必须强调的是，从任何意义上讲，typedef声明并没有创建一个新类型，它只是为某个已存在的类型增加了一个新的名称而已。typedef声明也没有增加任何新的语义:通过这种方式声明的变量与通过普通声明方式声明的变量具有完全相同的属性。实际上，typedef类似于#define语句，但由于typedef是由编译器解释的，因此它的文本替换功能要超过预处理器的能力。例如:

```c
typedef int(*PFI) (char * , char * );
```

该语句定义了类型PFI是“一个指向函数的指针，该函数具有两个 char *类型的参数，返回值类型为int”，它可用于某些上下文中，例如，可以用在第5章的排序程序中，如下所示:
PFI strcmp,numcmp;
除了表达方式更简洁之外，使用typedef还有另外两个重要原因。首先，它可以使程序参数化，以提高程序的可移植性。如果typedef声明的数据类型同机器有关，那么，当程序移植到其它机器上时，只需改变typedef类型定义就可以了。一个经常用到的情况是，对于各种不同大小的整型值来说，都使用通过typedef定义的类型名，然后，分别为各个不同的宿主机选择一组合适的 short、int 和 long类型大小即可。标准库中有一些例子，例如size_t和ptrdiff_t等。
&emsp;&emsp;typedef 的第二个作用是为程序提供更好的说明性——Treeptr类型显然比一个声明为指向复杂结构的指针更容易让人理解。


## 符号常量 #define
摘自 《C程序设计语言》1.4节
&emsp;&emsp;在结束讨论温度转换程序前，我们再来看一下符号常量。在程序中使用300、20等类似的“幻数”并不是一个好习惯，它们几乎无法向以后阅读该程序的人提供什么信息，而且使程序的修改变得更加困难。处理这种幻数的一种方法是赋予它们有意义的名字。#define指令可以把符号名（或称为符号常量）定义为一个特定的字符串:

```c
#define  名字  替换文本
```

在该定义之后，程序中出现的所有在#define 中定义的==名字（既没有用引号引起来，也不是其它名字的一部分）==都将用相应的==替换文本==替换。其中，名字与普通变量名的形式相同:它们都是以字母打头的字母和数字序列;替换文本可以是任何字符序列，而不仅限于数字。

```c
#include <stdio.h>
#define LOWER 0   /*lower limit of table */
#define UPPER 300  /*upper limit */
#define STEP 20   /*step size * /
/*print Fahrenheit-Celsius table */
main()
{
	int fahr;
	for (fahr = LOWER; fahr <= UPPER; fahr = fahr + STEP)
		printf ("%3d %6.1f\n", fahr, (5.0/9.0)* (fahr-32 ));
}
```

其中，LOWER、UPPER与 STEP都是符号常量，而非变量，因此不需要出现在声明中。符号常量名通常用大写字母拼写，这样可以很容易与用小写字母拼写的变量名相区别。注意，#define指令行的末尾没有分号。
有关宏的更多解释参考：
https://blog.csdn.net/Wmll1234567/article/details/109681262
### 宏函数
优点：
预编译的时候展开，不需要每次运行时载入，这种情况效率比函数高
缺点：

预编译后产生的文件比函数调用要大，
宏表达式中不能出现递归定义，这点区别于函数，因为宏只做简单的文本替换，且只替换一次，如果出现递归定义，就会无法被完全替换，导致后续编译时原宏名被当作函数；

#### 可变参数宏（__VA_ARGS__、##__VA_ARGS__）
`#define debug(...) printf(__VA_ARGS__)`
缺省号（...）代表一个可以变化的参数表。使用保留名 __VA_ARGS__ 把参数传递给宏。当宏的调用展开时，实际的参数就传递给 printf()了
例如：调用Debug("Y = %d\n", y); 而处理器会把宏的调用替换成:printf("Y = %d\n", y);

 因为debug()是一个可变参数宏，你能在每一次调用中传递不同数目的参数:

 ##__VA_ARGS__ 作用与__VA_ARGS__类似

 ##__VA_ARGS__ 宏前面加上##的作用在于，当可变参数的个数为0时，这里的##起到把前面多余的","去掉


## union
&emsp;&emsp;联合（union）的声明和结构与结构体类似，但是本质不同。联合的所有成员引用的是内存中的相同位置。当你想在不同时刻把不同的东西存储于同一位置时，就可以使用联合。

```c
#include<stdio.h>  
union var{  
        long int l;  
        int i;  
};  
main(){  
        union var v;  
        v.l = 5;  
        printf("v.l is %d\n",v.i);  
        v.i = 6;  
        printf("now v.l is %ld! the address is %p\n",v.l,&v.l);  
        printf("now v.i is %d! the address is %p\n",v.i,&v.i);  
}  
//结果：  
//v.l is 5  
//now v.l is 6! the address is 0xbfad1e2c  
//now v.i is 6! the address is 0xbfad1e2c  
```

## enum
枚举常量是另外一种类型的常量。枚举是一个常量整型值的列表，例如:
```c
enum boolean {NO,YES };
```
在没有显式说明的情况下，enum类型中第一个枚举名的值为0，第二个为1，依此类推。如果只指定了部分枚举名的值，那么未指定值的枚举名的值将依着最后一个指定值向后递增，参看下面两个例子中的第二个例子:

```c
enum escapes { 
	BELL = 'la '，BACKSPACE = '\b ',
	TAB = '\t ' ,NEWL工NE = '\n ' , 
	VTAB = '\v',RETURN = 'r' 
	};
enum months { 
JAN = 1，FEB,
MAR,APR,MAY,
JUN,JUL,AUG,
SEP,OCT,Nov,DEC  
} ;
/*FEB的值为2，MAR的值为3，依此类推*/
```

不同枚举中的名字必须互不相同。同一枚举中不同的名字可以具有相同的值。
枚举为建立常量值与名字之间的关联提供了一种便利的方式。相对于#define语句来说，它的优势在于常量值可以自动生成。尽管可以声明enum类型的变量，但编译器不检查这种类型的变量中存储的值是否为该枚举的有效值。不过，枚举变量提供这种检查，因此枚举比#define更具优势。此外，调试程序可以以符号形式打印出枚举变量的值。
### 实例
```c
#include <stdio.h>
 
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
 
int main()
{
    enum DAY day;
    day = WED;
    printf("%d",day);
    return 0;
}// 3
```

## likely() and unlikely()
参考：https://my.oschina.net/moooofly/blog/175019
https://www.cnblogs.com/ydqblogs/p/13832051.html
__builtin_expect是gcc（version >= 2.96）引入的，作用是允许程序员将最有可能执行的分支告诉编译器，让编译器告诉CPU提前加载该分支下的指令。
写法为：__builtin_expect(EXP, N)，表示的意思是：EXP == N的概率很大
likely 和 unlikely 是 gcc 扩展的跟处理器相关的宏： 

```c
#define  likely(x)        __builtin_expect(!!(x), 1) 
#define  unlikely(x)      __builtin_expect(!!(x), 0)
__builtin_expect((x),1) 表示 x 的值为真的可能性更大；
__builtin_expect((x),0) 表示 x 的值为假的可能性更大。
```
&emsp;&emsp;现在处理器都是流水线的，有些里面有多个逻辑运算单元，系统可以提前取多条指令进行并行处理，但遇到跳转时，则需要重新取指令，这相对于不用重新去指令就降低了速度。 
&emsp;&emsp;所以就引入了 likely 和 unlikely ，目的是增加条件分支预测的准确性，cpu 会提前装载后面的指令，遇到条件转移指令时会提前预测并装载某个分 支的指令。unlikely 表示你可以确认该条件是极少发生的，相反 likely 表示该条件多数情况下会发生。编译器会产生相应的代码来优化 cpu 执行效率。
## extern 关键字用法
&emsp;&emsp;如果全局变量不在文件的开头定义，其有效的范围为其定义处到文件结束。如果想在定义点之前的函数引用该全局变量，则要在引用之前用关键字extern对该变量作“外部变量声明”，即表示该变量是一个已经定义的外部变量。这样就可以从声明处其，使用该外部变量。


```c
#include <stdio.h>
int max（int a,int b);
 
int main(void){
    int c;
    extern int x_a;
    extern int x_b;
    c = max(x_a,x_b);
    printf ("max %d\n",c);
    return 0;
}
 
int x_a = 1;
int x_b = 2;
 
int max(int a,int b){
   return (a>b? a:b);
} 
```
&emsp;&emsp;在上述代码中，因为全局变量x_a,x_b是main函数之后声明的，所以其作用范围不在main函数中。如果要在main函数中调用它们，便需要用extern对变量x_a,x_b进行外部变量声明。总的来说，如果在变量定义之前要使用该变量，就需要在使用之前用extern声明变量。
### 多个源文件之间的引用
如果一个工程由多个源文件组成，在一个源文件中想引用另一个源文件中已定义的外部变量，也需要在引用变量的源文件中用extern关键字声明变量，如下代码所示：

```c
/*************第一个源文件************/
#include <stdio.h>
extern int a;
extern int b;
int max(){
    return (a>b? a:b);
}
 
/*************第二个源文件************/
#include <stdio.h>
int a = 1;
int b = 2;
int max();
int main(void){
    int c;
    c = max();
    printf("max %d\n",c);
    return 0;
}
```

## static
参考：https://baijiahao.baidu.com/s?id=1684513244156239757&wfr=spider&for=pc
static三大作用：
### 1.修饰局部变量–静态局部变量
把局部变量改变为静态变量后是改变了它的存储方式即改变了它的生命周期。但不改变他的作用域。
static局部变量只被初始化一次，下一次依据上一次结果值；

未经static修饰时每次i被调用的值都为0；生命周期随着每次函数的调用而更新。

经static修饰后的局部变量，不随着函数的每次调用而更新，而是沿用上一次的值，被初始化一次。生命周期随着代码运行结束而结束。

### 2.修饰全局变量–静态全局变量
全局变量之前以static 修饰就构成了静态的全局变量。
当一个源程序由多个源文件组成时，非静态的全局变量在各个源文件中都是有效的。而静态全局变量则限制了其作用域， 即只在定义该变量的源文件内有效， 在同一源程序的其它源文件中不能使用它。由于静态全局变量的作用域局限于一个源文件内，只能为该源文件内的函数公用，因此可以避免在其它源文件中引起错误。
static全局变量只初使化一次，防止在其他文件单元中被引用。

未经static修饰的全局变量：
未经修饰时可跨文件使用，输出结果为10；

static修饰的全局变量：
不可跨文件使用全局变量，输出的是系统给的默认值0而非10。

### 3.修饰函数–静态函数
未经static修饰的函数：
可跨文件调用函数输出结果3。
static修饰的函数：
经static修饰后不可跨文件调用函数，并报错。

总结
- 静态函数会被自动分配在一个一直使用的存储区，直到程序结束才从内存消失，避免调用函数时压栈出栈，速度快很多
- 其他文件可以定义相同名字的函数，不会发生冲突
- 静态函数不能被其它文件调用，作用于仅限于本文件
- 当我们同时编译多个文件时，所有未加static前缀的全局变量和函数都具有全局可见性。如果加了static，就会对其它源文件隐藏。

## struct 学习
```c
#include <stdio.h>
struct{                //定义了这种结构体的一个变量--fieldsByDb1。
    int size; 
    int count; 
} fieldsByDb1;

struct A{              //定义了一种结构体，叫A，并定义这种结构体的一个变量--fieldsByDb2，若再定义新变量则应该struct A a;
    int size; 
    int count; 
} fieldsByDb2;

typedef struct B{      //定义了一种结构体，叫B，并定义这种结构体的一个变量--fieldsByDb3，若再定义新变量则应该struct A a;
    int size;          //并将struct B{...}取了一个别名叫fieldsByDb3，可以用fieldsByDb3 b;声明此类型的变量。
    int count; 
} fieldsByDb3;

int main()
{
    struct A a;
    fieldsByDb3 b;
    a.count=1;
    b.count=1;
    fieldsByDb1.count=1;
    fieldsByDb2.count=1;
    printf("%d\n",fieldsByDb.count);
}
```

## C 位域（结构体中带冒号）
有些信息在存储时，并不需要占用一个完整的字节，  
而只需占几个或一个二进制位。例如在存放一个开关量时，只有0和1   两种状态，
用一位二进位即可。为了节省存储空间，并使处理简便，
C语言又提供了一种数据结构，称为“位域”或“位段”。所谓“位域”是把一个字节中的二进位划分为几个不同的区域，
并说明每个区域的位数。每个域有一个域名，允许在程序中按域名进行操作。  
这样就可以把几个不同的对象用一个字节的二进制位域来表示。

```c
#include<stdio.h>
struct packed_struct {
  unsigned int f1:1;
  unsigned int f2:1;
  unsigned int f3:1;
  unsigned int f4:1;
  unsigned int type:4;
  unsigned int my_int:9;
  unsigned int my_int2:9;
  unsigned int my_int3:9;

} pack;
 struct   bs   
  {   
    int   a:8;   
    int   b:2;   
    int   c:6;   
  }data;     
  struct
{
  unsigned int widthValidated : 1;
  unsigned int heightValidated : 1;
} status;
int main()
{
    printf("%d\n",sizeof(pack));
    printf("%d\n",sizeof(data));
    printf("%d\n",sizeof(status));
    printf("%d\n",sizeof(unsigned int));

}
```
运行结果：
`
8
4
4
4
`
## 指针相关
### 指针的指针的应用
原文链接：https://blog.csdn.net/oqqHuTu12345678/article/details/60962223
（1）在子函数中修改主函数传过来的指针的指向

比如主函数申明一个指针变量，且不为其分配指向空间（只是指向NULL），然后取该指针变量的地址传参给子函数；
在子函数里根据需要申请一片空间；
对传参解引用后得到原来的指针变量，且把该指针变量指向刚才申请的空间（修改了原来指针的指向，这是本质）；
从而实现不确定返回值个数的编程方法（这是应用）；
例子1（本质）：
```c
#include<stdio.h>
int find(char *s, char src, char **rt)//从s中查询出src字符所在的位置并在rt中返回。
{
	int i = 0;
	while (*(s + i))
	{
		if (*(s + i) == src)//只会出现一次满足条件，此时将此地址给*rt，即p
		{
			* rt = s + i;//这里修改了p的指向
		}
		i++;
	}
	return 0;
 }
 
 
int main(void)
{
   char a[10] = "zhuyujia";
   char *p=NULL;
   find(a, 'y', &p);//改变p的指向，在子函数中实现
   printf("%s", p);
   getchar(); getchar();
   return 0;
}
 
/*
//补充：
打印指针的时候，会把指针所指向的内容以及至字符串末位的内容都打印出来
#include<stdio.h>
int main(void)
{
	char a[10] = "abcdefgff";
	char* p = &a[6];
	printf("%s", p);//会打印gff
	getchar(); getchar();
	return 0;
}
*/
```
例子2：
```c
#include <stdlib.h>
#include <time.h>
#include<stdio.h>
/*当然有必须使用二级指针才能解决的情况,如,某个函数的功能是
返回某个问题的计算结果,但是结果数据是不确定个数的值,所以
在调用此函数时不知道事先应分配多少空间来保存返回的数据,此时
的处理办法就是传递一个没有分配空间的指针的指针(地址)进去,
让函数自己根据计算的结果分配足够的空间来保存结果,并返回,
调用者使用了结果后,由调用者负责内存的释放,即,大家可能听说
过的"谁使用(调用)谁释放"之类的话,如下面的代码:*/
 
//返回不定结果个数的计算函数
//参数int **pResult  保存返回数据的指针的指针
//参数int &count     保存返回的结果个数
void Compute2(int **pResult, int &count)
{
	//使用随机数来模拟计算结果数的个数
	srand(time(NULL));
	count = rand() % 10;//控制个数在10个以内
 
	*pResult = new int[count];//*pResult相当于主函数传来的pResult指针，
							  //这里就修改了主函数中的pResult指向，因为还是指针，因此可以指向新开辟的空间
	for (int i = 0; i < count; i++)
	{
		(*pResult)[i] = rand();//给结果随即赋值
	}
}
 
//返回不定结果个数的计算函数(此函数不能返回数据)
//参数int *pResult   为保存返回数据的指针
//参数int &count     为保存返回的结果个数
void Compute1(int *pResult, int &count)
{
	//使用随机数来模拟计算结果数的个数
	srand(time(NULL));
	count = rand() % 10;//控制个数在10个以内
 
	pResult = new int[count];
	for (int i = 0; i < count; i++)
	{
		pResult[i] = rand();//给结果随即赋值
	}
}
 
int main(void)
{
	int *pResult = NULL;//待获取结果的指针，这里没有分配空间大小，因为不知道返回结果的个数
						//具体返回的个数在在子函数中确定，此时指针pResult指向也改变了
						//这就间接的说明“在子函数中修改主函数传来的指针”的意图
						//具体的应用就在于返回个数不确定的场景，这是后面编程的一个体会点
	int count = 0;//返回结果的个数
 
				  /*
				  Compute1(pResult,count);//pResult为指针,第二个参数使用引用传递,
				  //使用这个函数时,在函数内部分配的内存的指针并没有返回到主函数中
				  for ( int i = 0 ; i < count ; i++ )
				  printf("第 %d 个结果为 : %d\n",pResult[i]);//执行了Compute1()函数后,pResult的值还是为NULL
				  delete [] pResult;
				  pResult = NULL;
				  */
 
	Compute2(&pResult, count); //&pResult为指针的地址(即指针的指针),第二个参数使用引用传递
	for (int i = 0; i < count; i++)
		printf("第 %d 个结果为 : %d\n", i, pResult[i]);
 
	delete[] pResult;
	pResult = NULL;
 
	getchar();
	return 0;
}
```


### 函数指针
在hash.h原文代码中有这个，搞不懂：
```c
/* Convert a key to a u32 hash value */
typedef uint32_t (* HASH_KEY_FUNC)(const void *key);

/* Given a key does it match the element */
typedef int (* HASH_CMP_FUNC)(const void *key, const void *element);
```
学习过程：
&emsp;&emsp;网上说是函数指针
什么是函数指针：
&emsp;&emsp;如果在程序中定义了一个函数，那么在编译时系统就会为这个函数代码分配一段存储空间，这段存储空间的首地址称为这个函数的地址。而且函数名表示的就是这个地址。既然是地址我们就可以定义一个指针变量来存放，这个指针变量就叫作函数指针变量，简称函数指针。
那么这个指针变量怎么定义呢？虽然同样是指向一个地址，但指向函数的指针变量同我们之前讲的指向变量的指针变量的定义方式是不同的。例如：
```c
int(*p)(int, int);
```
&emsp;&emsp;这个语句就定义了一个指向函数的指针变量 p。首先它是一个指针变量，所以要有一个“ \* ”，即(\* p)其次前面的 int 表示这个指针变量可以指向返回值类型为 int 型的函数；后面括号中的两个 int 表示这个指针变量可以指向有两个参数且都是 int 型的函数。所以合起来这个语句的意思就是：定义了一个指针变量 p，该指针变量可以指向返回值类型为 int 型，且有两个整型参数的函数。p 的类型为 int(\*)(int，int)。

所以函数指针的定义方式为：

```c
函数返回值类型 (* 指针变量名) (函数参数列表);
```

&emsp;&emsp;“函数返回值类型”表示该指针变量可以指向具有什么返回值类型的函数；“函数参数列表”表示该指针变量可以指向具有什么参数列表的函数。这个参数列表中只需要写函数的参数类型即可。

#### typedef 函数指针 用法
原文链接：https://blog.csdn.net/qll125596718/article/details/6891881
使用typedef更直观更方便：typedef  返回类型(\*新类型)(参数表)
```c
typedef char (*PTRFUN)(int); 
PTRFUN pFun; 
char glFun(int a){ return;} 
void main() 
{ 
    pFun = glFun; 
    (*pFun)(2); 
} 
```
 
 typedef的功能是定义新的类型。第一句就是定义了一种PTRFUN的类型，并定义这种类型为指向某种函数的指针，这种函数以一个int为参数并返回char类型。后面就可以像使用int,char一样使用PTRFUN了。
 第二行的代码便使用这个新类型定义了变量pFun，此时就可以像使用形式1一样使用这个变量了。

## C++ 判断标准版本和编译器
参考：[C++ 判断标准版本和编译器](https://blog.csdn.net/m0_47534090/article/details/108591764)
**clang** 和 **gcc** 判断`__cplusplus`

版本     | `__cplusplus`的值
-------- | -----
C++ 17  | 201703L
C++ 14 | 201402L
C++ 11  | 201103L
C++ 03 以下|199711L

**msvc** 判断`_MSVC_LANG`

| Column 1 | `_MSVC_LANG`的值 |
|:--------:| -------------:|
| C++ 17 | 201703L |
| C++ 14 | 201703L |
| C++ 11 | 201103L |
| C++ 03 以下 | 199711L |
### 判断使用的编译器

编译器版本     | 宏
-------- | -----
msvc  | _MSC_ VER
clang | \__clang\__
gcc   | \__GNUC\__

实现
```c
// cpp.hpp
#ifndef CPP_HPP
#define CPP_HPP

#if defined(__clang__) || defined(__GNUC__)

	#define CPP_STANDARD __cplusplus
	
#elif defined(_MSC_VER)

	#define CPP_STANDARD _MSVC_LANG
	
#endif
 
#if CPP_STANDARD >= 199711L
	#define HAS_CPP_03 1
#endif
#if CPP_STANDARD >= 201103L
	#define HAS_CPP_11 1
#endif
#if CPP_STANDARD >= 201402L
	#define HAS_CPP_14 1
#endif
#if CPP_STANDARD >= 201703L
	#define HAS_CPP_17 1
#endif

#endif
```

## 关于编译器
参考：[详解三大编译器：gcc、llvm 和 clang ](https://zhuanlan.zhihu.com/p/357803433)
传统的编译器通常分为三个部分，前端（frontEnd），优化器（Optimizer）和后端(backEnd)。在编译过程中，前端主要负责词法和语法分析，将源代码转化为抽象语法树；优化器则是在前端的基础上，对得到的中间代码进行优化，使代码更加高效；后端则是将已经优化的中间代码转化为针对各自平台的机器代码。
### gcc (Linux)
GCC（GNU Compiler Collection，GNU 编译器套装），是一套由 GNU 开发的编程语言编译器。GCC 原名为 GNU C 语言编译器，因为它原本只能处理 C语言。GCC 快速演进，变得可处理 C++、Fortran、Pascal、Objective-C、Java 以及 Ada 等他语言。

### MSVC （Windows）
MinGW(Minimalist GNUfor Windows)，它是一个可自由使用和自由发布的Windows特定头文件和使用GNU工具集导入库的集合，允许你在Windows平台生成本地的Windows程序而不需要第三方C运行时(C Runtime)库。运行时库：支持程序运行的基本函数的集合，一般是静态库lib或动态库dll。

而MSVC，就是上文所说的第三方C运行时库：由微软开发的VC运行时库，被Visual Studio IDE所集成。所以我们使用VS时会附带MSVC编译器。所以可以看到啦，MinGW和MSVC都是Windows C/C++语言编译支持，配置环境时遇到两者择其一即可。

原文链接：https://blog.csdn.net/qq_51491920/article/details/123601566

### LLVM的clang (Apple)
LLVM是构架编译器(compiler)的框架系统，以C++编写而成，用于优化以任意程序语言编写的程序的编译时间(compile-time)、链接时间(link-time)、运行时间(run-time)以及空闲时间(idle-time)，对开发者保持开放，并兼容已有脚本。
LLVM计划启动于2000年，最初由美国UIUC大学的Chris Lattner博士主持开展。2006年Chris Lattner加盟Apple Inc.并致力于LLVM在Apple开发体系中的应用。Apple也是LLVM计划的主要资助者。目前LLVM已经被苹果IOS开发工具、Xilinx Vivado、Facebook、Google等各大公司采用。
Clang 是 LLVM 的前端，可以用来编译 C，C++，ObjectiveC 等语言。Clang 则是以 LLVM 为后端的一款高效易用，并且与IDE 结合很好的编译前端。
Clang 只支持C，C++ 和 Objective-C 三种语言。2007 年开始开发，C 编译器最早完成，而由于 Objective-C 只是 C 语言的一个简单扩展，相对简单，很多情况下甚至可以等价地改写为 C 语言对 Objective-C 运行库的函数调用，因此在 2009 年时，已经完全可以用于生产环境。C++ 在后来也得到了支持。
### 三者区别
-   msvc通常用于编译Windows应用，而gcc/clang则可以用来编译Windows/Linux/MacOS等所有平台的应用。
-   很多基于POSIX/GNU体系开发的应用并未在msvc上编译调试过。反之，很多纯windows应用并未在gcc/clang上编译调试过。所以两者的代码大概率是不可互换的，除非你写的恰好是跨平台代码（比如Qt）
-   msvc在字符编码的处理方面略有不同，它默认Win当前语言所属的MBCS字符集，如果使用utf-8通常需要加bom，当然即便加了bom，某些代码编译起来也会需要特别的处理。而gcc/clang是默认要用无bom的utf-8字符编码。
-   msvc/gcc/clang在具体的优化策略上会有不同，所以同样的程序代码会产生不同方向的优化，生成不同的机器代码。
-   如果去网上找开源代码，多数情况下不会同时支持所有编译器，要么是支持 gcc/clang 的，要么是支持 msvc 的，改换平台移植往往需要额外的工作量。如果你需要依赖的第三方代码更多是属于 gcc/clang 体系的，那么就建议选择gcc/clang，反之如果你需要依赖的第三方代码更多基于 msvc ，那就建议选择 msvc。
-  至于 gcc 跟 clang，两者代码标准大体上是兼容的，主要区别可能只是开发团队不同，设计理念不同。
链接：https://www.zhihu.com/question/445921363/answer/1763195868  

## 移位
### 左移<<
左移运算符（<<）将其左侧运算对象每一位的值向左移动其右侧运算对象指定的位数。
左侧运算对象移出左末端位的值丢失，==用0填充空出的位置==.

### 右移>>
右移运算符，将其左侧运算对象每一位的值向右移动其右侧运算对象指定的位数。
左侧运算对象移出右末端位的值丢失。
对于==无符号==类型，==用零填充空出的位置==；
对于==有符号==类型，其结果==取决于机器。==
空出的位置==可用0==填充，或者==用符号位（即最左端的位）的副本==填充：
（10001010）>> 2    //表达式，有符号值
（00100010）           //在某些系统中的结果值
（10001010）>> 2    //表达式，有符号值
（11100010）           //在另一些系统上的结果值
下面是无符号值的例子：
（10001010）>> 2    //表达式，无符号值
（00100010）           //所有系统都得到该结果值
每个位向右移动两个位置，空出的位用0填充。

- ==左移(<<)N位的本质是乘以2的N次方==
- ==右移(>>)N位的本质是除以2的N次方==
```c
#include<stdio.h>
int main()
{
    int a,b;
    a=13<<2;
    b=25>>3;
    printf("a=%d,b=%d\n",a,b);
    return 0;
}
```
结果：
`a=52,b=3`

## 位运算符
位运算符有：按位与、按位或、按位非、按位异或。

### 与
&  
只有两个位同时为1,才能得到1。

&0xff的作用：
简述：
& 0 清零(置0)
& 1 保留原值
- 1. A  &  0xff 代表取A的低八位，前面的都得0。
- 2. java专属，由于java没有unsigned类型，所以为了适应与其他语言二进制通讯时各种数据的一致性，需要做一些处理。
最直观的例子：int a = -127 & 0xFF ; // 等同于 unsigned int c = 129; (这里的-127与129是字节，只为了直观而写的具体数字)
这里要严格说明一点：再32位机器上，0xff实际上是 0x00000000 00000000 00000000 11111111，
而-127是 11111111 11111111 11111111 10000001 (补码形式), 那么-127 & 0xff的结果自然是
00000000 00000000 00000000 10000001 即 129.
==简而言之，该作用主要是为了将 有符号数转换为无符号数。==



### 或
|     有一个为1则为1

作用：
| 0 保留原值
| 1 置1

### 非(取反)
~     按位取反
### 异或
^    不同为1（如1和0，0和1），相同为0（0和0,1和1）

## main 函数
连接：[C语言main函数参数](http://c.biancheng.net/cpp/html/86.html)
`main(argc,**argv)`
- argc  表示参数的个数
- argv  argv[0]为自身运行目录路径和程序名，argv[1]指向第一个参数、argv[2]指向第二个参数……

例如有命令行为：  
    C:\>E24  BASIC  foxpro  FORTRAN  
由于文件名E24本身也算一个参数，所以共有4个参数，因此argc取得的值为4。argv参数是字符串指针数组，其各元素值为命令行中各字符串(参数均按字符串处理)的首地址。 指针数组的长度即为参数个数。数组元素初值由系统自动赋予。其表示如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2c3606ee4ff4c4cb970d12ce473f871.png)

`int main(int argc , char* argv[],char* envp[]);`
- char* envp[] 也是一个字符串数组，主要是保存这用户环境中的变量字符串，以NULL结束。envp[]的每一个元素都包含ENVVAR=value形式的字符串，其中ENVVAR为环境变量，value为其对应的值。envp一旦传入，它就只是单纯的字符串数组而已，不会随着程序动态设置发生改变。可以使用putenv函数实时修改环境变量，也能使用getenv实时查看环境变量，但是envp本身不会发生改变；平时使用到的比较少

原文链接：https://blog.csdn.net/z_ryan/article/details/80979432

## pthread 相关
文档：http://sourceware.org/pthreads-win32/manual/index.html
### 数据类型:

类型     | 说明
-------- | -----
pthread_t：|线程句柄 　　
pthread_attr_t：|线程属性

### 基本线程操作：

函数     | 说明
-------- | -----
pthread_create()	|创建线程开始运行相关线程函数，运行结束则线程退出
pthread_eixt()	|因为exit()是用来结束进程的，所以则需要使用特定结束线程的函数
pthread_join()  |挂起当前线程，用于阻塞式地等待线程结束，如果线程已结束则立即返回，0=成功
pthread_cancel()|发送终止信号给thread线程，成功返回0，但是成功并不意味着thread会终止
pthread_testcancel()|在不包含取消点，但是又需要取消点的地方创建一个取消点，以便在一个没有包含取消点的执行代码线程中响应取消请求.
pthread_setcancelstate()|设置本线程对Cancel信号的反应
pthread_setcanceltype()|设置取消状态 继续运行至下一个取消点再退出或者是立即执行取消动作
pthread_setcancel()|设置取消状态

### 互斥与同步机制基本函数

函数     | 说明
-------- | -----
pthread_mutex_init()	|互斥锁的初始化
pthread_mutex_lock()	|锁定互斥锁，如果尝试锁定已经被上锁的互斥锁则阻塞至可用为止
pthread_mutex_trylock()	|非阻塞的锁定互斥锁
pthread_mutex_unlock()	|释放互斥锁
pthread_mutex_destory()	|互斥锁销毁函数

### 信号量线程控制(默认无名信号量)

函数     | 说明
-------- | -----
sem_init(sem)	|初始化一个定位在sem的匿名信号量
sem_wait()	|把信号量减1操作，如果信号量的当前值为0则进入阻塞，为原子操作
sem_trywait()	|如果信号量的当前值为0则返回错误而不是阻塞调用(errno=EAGAIN),其实是sem_wait()的非阻塞版本
sem_post()	|给信号量的值加1，它是一个“原子操作”，即同时对同一个信号量做加1,操作的两个线程是不会冲突的
sem_getvalue(sval)	|把sem指向的信号量当前值放置在sval指向的整数上
sem_destory(sem)	|销毁由sem指向的匿名信号量

### 线程属性配置相关函数

函数     | 说明
-------- | -----
pthread_attr_init()	|初始化配置一个线程对象的属性,需要用pthread_attr_destroy函数去除已有属性
pthread_attr_setscope()	|设置线程属性
pthread_attr_setschedparam()	|设置线程优先级
pthread_attr_getschedparam()	|获取线程优先级

### pthread mutex 基本用法
参考：https://feng-qi.github.io/2017/05/08/pthread-mutex-basic-usage/
使用mutex的基本步骤就是：

定义muutex -> 初始化mutex -> 使用mutex(lock, unlock, trylock) -> 销毁mutex。
函数名也已经把它自己的功能描述的非常清楚了。只是有一些细节需要注意：

pthread_mutex_t的初始化有两种方法，一种是使用函数pthread_mutex_init，使用结
束需要调用函数pthread_mutex_destroy进行销毁，调用时mutex必须未上锁。

第二种方法是使用PTHREAD_MUTEX_INITIALIZER。根据[1]中的描述，似乎对使用这种方法
初始化的mutex调用pthread_mutex_destroy()会产生错误，对未上锁的mutex调用
pthread_mutex_unlock也会产生错误。

例子：
```c
#include <stdio.h>
#include <pthread.h>

/* pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; */
pthread_mutex_t mutex;

int count;

void * thread_run(void *arg)
{
    int i;
    pthread_mutex_lock(&mutex);
    for (i = 0; i < 3; i++) {
        printf("[%#lx]value of count: %d\n", pthread_self(), ++count);
    }
    pthread_mutex_unlock(&mutex);
    return 0;
}

int main(int argc, char *argv[])
{
    pthread_t thread1, thread2;
    pthread_mutex_init(&mutex, 0);
    pthread_create(&thread1, NULL, thread_run, 0);
    pthread_create(&thread2, NULL, thread_run, 0);
    pthread_join(thread1, 0);
    pthread_join(thread2, 0);
    pthread_mutex_destroy(&mutex);
    return 0;
}
```

如果使用了第 4 行的初始化方法，可以删除 23 和 28 行。

### pthread_self
#### 声明 ：
```c
#include <pthread.h>
pthread_t pthread_self(void);
```
#### 描述：
获取本进程自身的 ID。进程 ID 类型是 pthread_t ，这个类型一般为long long 型，8个字节。


## char 和 unsigned char 的区别
原文链接：https://blog.csdn.net/guotianqing/article/details/77341657
相同点：在内存中都是一个字节，8位（2^8=256），都能表示256个数字
不同点：char的最高位为符号位，因此char能表示的数据范围是-128~127，unsigned char没有符号位，因此能表示的数据范围是0~255
实际使用中，如普通的赋值，读写文件和网络字节流都没有区别，不管最高位是什么，最终的读取结果都一样，在屏幕上面的显示可能不一样。

但是要把一个char类型的变量赋值给int、long等数据类型或进行类似的强制类型转换时时，系统会进行类型扩展，这时区别就大了。对于char类型的变量，系统会认为最高位为符号位，然后对最高位进行扩展，即符号扩展。若最高位为1，则扩展到int时高位都以1填充。对于unsigned char类型的变量，系统会直接进行无符号扩展，即0扩展。扩展的高位都以0填充。所以在进行类似的操作时，如果char和unsigned char最高位都是0，则结果是一样的，若char最高位为1，则结果会大相径庭。



## 快速互转16进制和原始字符串
参考：https://www.chenxublog.com/2020/03/08/c-fast-convert-hex-char-array.html

```c
#include <stdio.h>
const char hex_table[] = {
'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'
};
// s -> d: 从s的末尾开始，取4的低四位0100，即4，查表，得4，然后高四位0011查表得3，即34 // 34即4的asii码。 
void to_hex(char *s, int l, char *d)//s:1234,d:31323334
{                                   //4:0011 0100									
    while(l--)                    
    {
    	int x=2*l+1;
    	
        *(d+2*l+1) = hex_table[(*(s+l))&0x0f];
        *(d+2*l) = hex_table[(*(s+l))>>4];
    }
}
//取倒数第一的值的后四位，与倒数第二值左移四位再相或 
void from_hex(char *s, int l, char *d)
{
	//s:"31323334"
	//d:1234
    while(l--)//4:0011 0100,3:0011 0011
    {         //取4的低四位0100，与3左移四位后（0011 0000）相或，即 0000 0100 | 0011 0000
        char* p = s+l;//相当于3乘16再加4，得到4的asii值。
        char* p2 = p-1;
        printf("%d %d %d,%d  %d \n",*(p),*(p2),(*p)&0x0f,(*p2)<<4,(*p)&0x0f|(*p2)<<4);
        *(d+l/2) =( (*p>'9'? *p+9 : *p) & 0x0f ) | ((*p2>'9'? *p2+9 : *p2) << 4 );  
        l--;
    }
}

int main () 
{
    char s[]= "1234";
    char s2[5];
    char d[9];
    s2[4]='\0';
    d[8] = '\0';
    to_hex(s,4,d);
    printf("%s\n",d);
    
    from_hex(d,8,s2);
	printf("%s",s2);  
    return 0;
}
```
提升版：
2020.3.9：Antecer带来了更高效的hex转数组代码

```c
#include <stdio.h>

const char hex_table[] = {
'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'
};
void to_hex(char *s, int l, char *d)
{
    while(l--)
    {
        *d = hex_table[*s >> 4];
        d++;
        *d = hex_table[*s & 0x0f];
        s++;
        d++;
    }
}

void from_hex(char *s, int l, char *d)
{
    while(l--)
    {
        *d = (*s>'9' ? *s+9 : *s) << 4;
        ++s;
        *d |= (*s>'9' ? *s+9 : *s) & 0x0F;
        ++s;
        ++d;
    }
}
```

2020.3.10：稀饭放姜发现内嵌“++”操作比单独写一行运行要快
```c
#include <stdio.h>
const char hex_table[] = {
'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'
};
void to_hex(char *s, int l, char *d)
{
    while(l--)
    {
        *(d++) = hex_table[*s >> 4];
        *(d++) = hex_table[*(s++) & 0x0f];
    }
}
void from_hex(char *s, int l, char *d)
{
    while(l--)
    {
        *(d++) = ( (*s>'9' ? *(s++)+9 : *(s++)) << 4 )
        | ( (*s>'9' ? *(s++)+9 : *(s++)) & 0x0F );
    }
}
```




## PRId64
参考：https://blog.csdn.net/lyndon_li/article/details/113755110
PRIu64 的定义在 inttypes.h 头文件里。

```c
# if __WORDSIZE == 64
#  define __PRI64_PREFIX	"l"
# else
#  define __PRI64_PREFIX	"ll"
# endif

# define PRIu64		__PRI64_PREFIX "u"

```

可以看到，
32 位编译器，会把 "%"PRIu64 扩展为 "%lld"，
64 位编译器，会把 "%"PRIu64 扩展为 "%ld"，
使得uint64_t兼容32位和64位机器。
## 格式控制符
### %p
格式控制符“%p”中的p是pointer（指针）的缩写。指针的值是语言实现（编译程序）相关的，但几乎所有实现中，指针的值都是一个表示地址空间中某个存储器单元的整数。printf函数族中对于%p一般以十六进制整数方式输出指针的值，附加前缀0x。
示例：
```c
int i = 1;
printf("%p",&i);
```
相当于
```c
int i = 1;
printf("0x%x",&i);
```

# 信号量相关
可参考网盘中的Linux系统编程笔记
## 互斥锁mutex
### 主要应用函数：
- pthread_mutex_init 函数
- pthread_mutex_destory 函数
- pthread_mutex_lock 函数
- pthread_mutex_trylock 函数
- pthread_mutex_unlock 函数
以上 5 个函数的返回值都是：成功返回 0，失败返回错误号
pthread_mutex_t 类型，其本质是一个结构体。为简化理解，应用时可忽略其实现细节，简单当成整数看待
pthread_mutex_t mutex；变量 mutex 只有两种取值：0,1
### 使用步骤
使用 mutex(互斥量、互斥锁)一般步骤：
pthread_mutex_t 类型。
1. pthread_mutex_t lock; 创建锁
2 pthread_mutex_init; 初始化 1
3. pthread_mutex_lock;加锁 1-- --> 0
4. 访问共享数据（stdout)
5. pthrad_mutext_unlock();解锁 0++ --> 1
6. pthead_mutex_destroy；销毁锁

`int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr)`

这里的 restrict 关键字，表示指针指向的内容只能通过这个指针进行修改
restrict 关键字：
用来限定指针变量。被该关键字限定的指针变量所指向的内存操作，必须由本指针完成。
初始化互斥量：
pthread_mutex_t mutex;
1. pthread_mutex_init(&mutex, NULL); 动态初始化。
2. pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; 静态初始化。



## 条件变量cond
本身不是锁！ 但是通常结合锁来使用。 mutex
### 主要应用函数:
pthread_cond_init   函数。
pthread_cond_destroy   函数,
pthread_cond_wait   函数。
pthread_cond_timedwait  函数,
pthread_cond_signal   函数。
pthread_cond_broadcast   函数
以上6个函数的返回值都是。成功返回o，失败直接返回错误号。v
pthread cond t类型用于定义条件变量。
pthread cond t cond;

初始化条件变量：
1. pthread_cond_init(&cond, NULL); 动态初始化。
2. pthread_cond_t cond = PTHREAD_COND_INITIALIZER; 静态初始化。

阻塞等待条件：
pthread_cond_wait(&cond, &mutex);
作用：
1） 阻塞等待条件变量满足
2） 解锁已经加锁成功的信号量 （相当于 pthread_mutex_unlock(&mutex)），12 两步为
一个原子操作
3) 当条件满足，函数返回时，解除阻塞并重新申请获取互斥锁。重新加锁信号量 （相
当于， pthread_mutex_lock(&mutex);）


### pthread_cond_wait
#### 定义
`int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);`
#### 描述
用于阻塞当前线程，等待别的线程使用pthread_cond_signal()或pthread_cond_broadcast来唤醒它。 pthread_cond_wait() 必须与pthread_mutex
配套使用。
pthread_cond_wait()函数一进入wait状态就会自动release mutex。当其他线程通过pthread_cond_signal()或pthread_cond_broadcast，把该线程唤醒，使pthread_cond_wait()通过（返回）时，该线程又自动获得该mutex。
```
pthread_cond_wait atomically unlocks the mutex (as per pthread_unlock_mutex) and waits for the condition variable cond to be signalled. 
Pthread_cond_wait 自动解锁互斥锁(按每个 pthread_unlock_mutex 解锁)并等待条件变量 cond 发出信号。
The thread execution is suspended and does not consume any CPU time until the condition variable is signalled. 
线程执行暂停，在条件变量发出信号之前不消耗任何 CPU 时间。
The mutex must be locked by the calling thread on entrance to pthread_cond_wait. 
必须在 pthread_cond_wait 入口处由调用线程锁定互斥对象。
Before returning to the calling thread, pthread_cond_wait re-acquires mutex (as per pthread_lock_mutex).
在返回到调用线程之前，pthread _ cond _ wait 重新获取互斥对象(如 pthread_lock_mutex)。
```

### 生产者消费者代码预览

```c
1./*借助条件变量模拟 生产者-消费者 问题*/  
2.#include <stdlib.h>  
3.#include <unistd.h>  
4.#include <pthread.h>  
5.#include <stdio.h>  
6.  
7./*链表作为公享数据,需被互斥量保护*/  
8.struct msg {  
9.    struct msg *next;  
10.    int num;  
11.};  
12.  
13.struct msg *head;  
14.  
15./* 静态初始化 一个条件变量 和 一个互斥量*/  
16.pthread_cond_t has_product = PTHREAD_COND_INITIALIZER;  
17.pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;  
18.  
19.void *consumer(void *p)  
20.{  
21.    struct msg *mp;  
22.  
23.    for (;;) {  
24.        pthread_mutex_lock(&lock);  
25.        while (head == NULL) {           //头指针为空,说明没有节点    可以为if吗  
26.            pthread_cond_wait(&has_product, &lock);   // 解锁，并阻塞等待  
27.        }  
28.        mp = head;        
29.        head = mp->next;                 //模拟消费掉一个产品  
30.        pthread_mutex_unlock(&lock);  
31.  
32.        printf("-Consume %lu---%d\n", pthread_self(), mp->num);  
33.        free(mp);  
34.        sleep(rand() % 5);  
35.    }  
36.}  
37.  
38.void *producer(void *p)  
39.{  
40.    struct msg *mp;  
41.  
42.    for (;;) {  
43.        mp = malloc(sizeof(struct msg));  
44.        mp->num = rand() % 1000 + 1;        //模拟生产一个产品  
45.        printf("-Produce ---------------------%d\n", mp->num);  
46.  
47.        pthread_mutex_lock(&lock);  
48.        mp->next = head;  
49.        head = mp;  
50.        pthread_mutex_unlock(&lock);  
51.  
52.        pthread_cond_signal(&has_product);  //将等待在该条件变量上的一个线程唤醒  
53.        sleep(rand() % 5);  
54.    }  
55.}  
56.  
57.int main(int argc, char *argv[])  
58.{  
59.    pthread_t pid, cid;  
60.    srand(time(NULL));  
61.  
62.    pthread_create(&pid, NULL, producer, NULL);  
63.    pthread_create(&cid, NULL, consumer, NULL);  
64.  
65.    pthread_join(pid, NULL);  
66.    pthread_join(cid, NULL);  
67.  
68.    return 0;  
69.}  
```


## 信号量sem
seamphore --信号量
应用于线程、进程间同步。
相当于 初始化值为 N 的互斥量。 N 值，表示可以同时访问共享数据区的线程数。
函数：
sem_t sem;	定义类型。
`int sem_init(sem_t *sem, int pshared, unsigned int value);`
参数：
sem： 信号量 
pshared：	0： 用于线程间同步
					1： 用于进程间同步
value：N值。（指定同时访问的线程数）
sem_destroy();
sem_wait();		一次调用，做一次-- 操作， 当信号量的值为 0 时，再次 -- 就会阻塞。 （对比 pthread_mutex_lock）
sem_post();		一次调用，做一次++ 操作. 当信号量的值为 N 时, 再次 ++ 就会阻塞。（对比 pthread_mutex_unlock）
