Capture is a multithreaded glib2 application
Capture是一个多线程的glib2应用程序
# Threads

In general capture tries to not use locks for anything but queues when communicating between threads.
When possible we use read only complex data structures shared across threads.
When those data structures need to be updated we create a new one and replace the old one, which is scheduled to be freed at a later time (moloch_free_later) so any current readers don't crash.

通常，在线程之间通信时，capture会尝试对队列以外的任何对象使用锁。
如果可能，我们使用线程间共享的只读复杂数据结构。
当这些数据结构需要更新时，我们会创建一个新的数据结构并替换旧的数据结构，旧的数据结构计划稍后释放（moloch_free_later），这样当前的任何读卡器都不会崩溃。

## capture
The main thread, all http requests are on the main thread.  主线程，所有http请求都在主线程上。
Since sessions aren't locked, any sessions actions need to be added to the packet threads. 

由于会话没有被锁定，任何会话操作都需要添加到数据包线程中。

## moloch-stats
Simple thread that just calculates all the stats occasionally and sends to ES.
简单的线程，只是偶尔计算所有的统计数据并发送给ES。
## moloch-pkt##
The packet threads controlled by packetThreads. 由packetThreads控制的数据包线程。
These threads are responsible for processing packets that are passed to it in batches. 这些线程负责处理分批传递给它的数据包。
Sessions are hashed across packet threads and all packets are processed by where the session is.
会话在数据包线程之间进行散列，所有数据包都按会话所在的位置进行处理。
Any operations to a session has to happen in the packet thread since sessions don't have locks.
对会话的任何操作都必须在数据包线程中进行，因为会话没有锁。
Use moloch_session_add_cmd to schedule a session task from a different thread.
使用moloch_session_add_cmd从不同线程调度会话任务。

## moloch-pcap#
When using the libpcap reader a thread is created for each interface.  使用libpcap读取器时，会为每个接口创建一个线程。
These threads are responsible for reading in the packets and batch adding them to the packet threads.
这些线程负责读入数据包并将其批量添加到数据包线程中。

## moloch-af3#-#
When using the afpacket reader a thread is created for each interface * tpacketv3NumThreads  
使用afpacket reader时，会为每个接口创建一个线程*tpacketv3NumThreads
These threads are responsible for reading in the packets and batch adding them to the packet threads.
这些线程负责读入数据包并将其批量添加到数据包线程中。

## moloch-simple
A single thread that is responsible for writing out to disk the completed pcap buffers.
一个线程，负责将完成的pcap缓冲区写入磁盘。

# Files

# Parsers vs Plugins

In reality there isn't much difference between parsers and plugins, other than when they are loaded and when they are initialized.
实际上，解析器和插件之间没有太大区别，除了加载和初始化的时间。

## Parsers
Anything in the parsers directories (parsersDir) are auto loaded and the moloch_parser_init function is called when loaded.
If files have the same in multiple directories, capture will load the first one found.
parsers目录（parsersDir）中的任何内容都是自动加载的，加载时会调用moloch_parser_init函数。
如果多个目录中有相同的文件，capture将加载找到的第一个。
## Plugins
Which plugins to use have to be explicitly listed in rootPlugins and plugins variables.  
使用哪些插件必须在rootPlugins和plugins变量中明确列出。
They are loaded from the plugins directories (pluginsDir) and the moloch_plugin_init function is called when loaded.
它们从plugins目录（pluginsDir）加载，加载时调用moloch_plugin_init函数。
If files have the same in multiple directories, capture will load the first one found.
如果多个目录中有相同的文件，capture将加载找到的第一个。
The rootPlugins are loaded first, before capture has dropped privileges.
在capture放弃特权之前，首先加载根插件。
The normal plugins are loaded after the parsers.
普通插件在解析器之后加载。

# Creating new parsers
创建新的解析器

Packets have two phases of life and many parsers will need to deal with both phases.
数据包有两个生命阶段，许多解析器将需要处理这两个阶段。
The first phase runs on the reader thread and should do very basic parsing and validation of the packet.
第一阶段在读卡器线程上运行，应该对数据包进行非常基本的解析和验证。
The second phase runs on the packet thread and does whatever decoding and SPI data generation needs to be done.
第二阶段在包线程上运行，执行需要执行的任何解码和SPI数据生成。
## Ethernet/IP Enqueue phase
Ethernet/IP Enqueue phase
以太网/IP排队阶段

This phase is responsible for   这个阶段负责
* basic decoding and verification of the packet  数据包的基本解码和验证
* setting the `mProtocol` field with the moloch protocol  使用moloch协议设置“mProtocol”字段
* setting the `hash` field with the hash of the session id  使用会话id的哈希设置“hash”字段

You only need to create a new enqueue callback for special ethernet and ip protocols, which can be set with the moloch_packet_set_ethernet_cb and moloch_packet_set_ip_cb..
您只需要为特殊的以太网和ip协议创建一个新的排队回调，可以使用moloch_packet_set_ethernet_cb和moloch_packet_set_ip_cb进行设置。。
Normal TCP/UDP traffic should NOT set an enqueue callback.
正常TCP/UDP通信不应设置排队回调。

## Ethernet/IP Process phase
以太网/IP处理阶段

This phase is responsible for actually processing the packets and generating the SPI data.
该阶段负责实际处理数据包并生成SPI数据。
You only need to create new process callbacks for special ethernet and ip protocols.
您只需要为特殊的以太网和ip协议创建新的进程回调。
The callbacks are set with the moloch_mprotocol_register
回调是用moloch_mprotocol_register设置的

moloch_mprotocol_register (char *name, int ses, create_session_id, pre_process, process)

* name - the name of this protocol 名称-此协议的名称
* ses - the SESSION_* type, usually SESSION_OTHER  ses——会话类型，通常是会话类型
* create_session_id - required - Given a packet, fill in the session id  必需-给定数据包，填写会话id
* pre_process - required - called before saving/rules. Given the session, packet, isNewSession - can be used to set any initial SPI data fields or packet direction
必需-保存前调用/规则。给定会话，packet，isNewSession-可用于设置任何初始SPI数据字段或数据包方向
* process - optional - called after saving to disk and rules.  Should generate most of the SPI data or enqueue for higher level protocol decoding.  Returns if the packet should be freed or not yet.
process - 可选-保存到磁盘和规则后调用。应生成大部分SPI数据或排队以进行更高级别的协议解码。返回是否应释放数据包。
* free - optional - called when the session is being freed 可选-在释放会话时调用

## TCP/UDP parsing
TCP/UDP解析

TCP/UDP parsing and classification is a two step process.  TCP/UDP解析和分类是一个两步过程。
* Classify - Look
* Parsing - 


moloch_parsers_register2
define moloch_parsers_register

#define moloch_parsers_classifier_register_tcp
#define moloch_parsers_classifier_register_udp

#define moloch_parsers_classifier_register_port
