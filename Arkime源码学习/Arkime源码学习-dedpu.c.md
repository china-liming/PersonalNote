## 注释
```
dedpu --去重；重复数据删除；删除备份

Circular array of chained hashtable where space is preallocated and there is a count array of elements per hashtable slot. 
链式散列表的循环数组，其中空间是预先分配的，每个散列表槽有一个元素计数数组。

We stop searching the oldest hashtable and wipe its count array.
 我们停止搜索最老的哈希表并擦除它的计数数组。
 
We size the hashtable assuming DEDUP_SLOT_FACTOR elements per slot, but actually allow DEDUP_SIZE_FACTOR elements.
我们假设每个 slot 有 DEDUP_slot_factor 元素，但是实际上允许 DEDUP_size_factor 元素。

Currently use md5 on ip + tcp/udp hdr
```


## arkime_dedup_init
```c
void arkime_dedup_init()
{
    if (!config.enablePacketDedup)
        return;

    dedupSeconds   = moloch_config_int(NULL, "dedupSeconds", 2, 0, 30) + 1; // + 1 because a slot isn't active before being replaced
    dedupPackets   = moloch_config_int(NULL, "dedupPackets", 0xfffff, 0xffff, 0xffffff);
    dedupSlots     = moloch_get_next_prime(dedupPackets/DEDUP_SLOT_FACTOR);
    dedupSize      = dedupSlots * DEDUP_SIZE_FACTOR;

    if (config.debug)
        LOG("seconds = %u packets = %u slots = %u size = %u mem=%u", dedupSeconds, dedupPackets, dedupSlots, dedupSize, dedupSeconds *(dedupSlots + dedupSize*16));

    seconds = MOLOCH_SIZE_ALLOC0("dedup seconds", sizeof(DedupSeconds_t) * dedupSeconds);
    for (uint32_t i = 0; i < dedupSeconds; i++) {
        MOLOCH_LOCK_INIT(seconds[i].lock);
        seconds[i].counts = MOLOCH_SIZE_ALLOC("dedup counts", dedupSlots);
        seconds[i].md5s = MOLOCH_SIZE_ALLOC("dedup counts", dedupSize * 16);
    }
}

```