## select、poll、epoll

### 简介

select , poll , epoll 都是 IO多路复用 的机制。I/O多路复用 就是通过一种机制，一个进程可以
监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。
但 select , poll , epoll 本质上都是 同步I/O，因为它们都需要在读写事件就绪后自己负责进行读写，
也就是说这个读写过程都是阻塞的，而 异步I/O 无需自己负责进行读写，异步I/O 的实现会负责把数据从
内核拷贝到用户空间。



### 提纲

#### 1. select

select 函数监视的文件描述符分 3类，分别是 writefds、readfds 和 exceptfds。调用后 select函数
会阻塞，直到有描述符就绪（有数据 可读、可写、或者有 except），或者超时（timeout 指定等待时间，如果
立即返回设为 null 即可），函数返回。当 select 函数返回后，可以通过遍历 fdset，来找到就绪的描述符。

select 目前几乎在所有的平台上支持，其良好跨平台支持也是它的一个有点。select 的一个缺点在于单个进程
能够监视的文件描述符存在最大限制，在 Linux 上一般为1024，可以通过修改宏定义甚至重新编译内核的方式，
提升这一限制，但是这一也会造成效率的降低。

#### 2. poll

不同于 select 使用三个位图来表示三个 fdset 的方式，poll 使用一个 pollfd 的指针实现。

pollfd 结构包含了要监视的 event 和发生的 event，不要再使用 “参数-值” 传递的方式。同时，
pollfd 并没有最大数量限制（但是数量过大后性能也是会下降）。和 select 函数一样，poll 返回后，
需要轮询 pollfd 来获取就绪的描述符。

从上面看，select 和 poll 都需要在返回后，通过遍历文件描述符来获取已经就绪的 socket。事实上，
同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率
也会线性下降。

#### 3. epoll

epoll 是在 2.6 内核中提出的，是之前的 select 和 poll 的增强版本。相对于 select 和 poll 来说，
epoll 更加灵活，没有描述符限制。epoll 使用一个文件描述符管理多个描述符，将用户关系的文件描述符的
事件存放到内核的一个事件表中，这样在用户空间和内核空间的 copy 只需一次。

题外话，由上面特性，我们可以看出：

- epoll 适用于 web服务器（一旦建立连接，活跃度低）
- select 适用于 游戏服务器（一旦建立连接，活跃度高）


###  epoll 详细工作原理

epoll是一种IO多路复用技术，可以非常高效的处理数以百万计的socket句柄，比起以前的select和poll效率高大发了。
它到底为什么可以高速处理这么多并发连接呢？

C库封装的3个epoll系统调用。

```cpp
int epoll_create(int size);  
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);  
int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout);  
```

使用起来很清晰，首先要调用 epoll_create 建立一个 epoll 对象，参数 size 是内核保证能够正确处理的
最大句柄数，多于这个最大树时内核可不保证效果。

epoll_ctl 可以操作上面建立 epoll，例如，将刚建立的 socket 加入到 epoll 中让其监控，或者把 epoll
正在 监控的某个 socket 句柄移出 epoll，不再监控它等等。

epoll_wait 在调用时，在给定的 timeout 时间内，当在监控的所有句柄中有事件发生时，就返回用户态的进程。


从上面的调用方式就可以看到 epoll 比 select/poll 的优越之处：因为后者每次调用时都要传递你所要监控的
所有 socket 给 select/poll 系统调用，这意味着需要将用户态的 socket 列表 copy 到内核态，如果以万计
的句柄会导致每次都要 copy 几十几百KB的内存到内核态，非常低效。而我们调用 epoll_wait 时就相当于以往调用 select/poll，
但是这时却不用传递 socket 句柄给内核，因为内核已经在 epoll_ctl 中拿到了要监控的句柄列表。

所以，实际上在调用 epoll_create 后，内核就已经在内核态开始准备帮你存储要监控的句柄了，每次调用 epoll_ctl 只是
在往内核的数据结构里塞入新的 socket 句柄。

在内核里，一切皆文件。所以，epoll 向内核注册了一个文件系统，用于存储上述的被监控 socket。当你调用 epoll_create时，
就会在这个虚拟的 epoll 文件系统里创建一个 file 节点。当然这个 file 不是普通文件，它只服务与 epoll。

epoll 在被内核初始化时（操作系统启动），同时会开辟出 epoll 自己的内核高速 cache 区，用于安置每一个我们想
监控的 socket，这些 socket 会以红黑树的形式保存在内核 cache 里，以支持快速的查找、插入、删除。这个内核高速
cache 区，就是建立连续的物理内存页，然后在之上建立 slab 层，简单地说，就是物理上分配好你想要的 size 的内存对象，
每次使用时都是使用空闲的已分配好的对象。


