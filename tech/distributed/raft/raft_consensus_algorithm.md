## Raft 一致性算法

### 前言

本文内容几乎全部摘自 

- [Raft 英文 paper pdf](https://ramcloud.atlassian.net/wiki/download/attachments/6586375/raft.pdf)

- [Raft 论文译文](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)

paper 写的比较完备，无可挑剔 : )

### 目录

[1. Raft 复制状态机](#1-Raft-复制状态机)

[2. 关于 Raft 一致性算法的浓缩总结](#2-关于-Raft-一致性算法的浓缩总结)

[3. Raft 在任何时候都保证以下的各个特性](#3-Raft-在任何时候都保证以下的各个特性)


### 1. Raft 复制状态机

![图 1 ](./images/raft_consensus_algorithm_1.png)

> 图 1 ：复制状态机的结构。一致性算法管理着来自客户端指令的复制日志。状态机从日志中处理相同顺序的相同指令，所以产生的结果也是相同的。

复制状态机通常都是基于复制日志实现的，如图 1。每一个服务器存储一个包含一系列指令的日志，并且按照日志的顺序进行执行。
每一个日志都按照相同的顺序包含相同的指令，所以每一个服务器都执行相同的指令序列。
因为每个状态机都是确定的，每一次执行操作都产生相同的状态和同样的序列。

保证复制日志相同就是一致性算法的工作了。
在一台服务器上，一致性模块接收客户端发送来的指令然后增加到自己的日志中去。
它和其他服务器上的一致性模块进行通信来保证每一个服务器上的日志最终都以相同的顺序包含相同的请求，尽管有些服务器会宕机。
一旦指令被正确的复制，每一个服务器的状态机按照日志顺序处理他们，然后输出结果被返回给客户端。
因此，服务器集群看起来形成一个高可靠的状态机。

实际系统中使用的一致性算法通常含有以下特性：

* 安全性保证（绝对不会返回一个错误的结果）：在非拜占庭错误情况下，包括网络延迟、分区、丢包、冗余和乱序等错误都可以保证正确。  
* 可用性：集群中只要有大多数的机器可运行并且能够相互通信、和客户端通信，就可以保证可用。因此，一个典型的包含 5 个节点的集群可以容忍两个节点的失败。服务器被停止就认为是失败。他们当有稳定的存储的时候可以从状态中恢复回来并重新加入集群。  
* 不依赖时序来保证一致性：物理时钟错误或者极端的消息延迟只有在最坏情况下才会导致可用性问题。  
* 通常情况下，一条指令可以尽可能快的在集群中大多数节点响应一轮远程过程调用时完成。小部分比较慢的节点不会影响系统整体的性能。  

### 2. 关于 Raft 一致性算法的浓缩总结（不包括成员变换和日志压缩）

> Raft 是一种用来管理[Raft 复制状态机](#1-Raft-复制状态机)中描述的复制日志的算法。

### 2.1 状态

|状态|所有服务器上持久存在的|
|-------|------|
|currentTerm | 服务器最后一次知道的任期号（初始化为 0，持续递增）|
|votedFor | 在当前获得选票的候选人的 Id|
| log[] | 日志条目集；每一个条目包含一个用户状态机执行的指令，和收到时的任期号 |

|状态|所有服务器上经常变的|
|-------|------|
| commitIndex| 已知的最大的已经被提交的日志条目的索引值|
| lastApplied| 最后被应用到状态机的日志条目索引值（初始化为 0，持续递增）|

| 状态 | 在领导人里经常改变的 （选举后重新初始化）|
|----|--------|
| nextIndex[] | 对于每一个服务器，需要发送给他的下一个日志条目的索引值（初始化为领导人最后索引值加一）|
| matchIndex[] | 对于每一个服务器，已经复制给他的日志的最高索引值|


### 2.2 附加日志 RPC

由领导人负责调用来复制日志指令；也会用作 heartbeat  

| 参数 | 解释 |
|----|----|
|term| 领导人的任期号|
|leaderId| 领导人的 Id，以便于跟随者重定向请求|
|prevLogIndex|新的日志条目紧随之前的索引值|
|prevLogTerm|prevLogIndex 条目的任期号|
|entries[]|准备存储的日志条目（表示心跳时为空；一次性发送多个是为了提高效率）|
|leaderCommit|领导人已经提交的日志的索引值|

| 返回值| 解释|
|---|---|
|term|当前的任期号，用于领导人去更新自己|
|success|跟随者包含了匹配上 prevLogIndex 和 prevLogTerm 的日志时为真|

接收者实现：

1. 如果 `term < currentTerm` 就返回 false  
2. 如果日志在 prevLogIndex 位置处的日志条目的任期号和 prevLogTerm 不匹配，则返回 false  
3. 如果已经存在的日志条目和新的产生冲突（索引值相同但是任期号不同），删除这一条和之后所有的  
4. 附加日志中尚未存在的任何新条目  
5. 如果 `leaderCommit > commitIndex`，令 commitIndex 等于 leaderCommit 和 新日志条目索引值中较小的一个  

### 2.3 请求投票 RPC

由候选人负责调用用来征集选票

| 参数 | 解释|
|---|---|
|term| 候选人的任期号|
|candidateId| 请求选票的候选人的 Id |
|lastLogIndex| 候选人的最后日志条目的索引值|
|lastLogTerm| 候选人最后日志条目的任期号|

| 返回值| 解释|
|---|---|
|term| 当前任期号，以便于候选人去更新自己的任期号|
|voteGranted| 候选人赢得了此张选票时为真|

接收者实现：

1. 如果`term < currentTerm`返回 false  
2. 如果 votedFor 为空或者为 candidateId，并且候选人的日志至少和自己一样新，那么就投票给他  

### 2.4 所有服务器需遵守的规则

所有服务器：

* 如果`commitIndex > lastApplied`，那么就 lastApplied 加一，并把`log[lastApplied]`应用到状态机中
* 如果接收到的 RPC 请求或响应中，任期号`T > currentTerm`，那么就令 currentTerm 等于 T，并切换状态为跟随者

跟随者：

* 响应来自候选人和领导者的请求
* 如果在超过选举超时时间的情况之前都没有收到领导人的心跳，或者是候选人请求投票的，就自己变成候选人

候选人：

* 在转变成候选人后就立即开始选举过程
	* 自增当前的任期号（currentTerm）
	* 给自己投票
	* 重置选举超时计时器
	* 发送请求投票的 RPC 给其他所有服务器
* 如果接收到大多数服务器的选票，那么就变成领导人
* 如果接收到来自新的领导人的附加日志 RPC，转变成跟随者
* 如果选举过程超时，再次发起一轮选举

领导人：

*  一旦成为领导人：发送空的附加日志 RPC（心跳）给其他所有的服务器；在一定的空余时间之后不停的重复发送，以阻止跟随者超时
*  如果接收到来自客户端的请求：附加条目到本地日志中，在条目被应用到状态机后响应客户端
*  如果对于一个跟随者，最后日志条目的索引值大于等于 nextIndex，那么：发送从 nextIndex 开始的所有日志条目：
	* 如果成功：更新相应跟随者的 nextIndex 和 matchIndex
	* 如果因为日志不一致而失败，减少 nextIndex 重试
* 如果存在一个满足`N > commitIndex`的 N，并且大多数的`matchIndex[i] ≥ N`成立，并且`log[N].term == currentTerm`成立，那么令 commitIndex 等于这个 N

### 3. Raft 在任何时候都保证以下的各个特性

| 特性| 解释|
|---|---|
|选举安全特性| 对于一个给定的任期号，最多只会有一个领导人被选举出来|
|领导人只附加原则| 领导人绝对不会删除或者覆盖自己的日志，只会增加|
|日志匹配原则| 如果两个日志在相同的索引位置的日志条目的任期号相同，那么我们就认为这个日志从头到这个索引位置之间全部完全相同|
|领导人完全特性|如果某个日志条目在某个任期号中已经被提交，那么这个条目必然出现在更大任期号的所有领导人中|
|状态机安全特性| 如果一个领导人已经在给定的索引值位置的日志条目应用到状态机中，那么其他任何的服务器在这个索引位置不会提交一个不同的日志|


### 相关博文

- [Raft 日志复制](/tech/distributed/raft/raft_log_replication.md)

- [Raft 领导选举](/tech/distributed/raft/raft_leader_election.md)

- [Golang 操作 etcd（上）](/tech/distributed/etcd/etcd_usage_golang_1.md)

- [Golang 操作 etcd（下）](/tech/distributed/etcd/etcd_usage_golang_2.md)


#### 感谢

- [Raft 英文 paper pdf](https://ramcloud.atlassian.net/wiki/download/attachments/6586375/raft.pdf)

- [Raft 作者讲解视频](https://www.youtube.com/watch?v=YbZ3zDzDnrw&feature=youtu.be)

- [Raft 作者讲解视频对应的 PPT](https://ramcloud.atlassian.net/wiki/download/attachments/6586375/raft.pdf)

- [Raft 论文译文](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)

- [Raft 动画](http://thesecretlivesofdata.com/raft/)


