## TCP

### 1. TCP 的三次握手 

#### 1.1 传输控制协议 TCP 简介

- 面向连接的、可靠的、基于字节流的传输层通信协议

- 将应用层的数据流分割成报文段并发送诶目标节点的 TCP 层

- 数据包都有序号，对方收到则发送 ACK 确认，未收到则重传

- 使用校验和来检验数据在传输过程中是否有误
    
题外话：

- 进程间通信：管道，内存共享，信号量，消息队列等

#### 1.2 TCP Flags

- URG：紧急指针标志

- **ACK：确认序号标志**

- PSH：push标志

- RST：重置连接标志

- **SYN：同步序号，用于建立连接过程**

- **ACK：finish 标志，用于释放连接**

题外话：

- TCP：全双工通信


#### 1.3 TCP 三次握手

```
+------------+ Client                                 Server  +------------+
|                                                                          |
|           +----------+                           +----------+            |
|主 动 打 开 |  CLOSED  +                           |  CLOSED  |被 动 打 开  |
+---------->----------+XXXX  ACK=1,seq=x           +-----------<-----------+
            |          +   XXXX                    |          |
            | SYN-SENT |       XXXXX               |  LISTEN  |
            |          |            XXXXXXXXXXX    |          |
            |          |                      XXXXXX----------+
            |          |SYN=1,ACK=1,seq=y,ack=x+1 XX          |
            |          |               XXXXXXXXXXXX+ SYN-RECV |
            |          |        XXXXXXX            |          |
            |          +  XXXXXX                   |          |
            +---------+XXX                         |          |
            |          +XX   ACK=1,seq=x+1,ack=y+1 |          |
            |  ESTAB-  |  XXX                      |          |
            |  LISHED  |     XXXXXX                |          |
            |          |          XXXXXXXXXXXX     |          |
            |          |                     XXXXXXX----------+
            |          |                           |          |
            |          |       data transfer       | ESTAB-   |
            |          +<------------------------->+ LISHED   |
            |          |                           |          |
            +----------+                           +----------+

```

- 在 TCP/IP 协议中，TCP 协议提供可靠的连接服务，采用三次握手建议一个连接。

- 第一次握手：建立连接时，客户端发送 SYN 包（SYN=1）到服务器，并进入 SYN_SENT 状态，等待服务器确认。

- 第二次握手：服务端收到 SYN 包，必须确认客户端的 SYN(ack=j+1)，同时自己也发送一个 SYN 包 (syn=k)，即
SYN + ACK 包，此时服务器进入SYN_RECV 状态

- 第三次握手：客户端收到服务器的 SYN + ACK 包，向服务器发送确认包 ACK（ack=k+1），此包发送完毕，
客户端和服务器进入 ESTABLISHED 状态，完成三次握手。

#### 1.4 为什么需要三次握手才能建立起连接

为了初始化 Sequence Number 的初始值

#### 1.5 首次握手的隐患 —— SYN 超时

1.5.1 问题起因分析

- Server 收到 Client 的 SYN，回复 SYN-ACK 的时候未收到 ACK 确认

- Server 不断重试直至超时，Linux 默认等待 63 秒（1+2+4+8+16+32=63s 默认5次重试包） 才能断开连接

1.5.2 针对 SYN Flood 的防护措施

- SYN 队列满后，通过 tcp_syncookies 参数回发 SYN Cookie

- 若为正常连接则 Client 会回发 SYN Cookie，直接建立连接

#### 1.6 建立连接后，Client 出现故障怎么办

1.6.1 保活机制

- 向对方发送保活探测报文，如果未收到响应则继续发送

- 尝试次数达到保活探测数 仍未收到响应则中断连接


### 2. TCP 的四次挥手 

#### 2.1 TCP 四次挥手

```
+------------+ Client                                 Server  ----------------+
|                                                             <------------+  |
|           +----------+                           +----------+            |  |            
|主 动 关 闭 |  ESTAB-  |                           |          |            |  | 
|           |  LISHED  |                           |          |            |  | 
+---------->+----------+XXXX  FIN=1,seq=u          |  ESTAB-  |            |  |
            |          +   XXXX                    |  LISHED  |            |  |
            |  FIN-    |       XXXXX               |          |            |  |
            |  WAIT-1  |            XXXXXXXXXXX    |          | 通知应用进程 |  |
            |          |                       XXXXX----------+------------+  |
            |          |  ACK=1,seq=v,ack=u+1    XX|          |               |
            |          |             XXXXXXXXXXXXX |          |               |
            |----------|XXXXXXXXXXXXX              | CLOSE-   |               |
            |          |       data transfer       |  WAIT    |               |
            |          +<------------------------->+          |               |
            |  FIN-    |  FIN=1,ACK=1,seq=w,ack=u+1|          |  被 动 关 闭   |
            |  WAIT-2  |                  XXXXXXXXXX----------+<--------------+
            |          |    XXXXXXXXXXXXXXX        |          |
|-----------+----------+XXXX                       |          |
| wait 2 MSL|          |  XX FIN=1,seq=u+1,ack=w+1 |  LAST-   |
|           |  TIME-   |   XX                      |   ACK    |
|           |  WAIT    |     XXXXXX                |          |
|           |          |          XXXXXXXXXXXX     |          |
|           |          |                     XXXXXXX----------+
|           |          |                           | CLOSED   |
+---------->+----------+                           +----------+
            |  CLOSED  |                           
            +----------+                                       

注：2 MSL，MSL 即 MaximumSegmentLifetime，一个数据分片（报文）在网络中能够生存的最长时间

RFC 定义 MSL= 2min
Linux 定义 MSL = 30s

```


TCP 采用四次挥手来释放连接

- 第一次挥手：Client 发送一个 FIN，用来关闭 Client 到 Server 的数据传输，Client 进入 FIN_WAIT_1 状态

- 第二次挥手：Server 收到 FIN 后，发送一个 ACK 给 Client，确认序列号为收到序列号 +1（与 SYN相同，一个FIN 占用一个序号），
Server 进入 CLOSE_WAIT 状态；

- 第三次挥手：Server 发送一个 FIN，用来关闭 Server 到 Client 的数据传输，Server 进入 LAST_ACK 状态；

- 第四次挥手：Client 收到 FIN 后，Client 进入 TIME_WAIT 状态，接着发送一个 ACK 给 Server，确认序列号为收到序列号+1，
Server 进入 CLOSED 状态，完成四次挥手。

#### 2.2 为什么有 TIME_WAIT 状态

- 确保有足够的时间让对方收到 ACK 包

- 避免新旧连接混淆

#### 2.3 为什么需要四次握手才能断开连接

因为全双工，发送方和接收方都需要 FIN 报文和 ACK 报文

#### 2.4 服务器出现大量 CLOSE_WAIT 状态的原因

- 对方关闭 socket 连接，我方忙于读或写，没有及时关闭连接

- 检查代码，特别是释放资源的代码

- 检查配置，特别是处理请求的线程配置

```shell
netstat -n | awk '/^TCP/{++S[$NF]}END{for(a in S) print a,S[a]}'

>>>
CLOSE_WAIT 6
FIN_WAIT_1 2
ESTABLISHED 28
SYN_SENT 3

$NF Filed 代表最后最后一列

Windows：
　　“/”是表示参数，“\”是表示本地路径。
Linux和Unix：
　　“/”表示路径，“\”表示转义，“-”和“--”表示参数。
网络：
　　由于网络使用Unix标准，所以网络路径用“/”。　　


连接太多最终可能会报：
too many open files
```

## 感谢

...