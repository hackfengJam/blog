# 原生爬虫

源码仓库：[这里](https://github.com/hackfengJam/golearn/tree/master/project/crawler)

---


通过 channel - 爬取珍爱网

- 【engine】
    - [引擎](https://github.com/hackfengJam/golearn/tree/master/project/crawler/engine)
- 【fetcher】
    - [抓取器](https://github.com/hackfengJam/golearn/tree/master/project/crawler/fetcher)
- 【scheduler】
    - [调度器](https://github.com/hackfengJam/golearn/tree/master/project/crawler/scheduler)
- 【parser】
    - [解析器](https://github.com/hackfengJam/golearn/tree/master/project/crawler/zhenai/parser)
- 【爬虫架构】
    - [单任务版爬虫架构](https://github.com/hackfengJam/golearn/tree/master/project/crawler/zhenai/parser)
    - [并发版爬虫架构](#并发版爬虫架构)
    - 【并发版爬虫架构】
        - [实现I](#实现i)
        - [实现II：并发分发Request](#实现ii并发分发request)
        - [实现III：Request队列和Worker队列](#实现iiirequest队列和worker队列)

#### 单任务版爬虫架构
```
+--------------+            +--------------+      response      +-----------------+
|              |  request   |              +------------------->+                 |
|     Seed     +----------->+    Engine    |  requests,items    |     Parser      |
|              |            |              |<-------------------|                 |
+--------------+            ++-----------+-+                    +-----------------+
                     +------^^           ^ ^------+                       
                     |-------|           |        |
                     ||                  |        |
                     ||                  |url     |response
                     vv                  |        |
     +---------------++------+           | +------+--------------+
     |                       |           | |                     |
     |     Task Queue        |           +->       Fetcher       |
     |                       |             |                     |
     +-----------------------+             +---------------------+

```

#### 并发版爬虫架构
```
                                       +----------------------------------------------+
                                       |                                              |
                                       |                                     Worker   |
                                       |                                              |
                                       |                                              |
+--------------+            +--------------+      response      +------- ---------+   |
|              |  request   |          |   +------------------->+                 |   |
|     Seed     +----------->+    Engine|   |  requests,items    |     Parser      |   |
|              |            |          |   +<-------------------+                 |   |
+--------------+            +------------+-+                    +-----------------+   |
                     +------->         | > ^------+                                   |
                     |-------+         | |        |                                   |
                     ||                | |        |                                   |
                     ||                | |url     |response                           |
                     <>                | |        |                                   |
     +-----------------------+         | |        +-------------+------------------+  |
     |                       |         | |                      |                  |  |
     |     Task Queue        |         | ----------------------->     Fetcher      |  |
     |                       |         |                        |                  |  |
     +-----------------------+         |                        +------------------+  |
                                       |                                              |
                                       +----------------------------------------------+

```

#### 实现I:
```

                           +---------------+
                           |   OutPut      |
                           |               |
                           +---------------+
                                  <>
                                  ||
                                  || Items
                                  ||
                                  ||
+--------------+            +--------------+                    +-----------------+
|              |  request   |              |   requests,items   | +-----------------+
|     Seed     +----------->+    Engine    +--------------------+ | +-----------------+
|              |            |              |                    | | |                 |
+--------------+            +-----------+--+                    +-+ |      Worker     |
                                        |                         +-+                 |
                                        | request                   +--+--------------+
                                        |                              |
                                        |                              |
                                        v                              |
                                      +-+------------------------------+----+
                                      |                                     |
                                      |               Scheduler             |
                                      |                                     |
                                      +-------------------------------------+

```

#### 实现II：并发分发Request

```
                                       +-----------------+
                                       | +-----------------+
                                       | | +-----------------+
                                       | | |                 |
         +                             +-+ |      Worker     |
         | request                       +-+                 |
         |                                 +--+--------------+
         |                                    ^
         |                                    | request
         v                                    |
+--------+---------+    create for     +------------------+
|                  |    each request   | +------------------+
|     Scheduler    +------------------>+ | +-------------------+
|                  |                   | | |                   |
+------------------+                   +-+ |     Goroutine     |
                                         +-+                   |
                                           +-------------------+
```

#### 实现III：Request队列和Worker队列
```
                +
                |
                | request
                |
                |
                v                          +--------------------------------+
        +-------+---------------+          |                                |
        |                       |          |  +----------+     +----------+ |
        |       Scheduler       +--------->+  |  Request +---->+  Worker  | |
        |                       |          |  +----------+     +----------+ |
        +-+-------------------+-+          |                                |
          |                   |            +--------------------------------+
          |                   |
          v                   |
+---------+-------+  +--------+---------+                   +------------------+
|                 |  |                  |                   | +------------------+
|  Request Queue  |  |   Worker Queue   +-------------------+ | +-------------------+
|                 |  |                  |                   | | |                   |
+-----------------+  +------------------+                   +-+ |      Worker       |
                                                              +-+                   |
                                                                +-------------------+
```


#### 感谢


