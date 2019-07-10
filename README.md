# 个人博客

本仓库下存放个人博客的源文件。持续更新，欢迎 `star`。

如果大家觉得那里写的不合适的可以给我提 `Issue`

---

## 前言

程序员的自我修养：

1. 一切语言、技术或者框架，本质都是工具，工具的价值在于为使用者提供竞争优势。

2. 如果真的有一种语言或者框架很牛逼，那么荣耀或者赞誉应该属于创造它的人，与使用者没有半毛钱关系；
使用者的荣耀应该来自；针对恰当的需求使用恰当的语言或者框架，并做到按时交付以及高质量。

3. 大部分人并不是天生有选择恐惧症，也不是天生的杠精，在我看来所有关于选择的迷惑或者争吵，大都因为：
    1. 没有设定清晰的标准；  
    2. 标准不唯一；  
    3. 标准之间没有优先级或者权重；  

---

## 目录简介

<details open>
<summary>展开目录简介</summary>

- [语言基础](./contents/basic.md)
    - [Python](./contents/basic.md#python)
    - [Golang](./contents/basic.md#golang)
    - [Java](./contents/basic.md#java)

- [数据结构与算法](./contents/algorithm.md)
    - [我的专题仓库](./contents/algorithm.md#我的专题仓库helloalgorithm有介绍点击这里)

- [框架技术](./contents/framework.md)
    - [Scrapy](./contents/framework.md#scrapy)
    - [Elasticsearch](./contents/framework.md#elasticsearch)
    - [SpringCloud](./contents/framework.md#springcloud)
    
- [各种技术](./contents/misc.md)
    - [Redis](./contents/misc.md#redis)
    - [Kafka](./contents/misc.md#kafka)
    - [MySQL](./contents/misc.md#mysql)
    - [Nginx](./contents/misc.md#nginx)
    - [分布式](./contents/misc.md#分布式)
    - [Linux](./contents/misc.md#linux)
    - [网络](./contents/misc.md#网络)

- [设计与实战](./contents/design.md)
    - [搭建并行处理管道 - Golang](./contents/design.md#搭建并行处理管道---golang)
    - [原生爬虫 - Golang](./contents/design.md#原生爬虫---golang)
    - [流量统计系统 - Golang](./contents/design.md#流量统计系统---golang)
    - [开发分布式任务调度 - Golang](./contents/design.md#开发分布式任务调度---golang)

- [反省与总结](./contents/reflection_and_summary.md)
    - [面试](./contents/reflection_and_summary.md#面试)
    - [其他](./contents/reflection_and_summary.md#其他)

- [工具](./contents/tools.md)
    - [Pycharm上传到码云或者GitHub](./contents/tools.md#pycharm上传到码云或者github)
    - [欢迎使用CSDN-markdown编辑器](./contents/tools.md#欢迎使用csdn-markdown编辑器)

- [chinese](./contents/chinese.md)
    - [chinese_300_tang_poems](./contents/chinese.md#chinese_300_tang_poems)

</details>


## 目录

<details>
<summary>目录详情</summary>

#### 【语言基础】

- ##### Python
    - 基础知识
        - [变量对象和引用](./basic/python/basic/变量对象和引用.md)
        - [Python学习之Queue](./basic/python/basic/Python学习之Queue.md)
        - [Python中的作用域及global用法](./basic/python/basic/Python中的作用域及global用法.md)
    - Requests 库
        - [Requests模块学习之一-发送请求](./basic/python/requests/Python学习之Requests模块学习之一-发送请求.md)
        - [Requests模块学习之二-处理响应](./basic/python/requests/Python学习之Requests模块学习之二-处理响应.md)
        - [Requests模块学习之三-进阶话题](./basic/python/requests/Python学习之Requests模块学习之三-进阶话题.md)
    - 并发
        - [Python_多进程_进程池](./basic/python/concurrency/Python_多进程_进程池.md)
        - [Python之路_异步IO_队列_缓存](./basic/python/concurrency/Python_多进程_进程池.md)
        - [Python之路_进程_线程](./basic/python/concurrency/Python_多进程_进程池.md)

- ##### Golang
    - [用 golang 实现 nginx 反向代理及负载均衡](./basic/golang/用go实现nginx反向代理及负载均衡.md)
    
- ##### Java
    - Pending

#### 【数据结构与算法】

- ##### 我的专题仓库「HelloAlgorithm」有介绍：[点击这里](https://github.com/hackfengJam/HelloAlgorithm)

#### 【框架技术】

- ##### Scrapy
    - [Scrapy - 第一步框架原理](./framework/scrapy/第一步Scrapy框架原理.md)
- ##### Elasticsearch
    - [Elasticsearch - 介绍及开发环境搭建](./framework/elasticsearch/Elasticsearch环境搭建.md)
    - [Elasticsearch - 搜索引擎_pending](./framework/elasticsearch/搜索引擎_Elasticsearch_pending.md)
- ##### SpringCloud
    - [Eureka 的介绍](tech/springcloud/Eureka介绍.md)
    - [Eureka Server 和 Client 之间的信息维护（注册和续约）](tech/springcloud/Eureka_Server_和_Client_之间的信息维护（注册和续约）.md)
    - [Zuul 的介绍](tech/springcloud/Zuul介绍.md)

#### 【各种技术】

- ##### Redis
    - [Redis的正确打开方式](tech/redis/Redis的正确打开方式.md)
    - [高并发情况下Redis做缓存的一系列问题_pending](tech/redis/高并发情况下Redis做缓存的一系列问题_pending.md)
- ##### Kafka
    - [初识kafka_pending](tech/kafka/初识kafka_pending.md)
- ##### MySQL
    - [数据库优化 - 索引优化](tech/mysql/数据库优化——索引优化.md)
    - [Mysql大表处理_pending](tech/mysql/Mysql大表处理_pending.md)
    - [数据库 - 如何设计一个关系型数据库](tech/mysql/数据库——1_数据库架构.md)
    - [数据库 - 索引管理](tech/mysql/数据库——2_索引管理.md)
    - [数据库 - 锁管理](tech/mysql/数据库——3_锁管理.md)
- ##### Nginx
    - [nginx - 使用之总体简介](tech/nginx/nginx使用之总体简介.md)
    - [nginx - 使用之配置文件的组成及主配置段的指令之一](tech/nginx/nginx使用之配置文件的组成及主配置段的指令之一.md)
    - [nginx - 使用之配置文件的组成及主配置段的指令之二](tech/nginx/nginx使用之配置文件的组成及主配置段的指令之二.md)
- ##### 分布式
    - [分布式id生成算法 - SnowFlake](tech/distributed/algo/分布式id生成算法SnowFlake.md)
    - Raft
        - [Raft 领导选举](tech/distributed/raft/raft_leader_election.md)
        - [Raft 一致性算法](tech/distributed/raft/raft_consensus_algorithm.md)
        - [Raft 日志复制](tech/distributed/raft/raft_log_replication.md)
    - Celery
        - [Celery 的正确打开方式 - 结合「官方文档」及「实际用例」了解 Celery](tech/distributed/celery/celery_opens_correct_way.md)「**关键词：分布式任务队列；Celery**」
    - etcd
        - [什么是 etcd?](tech/distributed/etcd/etcd_study_1_what_is_etcd.md)
        - [etcd 功能与原理](tech/distributed/etcd/etcd_function_and_principle.md)
        - [Golang 操作 etcd（上）](tech/distributed/etcd/etcd_usage_golang_1.md)
        - [Golang 操作 etcd（下）](tech/distributed/etcd/etcd_usage_golang_2.md)
- ##### Linux
    - [Linux - find、grep、awk、sed 常用命令](./tech/linux/Linux.md)
    - [select、poll、epoll](./tech/linux/select_poll_epoll.md)
    - [零拷贝 - NIO](./tech/linux/零拷贝_NIO.md)
- ##### 网络
    - [TCP 三次握手、四次挥手详解](./tech/network/tcp.md)
    - [HTTP 与 HTTPS 详解与区别](./tech/network/http与https.md)
    - [HTTPS 如何做到安全](./tech/network/https.md)
- ##### 架构
    - [Git workflow](./tech/architecture/git_workflow.md)
    - [你的项目应该如何分层？](./tech/architecture/how_should_your_project_be_stratified.md)
    - [什么是扇入和扇出？](./tech/architecture/fanout_and_fanin.md)

#### 【设计与实战】

- ##### [搭建并行处理管道 - Golang](./design/golang_pipeline/golang_pipeline.md)
- ##### [原生爬虫 - Golang](./design/golang_crawler/golang_crawler.md)
- ##### [流量统计系统 - Golang](./design/golang_analysis/golang_analysis.md)
- ##### [开发分布式任务调度 - Golang](./design/golang_crontab/golang_crontab.md)
- ##### [微信抢红包功能设计 - Golang](./design/red_envelope/red_envelope.md)

    
#### 【反省与总结】

- ##### 面试
    - [Python面试题精选](./reflection_and_summary/interview/Python面试题精选.md)
    - [面试-复习](./reflection_and_summary/interview/面试-复习.md)
    - [某不知名小厂面经](./reflection_and_summary/interview/某不知名小厂面经.md)
- ##### 其他    
    - [给学弟学妹们总的方向及建议](./reflection_and_summary/misc/给学弟学妹们总的方向及建议.md)

#### 【工具】

- ##### [Pycharm上传到码云或者GitHub](./tools/Pycharm上传到码云或者GitHub.md)
- ##### [欢迎使用CSDN-markdown编辑器](./tools/欢迎使用CSDN-markdown编辑器.md)

#### 【chinese】

- ##### chinese_300_tang_poems
    - 五言古诗_三十三首
        - [张九龄 - 感遇_二首](./chinese/chinese_300_tang_poems/五言古诗_三十三首/张九龄/感遇_二首.md)

</details>


## 开源社区链接

[Gitee](https://gitee.com/hackfun)

[GitHub](https://github.com/hackfengJam)

## TODO list

<details>
<summary>展开查看</summary>

- 有个webhook接口：目前直接返回200，并调用异步任务系统。
  现在 有三个 HTTP 请求（1:create, 2:modify, 3:delete） 过来（三个请求时间间隔不一定，可能没有 2:modify），需要它们三个异步任务顺序执行。（给每一个请求 一个id，通过id hash发到执行的机器，执行机器分配线程执行是拿到 id 存在已分配的线程中 ）
  
- 初识 kafka 

- kafka 高级特性之消息事务

- Mysql大表处理

- 高并发情况下 Redis 做缓存的一系列问题

- 数据库如何建索引，如何分库分表

- LRU 的实现，原理、数据结果和过程结果

- QPS 限流 （缓存，滑动窗口？）

- 标签 推荐算法实现

- HTTPS 如何做到安全

- 根据二叉树前序遍历生成 AVL 树

- Redis 主从同步和持久化方式

- Redis 高可用方案

- Redis 缓存集中过期如何处理

- Redis 的集群有哪些

- 用 redis 做限流

- 统计大量访问日志（分几百M 和 几百G的场景）；得出访问次数最多的前 K 个人 （单台机器实现）

- 8G 文件 1G 内存，查找重复的数字

- 对称性加密跟非对称性加密的比较、使用场景

- RSA 加密算法

- 搜索引擎倒排原理及实现方式

- XSS 和 CSRF

- ctrl+c 后操作系统做了什么

- T级别大日志文件，如何找出一段时间内日志

- 求无向无环图的直径

- python GIL 以及为什么有GIL，还需要 threading 

- 自增ID 与 uuid 的优劣

- B、B+、跳跃表 区别 时间复杂度

- java fail-fast 机制

- Mac 地址如果被改会怎么样

- 路由寻址过程做了哪些事

- HTTP 1.1、2 特性

- HTTP 里面的各种攻击及应对策略

- 如何信任 CA

- 分布式事务，两阶段提交协议，失败重试补偿

- 在微服务架构中，如何能保证接口的可靠性。（幂等性校验？安全角度？）

- 程序设计
    - Golang 开发分布式任务调度
    
    - 微信抢红包功能设计
    
    - 推送的频率控制
    
    - 抖音评论列表的设计及缓存实现
    
    - 假设是一个抽奖的游戏，不同的人是有不同的概率倍数，是一个整数，例如：1、3、5...。输入100万人，要求抽奖抽出2万个人；
    并假设每个人都有一个唯一id，写一个函数做下抽奖，输入和输出的数据结构自己设计
    
    - 设计群消息已读功能
    
    - HTTP 301 实现原理，设计一个短链服务
    
    - 给一个亿级用户登录登出时间戳日志，统计用户在线量峰值及持续时间，代码实现
    
    - 消息队列如何保证可靠
    
    - 设计秒杀系统要求保证公平
    
    - 如何限制每分钟每个手机号短信发送数
    
    - 发短信业务，1分钟内一个号一个业务 1000 条
    
    - 多人联机贪吃蛇设计
    
    - 链表逆序，设计一个王者的组队系统
    
    - 头条文章向用户推送避免重复推送问题
    
    - 如何实现音乐随机播放
    
    - 系统设计：微信扫码登录
    
    - 微博的热门评论，在分页到很深的时候，如何进行优化
    
    - 一个分布式不安全的文件系统，如何保证每次只有一个请求进行读写

</details>

## Tips:

- 为方便查阅博客，可以在浏览器安装 [Octotree](https://github.com/buunguyen/octotree) 插件
