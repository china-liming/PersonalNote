[toc]
# moloch-stats
## 简介
Simple thread that just calculates all the stats occasionally and sends to ES.
简单的线程，只是偶尔计算所有的统计数据并发送给ES。

实例
仅出现一处：db.c --
```c
g_thread_unref(g_thread_new("moloch-stats", &moloch_db_stats_thread, NULL));
```

# moloch-pkt i
```c
packet.c --1624
snprintf(name, sizeof(name), "moloch-pkt%d", t);
#ifndef FUZZLOCH
g_thread_unref(g_thread_new(name, &moloch_packet_thread, (gpointer)(long)t));
#endif
```

# moloch-daq i
```c
//reader-daq.c --112
char name[100];
snprintf(name, sizeof(name), "moloch-daq%d", i);
g_thread_unref(g_thread_new(name, &reader_daq_thread, NULL));
```

# moloch-pcap i
```c
reader.c --112
char name[100];
snprintf(name, sizeof(name), "moloch-pcap%d", i);
g_thread_unref(g_thread_new(name, &reader_libpcap_thread, (gpointer)(long)i));
```

# moloch-pfring i
```c
reader-snf.c  --133
char name[100];
        snprintf(name, sizeof(name), "moloch-pfring%d", i);
        g_thread_unref(g_thread_new(name, &reader_pfring_thread, (gpointer)(long)i));
```

# moloch-snf i-r
```c
//reader-snf.c  --133
char name[100];
snprintf(name, sizeof(name), "moloch-snf%d-%d", i, r);
g_thread_unref(g_thread_new(name, &reader_snf_thread, (gpointer)(long)(i | r << 8)));
  
```
# moloch-af3 i-t
```c
reader-tpacketv3.c --178
snprintf(name, sizeof(name), "moloch-af3%d-%d", i, t);
g_thread_unref(g_thread_new(name, &reader_tpacketv3_thread, (gpointer)(long)i));

```

# moloch-simple学习
## 简介
moloch-simple
A single thread that is responsible for writing out to disk the completed pcap buffers.
一个线程，负责将完成的pcap缓冲区写入磁盘。

实例
moloch-simple 仅出现一处，在writer-simple.c中的writer_simple_init中：
[[Arkime源码学习-writer-simple.c#writer_simple_init]]















