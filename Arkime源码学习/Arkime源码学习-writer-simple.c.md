## 简介
```c
 writer-simple.c  -- Simple Writer Plugin  简单编写器插件
This writer just creates a file per packet thread and queues buffers to be written to disk in a single output thread.
该编写器只需为每个数据包线程创建一个文件，并在单个输出线程中将要写入磁盘的缓冲区排队。
```

```c
// Information about the current file being written to, all items that are constant per file should be here
//有关要写入的当前文件的信息，每个文件的所有常量项都应在此处
typedef struct {
    EVP_CIPHER_CTX      *cipher_ctx;
    uint64_t             pos;
    uint64_t             blockStart;
    uint64_t             packetBytesWritten;
    uint32_t             packets;
    uint32_t             posInBlock;
    uint32_t             id;
    int                  fd;
    uint8_t              dek[256];
    z_stream             z_strm;
} MolochSimpleFile_t;
// Information about the current buffer being written to, there can be multiple buffers per file
//有关当前正在写入的缓冲区的信息，每个文件可以有多个缓冲区
// NOTE this points to the file structure, kind of backwards
//注意，这指向文件结构，有点向后
typedef struct molochsimple {
    struct molochsimple *simple_next, *simple_prev;
    char                *buf;
    MolochSimpleFile_t  *file;
    uint32_t             bufpos;
    uint8_t              closing;
    uint8_t              thread;
} MolochSimple_t;
typedef struct {
    struct molochsimple *simple_next, *simple_prev;
    int                  simple_count;
    MOLOCH_LOCK_EXTERN(lock);//pthread_mutex_t lock_mutex;
} MolochSimpleHead_t;

LOCAL  MolochSimpleHead_t simpleQ;
LOCAL  MOLOCH_LOCK_DEFINE(simpleQ);
LOCAL  MOLOCH_COND_DEFINE(simpleQ);

enum MolochSimpleMode { MOLOCH_SIMPLE_NORMAL, MOLOCH_SIMPLE_XOR2048, MOLOCH_SIMPLE_AES256CTR};

LOCAL MolochSimple_t        *currentInfo[MOLOCH_MAX_PACKET_THREADS];
LOCAL MolochSimpleHead_t     freeList[MOLOCH_MAX_PACKET_THREADS];
LOCAL uint32_t               pageSize;
LOCAL enum MolochSimpleMode  simpleMode;
LOCAL int                    simpleMaxQ;
LOCAL const EVP_CIPHER      *cipher;
LOCAL int                    openOptions;
LOCAL struct timeval         lastSave[MOLOCH_MAX_PACKET_THREADS];
LOCAL struct timeval         fileAge[MOLOCH_MAX_PACKET_THREADS];
LOCAL uint32_t               firstPacket[MOLOCH_MAX_PACKET_THREADS];

#define INDEX_FILES_CACHE_SIZE (MOLOCH_MAX_PACKET_THREADS-1)
struct {
    int64_t  fileNum;
    FILE    *fp;
} indexFiles[MOLOCH_MAX_PACKET_THREADS][INDEX_FILES_CACHE_SIZE];

/*
 * Compression design inspired by Philip Gladstone and others.
 * 灵感来自菲利普·格莱斯顿等人的压缩设计。
 * A compressed file is made up of compressed blocks.
 * 压缩文件由压缩块组成。
 * You can only start reading a compressed file at the beginning of a block.
 * 只能从块的开头开始读取压缩文件。
 * Blocks are variable sized, with the max UNCOMPRESED data per block
 * controlled by simpleGzipBlockSize.
 * 块的大小是可变的，每个块的最大未复制数据由simpleGzipBlockSize控制。
 * uncompressedBits is calculated so it can hold simpleGzipBlockSize.
 * uncompressedBits是经过计算的，因此它可以保存simpleGzipBlockSize。
 * The file pos for each packet is made of two parts
 *   X the location in the file of the start of the compress block, which
 *   is shifted uncompressedBits
 *   Y the location inside the uncompressed block of the packet start
 * 每个数据包的文件pos由两部分组成，X是压缩块开始的文件中的位置，Y是数据包开始的未压缩块中的位置
 * A larger simpleGzipBlockSize leads to better compression but slower read time.
 * simpleGzipBlockSize越大，压缩效果越好，但读取时间越慢。
 */
LOCAL int      uncompressedBits;// Number of bits used in filepos to store location in block
LOCAL uint32_t simpleGzipBlockSize; // Max data that we try and compress, can be represented by uncompressedBits


```

