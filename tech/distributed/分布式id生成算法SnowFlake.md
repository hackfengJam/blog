# 分布式id生成算法 - SnowFlake


#### 概述

分布式系统中，有一些需要使用全局唯一ID的场景，这种时候为了防止ID冲突可以使用36位的UUID，但是UUID有一些缺点，首先他相对比较长，另外UUID一般是无序的，
由于采用的是无意义的字符串，推测会在数据量增大时造成访问过慢，在基础互联网的系统设计中都不推荐采用。
有些时候我们希望能使用一种简单一些的ID，并且希望ID能够按照时间有序生成。

而twitter的snowflake解决了这种需求，最初Twitter把存储系统从MySQL迁移到Cassandra，因为Cassandra没有顺序ID生成机制，所以开发了这样一套全局唯一ID生成服务。

自增ID的方法虽然比较适合大数据量的场景，当时由于自增ID是按照顺序增加的，数据记录都是可以根据ID号进行推测出来，对于一些数据敏感的场景，不建议采用。


#### Snowflake 算法核心
把***时间戳***，***工作机器id***，***序列号***组合在一起。

snowflake的结构如下(每部分用-分开):

![Alt text](images/snowflake-64bit.jpg)


第一位为未使用，接下来的41位为毫秒级时间(41位的长度可以使用69年)，然后是5位datacenterId和5位workerId(10位的长度最多支持部署1024个节点） ，最后12位是毫秒内的计数（12位的计数顺序号支持每个节点每毫秒产生4096个ID序号）

一共加起来刚好64位，为一个Long型。(转换成字符串后长度最多19)

snowflake生成的ID整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞（由datacenter和workerId作区分），并且效率较高。经测试snowflake每秒能够产生26万个ID。



#### 感谢

[twitter开源项目 - Scala版](https://github.com/twitter/snowflake)

[pysnowflake - Python版](https://github.com/erans/pysnowflake)

[理解分布式id生成算法SnowFlake](https://segmentfault.com/a/1190000011282426)

[64位自增ID算法详解](https://www.lanindex.com/twitter-snowflake%EF%BC%8C64%E4%BD%8D%E8%87%AA%E5%A2%9Eid%E7%AE%97%E6%B3%95%E8%AF%A6%E8%A7%A3/)
