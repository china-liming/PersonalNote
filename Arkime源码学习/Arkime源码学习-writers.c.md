[toc]
## 简介
writers.c  -- Functions dealing with writers
 处理编写器的函数
## 描述
- 声明了如下几个函数指针，这几个函数指针分别在不同种类的writer中进行具体定义。
  * MolochWriterQueueLength     moloch_writer_queue_length;
  *   MolochWriterWrite     moloch_writer_write;
  * MolochWriterExit          moloch_writer_exit;
  * MolochWriterIndex       moloch_writer_index;
- 声明writersHash哈希表，此哈希表是MolochStringHashStd_t类型。
- 定义moloch_writers_start() ,moloch_writers_add(),moloch_writers_init()这几个函数。
- 声明了如下几个初始化函数，这几个函数分别在不同种类的writer中被定义，在moloch_writers_init()中被调用，进行writer的初始化。
  - writer_disk_init(char*)，//好像没有用
  - writer_null_init(char*),
  - writer_inplace_init(char*),
  - writer_simple_init(char*);

## 源码
其中，[[Arkime源码学习-main.c#moloch_string_add|moloch_string_add();]]
```c
#include "moloch.h"
#include <inttypes.h>
#include <errno.h>

MolochWriterQueueLength moloch_writer_queue_length;
MolochWriterWrite moloch_writer_write;
MolochWriterExit moloch_writer_exit;
MolochWriterIndex moloch_writer_index;

/******************************************************************************/
extern MolochConfig_t        config;

LOCAL  MolochStringHashStd_t writersHash;

/******************************************************************************/
void moloch_writers_start(char *name) 
{
    MolochString_t *str;
    char *freeIt = NULL;

    if (!name)
        name = freeIt = moloch_config_str(NULL, "pcapWriteMethod", "simple");


    HASH_FIND(s_, writersHash, name, str);
    if (!str) 
	{
        CONFIGEXIT("Couldn't find pcapWriteMethod %s implementation", name);
    }
    MolochWriterInit func = str->uw;
    func(name);
    moloch_add_can_quit((MolochCanQuitFunc)moloch_writer_queue_length, "writer queue length");
    g_free(freeIt);
}
/******************************************************************************/
void moloch_writers_add(char *name, MolochWriterInit func) 
{
    moloch_string_add(&writersHash, name, func, TRUE);
}
/******************************************************************************/
void writer_disk_init(char*);
void writer_null_init(char*);
void writer_inplace_init(char*);
void writer_simple_init(char*);

void moloch_writers_init()
{
    HASH_INIT(s_, writersHash, moloch_string_hash, moloch_string_cmp);
    moloch_writers_add("null", writer_null_init);
    moloch_writers_add("inplace", writer_inplace_init);
    moloch_writers_add("simple", writer_simple_init);
    moloch_writers_add("simple-nodirect", writer_simple_init);
}

```


## MolochStringHashStd_t
 ```c
 LOCAL  MolochStringHashStd_t writersHash; //--30
 
 typedef HASH_VAR(s_, MolochStringHashStd_t, MolochStringHead_t, 13);//moloch.h--121
 
// 扩展到
typedef struct
{
    HASH_KEY_FUNC hash;
    HASH_CMP_FUNC cmp;
    int size;
    int count;
    MolochStringHead_t buckets[13];
} MolochStringHashStd_t

 ```
 
## moloch_writers_init()
### 描述
初始化writersHash哈希表，并向哈希表中添加null,inplace,simple,simple-nodirect
### 源码
 ```c
 void moloch_writers_init()
{
    HASH_INIT(s_, writersHash, moloch_string_hash, moloch_string_cmp);
    moloch_writers_add("null", writer_null_init);
    moloch_writers_add("inplace", writer_inplace_init);
    moloch_writers_add("simple", writer_simple_init);
    moloch_writers_add("simple-nodirect", writer_simple_init);
}
```

## moloch_writers_add
### 源码
其中 moloch_string_add 函数链接[[Arkime源码学习-main.c#moloch_string_add]]
```c
void moloch_writers_add(char *name, MolochWriterInit func) {
    moloch_string_add(&writersHash, name, func, TRUE);
}
```

## moloch_string_hash
[[Arkime源码学习-main.c#moloch_string_hash]]
 ```c
 //main.c--453
 SUPPRESS_UNSIGNED_INTEGER_OVERFLOW
uint32_t moloch_string_hash(const void *key)
{
    unsigned char *p = (unsigned char *)key;
    uint32_t n = 0;
    while (*p) {
        n = (n << 5) - n + *p;
        p++;
    }

    n ^= hashSalt;

    return n;
}
```

## moloch_string_cmp
```c
int moloch_string_cmp(const void *keyv, const void *elementv)
{
    char *key = (char*)keyv;
    MolochString_t *element = (MolochString_t *)elementv;

    return strcmp(key, element->str) == 0;
}
```

## moloch_writers_start 
此函数在moloch.h中进行声明，在writers.c中进行定义。
### 描述
初步理解：从writerhash里面找到名叫name的一个MolochString_t结构体——str，找不到则报错退出，找到则调用str中的uw函数，uw函数的参数为name，并调用main.c中的moloch_add_can_quit函数。

### 源码
```c
void moloch_writers_start(char *name) 
{
    MolochString_t *str;
    char *freeIt = NULL;

    if (!name)
        name = freeIt = moloch_config_str(NULL, "pcapWriteMethod", "simple");

    HASH_FIND(s_, writersHash, name, str);
    if (!str) 
    {
        CONFIGEXIT("Couldn't find pcapWriteMethod %s implementation", name);
    }
    MolochWriterInit func = str->uw;
    func(name);
    moloch_add_can_quit((MolochCanQuitFunc)moloch_writer_queue_length, "writer queue length");
    g_free(freeIt);
}
```

### g_free;
```c
//glib
g_free(freeIt);

void g_free (gpointer mem)
描述：
释放mem所指向的内存。
如果mem是NULL，它会简单地返回，所以在调用这个函数之前不需要检查mem是否为NULL。

```

### MolochString_t --moloch
[[Arkime源码学习-moloch.h#moloch_string]]

### moloch_config_str --config.c
[[Arkime源码学习-config.c#moloch_config_str]]

```
name = freeIt = moloch_config_str(NULL, "pcapWriteMethod", "simple");
LOCAL GKeyFile             *molochKeyFile;
```

### HASH_FIND --hash.h
[[Arkime源码学习-hash.h#HASH_FIND]]
```c
HASH_FIND(s_, writersHash, name, str);

```
### CONFIGEXIT --moloch.h
```c
#define CONFIGEXIT(...)                              \
    do                                               \
    {                                                \
        printf("FATAL CONFIG ERROR - " __VA_ARGS__); \
        printf("\n");                                \
        exit(1);                                     \
    } while (0) /* no trailing ; */
//实例
CONFIGEXIT("Couldn't find pcapWriteMethod %s implementation", name);
//扩展到：
do
{
    printf("FATAL CONFIG ERROR - "
           "Couldn't find pcapWriteMethod %s implementation",
           name);
    printf("\n");
    exit(1);
} while (0)
```

### MolochWriterInit
```c
typedef void (*MolochWriterInit)(char *name);
```
### moloch_add_can_quit
[[Arkime源码学习-main.c#moloch_add_can_quit]]
```c
//实例：
moloch_add_can_quit((MolochCanQuitFunc)moloch_writer_queue_length, "writer queue length");
 
```

### 分析实例
在main.c中，LLVMFuzzerInitialize 和 moloch_ready_gfunc两个函数用到了moloch_writers_start，分别如下：
#### LLVMFuzzerInitialize 中
```c
moloch_writers_start("null");//774行
```

#### moloch_ready_gfunc中
```c
    moloch_writers_start("inplace");//670行
    moloch_writers_start(NULL);//672行
    moloch_writers_start("null");//677行
```

#### 这两个函数的完整函数体如下：
```c
int LLVMFuzzerInitialize(int *UNUSED(argc), char ***UNUSED(argv))
{
    config.configFile = g_strdup("config.test.ini");
    config.dryRun = 1;
    config.pcapReadOffline = 1;
    config.hostName = strdup("fuzz.example.com");
    config.nodeName = strdup("fuzz");

    hashSalt = 0;
    pcapFileHeader.dlt = DLT_EN10MB;

    moloch_free_later_init();
    moloch_hex_init();
    moloch_config_init();
    moloch_writers_init();
    moloch_writers_start("null");
    moloch_readers_init();
    moloch_readers_set("null");
    moloch_plugins_init();
    moloch_field_init();
    moloch_http_init();
    moloch_db_init();
    moloch_packet_init();
    moloch_config_load_local_ips();
    moloch_config_load_packet_ips();
    moloch_yara_init();
    moloch_parsers_init();
    moloch_session_init();
    moloch_plugins_load(config.plugins);
    moloch_rules_init();
    moloch_packet_batch_init(&batch);
    return 0;
}
```

```c
gboolean moloch_ready_gfunc (gpointer UNUSED(user_data))
{
    if (moloch_http_queue_length(esServer))
        return TRUE;

    if (config.debug)
        LOG("maxField = %d", config.maxField);

    if (config.pcapReadOffline) {
        if (config.dryRun || !config.copyPcap) {
            moloch_writers_start("inplace");
        } else {
            moloch_writers_start(NULL);
        }

    } else {
        if (config.dryRun) {
            moloch_writers_start("null");
        } else {
            moloch_writers_start(NULL);
        }
    }
    moloch_readers_start();
    if (!config.pcapReadOffline && (pcapFileHeader.dlt == DLT_NULL || pcapFileHeader.snaplen == 0))
        LOGEXIT("ERROR - Reader didn't call moloch_packet_set_dltsnap");
    return FALSE;
}
```


