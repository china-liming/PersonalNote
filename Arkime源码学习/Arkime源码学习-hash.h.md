[toc]
## 源码介绍
```c
   hash.h  -- Hashtable
 * Simple macro based hashtable using DLL buckets with counts that supports 
 * having a single item in multiple lists or hash tables by using a prefix.  
 * Every element in the hash table needs to follow the DLL rules.
 *
 * To Use:
 * Create item structure and optional head structure for use with a DLL
 * Create the key function and cmp function.
 * Use HASH_VAR to declare the actual variable.  Can be global or in a structure
 * Use HASH_INIT to initialze the hashtable
 *
 * A key can also just be the element
 *
 * The same WARNING in dll.h applies since hash.h just uses DLL
简单的基于宏的哈希表，使用带有计数的DLL存储桶，支持在多个列表或哈希表中使用前缀包含单个项。
哈希表中的每个元素都需要遵循DLL规则。
使用：
创建项结构和可选的头结构，以便与DLL一起使用
创建键函数和cmp函数。
使用HASH_VAR声明实际变量。可以是全局的，也可以是结构中的
使用HASH_INIT初始化哈希表
键也可以只是元素
dll.h中也有同样的警告,因为hash.h只是使用了DLL
```
## 函数指针
原文代码中有这个，搞不懂：
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
&emsp;&emsp;这个语句就定义了一个指向函数的指针变量 p。首先它是一个指针变量，所以要有一个“ * ”，即(* p)其次前面的 int 表示这个指针变量可以指向返回值类型为 int 型的函数；后面括号中的两个 int 表示这个指针变量可以指向有两个参数且都是 int 型的函数。所以合起来这个语句的意思就是：定义了一个指针变量 p，该指针变量可以指向返回值类型为 int 型，且有两个整型参数的函数。p 的类型为 int(*)(int，int)。

所以函数指针的定义方式为：

```c
函数返回值类型 (* 指针变量名) (函数参数列表);
```

&emsp;&emsp;“函数返回值类型”表示该指针变量可以指向具有什么返回值类型的函数；“函数参数列表”表示该指针变量可以指向具有什么参数列表的函数。这个参数列表中只需要写函数的参数类型即可。

## typedef 函数指针 用法
原文链接：https://blog.csdn.net/qll125596718/article/details/6891881
使用typedef更直观更方便：typedef  返回类型(*新类型)(参数表)
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

## HASH_VAR
定义了一个叫varname的结构体，这个结构体包含创建一个hash表所需的所有元素，hash表的大小为num。
### 源码
```c
//说明：Use HASH_VAR to declare the actual variable.  Can be global or in a structure
#define HASH_VAR(name, varname, elementtype, num) \
   struct \
   { \
       HASH_KEY_FUNC hash; \
       HASH_CMP_FUNC cmp; \
       int size; \
       int count; \
       elementtype buckets[num]; \
   } varname
```
### 实例：
```c
//field.c --25行
HASH_VAR(d_, fieldsByDb, MolochFieldInfo_t, 307);

//扩展到：
struct{
	HASH_KEY_FUNC hash; 
	HASH_CMP_FUNC cmp; 
	int size; 
	int count; 
	MolochFieldInfo_t buckets[307]; 
} fieldsByDb
```

## HASH_INIT
初始化hash表，varname是一个结构体，必须要有size、hash、cmp、count、buckets 这几个成员变量。其中size是 varname 的 buckets 数组的大小，hash 是一个函数指针，cmp是一个函数指针。count 是此时hash表中元素的个数。buckets 是哈希表。
### 源码
```c
//Use HASH_INIT to initialze the hashtable
#define HASH_INIT(name, varname, hashfunc, cmpfunc) \
  do { \
       (varname).size = sizeof((varname).buckets)/sizeof((varname).buckets[0]); \
       (varname).hash = hashfunc; \
       (varname).cmp = cmpfunc; \
       (varname).count = 0; \
       for (int _i = 0; _i < (varname).size; _i++) { \
           DLL_INIT(name, &((varname).buckets[_i])); \
       } \
     } while (0)
```
### 实例：
```c
//config.c --925行
HASH_INIT(s_, config.dontSaveTags, moloch_string_hash, moloch_string_cmp);
扩展为：
do { 
	(config.dontSaveTags).size = sizeof(
		(config.dontSaveTags).buckets)/sizeof((config.dontSaveTags).buckets[0]
	); 
	(config.dontSaveTags).hash = moloch_string_hash; 
	(config.dontSaveTags).cmp = moloch_string_cmp; 
	(config.dontSaveTags).count = 0; 
	for (int _i = 0; _i < (config.dontSaveTags).size; _i++) 
	{ 
		DLL_INIT(s_, &((config.dontSaveTags).buckets[_i])); 
	} 
} while (0)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c31bd6b7fab34f2da93b853a2d2f3302.png)

## HASHP_VAR
### 源码
```c
#define HASHP_VAR(name, varname, elementtype) \
   struct \
   { \
       HASH_KEY_FUNC hash; \
       HASH_CMP_FUNC cmp; \
       int size; \
       int count; \
       elementtype *buckets; \
   } varname
