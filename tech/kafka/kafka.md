## Kafka

### 1. 生产者

#### 1.1 分区策略

**1) 分区的原因**

- 方便在集群中扩展。每个 Partition 可以通过调整以适应他所在的机器，而一个 topic 可以用多个 Partition 组成，因此整个集群就可以适应任意大小的数据了；

- 可以提高并发。可以以 Partition 为单位读写

**2) 分区的原则**

1) 指明 partition 情况，直接将指明的值作为 partition 值；

2) 没有指明 partition 但有 key 情况，将 key hash 值与 topic 的 partition 数取余

3) 既没有 partition 有没有 key 值，第一次调用随机生成一个整数（后面每次调用在这个整数上自增），
将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法

4) 可以自定义分区策略

#### 1.2 数据可靠性保证

**1）副本数据同步策略**

| 方案 | 优点 | 缺点 |
|------|-----|-----|
| 半数以上完成同步，就发送 ack | 延迟低 | 选举新的 leader 时，容忍 n 太节点的故障，需要 2n+1 个副本
| 全部完成同步，才发送 ack | 选举新的 leader 时，容忍 n 台节点的故障，需要 n+1 个副本 | 延迟高 |

Kakfa 选择的第二中方案，原因如下：

1. 同样为了容忍 n 太节点故障，第一种方案需要 2n + 1 个副本，而第二种方案
需要 n + 1 个副本，而 Kafka 的每个分区都有大量的数据，第一种方案会造成大量的数据冗余

2. 虽然第二种方案的网络延迟比较高，但网络延迟对 Kafka 的影响较小。

**2) ISR**

> 第二种方案，存在问题：如果有一个 follower 因为故障迟迟不能与 Leader 同步，那么 Leader 就会 一直等下去，
直到它完成同步，才能发送 ack。

Leader 维护了一个动态的 in-sync replica set(ISR)，意为何 Leader 保持同步的 follower 集合。
当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。
如果 follower 长时间未向 leader 同步数据，则该 follower 将会被踢出 ISR，时间阈值
由 `replica.lag.time.max.ms` 参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 Leader。


- ZK 和 内存都有  

* ISR 加入条件
    * 拉取条数    
    * 同步时间     
* 新版本(>=0.9)只保留了一个，条件   
    * 同步时间(replica.lag.time.max.ms)  
    * 踢出拉取`拉取条数`条件。避免频繁的加入/踢出 ISR
        * 避免频繁和 zk 交互


**3) ack应答机制**


* 0(不接收 ack，不重试)   
    - broker 一接收到还没有写入磁盘就已经返回，当 broker 故障时，有可能 **丢失数据**  
    
* 1(Leader 收到后，返回 ack)    
    - partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功前 leader 故障，那么将会 **丢失数据**  
    
* -1(ISR 全部同步完，返回ack)  
    - producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才返回 ack。但是如何在 follower 同步完成后，broker 发送 ack 之前，leader 发生故障之前，那么会造成 **数据重复**
    - 注：
        * replica set = 1，退化到 1（比如 ISR 里节点全被踢出）；此时也有可能 **丢失数据**    
            - ISR 里面节点特别慢的情况下，全部被踢出了，只剩 Leader；此时 Leader 挂了。也会丢数据  
        * 可能重复数据  
            - Follower 同步完了，还未发 ack，此时 Leader 挂了。 Producer 会重发数据。导致重复数据问题  


**4) 故障处理细节**

- LEO：指的是每个副本最大的 offset  
- HW： 指的是消费者能见到的最大的 offset，ISR 队列中最小的 LEO   

1. follower 故障

follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后，follower 会读取本地磁盘
记录的上次 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。
等该 **follower 的 LEO ≥ 该 Partition 的 HW**，即 follower 追上 leader 之后，就可以
重新加入 ISR 了。

2. leader 故障

leader 发生故障之后，会从 ISR 中选出一个新的 leader 之后，为保证多个副本之间的数据一致性，
其余的 follower 会先将各自的 log 文件高于 HW 的部分截取掉，然后从新的 leader 同步数据。

**注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。**

3. 这里的一致性指的是  
* 消费一致性  
* 存储一致性  


* HW (High Watermark 高水位)    
    * 副本中最小的 LEO  
    * 消费者可见的最大可见的 offset  
    * Leader 重启后，通知 Follower 截取自己 HW 之后的 Log  
* LEO(Log End Offset)  
    * 每个副本最后一位 log offset  
    