```cpp
static int __init eventpoll_init(void)  
{  
    ... ...  
  
    /* Allocates slab cache used to allocate "struct epitem" items */  
    epi_cache = kmem_cache_create("eventpoll_epi", sizeof(struct epitem),  
            0, SLAB_HWCACHE_ALIGN|EPI_SLAB_DEBUG|SLAB_PANIC,  
            NULL, NULL);  
  
    /* Allocates slab cache used to allocate "struct eppoll_entry" */  
    pwq_cache = kmem_cache_create("eventpoll_pwq",  
            sizeof(struct eppoll_entry), 0,  
            EPI_SLAB_DEBUG|SLAB_PANIC, NULL, NULL);  
  
 ... ...  
```

epoll的高效就在于，当我们调用 epoll_ctl 往里塞入百万个句柄时，epoll_wait 仍然可以飞快的返回，并有效的将发生事件
的句柄给我们用户。这是由于我们在调用 epoll_create 时，内核除了帮我们在 epoll 文件系统里建了个 file 结点，在内核 cache
里建了个红黑树用于存储以后 epoll_ctl 传来的 socket 外，还会再建立一个 list链表，用于存储准备就绪的事件，当 epoll_wait 调用时，
仅仅观察这个 list链表 里有没有数据即可。有数据就返回，没有数据就 sleep ，等到 timeout 时间到后即使链表没数据也返回。所以，
epoll_wait 非常高效。


而且，通常情况下即使我们要监控百万计的句柄，大多一次也只返回很少量的准备就绪句柄而已，所以，epoll_wait 仅需要从内核态
copy 少量的句柄到用户态而已，因此相当高效。

那么，这个准备就绪 list链表 是怎么维护的呢？当我们执行 epoll_ctl 时，除了把 socket 放到 epoll文件系统 里 file 对象对应的红黑树上之外，
还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪 list链表 里。
所以，当一个 socket 上有数据到了，内核在把网卡上的数据 copy 到内核中后，就来把 socket 插入到准备就绪链表里了。

如此，一颗红黑树，一张准备就绪句柄链表，少量的内核 cache，就帮我们解决了大并发下的 socket 处理问题。
执行 epoll_create 时，创建了红黑树和就绪链表，执行 epoll_ctl 时，如果增加 socket 句柄，则检查在红黑树中是否存在，
存在立即返回，不存在则添加到树干上，然后向内核注册回调函数，用于当中断事件来临时向准备就绪链表中插入数据。
执行 epoll_wait 时立刻返回准备就绪链表里的数据即可。

最后看看 epoll 独有的两种模式 LT 和 ET 。无论是 LT 和 ET 模式，都适用于以上所说的流程。区别是，LT模式下，
只要一个句柄上的事件一次没有处理完，会在以后调用 epoll_wait 时次次返回这个句柄，而ET模式仅在第一次返回。

这件事怎么做到的呢？当一个 socket 句柄上有事件时，内核会把该句柄插入上面所说的准备就绪 list链表，这时我们调用 epoll_wait，
会把准备就绪的 socket 拷贝到用户态内存，然后清空准备就绪 list链表，最后，epoll_wait 干了件事，就是检查这些 socket，
如果不是ET模式（就是LT模式的句柄了），并且这些 socket 上确实有未处理的事件时，又把该句柄放回到刚刚清空的准备就绪链表了。
所以，非ET的句柄，只要它上面还有事件，epoll_wait 每次都会返回。而ET模式的句柄，除非有新中断到，即使 socket 上的事件没有处理完，
也是不会次次从 epoll_wait 返回的。

题外话：

- LT(level triggered) 是缺省的工作方式，并且同时支持 block 和 no-block socket.在这种做法中，内核告诉你一个文件描述符是否就绪了，
然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的，所以，这种模式编程出错误可能性要小一点。
传统的 select/poll 都是这种模型的代表。

- ET (edge-triggered) 是高速工作方式，只支持 no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过 epoll 告诉你。
然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了，
但是请注意，如果一直不对这个 fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once)。


## 感谢

[epoll和select区别](https://blog.csdn.net/ysu108/article/details/7570571)

[epoll 详细工作原理](https://blog.csdn.net/hdutigerkin/article/details/7517390)

[linux 内核poll/select/epoll实现剖析](https://blog.csdn.net/lishenglong666/article/details/45536611)
