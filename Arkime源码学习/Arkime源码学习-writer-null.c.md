## 简介
writer-null.c  -- writer that doesn't do anything
## 源码
```c
#define _FILE_OFFSET_BITS 64
#include "moloch.h"
#include <errno.h>
#include <fcntl.h>
#include <inttypes.h>
#include <pthread.h>
#include <sys/stat.h>
#include <sys/mman.h>

extern MolochConfig_t        config;

LOCAL  uint64_t              outputFilePos = 24;

/******************************************************************************/
LOCAL uint32_t writer_null_queue_length()
{
    return 0;
}
/******************************************************************************/
LOCAL void writer_null_exit()
{
}
/******************************************************************************/
LOCAL void writer_null_write(const MolochSession_t * const UNUSED(session), MolochPacket_t * const packet)
{
    packet->writerFileNum = 0;
    packet->writerFilePos = outputFilePos;
    outputFilePos += 16 + packet->pktlen;
}
/******************************************************************************/
void writer_null_init(char *UNUSED(name))
{
    moloch_writer_queue_length = writer_null_queue_length;
    moloch_writer_exit         = writer_null_exit;
    moloch_writer_write        = writer_null_write;
}

```