- 补充
    - [kafka分区选主机制](https://blog.csdn.net/lixiang987654321/article/details/99674419)    
    - [kafka的HA设计](https://www.jianshu.com/p/83066b4df739)    
    - [Kafka controller重设计](https://www.cnblogs.com/huxi2b/p/6980045.html)   

#### 1.3 Exactly Once 语义

> 将服务器的 ACK 级别设置为 -1。At Least Once + 幂等性 = Exactly Once

启用幂等性， `enable.idompotence` 设置为 true（ack 会被默认设置为 -1）。开启幂等性的
Producer 在初始化的时候会被分配一个 PID，发往同一个 Partition 的消息会附带 Sequence Number。
而 Broker 端会对 <PID, Partition, SeqNumber> 做缓存，当具有相同主键的消息提交时，Broker 只会持久化一条。

但是 PID 重启就会变好，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨回会话的 Exactly Once。（关于：无法保证跨分区跨回会话；事务可以保证，后续介绍）


#### 1.4 生产者总结

* 生产者分配策略  
    - partition  
        * index 文件  
        * log 文件  
* ACK  
    * 0(不接收 ack，不重试)  
    * 1(Leader 收到后，返回 ack)  
    * -1(ISR 全部同步完，返回ack)  
        * replica set = 1，退化到 1  
        * 可能重复数据  
* ISR(In-sync-replica set)  
    * HW (高水位)    
        * 副本中最小的 LEO  
        * 消费者可见的最大可见的 offset  
        * Leader 重启后，通知 Follower 截取自己 HW 之后的 Log  
    * LEO(Log End Offset)  
        * 每个副本最后一位 log offset  
* 幂等（Exactly Once）  
    * PID, partition, seq 保证幂等。重启后 Producer  PID 重新生成    
        * 跨会话、跨分区 不保证幂等(事务支持重启幂等)   


### 2. 消费者

2.1 消费方式

采用 pull 模式 从broker 中读取。

2.1 分区分配策略

- RoundRobin   
    - 按 Group 分配  
    - 要保证 同一个消费者组 订阅同一个Topic 
- Range（默认）  
    - 按 Topic 分配  
    - 可能出现，消费者消费数据不对等问题（随着订阅 Topic 越多，不对等越严重）  
    - 触发 Range 时机  
        - 消费者数量发生变化时，触发重新分配  
    - 当消费者数量和 Partition 数量一致时，再增加一个消费者，依然触发重新分配。有一个消费者可能分配不到  
    
    

* Kafka Ranger 分区再分析  
    * RoundRobin  
        * 直接看 Group  
    * Range  
        * 先找到订阅 Topic 的 consumer，然后再考虑 consumer Group   

### 3. 消息发送流程

- Kafka 的 Producer 发送消息采用的是 异步发送的方式  
    - 涉及对象  
        - main 线程  
        - Sender 线程  
        - 一个 线程共享变量 —— RecordAccumulator（存放待发送的数据）  
    - main 线程将消息发送给 RecordAccumulator，Sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka broker  

- send(ProducerRecord)
    - Interceptors -> Serializer -> Partitioner
    
- 相关参数   
    - `batch.size`: 只有数据积累到 `batch.size` 之后， sender 才会发送数据   
    - `linger.ms`: 如果数据迟迟未达到 `batch.size`， sender 等待 `linger.time` 之后就会发送数据


### 一些问题

kafka 选主怎么做的？  
kafka 与 rabbitmq区别  
kafka 分区怎么同步的  
kafka 怎么保证不丢消息的  
kafka 为什么可以扛住这么高的qps  

1.Kafka 中的 ISR(InSyncRepli)、OSR(OutSyncRepli)、AR(AllRepli)代表什么？  
2.Kafka 中的 HW、LEO 等分别代表什么？  
3.Kafka 中是怎么体现消息顺序性的？  
4.Kafka 中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？  
5.Kafka 生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？  
6.“消费组中的消费者个数如果超过 topic 的分区，那么就会有消费者消费不到数据”这句话是否正确？    
7.消费者提交消费位移时提交的是当前消费到的最新消息的 offset 还是 offset+1？    
8.有哪些情形会造成重复消费？  
9.那些情景会造成消息漏消费？  
10.当你使用 kafka-topics.sh 创建（删除）了一个 topic 之后，Kafka 背后会执行什么逻辑？  
 1）会在 zookeeper 中的/brokers/topics 节点下创建一个新的 topic 节点，如：  
/brokers/topics/first  
 2）触发 Controller 的监听程序  
 3）kafka Controller 负责 topic 的创建工作，并更新 metadata cache  
11.topic 的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？  
12.topic 的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？  
13.Kafka 有内部的 topic 吗？如果有是什么？有什么所用？  
14.Kafka 分区分配的概念？  
15.简述 Kafka 的日志目录结构？  
16.如果我指定了一个 offset，Kafka Controller 怎么查找到对应的消息？  
17.聊一聊 Kafka Controller 的作用？  
18.Kafka 中有那些地方需要选举？这些地方的选举策略又有哪些？  
19.失效副本是指什么？有那些应对措施？  
20.Kafka 的哪些设计让它有如此高的性能？  
    

### 补充

- [Kafka 官方文档](https://kafka.apache.org/documentation/)    
- [Kafka controller重设计](https://www.cnblogs.com/huxi2b/p/6980045.html)    
- [kafka分区选主机制](https://blog.csdn.net/lixiang987654321/article/details/99674419)    
- [kafka的HA设计](https://www.jianshu.com/p/83066b4df739)    

### 感谢

- TODO