## writer_simple_init()
### 声明
在writers.c中 58行声明
```c
void writer_simple_init(char*);
```
### 源码
 writer-simple.c --784
```c
void writer_simple_init(char *name)
{
    moloch_writer_queue_length = writer_simple_queue_length;
    moloch_writer_exit         = writer_simple_exit;
    moloch_writer_write        = writer_simple_write;

    simpleMaxQ = moloch_config_int(NULL, "simpleMaxQ", 2000, 50, 0xffff);
    simpleGzipLevel = moloch_config_int(NULL, "simpleGzipLevel", 6, 1, 9);
    char *mode = moloch_config_str(NULL, "simpleEncoding", NULL);

    simpleGzipBlockSize = moloch_config_int(NULL, "simpleGzipBlockSize", 0, 0, 0xfffff);
    if (simpleGzipBlockSize > 0) {
        gzip = TRUE;
        if (simpleGzipBlockSize < 8192) {
            simpleGzipBlockSize = 8191;
            LOG ("INFO: Reseting simpleGzipBlockSize to %d, the minimum value", simpleGzipBlockSize);
        }
        uncompressedBits = ceil(log2(simpleGzipBlockSize));

        // simpleGzipBlockSize can't be a power of 2
        if ((uint32_t)pow(2, uncompressedBits) == simpleGzipBlockSize)
            simpleGzipBlockSize--;

        if (simpleGzipBlockSize > config.pcapWriteSize) {
            config.pcapWriteSize = simpleGzipBlockSize + 1;
            LOG ("INFO: Reseting pcapWriteSize to %d, so it is larger than simpleGzipBlockSize", config.pcapWriteSize);
        }

        if (config.debug)
            LOG("Will gzip - blocksize: %u bits: %u", simpleGzipBlockSize, uncompressedBits);
    }

    if (mode == NULL || !mode[0]) {
    } else if (strcmp(mode, "aes-256-ctr") == 0) {
        simpleMode = MOLOCH_SIMPLE_AES256CTR;
        cipher = EVP_aes_256_ctr();
        if (config.maxFileSizeB > 64*1024LL*1024LL*1024LL) {
            LOG ("INFO: Reseting maxFileSizeG since %lf is greater then the max 64G in aes-256-ctr mode", config.maxFileSizeG);
            config.maxFileSizeG = 64.0;
            config.maxFileSizeB = 64LL*1024LL*1024LL*1024LL;
        }
    } else if (strcmp(mode, "xor-2048") == 0) {
        LOG("WARNING - simpleEncoding of xor-2048 is NOT actually secure");
        simpleMode = MOLOCH_SIMPLE_XOR2048;
    } else {
        CONFIGEXIT("Unknown simpleEncoding '%s'", mode);
    }

    if (mode) {
        g_free(mode);
    }

    // Since we are doing direct IO must be a multiple of pagesize;
    pageSize = getpagesize();
    if (config.pcapWriteSize % pageSize != 0) {
        config.pcapWriteSize = ((config.pcapWriteSize + pageSize - 1) / pageSize) * pageSize;
        LOG ("INFO: Reseting pcapWriteSize to %u since it must be a multiple of %u", config.pcapWriteSize, pageSize);
    }

    openOptions = O_NOATIME | O_WRONLY | O_CREAT | O_TRUNC;
    if (strcmp(name, "simple") == 0) {
#ifdef O_DIRECT
        openOptions |= O_DIRECT;
#else
        LOG("No O_DIRECT defined, skipping");
#endif
    } else {
        LOG("Not using O_DIRECT by config");
    }

    config.gapPacketPos = moloch_config_boolean(NULL, "gapPacketPos", TRUE);

    simpleShortHeader = moloch_config_boolean(NULL, "simpleShortHeader", FALSE);
    if (simpleShortHeader && config.maxFileTimeM > 60) {
        config.maxFileTimeM = 60;
        LOG ("INFO: Reseting maxFileTimeM to 60 since using simpleShortHeader");
    }

    localPcapIndex = moloch_config_boolean(NULL, "localPcapIndex", FALSE);
    if (localPcapIndex) {
        if (config.pcapDir[1]) {
            LOG("WARNING - Will always use first pcap directory for local index");
        }

        config.maxFileSizeB = MIN(config.maxFileSizeB, 0x07ffffffffL);
        config.gapPacketPos = FALSE;
        moloch_writer_index = writer_simple_index;
    }

    DLL_INIT(simple_, &simpleQ);

    struct timeval now;
    gettimeofday(&now, NULL);

    int thread;
    for (thread = 0; thread < config.packetThreads; thread++) {
        lastSave[thread] = now;
        fileAge[thread] = now;
        DLL_INIT(simple_, &freeList[thread]);
        MOLOCH_LOCK_INIT(freeList[thread].lock);
    }

    g_thread_unref(g_thread_new("moloch-simple", &writer_simple_thread, NULL));

    g_timeout_add_seconds(1, writer_simple_check_gfunc, 0);
}



```




