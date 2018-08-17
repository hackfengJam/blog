elasticsearch   mysql
index（索引）    数据库
type（类型）      表
document（文档）  行
fields           列


方法
GET
POST
PUT
DELETE


倒排索引
倒排索引源于实际应用中需要根据属性的值来查找记录，这种索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引(inverted index)。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件(inverted file)。

例子
文件A： 通过python django搭建网站


映射(mapping)
当我们创建索引的时候，可以预先定义字段的类型以及相关属性

elasticsearch会根据JSON源数据的基础类型猜测你想要的字段映射。将输入的数据转变成可搜索的索引项。Mapping就是我们自己定义的字段的数据类型，同时告诉elasticsearch如何索引数据以及是否可以被搜索。

作用：会让索引建立的更加细致和完善

类型：静态映射和动态映射

内置类型
string类型：text，keyword（string类型在es5开始已经废弃）
数字类型：long，integer，short，byte，double，float
日期类型：date
bool类型：boolean
binary类型：binary
复杂类型：object（对象），nested（对象集）
geo类型：geo-point，geo-shape
专业类型：ip，competition

属性   					描述     						适合类型
store  	值为yes表示存储，为no表示不存储，默认为no			all

index	yes表示分析，no表示不分析，默认值为true				string

null_value	如果字段为空，可以设置一个默认值，比如"NA"		all

analyzer	可以设置索引和索引时用的分析器，默认使用的是standard分析器，还可以使用                			whitespace、simple、english					all

include_in_all	默认es为每个文档定义一个特殊域_all，它的作用是让每个字段被搜索到，如果不想某个字段被搜索到，可以设置为false							all

format	时间格式字符串的模式								date


查询
elasticsearch是功能非常强大的搜索引擎，使用它的目的就是为了快速的查询到需要的数据。

查询分类：
- 基本查询：使用elasticsearch内置查询条件进行查询
- 组合查询：把多个查询组合在一起进行复合查询
- 过滤：查询同时，通过filter条件下在不影响打分的情况下筛选数据

analyzer："ik_max_word"


