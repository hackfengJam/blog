# 数据库索引优化

## MySQL支持的索引类型

1. [BTree索引](#1B-tree索引)
2. [Hash索引](#2Hash索引)


### 1.B-tree索引

#### 1.1 B-tree索引的特点（顺序存储）

- B-tree 索引能够加快数据的查询速度
- B-tree 索引更适合进行范围查找（顺序存储）

#### 1.2 在什么情况下可以用到B树索引

- 全职匹配的查询
	- order_sn = '9876432119900'
- 匹配最左前缀的查询
- 匹配列前缀查询
	- order_sn like '9876%'
- 匹配范围值的查询
	- order_sn > '9876432119900' and order_sn < '9876432119999'
- 精确匹配左前列并范围匹配另外一列
- 只访问索引的查询

#### 1.3 Btree索引的使用限制

- 如果不是按照索引最左烈开始查找，则无法使用索引
- 使用索引时不能跳过索引中的列
- not in 和 <> 操作无法使用索引
- 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引
 
### 2.Hash索引

#### 2.1 Hash索引的特点
- Hash索引时基于Hash表实现的，只有查询条件精确匹配Hash索引中的所有列时，在能够使用到hash索引。
- 对于Hash索引中的所有列，存储引擎都会为每一行计算一个Hash码，Hash索引中存储的就是Hash码。

#### 2.2 Hash索引的限制
- Hash索引必须进行二次查找
- Hash索引无法用于排序
- Hash索引不支持部分索引查找，也不支持范围查找
- Hash索引中Hash码的计算可能存在Hash冲突

### 3.为什么要使用索引

- 索引大大减少了存储引擎需要扫描的数据量
- 索引可以帮助我们进行排序以避免使用临时表（避免自排临时表）
- 索引可以把随机I/O变为顺序I/O
- 注：innodb索引最大767字节，myisam索引最大1000字节

### 4. 索引是不是越多越好

- 索引会增加写操作的成本
- 太多的索引会增加查询优化器的选择时间

### 5. 查询优化策略

#### 5.1 索引列上不能使用表达式或函数

	select ... from product where to_days(out_date)-to_days(current_date)<=30
	优化后
	select ... from product where out_date <= date_add(current_date,interval 30 day)

#### 5.2 前缀索引和索引列的选择性
	create index index_name on table(col_name(n));
	索引的选择性是不重复的所有制和表的记录的比值
	mysql5.0之前，每一个查询只能使用到一个列上的索引。


#### 5.3 联合索引
	如何选择索引列的顺序（优先级从上往下递减）
	（1）经常会被使用到的列优先
	（2）选择性高的列优先
	（3）宽度小的列优先

#### 5.4 覆盖索引
	（1）优点
		①.可以优化缓存，减少磁盘IO操作
		②.可以减少随机IO，变随机IO操作为顺序IO操作
		③.可以避免对Innodb主键索引的二次查询
		④.可以避免MyISAM表进行系统调用
	（2）无法使用覆盖索引的情况
		①.存储引擎不支持覆盖索引
		②.查询使用了太多的列
		③.使用了双%号的like查询

### 6.使用索引来优化查询
	（1）使用索引扫描来优化排序
		①.索引的列顺序和Order by子句的顺序完全一致
		②.索引中索引列的方向（升序，降序）和Order By 子句完全一致
		③.Order By中的字段全部在关联表中的第一张表中
	（2）模拟Hash索引优化查询
		①.只能处理键值的全值匹配查找
		③.所使用的Hash函数决定着索引键的大小
	（3）利用索引优化锁
		①.索引可以减少锁定的行数
		③.索引可以加快处理速度，同时也加快了所的释放

### 7.索引的维护和优化
	（1）删除重复和冗余的索引
		①.primary key(id), unique_id key(id), index(id)   主键和唯一主键都会建立索引不需要再建立额外索引
		②.index(a), index(a,b)  联合索引：index(a,b)实则也是与index(a)冗余
		③.primary key(id), index(a,b)  同②
		注：当然，当联合索引很大时，可以建立一个冗余索引。比如：index(a,b,c,d,..) 此时我们可以适当建立 index(a)
		③. pt-duplicate-key-checker h=127.0.0.1,使用工具查看重复和冗余索引（pt-duplicate-key-checker这款工具也是percona-toolkit中一款非常适用的工具)
	（2）查找未被使用过的索引
		select object_schema,object_name,index_name,b.`TABLE_ROWS` FROM performance_schema.table_io_waits_summary_by_index_usage a JOIN information_schema.tables b ON a.`OBJECT_SCHEMA`=b.`TABLE_SCHEMA` AND a.`OBJECT_NAME`=b.`TABLE_NAME` WHERE index_name IS NOT NULL and count_star=0 ORDER BY object_schema,object_name;
	（3）更新索引统计信息及减少索引碎片
		analyze table table_name
		innodb是随机，并不是全盘扫描，只做评估，效率高，但不准确
		myisam是全盘扫描，准确，但锁表时间长，效率低
		
		定期维护索引碎片
		定期维护表碎片 optimize table table_name （注：使用不当会导致锁表）