### 实例
在writers.c中moloch_writers_init()中使用。
[[Arkime源码学习-writers.c#moloch_writers_init]]


## writer_simple_write()
```c
LOCAL void writer_simple_write(const MolochSession_t * const session, MolochPacket_t * const packet)
{
    MolochSimple_t *info;

    if (DLL_COUNT(simple_, &simpleQ) > simpleMaxQ) {
        static uint32_t lastError;
        static uint32_t notSaved;
        packet->writerFilePos = 0;
        notSaved++;
        if (packet->ts.tv_sec > lastError + 60) {
            lastError = packet->ts.tv_sec;
            LOG("WARNING - Disk Q of %d is too large and exceed simpleMaxQ setting so not saving %u packets. Check the Arkime FAQ about (https://arkime.com/faq#why-am-i-dropping-packets) testing disk speed", DLL_COUNT(simple_, &simpleQ), notSaved);
        }
        return;
    }

    int thread = session->thread;

    // Need to open a new file
    if (!currentInfo[thread]) {
        char  dekhex[1024];
        char *name = 0;
        char *kekId;
        char *packetPosEncoding = MOLOCH_VAR_ARG_SKIP;
        char *uncompressedBitsArg = MOLOCH_VAR_ARG_SKIP;
        char  indexFilename[1024];

        indexFilename[0] = 0;
        if (localPcapIndex) {
            packetPosEncoding = "localIndex";
            snprintf(indexFilename, sizeof(indexFilename), "%s/%s-#NUM#.index", config.pcapDir[0], config.nodeName);
        } else if (config.gapPacketPos) {
            packetPosEncoding = "gap0";
        }

        info = currentInfo[thread] = writer_simple_alloc(thread, NULL);
        info->file = MOLOCH_TYPE_ALLOC0(MolochSimpleFile_t);

        if (gzip) {
            uncompressedBitsArg = (gpointer)(long)uncompressedBits;

            info->file->z_strm.next_out = (Bytef *) info->buf;
            info->file->z_strm.avail_out = config.pcapWriteSize + MOLOCH_PACKET_MAX_LEN;
            deflateInit2(&info->file->z_strm, simpleGzipLevel, Z_DEFLATED, 16 + 15, 8, Z_DEFAULT_STRATEGY);
        }

        switch(simpleMode) {
        case MOLOCH_SIMPLE_NORMAL:
            if (simpleShortHeader)
                name = ".arkime";
            else if (gzip)
                name = ".pcap.gz";
            else
                name = ".pcap";
            name = moloch_db_create_file_full(packet->ts.tv_sec, name, 0, 0, &info->file->id,
                                              "packetPosEncoding", packetPosEncoding,
                                              "uncompressedBits", uncompressedBitsArg,
                                              "indexFilename", indexFilename[0] ? indexFilename : MOLOCH_VAR_ARG_SKIP,
                                              (char *)NULL);
            break;
        case MOLOCH_SIMPLE_XOR2048:
            name = ".arkime";
            kekId = writer_simple_get_kekId();
            RAND_bytes(info->file->dek, 256);
            writer_simple_encrypt_key(kekId, info->file->dek, 256, dekhex);
            name = moloch_db_create_file_full(packet->ts.tv_sec, name, 0, 0, &info->file->id,
                                              "encoding", "xor-2048",
                                              "dek", dekhex,
                                              "kekId", kekId,
                                              "packetPosEncoding", packetPosEncoding,
                                              "uncompressedBits", uncompressedBitsArg,
                                              "indexFilename", indexFilename[0] ? indexFilename : MOLOCH_VAR_ARG_SKIP,
                                              (char *)NULL);
            g_free(kekId);
            break;
        case MOLOCH_SIMPLE_AES256CTR: {
            info->file->cipher_ctx = EVP_CIPHER_CTX_new();
            name = ".arkime";
            uint8_t dek[32];
            uint8_t iv[16];
            char    ivhex[33];
            RAND_bytes(iv, 12);
            RAND_bytes(dek, 32);
            memset(iv+12, 0, 4);
            kekId = writer_simple_get_kekId();
            writer_simple_encrypt_key(kekId, dek, 32, dekhex);
            moloch_sprint_hex_string(ivhex, iv, 12);
            EVP_EncryptInit(info->file->cipher_ctx, cipher, dek, iv);
            name = moloch_db_create_file_full(packet->ts.tv_sec, name, 0, 0, &info->file->id,
                                              "encoding", "aes-256-ctr",
                                              "iv", ivhex,
                                              "dek", dekhex,
                                              "kekId", kekId,
                                              "packetPosEncoding", packetPosEncoding,
                                              "uncompressedBits", uncompressedBitsArg,
                                              "indexFilename", indexFilename[0] ? indexFilename : MOLOCH_VAR_ARG_SKIP,
                                              (char *)NULL);
            g_free(kekId);
            break;
        }
        default:
            CONFIGEXIT("Unknown simpleMode %d", simpleMode);
        }

        /* If offline pcap honor umask, otherwise disable other RW */
        if (config.pcapReadOffline) {
            currentInfo[thread]->file->fd = open(name,  openOptions, S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH);
        } else {
            currentInfo[thread]->file->fd = open(name,  openOptions, S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP);
        }
        if (currentInfo[thread]->file->fd < 0) {
            LOGEXIT("ERROR - pcap open failed - Couldn't open file: '%s' with %s  (%d) -- You may need to check directory permissions or set pcapWriteMethod=simple-nodirect in config.ini file.  See https://arkime.com/settings#pcapwritemethod", name, strerror(errno), errno);
        }

        if (simpleShortHeader) {
            firstPacket[thread] = packet->ts.tv_sec - 60; // Allow slightly out of sync clocks
            MolochPcapFileHdr_t   pcapFileHeader2;
            memcpy(&pcapFileHeader2, &pcapFileHeader, 24);
            pcapFileHeader2.magic = 0xa1b2c3d5;
            pcapFileHeader2.thiszone = firstPacket[thread];
            writer_simple_write_output(info, (unsigned char *)&pcapFileHeader2, 20);
        } else {
            writer_simple_write_output(info, (unsigned char *)&pcapFileHeader, 20);
        }

        uint32_t linktype = moloch_packet_dlt_to_linktype(pcapFileHeader.dlt);
        writer_simple_write_output(info, (unsigned char *)&linktype, 4);
        if (config.debug)
            LOG("opened %d %s %d", thread, name, info->file->fd);
        g_free(name);

        // Make a new block for start of packets
        if (gzip)
            writer_simple_gzip_make_new_block(info);
        gettimeofday(&fileAge[thread], NULL);
    } else {
        info = currentInfo[thread];
    }

    packet->writerFileNum = info->file->id;

    if (gzip) {
        if (info->file->posInBlock >= simpleGzipBlockSize) {
            writer_simple_gzip_make_new_block(info);
        }

        packet->writerFilePos = (info->file->blockStart << uncompressedBits) + info->file->posInBlock;
    } else {
        packet->writerFilePos = info->file->pos;
    }

    info->file->packets++;
    if (simpleShortHeader) {
        char header[6];
        // LLLL LLLL LLLL LLLL
        memcpy(header, &packet->pktlen, 2);

        // SSSS SSSS SSSS MMMM MMMM MMMM MMMM MMMM
        uint32_t t;
        if (firstPacket[thread] > packet->ts.tv_sec) {
            LOG("WARNING - timing moving backwards, simpleShortHeader should be disabled");
            // Time stamp is too early, just prented its at firstPacket time
            t = packet->ts.tv_usec;
        } else {
            t = ((packet->ts.tv_sec - firstPacket[thread]) << 20) | packet->ts.tv_usec;
        }

        memcpy(header+2, &t, 4);

        writer_simple_write_output(info, (unsigned char *)&header, 6);
    } else {
        struct moloch_pcap_sf_pkthdr hdr;

        hdr.ts.tv_sec  = packet->ts.tv_sec;
        hdr.ts.tv_usec = packet->ts.tv_usec;
        hdr.caplen     = packet->pktlen;
        hdr.pktlen     = packet->pktlen;
        writer_simple_write_output(info, (unsigned char *)&hdr, 16);
    }
    writer_simple_write_output(info, packet->pkt, packet->pktlen);

    if (info->bufpos > config.pcapWriteSize) {
        writer_simple_process_buf(thread, 0);
    } else if (info->file->packetBytesWritten >= config.maxFileSizeB) {
        writer_simple_process_buf(thread, 1);
    }
}
/******************************************************************************/
```
## writer_simple_exit()
```c
LOCAL void writer_simple_exit()
{
    int thread;

    for (thread = 0; thread < config.packetThreads; thread++) {
        if (currentInfo[thread]) {
            writer_simple_process_buf(thread, 1);
        }
    }

    // Pause the main thread until our thread finishes
    while (writer_simple_queue_length() > 0) {
        usleep(10000);
    }

    for (thread = 0; thread < config.packetThreads; thread++) {
        for (int p = 0; p < INDEX_FILES_CACHE_SIZE; p++) {
            if (indexFiles[thread][p].fp) {
                fclose(indexFiles[thread][p].fp);
                indexFiles[thread][p].fp = 0;
            }
        }
    }
}
/******************************************************************************/

```
## moloch_config_int()
```c
uint32_t moloch_config_int(GKeyFile *keyfile, char *key, uint32_t d, uint32_t min, uint32_t max)
{
    char     *result;
    uint32_t  value = d;

    if (!keyfile)
        keyfile = molochKeyFile;

    if (config.override && keyfile == molochKeyFile && (result = g_hash_table_lookup(config.override, key))) {
        value = atol(result);
    } else if (g_key_file_has_key(keyfile, config.nodeName, key, NULL)) {
        value = g_key_file_get_integer(keyfile, config.nodeName, key, NULL);
    } else if (config.nodeClass && g_key_file_has_key(keyfile, config.nodeClass, key, NULL)) {
        value = g_key_file_get_integer(keyfile, config.nodeClass, key, NULL);
    } else if (g_key_file_has_key(keyfile, "default", key, NULL)) {
        value = g_key_file_get_integer(keyfile, "default", key, NULL);
    }

    if (value < min) {
        LOG ("INFO: Reseting %s since %u is less then the min %u", key, value, min);
        value = min;
    }
    if (value > max) {
        LOG ("INFO: Reseting %s since %u is greater then the max %u", key, value, max);
        value = max;
    }

    if (config.debug) {
        LOG("%s=%u", key, value);
    }

    return value;
}

/******************************************************************************/

```
## moloch_config_str()
```c
gchar *moloch_config_str(GKeyFile *keyfile, char *key, char *d)
{
    char *result;

    if (!keyfile)
        keyfile = molochKeyFile;

    if (config.override && keyfile == molochKeyFile && (result = g_hash_table_lookup(config.override, key))) {
        if (result[0] == 0)
            result = NULL;
        else
            result = g_strdup(result);
    } else if (g_key_file_has_key(keyfile, config.nodeName, key, NULL)) {
        result = g_key_file_get_string(keyfile, config.nodeName, key, NULL);
    } else if (config.nodeClass && g_key_file_has_key(keyfile, config.nodeClass, key, NULL)) {
        result = g_key_file_get_string(keyfile, config.nodeClass, key, NULL);
    } else if (g_key_file_has_key(keyfile, "default", key, NULL)) {
        result = g_key_file_get_string(keyfile, "default", key, NULL);
    } else if (d) {
        result = g_strdup(d);
    } else {
        result = NULL;
    }

    if (result)
        g_strstrip(result);

    if (config.debug) {
        LOG("%s=%s", key, result?result:"(null)");
    }

    return result;
}


```
## LOG(...)
```c
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
```



## writer_simple_thread
```c
LOCAL void *writer_simple_thread(void *UNUSED(arg))
{
    MolochSimple_t *info;

    if (config.debug)
        LOG("THREAD %p", (gpointer)pthread_self());

    while (1) 
    {
        MOLOCH_LOCK(simpleQ);
        while (DLL_COUNT(simple_, &simpleQ) == 0) 
        {
            MOLOCH_COND_WAIT(simpleQ);
        }
        DLL_POP_HEAD(simple_, &simpleQ, info);
        MOLOCH_UNLOCK(simpleQ);

        uint32_t pos = 0;
        uint32_t total = info->bufpos;
        if (info->closing) 
        {
            // Round up to next page size
            if (total % pageSize != 0)
                total = ((total/pageSize)+1)*pageSize;
        }

        switch(simpleMode) 
        {
        case MOLOCH_SIMPLE_NORMAL:
            break;
        case MOLOCH_SIMPLE_XOR2048: 
	        {
	            uint32_t i;
	            for (i = 0; i < total; i++)
	                info->buf[i] ^= info->file->dek[i % 256];
	            break;
	        }
        case MOLOCH_SIMPLE_AES256CTR: 
	        {
	            int outl;
	            if (!EVP_EncryptUpdate(info->file->cipher_ctx, (uint8_t *)info->buf, &outl, (uint8_t *)info->buf, total))
	                LOGEXIT("ERROR - Encrypting data failed");
	            if ((int)total != outl)
	                LOGEXIT("ERROR - Encryption in (%u) and out (%d) didn't match", total, outl);
	            break;
	        }
        }

        while (pos < total) 
        {
            int len = write(info->file->fd, info->buf + pos, total - pos);
            if (len >= 0) 
            {
                pos += len;
            } 
            else 
            {
                LOGEXIT("ERROR - writing %d %s", len, strerror(errno));
            }
        }
        if (info->closing) 
        {
            if (ftruncate(info->file->fd, info->file->pos) < 0 && config.debug)
                LOG("Truncate failed");
            close(info->file->fd);
            moloch_db_update_filesize(info->file->id, info->file->pos, info->file->packetBytesWritten, info->file->packets);
        }

        writer_simple_free(info);
    }
    return NULL;
}

```