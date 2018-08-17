## Redis的正确打开方式

### 提纲

- 引入：后端开发中最常遇到的问题————共享数据
	- 多线程下共享数据————共享变量即可
	- 多进程/多进程服务器共享数据————外部共享存储
		- MySQL：适合存储关系型数据，一般用于存储业务数据
		- Redis：适合存储离散，简单的数据，操作丰富，速度更快

- 简单、快速、好用的外部共享存储————Redis
	- 认证令牌、验证码、密码输错次数等————基本GET、SET、INCR操作
		- SETEX实现自动过期的登录令牌、验证码
		- INCR、EXPIRE操作实现限制密码输错次数

	- 缓存MySQL数据库记录
		- 方式1：SET、GET保存完整JSON字符串 （参考：http://redis.cn/commands/set.html）
		- 方式2：HSET、HMSET、HGET、HMGET、HGETALL拆分保存数据 （参考：http://redis.cn/commands/hset.html）

	- 简单队列/栈、全局轮询————LIST操作 （参考：http://redis.cn/commands/lpush.html 、 http://redis.cn/commands/rpoplpush.html）
		- LPUSH、RPOP作为队列使用
		- LPUSH、BRPOP实现生产者-消费者模型
		- LPUSH、LPOP作为栈使用
		- RPOPLPUSH作为循环列表使用
		- RPOPLPUSH实现可靠队列 （参考：https://www.zhihu.com/question/32318171/answer/98280219）
	- 消息订阅、发布————PUBSUB操作
		- Node.js中PUBSUB的正确使用方式（参考：https://www.npmjs.com/package/redis#publish--subscribe）
		- Python中PUBSUB的正确使用方式（参考：https://github.com/andymccurdy/redis-py#api-reference）
		- 使用PSUBSCRIBE批量订阅，减少监听客户端（参考：http://redis.cn/commands/psubscribe.html）

- 题外话：Redis的Key设计/命名规则
	- 冒号「:」分隔，方便使用「keys *」通配
	- 添加「#」、「@」分隔应⽤用名称、键⽤用途，增加可读性。 *articlespider#cache@:user:1:* or *hfonline#cache@:user:1:*

- 全局自增长————INCR
- Redis事务 （参考：http://redis.cn/topics/transactions.html）
	- 使用WATCH、MULTI、EXEC实现Check-And-Set（乐观锁）

- 优先级/延迟队列————ZSET操作
	- SCORE作为启动时间/优先级的有序集合
	- 利用Redis事务实现ZPOP

- 使用HyperLogLog算法————PFADD、PFCOUNT （参考：http://www.runoob.com/redis/redis-hyperloglog.html）
	- 文章阅读量统计

- 缓存常见套路 （参考：https://coolshell.cn/articles/17416.html）
	- Cache Aside————最常用，需要增加失效时间
	- Read/Write Through————对应用层透明
	- Write Behind Caching （Write Back）————速度飞快、非强一致性

- 完整Redis操作介绍————文档多么齐全！
	- 注意 「可⽤用版本」
	- 注意 「时间复杂度」


## 感谢
[Redis 官方文档](https://redis.io/commands)

[周逸灵](https://www.zhihu.com/people/zhou-yi-ling-31/activities)