```
### 实例：
这个宏定义只发现了这一个用法：
```c
//session.c --35行
typedef HASHP_VAR(h_, MolochSessionHash_t, MolochSessionHead_t);
```
扩展到:
```c
struct 
{
	HASH_KEY_FUNC hash;
	HASH_CMP_FUNC cmp;
    int size;
    int count;
	MolochSessionHead_t *buckets;
} MolochSessionHash_t
```
## HASHP_INIT
### 源码
```c
#define HASHP_INIT(name, varname, sz, hashfunc, cmpfunc) \
  do { \
       (varname).size = sz; \
       (varname).hash = hashfunc; \
       (varname).cmp = cmpfunc; \
       (varname).count = 0; \
       (varname).buckets = malloc(sz * sizeof((varname).buckets[0])); \
       for (int _i = 0; _i < (varname).size; _i++) { \
           DLL_INIT(name, &((varname).buckets[_i])); \
       } \
     } while (0)
```
### 实例：
```c
session.c --635
HASHP_INIT(h_, sessions[t][s], primes[s], moloch_session_hash, (HASH_CMP_FUNC)moloch_session_cmp);
```

```c
扩展到:
do { 
	(sessions[t][s]).size = primes[s]; 
	(sessions[t][s]).hash = moloch_session_hash; 
	(sessions[t][s]).cmp = (HASH_CMP_FUNC)moloch_session_cmp; 
	(sessions[t][s]).count = 0; 
	(sessions[t][s]).buckets = malloc(primes[s] * sizeof((sessions[t][s]).buckets[0])); 
	for (int _i = 0; _i < (sessions[t][s]).size; _i++) 
	{ 
		DLL_INIT(h_, &((sessions[t][s]).buckets[_i])); 
	} 
} while (0)
```
## HASH_HASH
### 源码
```c
#define HASH_HASH(varname, key) (varname).hash(key)
```
用法：

```c
#define HASH_ADD_HASH(name, varname, h, key, element) \
  do { \
      const uint32_t _hh = element->name##hash = h; \
      const int _b = element->name##bucket = element->name##hash % (varname).size; \
      const void *_end = (void*)&((varname).buckets[_b]); \
      for (element->name##next = (varname).buckets[_b].name##next; element->name##next != _end; element->name##next = element->name##next->name##next) { \
          if (_hh > element->name##next->name##hash) break; \
      }\
     element->name##prev             = element->name##next->name##prev; \
     element->name##prev->name##next = element; \
     element->name##next->name##prev = element; \
     (varname).buckets[_b].name##count++;\
     (varname).count++; \
  } while(0)


```
### 实例：
```c
只发现两处用法：
#define HASH_ADD(name, varname, key, element) HASH_ADD_HASH(name, varname, HASH_HASH(varname, key), key, element)
#define HASH_FIND(name, varname, key, element) HASH_FIND_HASH(name, varname, HASH_HASH(varname, key), key, element)
//config.c --void moloch_config_add_header(MolochStringHashStd_t *hash, char *key, int pos)
//701行
HASH_ADD(s_, *hash, hstring->str, hstring);

扩展到：
//第一步扩展：
HASH_ADD_HASH(s_, *hash, HASH_HASH(*hash, hstring->str), hstring->str, hstring)
//第二步扩展
do{ 
	 const uint32_t _hh = hstring->s_hash = (*hash).hash(hstring->str);
	 const int _b = hstring->s_bucket = hstring->s_hash % (*hash).size; 
	 const void *_end = (void*)&((*hash).buckets[_b]); 
	 for (hstring->s_next = (*hash).buckets[_b].s_next; hstring->s_next != _end; hstring->s_next = hstring->s_next->s_next) 
	{ 
	 	if (_hh > hstring->s_next->s_hash) 
	 		break; 
	} 
	 	hstring->s_prev = hstring->s_next->s_prev; 
	 	hstring->s_prev->s_next = hstring; 
	 	hstring->s_next->s_prev = hstring; 
	 	(*hash).buckets[_b].s_count++; 
	 	(*hash).count++; 
	 } while(0)
```
## HASH_ADD_HASH
### 源码
```c
#define HASH_ADD_HASH(name, varname, h, key, element) \
  do { \
      const uint32_t _hh = element->name##hash = h; \
      const int _b = element->name##bucket = element->name##hash % (varname).size; \
      const void *_end = (void*)&((varname).buckets[_b]); \
      for (element->name##next = (varname).buckets[_b].name##next; element->name##next != _end; element->name##next = element->name##next->name##next) { \
          if (_hh > element->name##next->name##hash) break; \
      }\
     element->name##prev             = element->name##next->name##prev; \
     element->name##prev->name##next = element; \
     element->name##next->name##prev = element; \
     (varname).buckets[_b].name##count++;\
     (varname).count++; \
  } while(0)

```
### 实例

```c
//hash.h --91行
#define HASH_ADD(name, varname, key, element) HASH_ADD_HASH(name, varname, HASH_HASH(varname, key), key, element)

//session.c --486
HASH_ADD_HASH(h_, sessions[thread][ses], hash, sessionId, session);
//扩展到：
do{ 
		const uint32_t _hh = session->h_hash = hash; 
		const int _b = session->h_bucket = session->h_hash % (sessions[thread][ses]).size; 
		const void *_end = (void*)&((sessions[thread][ses]).buckets[_b]); 
		for (session->h_next = (sessions[thread][ses]).buckets[_b].h_next; session->h_next != _end; session->h_next = session->h_next->h_next) 
		{ 
			if (_hh > session->h_next->h_hash) break; 
		}
		session->h_prev = session->h_next->h_prev; 
		session->h_prev->h_next = session; 
		session->h_next->h_prev = session; 
		(sessions[thread][ses]).buckets[_b].h_count++; 
		(sessions[thread][ses]).count++; 
} while(0)


```

## HASH_ADD
向hash这个hash表中添加值为i的元素，元素的结构体为hint
### 源码
```c
#define HASH_ADD(name, varname, key, element) HASH_ADD_HASH(name, varname, HASH_HASH(varname, key), key, element)
```
### 实例

```c

//field.c--771
HASH_ADD(i_, *hash, (void *)(long)i, hint);
//扩展到：
do
{
    const uint32_t _hh = hint->i_hash = (*hash).hash((void *)(long)i);
    const int _b = hint->i_bucket = hint->i_hash % (*hash).size;
    const void *_end = (void *)&((*hash).buckets[_b]);
    for (hint->i_next = (*hash).buckets[_b].i_next; hint->i_next != _end; hint->i_next = hint->i_next->i_next)
    {
        if (_hh > hint->i_next->i_hash)
            break;
    }
    hint->i_prev = hint->i_next->i_prev;
    hint->i_prev->i_next = hint;
    hint->i_next->i_prev = hint;
    (*hash).buckets[_b].i_count++;
    (*hash).count++;
} while (0)


//config.c --void moloch_config_add_header(MolochStringHashStd_t *hash, char *key, int pos)
//701行
HASH_ADD(s_, *hash, hstring->str, hstring);

扩展到：
//第一步扩展：
HASH_ADD_HASH(s_, *hash, HASH_HASH(*hash, hstring->str), hstring->str, hstring)
//第二步扩展
do{ 
	 const uint32_t _hh = hstring->s_hash = (*hash).hash(hstring->str);
	 const int _b = hstring->s_bucket = hstring->s_hash % (*hash).size; 
	 const void *_end = (void*)&((*hash).buckets[_b]); 
	 for (hstring->s_next = (*hash).buckets[_b].s_next; hstring->s_next != _end; hstring->s_next = hstring->s_next->s_next) 
	{ 
	 	if (_hh > hstring->s_next->s_hash) 
	 		break; 
	} 
	 	hstring->s_prev = hstring->s_next->s_prev; 
	 	hstring->s_prev->s_next = hstring; 
	 	hstring->s_next->s_prev = hstring; 
	 	(*hash).buckets[_b].s_count++; 
	 	(*hash).count++; 
   } while(0)

```

## HASH_REMOVE
### 源码
```c
#define HASH_REMOVE(name, varname, element) \
  do { \
      DLL_REMOVE(name, &((varname).buckets[element->name##bucket]), element); \
      (varname).count--; \
  } while(0)

```
### 实例

```c
//field.c --384
HASH_REMOVE(d_, fieldsByDb, minfo);
//扩展到
do{ 
	DLL_REMOVE(d_, &((fieldsByDb).buckets[minfo->d_bucket]), minfo); 
	(fieldsByDb).count--; 
  } while(0)
```

## HASH_FIND_HASH
### 源码
```c
#define HASH_FIND_HASH(name, varname, h, key, element) \
  do { \
      const uint32_t _hh = h; \
      const int _b = _hh % (varname).size; \
      const void *_end = (void*)&((varname).buckets[_b]); \
      for (element = (varname).buckets[_b].name##next; element != _end; element = element->name##next) { \
          if (_hh == element->name##hash && (varname).cmp(key, element)) break; \
          if (_hh > element->name##hash) {element = 0; break;} \
      } \
      if (element == _end) element = 0; \
  } while(0)
```
### 实例

## HASH_FIND_INT
### 源码
```c
#define HASH_FIND_INT(name, varname, key, element) HASH_FIND_HASH(name, varname, (uint32_t)key, (void*)(long)key, element)
```
### 实例
## HASH_FIND
### 描述 
通过key的值从varname哈希表中，找到对应的结构体，放到element中。
### 参数
- name              结构体中的名字，比如s_next、s_hash中的s_  
varname           哈希表的名字
key                    需要查找的名字
element             查找到的结果，通过此返回。
### 返回值
无

### 源码
```c
#define HASH_HASH(varname, key) (varname).hash(key)

#define HASH_FIND_HASH(name, varname, h, key, element) \
  do { \
      const uint32_t _hh = h; \
      const int _b = _hh % (varname).size; \
      const void *_end = (void*)&((varname).buckets[_b]); \
      for (element = (varname).buckets[_b].name##next; element != _end; element = element->name##next) { \
          if (_hh == element->name##hash && (varname).cmp(key, element)) break; \
          if (_hh > element->name##hash) {element = 0; break;} \
      } \
      if (element == _end) element = 0; \
  } while(0)

#define HASH_FIND(name, varname, key, element) HASH_FIND_HASH(name, varname, HASH_HASH(varname, key), key, element)
```
### 实例
```c
writers.c --41
HASH_FIND(s_, writersHash, name, str);

//扩展到
do
{
    const uint32_t _hh = (writersHash).hash(name);     //_hh 为将name进行哈希运算后的结果。
    const int _b = _hh % (writersHash).size;           //_b 为对size取余得出的数组下标。
    const void *_end = (void *)&((writersHash).buckets[_b]); //_end 为末尾的值
    for (str = (writersHash).buckets[_b].s_next; str != _end; str = str->s_next) //查找该数组下标下的链表。
    {
        if (_hh == str->s_hash && (writersHash).cmp(name, str))   //s_hash 应该为一次计算后的哈希值
            break;                                                 //找到。
        if (_hh > str->s_hash)                                     //没找到。
        {
            str = 0;
            break;
        }
    }
    if (str == _end)                                                //没找到
        str = 0;
} while (0)

```
## HASH_COUNT
### 源码
```c
#define HASH_COUNT(name, varname) ((varname).count)
```
### 实例

## HASH_BUCKET_COUNT
### 源码
```c
#define HASH_BUCKET_COUNT(name, varname, h) ((varname).buckets[((uint32_t)h) % (varname).size].name##count)
```
### 实例

## HASH_FORALL_POP_HEAD
### 源码
```c
#define HASH_FORALL_POP_HEAD(name, varname, element, code) \
  for ( int _##name##b = 0;  _##name##b < (varname).size;  _##name##b++) {\
      while((varname).buckets[_##name##b].name##count) { \
          DLL_POP_HEAD(name, &((varname).buckets[_##name##b]), element); \
          (varname).count--; \
          code \
      } \
  }
```
### 实例

## HASH_FORALL
### 源码
```c
#define HASH_FORALL(name, varname, element, code) \
  for ( int _##name##b = 0;  _##name##b < (varname).size;  _##name##b++) {\
      for (element = (varname).buckets[_##name##b].name##next; element != (void*)&((varname).buckets[_##name##b]); element = element->name##next) { \
          code \
      } \
  }
```
### 实例



