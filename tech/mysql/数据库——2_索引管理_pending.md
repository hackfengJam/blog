# 2. 索引管理
 
### 索引管理
 
 
#### 1.提出问题。

1.1 问题回顾与总结
 - 为什么要使用索引？
 - 什么样的信息能成为索引？
 - 索引的数据结构？
 - 密集索引和稀疏索引的区别？
 
1.2 衍生出来的问题，以mysql为例
 - 如何定位并优化慢查询 Sql？
 - 联合索引的最左匹配原则的成因？
 - 索引是建立的越多越好吗？

#### 2. 寻找答案

2.1 问题回顾与总结
 - 为什么要使用索引？
    - 快速查询数据
 - 什么样的信息能成为索引？
    - 主键、唯一键以及普通键等
 - 索引的数据结构？
    - 生成索引，建立 二叉查找树 进行二分查找
    - 生成索引，建立 B-Tree 结构进行查找
    - 生成索引，建立 B+-Tree 结构进行查找
    - 生成索引，建立 Hash 结构进行查找
 - 密集索引和稀疏索引的区别？
    - 密集索引文件中的每个搜索码值都对应一个索引值
    - 稀疏索引文件只为索引码的某些值建立索引项

2.2 衍生出来的问题，以mysql为例
 - 如何定位并优化慢查询 Sql？
    1. 根据日志定位慢查询 sql
    2. 使用 explain 等工具分析 sql
    2. 修改 sql 或者尽量让 sql 走索引


#### 3. 实际操作

3.1 mysql 日志

3.1.1 查看慢查询相关变量

```sql
show variables like '%query%';
```

结果如下（其余行暂时忽略）：

| Variable_name     |              Value             |
|-------------------|--------------------------------|
|long_query_time    |          10.000000             |
|slow_query_log     |             OFF                |
|slow_query_log_file|/usr/local/xxx/mysql/...slow.log|

需要关注的参数有：**long_query_time** 、**slow_query_log** 、 **slow_query_log_file**


3.1.2 查看当前慢查询数

```sql
show status like '%slow_queries%';
```

将会出现 slow_queries 字段，表示当前存在的慢查询个数。

3.1.3 打开慢查询日志

```sql
set global slow_query_log = on;
```

3.1.4 设置慢查询最长时间（默认：10 s）

我们修改为 1s
```sql
set global long_query_time = 1;
```

再次查看，我们将发现各个参数已被修改

```sql
show variables like '%query%';
```


结果如下（其余行暂时忽略）：

| Variable_name     |              Value             |
|-------------------|--------------------------------|
|long_query_time    |           1.000000             |
|slow_query_log     |              ON                |
|slow_query_log_file|/usr/local/xxx/mysql/...slow.log|


**注意**：以上所有设置，都是在本次会话有效，当退出重连时，参数设置将失效，可以通过修改 MySQL 配置文件，保证改动重启后依然有效。

