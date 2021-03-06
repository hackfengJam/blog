## Redis 为什么这么快？


### 1. 纯内存操作，肯定快

数据存储在内存中，读取的时候不需要进行磁盘的 IO

### 2. 单线程，无锁竞争损耗

单线程保证了系统没有线程的上下文切换

使用单线程，可以避免不必要的上下文切换和竞争条件，没有多进程或多线程引起的切换和 CPU 的消耗，不必考虑各种锁的问题，没有锁释放或锁定操作，不会因死锁而降低性能；

### 3. C 语言实现，更接近底层操作

Redis 是用 C 语言开发完成的

### 4. 多路 I/O 复用模型，非阻塞 IO

采用多路 I/O 复用技术可以让单个线程高效的处理多个网络连接请求（尽量减少网络 IO 的时间消耗）

非阻塞 IO 内部实现采用 epoll，采用了 epoll+自己实现的简单的事件框架。epoll 中的读、写、关闭、连接都转化成了事件，然后利用 epoll 的多路复用特性，绝不在 io 上浪费一点时间。

### 5. 数据结构简单，底层又做了优化

数据结构简单，对数据操作也简单，Redis 中的数据结构是专门进行设计的；

### 6. 源码精湛、简短

扩展：
在 Redis 中，常用的 5 种数据结构和应用场景
String： 缓存、计数器、分布式锁等。
List： 链表、队列、微博关注人时间轴列表等。
Hash： 用户信息、Hash 表等。
Set： 去重、赞、踩、共同好友等。
Zset： 访问量排行榜、点击量排行榜等。
多路 I/O 复用模型
多路 I/O 复用模型是利用 select、poll、epoll 可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有 I/O 事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（epoll 是只轮询那些真正发出了事件的流），并且只依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。

这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。

采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络 IO 的时间消耗），且 Redis 在内存中操作数据的速度非常快，也就是说内存内的操作不会成为影响 Redis 性能的瓶颈，主要由以上几点造就了 Redis 具有很高的吞吐量。


### 感谢 

- [Redis 为什么这么快？](https://blog.csdn.net/qq_22871083/article/details/104511778)

