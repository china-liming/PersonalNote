[toc]
```c
/******************************************************************************/
 moloch.h -- General Moloch include file
 *
 * Copyright 2012-2017 AOL Inc. All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this Software except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
moloch.h--通用moloch文件
版权所有2012-2017 AOL Inc.保留所有权利。
根据Apache许可证2.0版（以下简称“许可证”）获得许可；
除非遵守许可证，否则不得使用本软件。
您可以通过以下网址获得许可证副本：
http://www.apache.org/licenses/LICENSE-2.0
除非适用法律要求或书面同意，否则根据许可证分发的软件按“原样”分发，
无任何明示或暗示的保证或条件。
请参阅许可证，了解管理许可证下权限和限制的特定语言
 
```

## unused(x)
```c
#define UNUSED(x) x __attribute((unused))
```
参考：[](https://blog.csdn.net/Rong_Toa/article/details/87926187)
这是最容易使用的属性之一，它将变量标记为故意未使用。这不仅让编译器免于发出未使用的变量警告，还告诉人类同样的事情：这是故意的。
Of course, it's a good idea to actually remove variables that you're not using, this is not always possible. A common case of the unused int argc parameter to main() is one, as are variables sometimes excluded by conditional compilation.
当然，实际上删除不使用的变量是个好主意，这并不总是可能的。main（）中未使用int argc参数的一种常见情况是，有时条件编译会排除变量。
“_attribute__;”添加在变量名之后，虽然它可能看起来很笨拙，但您可以习惯这种样式：
```c
int main(int argc __attribute__((unused)), char **argv)
{ ...
```

## 判断编译器版本
### 源码
```c
#if defined(__clang__)   //如果是clang 编译器
#define SUPPRESS_SIGNED_INTEGER_OVERFLOW __attribute__((no_sanitize("signed-integer-overflow")))
#define SUPPRESS_UNSIGNED_INTEGER_OVERFLOW __attribute__((no_sanitize("unsigned-integer-overflow")))
#define SUPPRESS_SHIFT __attribute__((no_sanitize("shift")))
#define SUPPRESS_ALIGNMENT __attribute__((no_sanitize("alignment")))
#define SUPPRESS_INT_CONVERSION __attribute__((no_sanitize("implicit-integer-sign-change")))
#elif __GNUC__ >= 5    //判断gcc的版本是否大于等于5
#define SUPPRESS_SIGNED_INTEGER_OVERFLOW
#define SUPPRESS_UNSIGNED_INTEGER_OVERFLOW
#define SUPPRESS_SHIFT __attribute__((no_sanitize_undefined()))
#define SUPPRESS_ALIGNMENT __attribute__((no_sanitize_undefined()))
#define SUPPRESS_INT_CONVERSION
#else                  //如果是MSVC (windows编译器)
#define SUPPRESS_SIGNED_INTEGER_OVERFLOW
#define SUPPRESS_UNSIGNED_INTEGER_OVERFLOW
#define SUPPRESS_SHIFT
#define SUPPRESS_ALIGNMENT
#define SUPPRESS_INT_CONVERSION
#endif

```
参考[[C语言学习#C 判断标准版本和编译器]]

## MOLOCH_TYPE_ALLOC0
```c
#define MOLOCH_TYPE_ALLOC0(type) (type *)(calloc(1, sizeof(type)))
```
calloc函数：[[C语言函数学习#calloc]]


## LOG()
### 源码：
```c
//moloch.h --289
#define MOLOCH_LOCK_EXTERN(var)         pthread_mutex_t var##_mutex

//moloch.h --770
extern MOLOCH_LOCK_EXTERN(LOG);

//moloch.h --771
#define LOG(...) do { \
    if(config.quiet == FALSE) { \
        MOLOCH_LOCK(LOG); \
        time_t _t = time(NULL); \
        char   _b[26]; \
        printf("%15.15s %s:%d %s(): ",\
            ctime_r(&_t, _b)+4, __FILE__,\
            __LINE__, __FUNCTION__); \
        printf(__VA_ARGS__); \
        printf("\n"); \
        fflush(stdout); \
        MOLOCH_UNLOCK(LOG); \
    } \
} while(0) /* no trailing ; */

//moloch--191,192
#define MOLOCH_LOCK(var)                pthread_mutex_lock(&var##_mutex)
#define MOLOCH_UNLOCK(var)              pthread_mutex_unlock(&var##_mutex)
```
### 实例

```c
//writer-simple.c -799 
LOG ("INFO: Reseting simpleGzipBlockSize to %d, the minimum value", simpleGzipBlockSize);
//扩展为：
do { 
    if(config.quiet == FALSE) 
    { 
        pthread_mutex_lock(&LOG_mutex); 
        time_t _t = time((void *)O); 
        char _b[26];
        printf("%15.15s %s:%d %s(): " , ctime _r(&_t,_b)+4, " D:\Cprogream\ arkime-main\capture|[writer-simple.c", 799,); 
        printf("INFO: Reseting simpleGzipBlockSize to %d, the minimum value" , simpleGzipBlockSize); 
        printf("\n");
        flush((stdout)); 
        pthread_mutex_unlock(&LOG_mutex); 
    } 
    } while(0)

```
## struct MolochConfig_t
```c
typedef struct moloch_config {
    gboolean  quitting;
    char     *configFile;
    char     *nodeName;
    char     *hostName;
    char    **pcapReadFiles;
    char    **pcapReadDirs;
    char    **pcapFileLists;
    gboolean  pcapReadOffline;
    gchar   **extraTags;
    gchar   **extraOps;
    MolochFieldOps_t ops;
    gchar     debug;
    gchar     insecure;
    gboolean  quiet;
    gboolean  dryRun;
    gboolean  noSPI;
    gboolean  copyPcap;
    gboolean  pcapRecursive;
    gboolean  noStats;
    gboolean  tests;
    gboolean  pcapMonitor;
    gboolean  pcapDelete;
    gboolean  pcapSkip;
    gboolean  pcapReprocess;
    gboolean  flushBetween;
    gboolean  noLoadTags;
    gboolean  trackESP;
    gboolean  noLockPcap;
    gint      pktsToRead;

    GHashTable *override;

    uint64_t  ipSavePcap[4];
    uint64_t  etherSavePcap[1024];

    enum MolochRotate rotate;

    int       writeMethod;

    HASH_VAR(s_, dontSaveTags, MolochStringHead_t, 11);
    MolochFieldInfo_t *fields[MOLOCH_FIELDS_MAX];
    int                maxField;
    int                tagsStringField;

    int                numPlugins;

    GRegex  *offlineRegex;
    char     *prefix;
    char     *nodeClass;
    char     *elasticsearch;
    char    **interface;
    int       pcapDirPos;
    char    **pcapDir;
    char     *pcapDirTemplate;
    char     *bpf;
    char     *yara;
    char     *emailYara;
    char     *caTrustFile;
    char    **geoLite2ASN;
    char    **geoLite2Country;
    char     *rirFile;
    char     *ouiFile;
    char     *dropUser;
    char     *dropGroup;
    char    **pluginsDir;
    char    **parsersDir;

    char    **rootPlugins;
    char    **plugins;
    char    **smtpIpHeaders;

    double    maxFileSizeG;
    uint64_t  maxFileSizeB;
    uint32_t  maxFileTimeM;
    uint32_t  timeouts[SESSION_MAX];
    uint32_t  tcpSaveTimeout;
    uint32_t  maxStreams[SESSION_MAX];
    uint32_t  maxPackets;
    uint32_t  maxPacketsInQueue;
    uint32_t  dbBulkSize;
    uint32_t  dbFlushTimeout;
    uint32_t  maxESConns;
    uint32_t  maxESRequests;
    uint32_t  logEveryXPackets;
    uint32_t  pcapBufferSize;
    uint32_t  pcapWriteSize;
    uint32_t  maxWriteBuffers;
    uint32_t  maxFreeOutputBuffers;
    uint32_t  fragsTimeout;
    uint32_t  maxFrags;
    uint32_t  snapLen;
    uint32_t  maxMemPercentage;
    uint32_t  maxReqBody;
    int       packetThreads;

    char      logUnknownProtocols;
    char      logESRequests;
    char      logFileCreation;
    char      logHTTPConnections;
    char      parseSMTP;
    char      parseSMTPHeaderAll;
    char      parseSMB;
    char      ja3Strings;
    char      parseDNSRecordAll;
    char      parseQSValue;
    char      parseCookieValue;
    char      parseHTTPHeaderRequestAll;
    char      parseHTTPHeaderResponseAll;
    char      supportSha256;
    char      reqBodyOnlyUtf8;
    char      compressES;
    char      antiSynDrop;
    char      readTruncatedPackets;
    char      yaraEveryPacket;
    char     *pcapDirAlgorithm;
    char      corruptSavePcap;
    char      autoGenerateId;
    char      ignoreErrors;
    char      enablePacketLen;
    char      gapPacketPos;
    char      enablePacketDedup;
} MolochConfig_t;
```

## moloch_int
```c
#define HASH_VAR(name, varname, elementtype, num) \
    struct                                        \
    {                                             \
        HASH_KEY_FUNC hash;                       \
        HASH_CMP_FUNC cmp;                        \
        int size;                                 \
        int count;                                \
        elementtype buckets[num];                 \
    } varname

typedef struct moloch_int 
{
    struct moloch_int    *i_next, *i_prev;
    uint32_t              i_hash;
    short                 i_bucket;
} MolochInt_t;

typedef struct 
{
    struct moloch_int *i_next, *i_prev;
    int i_count;
} MolochIntHead_t;

typedef HASH_VAR(s_, MolochIntHash_t, MolochIntHead_t, 1);
typedef HASH_VAR(s_, MolochIntHashStd_t, MolochIntHead_t, 13);

typedef struct
{
    HASH_KEY_FUNC hash;
    HASH_CMP_FUNC cmp;
    int size;
    int count;
    MolochIntHead_t buckets[1];
}MolochIntHash_t

typedef struct
{
    HASH_KEY_FUNC hash;
    HASH_CMP_FUNC cmp;
    int size;
    int count;
    MolochIntHead_t buckets[13];
}MolochIntHashStd_t

```
### 实例
```c

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
     
#ifdef MOLOCH_USE_MALLOC
#define MOLOCH_TYPE_ALLOC(type) (type *)(malloc(sizeof(type)))
#else
#define MOLOCH_TYPE_ALLOC(type) (type *)(g_slice_alloc(sizeof(type)))

hash = MOLOCH_TYPE_ALLOC(MolochIntHashStd_t);
HASH_INIT(i_, *hash, moloch_int_hash, moloch_int_cmp);

//扩展到
do
{
    (*hash).size = sizeof((*hash).buckets) / sizeof((*hash).buckets[0]);
    (*hash).hash = moloch_int_hash;
    (*hash).cmp = moloch_int_cmp;
    (*hash).count = 0;
    for (int _i = 0; _i < (*hash).size; _i++)
    {
        (
            (&((*hash).buckets[_i]))->i_count = 0, 
            (&((*hash).buckets[_i]))->i_next = (&((*hash).buckets[_i]))->i_prev=(void*)&((*hash).buckets[_i])
        )
        DLL_INIT(i_, &((*hash).buckets[_i]));
    }
} while (0)

```
![image](https://img-blog.csdnimg.cn/5e20fbedc820440c87c2552ac14a2b2e.png)
## moloch_string
```c
typedef struct moloch_string 
{
    struct moloch_string *s_next, *s_prev;
    char                 *str;
    uint32_t              s_hash;
    gpointer              uw;
    short                 s_bucket;
    short                 len:15;
    short                 utf8:1;
} MolochString_t;

typedef struct 
{
    struct moloch_string *s_next, *s_prev;
    int s_count;
} MolochStringHead_t;

typedef HASH_VAR(s_, MolochStringHash_t, MolochStringHead_t, 1);
typedef HASH_VAR(s_, MolochStringHashStd_t, MolochStringHead_t, 13);
//扩展为：
typedef struct
{
    HASH_KEY_FUNC hash;
    HASH_CMP_FUNC cmp;
    int size;
    int count;
    MolochStringHead_t buckets[1];
} MolochStringHash_t

typedef struct
{
    HASH_KEY_FUNC hash;
    HASH_CMP_FUNC cmp;
    int size;
    int count;
    MolochStringHead_t buckets[13];
} MolochStringHashStd_t

```