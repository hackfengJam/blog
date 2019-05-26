## Linux

### 1. Linux 的体系结构

#### 1.1 Linux

- 体系结构主要分为用户态（用户上层活动）和内核态

- 内核：本质是一段管理计算机硬件设备的程序

- 系统调用：内核的访问接口，是一种能再简化的操作

    - man 2 syscalls 
    
    - man 2 acct 
    

- 公共函数库：系统调用的组合拳

- **Shell：命令解释器，可编程**

    - echo $SHELL
    
    - cat /etc/shells


### 2. 查找特定文件

#### 2.1 find

```shell
语法 find path [options] params
作用：在指定目录下查找文件
```

- 默认搜索当前目录

```shell
find -name "target.java"
```

- 从根开始搜索

```shell
find / -name "target.java"
```

- 通配符搜索

```shell
find / -name "target*"
```

- 忽略大小写

```shell
find / -iname "target*"
```

#### 2.2 总结，常用的方式

- find ~ -name "target.java" ：精确查找文件

- find ~ -name "target*" ：模糊查找文件

- find ~ -iname "target*" ：不区分文件名大小写模糊查找文件

- man find ：更多关于 find 指令的使用说明


### 3. 检索文件内容

#### 3.1 grep

```shell
语法 grep [options] pattern file
全称：Global Regular Expresssion Print
作用：查找文件里符合条件的字符串
```

- 检索文件

```shell
grep "haha*" target
```

- **管道操作符 |**

    - 可将指令连接起来，前一个指令的输出作为后一个指令的输入
    
    - command_1 STDOUT|STDIN command_2 STDOUT|STDIN command_3 

- 使用管道注意的要点

    - 只处理前一个命令的正确输出，不处理错误输出
    
    - 右边命令必须能够接受标准输入流，否则传递过程中数据会被抛弃
    
    - sed, awk, grep, cut, head, top, less, more, wc, join, sort, split 等


```shell
grep 'partial\[true\]' data.info.log | grep -o 'engine\[[0-9][a-z]*\]'

>>>
engine[ad75229ffd7d7df5f5d41d17b9a]
engine[bd75229ffd7d7df5f5d41d17b9a]
engine[cd75229ffd7d7df5f5d41d17b9a]
engine[dd75229ffd7d7df5f5d41d17b63]
engine[7e75229ffd7d7df5f5d41d17b18]
```

- 排除 -v 非过滤

```shell
ps -ef | grep tomcat | grep -v "grep"
```


#### 3.2 总结，常用的方式

- ```grep 'partial\[true\]' data.info.log | grep -o 'engine\[[0-9][a-z]*\]' ：精确查找文件```

- ```grep -o 'engine\[[0-9][a-z]*\]'```

- ```grep -v "grep"```


### 4. 对日志内容做统计

#### 4.1 awk

```shell
语法 awk [options] 'cmd' file
- 一次读取一行文本，按输入分隔符进行切片，切成多个组成部分
- 将切片直接保存在内建的变量中，$1, $2...（$0 表示行的全部）
- 支持对单个切片的判断，支持循环判断，默认分隔符为空格
```


```shell
cat netstat.txt

awk '{print $1,$4}' netstat.txt
>>>
Proto Local
tcp 115.28.159.6:ssh
tcp localhost:mysql
tcp localhost:mysql
udp localhost:40504
```


```shell
awk '$1=="tcp" && $2==1{print $0}' netstat.txt
```

```shell
cat test.txt
>>>
dafa,123123132
dafa,213
dafa,65432
dafa,3245
dafa,86453
dafa,123123132

awk -F "," {print $2}' test.txt
>>>
123123132
213
65432
3245
86453
123123132

```

```shell
grep 'partial\[true\]' data.info.log | grep -o 'engine\[[0-9][a-z]*\]' | awk '{enginearr[$1]++}END{for(i in enginearr)print i "\t" enginearr[i]}'
>>>
engine[ad75229ffd7d7df5f5d41d17b9a]  4
engine[bd75229ffd7d7df5f5d41d17b9a]  2
engine[cd75229ffd7d7df5f5d41d17b9a]  7
engine[dd75229ffd7d7df5f5d41d17b63]  10
engine[7e75229ffd7d7df5f5d41d17b18]  10
engine[7e75229ffd7d7df5f5d41d17b18]  8
```

