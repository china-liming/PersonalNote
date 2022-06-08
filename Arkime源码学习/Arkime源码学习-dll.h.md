[toc]
## 注释说明
```c
 dll.h  -- Double Linked List
 *
 * Simple macro based DLL with counts that supports having a single item in
 * multiple lists by using a prefix.  Every element in the list needs to 
 * have a [prefix]next and [prefix]prev elements.  The list head needs to have
 * a [prefix]count element.  
 *
 * WARNING: If using a seperate head structure AND a single item is in multiple 
 * lists the byte position in the structure of the next and prev elements MUST be
 * the same between the head structure and item structures.
dll.h--双链表
简单的基于宏的DLL，其计数支持通过使用前缀在多个列表中包含单个项。
列表中的每个元素都需要有[prefix]next和[prefix]prev元素。列表头需要有[prefix]计数元素。
警告：如果使用单独的头结构，且单个项位于多个列表中，
则头结构和项结构之间的next和prev元素在结构中的字节位置必须相同。
```

## DLL_INIT
```c
#define DLL_INIT(name,head) \
    ((head)->name##count = 0, \
     (head)->name##next = (head)->name##prev = (void *)head \
    )
```
假设这样调用：
```c
DLL_INIT(s_, &certs->alt);
```
会扩展到：

```c
(
	(&certs->alt)->s_count = 0, 
	(&certs->alt)->s_next = (&certs->alt)->s_prev = (void *)&certs->alt 
)
```
即创建了一个双链表。
## DLL_PUSH_TAIL
- 将一个新的节点加入到链表的尾部
```c
#define DLL_PUSH_TAIL(name,head,element) \
    ((element)->name##next          = (void *)(head), \
     (element)->name##prev          = (head)->name##prev, \
     (head)->name##prev->name##next = (element), \
     (head)->name##prev             = (element), \
     (head)->name##count++ \
    )
```
示例：
```c
DLL_PUSH_TAIL(s_, &certs->alt, element);
```
扩展为：
```c
(
	(element)->s_next = (void *)(&certs->alt), 
	(element)->s_prev = (&certs->alt)->s_prev, 
	(&certs->alt)->s_prev->s_next = (element), 
	(&certs->alt)->s_prev = (element),
	(&certs->alt)->s_count++ 
)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e42e0ce463f3444287be6b46f66ef71d.png)

## DLL_PUSH_TAIL_DLL

```c
#define DLL_PUSH_TAIL_DLL(name,head1,head2) \
    ((head1)->name##prev->name##next = (head2)->name##next, \
     (head2)->name##next->name##prev = (head1)->name##prev, \
     (head1)->name##prev             = (head2)->name##prev, \
     (head2)->name##prev->name##next = (void *)(head1), \
     (head1)->name##count+=(head2)->name##count, \
     DLL_INIT(name,head2) \
    )
```

```c
	DLL_PUSH_TAIL_DLL(packet_, &packetQ[t], &batch->packetQ[t]);
```

```c
(
		(&packetQ[t])->packet_prev->packet_next = (&batch->packetQ[t])->packet_next, 
		(&batch->packetQ[t])->packet_next->packet_prev = (&packetQ[t])->packet_prev, 
		(&packetQ[t])->packet_prev = (&batch->packetQ[t])->packet_prev, 
		(&batch->packetQ[t])->packet_prev->packet_next = (void *)(&packetQ[t]), 
		(&packetQ[t])->packet_count+=(&batch->packetQ[t])->packet_count, 
		DLL_INIT(packet_,&batch->packetQ[t]) 
)
```

## DLL_PUSH_HEAD

```c
#define DLL_PUSH_HEAD(name,head,element) \
    ((element)->name##next          = (head)->name##next, \
     (element)->name##prev          = (void *)(head), \
     (head)->name##next->name##prev = (element), \
     (head)->name##next             = (element), \
     (head)->name##count++ \
    )
```

## DLL_ADD_AFTER

```c
#define DLL_ADD_AFTER(name,head,after,element) \
    ((element)->name##next            = (after)->name##next, \
     (element)->name##prev            = (after), \
     (after)->name##next->name##prev  = (element), \
     (after)->name##next              = (element), \
     (head)->name##count++ \
    )
```
## DLL_ADD_BEFORE
```c
#define DLL_ADD_BEFORE(name,head,before,element) \
    ((element)->name##next            = (before), \
     (element)->name##prev            = (before)->name##prev, \
     (before)->name##prev             = (element), \
     (before)->name##prev->name##next = (element), \
     (head)->name##count++ \
    )
```

## DLL_REMOVE

```c
#define DLL_REMOVE(name,head,element) \
    ((element)->name##prev->name##next = (element)->name##next, \
     (element)->name##next->name##prev = (element)->name##prev, \
     (element)->name##prev             = 0, \
     (element)->name##next             = 0, \
     (head)->name##count-- \
    )
```
### 实例

```c
// drophash.c --192
DLL_REMOVE(dhg_, group, item);
//扩展到
(
	(item)->dhg_prev->dhg_next = (item)->dhg_next, 
	(item)->dhg_next->dhg_prev = (item)->dhg_prev, 
	(item)->dhg_prev = 0, (item)->dhg_next = 0, 
	(group)->dhg_count-- 
)
```

## DLL_MOVE_TAIL
```c
#define DLL_MOVE_TAIL(name,head,element) \
    ((element)->name##prev->name##next = (element)->name##next, \
     (element)->name##next->name##prev = (element)->name##prev, \
     (element)->name##next             = (void *)(head), \
     (element)->name##prev             = (head)->name##prev, \
     (head)->name##prev->name##next    = (element), \
     (head)->name##prev                = (element) \
    )
```

## DLL_POP_HEAD
### 描述
如果name双链表为空，element赋值为空，否则，element 赋值为头节点的下一个值，并且删除第一个元素。

```c
#define DLL_POP_HEAD(name, head, element) \
    ((head)->name##count == 0 ? ((element) = NULL, 0) : ((element) = (head)->name##next, DLL_REMOVE(name, (head), (element)), 1))

```

### 实例
```c
DLL_POP_HEAD(simple_, &simpleQ, info); --writer-simple.c --558
扩展到: 
(
	(&simpleQ)-> simple_count == 0 ? ((info) = ((void *)0),0) : ((info) = (&simpleQ)->simple_next, 
	(((info))->- simple. prev->simple_ next = (info))-> simple next, 
	(info))-> simple_ next- > simple_ prev = (info))-> simple_ prev, 
	(info))->simple_ prev = 0, 
	(nfo))> simple_ next = 0, 
	(&simpleQ))-> simple.count--), 1)
)


```