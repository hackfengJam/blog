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


**注意**：

    - 以上所有设置，都是在本次会话有效，当退出重连时，参数设置将失效，可以通过修改 MySQL 配置文件，保证改动重启后依然有效。
    
    - DDL 数据库定义语句，不会进入慢查询
    
    - DML 数据库修改语句，才会进入慢查询

3.1.5 之后可以通过 **slow_query_log_file** 文件，查看慢查询日志。

- 使用 explain 关键字段 对 sql分析。
    - type 字段
    
        连接类型。有多种类型，**重要且困难**。
        
        性能由高到低依次为：system > const > eq_ref > ref > fulltext > ref_or_null_index_merge > unique_subquery > index_subquery > range > **index** > **all**
        
        其中 index 和 all 都是扫描全表
        
        | type 项  |              说明             |
        |-------------------|--------------------------------|
        |   id     |  理解为，SQL执行的顺序的标识,SQL从大到小的执行           |
        |   all    |  对于每个来自于先前的表的行组合，进行完整的表扫描。如果表是第一个没标记const的表，这通常不好，并且通常在它情况下很差。通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或列值被检索出。           |
        |  index   |  该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）             |
            
    - extra 字段
    
        该列包含MySQL解决查询的详细信息。
    
        | extra 项     |              说明             |
        |-------------------|--------------------------------|
        |Using filesort     |   表示 MySQL 会对结果使用一个外部索引排序，而不是从表里按索引次序读到相关内容。可能在内存或者磁盘上进行排序。MySQL 中无法利用索引完成的排序操作称为"文件排序"            |
        |Using temporary    |  表示 MySQL 在对查询结果排序时使用临时表。常见于 order by 和 分组查询 group by。             |
        