- awk与sort妙用
```shell
echo '$SHELL' 单引号会原样输出
echo "$SHELL" 双引号会解析引号里面的变量
``单反号是执行命令

cat file1 file2 file3 > err.out 2>&1
2>&1是把错误信息打到文件里，没有则是不打印错误信息

查看sda1占用磁盘大小去掉%  -f ":" 以冒号为分隔符
awk是选择第几列，sed是过滤 格式sed 's/要替换的东东/用什么替换(可以为空)/'
df | grep /dev/sda1  | awk '{print $5}' | sed 's/%//'

sort默认是从小到大 -r逆序 -k指第几列 -n为按照数值大小排序
从小到大取最后三行
grep "2014-02-*" gpdata.txt | sort -n -k7 | tail -3
从大到小取头头三行
grep "2014-02-*" gpdata.txt | sort -n -k7 -r | head -3

```

- awk 之 asort 与 asorti 数组排序区别及演示

```shell
awk '{{enginearr[$1]++}END{slen=asorti(enginearr,b);for(i=1;i<=slen;i++) print i "\t" enginearr[i] "\t" enginearr[i]}'

报错：function asort never defined

sudo apt-get install gawk
```

#### 4.2 总结，常用的方式

- ```awk '{print $1,$4}' netstat.txt```

- ```awk '$1=="tcp" && $2==1 {print $0}' netstat.txt```

- ```awk '{enginearr[$1]++}END{for(i in enginearr)print i "\t" enginearr[i]}'```



### 5. 批量替换文本内容

#### 5.1 sed

```shell
语法 sed [option] 'sed cmd' filename
- 全名：stream editor，流编辑器
- 适合用于对文本的行内容进行处理
```


```shell
sed 's/^Str/String/' replace.java
>>> 
String a = "The xx   xxxxxxxxxxxxxx".
Str b = "The xx   jackxxxjackxxjackxxxx";
String c = "The xx   xxxxxxxxxxxxxx".


Integer bf = new Integer(2);
```

```shell
cat replace.java
>>>
Str a = "The xx   xxxxxxxxxxxxxx".
Str b = "The xx   jackxxxjackxxjackxxxx";
Str c = "The xx   xxxxxxxxxxxxxx".


Integer bf = new Integer(2);
```

```shell
sed -i 's/\.$/\;/' replace.java

cat replace.java
>>>
Str a = "The xx   xxxxxxxxxxxxxx";
Str b = "The xx   jackxxxjackxxjackxxxx";
Str c = "The xx   xxxxxxxxxxxxxx";


Integer bf = new Integer(2);
```

- / / /g 该行全局替换， 没有g，则代表只替换第一次匹配的字符串
```shell
sed -i 's/jack/me/' replace.java

cat replace.java
>>>
Str a = "The xx   xxxxxxxxxxxxxx";
Str b = "The xx   mexxxjackxxjackxxxx";
Str c = "The xx   xxxxxxxxxxxxxx";


Integer bf = new Integer(2);
```

```shell
sed -i 's/jack/me/g' replace.java

cat replace.java
>>>
Str a = "The xx   xxxxxxxxxxxxxx";
Str b = "The xx   mexxxmexxmexxxx";
Str c = "The xx   xxxxxxxxxxxxxx";


Integer bf = new Integer(2);
```

- / / /d 

```shell
sed -i '/Integer/d' replace.java

cat replace.java
>>>
Str a = "The xx   xxxxxxxxxxxxxx";
Str b = "The xx   mexxxmexxmexxxx";
Str c = "The xx   xxxxxxxxxxxxxx";
```


#### 5.2 总结，常用的方式

- ```sed 's/^Str/String/' replace.java```

- ```sed -i 's/\.$/\;/' replace.java```

- ```sed -i 's/jack/me/g' replace.java```

### 5. 总结

#### 5.1 常用的 Shell 指令

- find

- grep

- 管道操作符 |

- awk

- sed

## 感谢

[awk 之 asort 与 asorti 数组排序区别及演示](http://blog.chinaunix.net/uid-21374062-id-3189744.html)

[awk 数组排序教程(内置函数排序与自定义排序函数)](http://linux.it.net.cn/e/command/2015/0502/14989.html)