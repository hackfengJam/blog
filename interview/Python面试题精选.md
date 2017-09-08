## Python面试题

### 试题：NO.01
<strong>1.Python下多线程的限制以及多线程中传递参数的方式？</strong><br>

<p>python多线程有个全局解释器锁（global interpeter lock），这个锁的意思是任意时间只能有一个线程使用解释器，跟单cpu跑多个程序的意思，大家都是轮着用的，这叫“并发”，不是“并行”。
多进程间共享数据，可以使用multiprocessing.Value 和 multiprocessing.Array。
</p>
### 试题：NO.02
<strong>2.Python下是如何进行内存管理的？</strong>
<p>Python引用了一个内存池（Memory pool）机制，即Pymalloc机制（malloc：n.分配内存）,用于管理对小块内存的申请和释放。</p>
<p>内存池（Memory pool）的概念当创建大量消耗小内存的对象时，频繁调用new/malloc会导致大量的内存碎片，致使效率降低。内存池的概念就是预先在内存中申请一定数量的，大小相等的内存块留作备用，当有新的内存需求时，就先从内存池中分配给这个需求，不够了，之后再申请新的内存。这样做最显著的优势就是能够减少内存碎片，提升效率。内存池的实现方式有很多，性能和适用范围也不一样。</p>
<p>Python中内存管理机——Pymalloc：python中的内存管理机制都有两套实现，一套是针对小对象，就是大小小于256bits时，pymalloc会在内存池中申请内存空间；当大于256bits，则会直接执行new/malloc的行为来申请内存空间。关于释放内存方面，当一个对象的引用计数变为0时，python就会调用它的析构函数。在析构时，也采用了内存池机制，从内存池来的内存会被归还到内存池中，以避免频繁地释放动作。
</p>
### 试题：NO.03
<strong>3.什么是lambad函数？它有什么好处？</strong>
<p>lambda 函数式一个可以接受任意多个参数（包括可选参数）并且返回单个表达式不能超过一个。不要试图向lambda函数中塞入太多的东西；如果你需要更复杂的东西，应该定义一个普通函数，然后想它多长就多长</p>
### 试题：NO.04
<strong>4.如何Python输出一个Fibonacci数列？</strong>
<pre>
a, b = 0, 1  
while b < 100:
    print(b)
    a, b = b, a+b
</pre>
### 试题：NO.05
<strong>5.介绍一下Python中webbrowser的用法</strong><br>
webbrowser模块提供一个高级接口来显示基于Web的文档，大部分情况下只需要简单的调用*open()*方法。<br>
webbrowser定义了如下异常：
exception webbrowser.Error，当浏览器控件发生错误是会抛出这个异常
webbrowser有一下方法：webbrowser.open("url", new=0, autoraise=1)这个方法是在默认的浏览器中显示url，如果new = 0，那么url会在同一个浏览器窗口下打开，如果new = 1，会打开一个新的窗口，如果new = 2，会打开一个新的tab，如果autoraise=true，窗口会自动增长。
webbrowser.open_new(url)
在默认浏览器中打开一个新的窗口来显示url，否则，在仅的浏览器窗口中打开url

webbrowser.open_new_tab(url)
在默认浏览器中挡开一个新的tab来显示url，否则跟*open_new()*一样

webbrowser.get([name]) 根据name返回一个浏览器对象，如果name为空，则返回默认的浏览器
>
<pre>
chromePath = r'你的浏览器目录'
webbrowser.register('chrome', None, webbrowser.BackgroundBrowser(chromePath))
webbrowser.get('chrome').open('www.baidu.com', new=1, autoraise=True)
</pre>

注册一个名字为*chrome*的浏览器，如果这个浏览器类型被注册就可以用`get()`方法来获取
### 试题：NO.06
<strong>6.解释一下python的and-or语法</strong>
<p>与C表达式 <code>bool ? a : b </code>类似，但是<code>bool and a or b</code>,当a 为假时，不会像C表达式<code>bool ? a : b </code>一样工作</p>应该将and-or技巧封装成一个函数：
<pre>
def choose(bool, a, b):
&nbsp;&nbsp;&nbsp;&nbsp;return (bool and [a] or [b])[0]
print(choose(True,1,2))
>>>
1
</pre>

因为[a] 是一个非空列表，它永远不会为假。甚至a是 `0` 或 `''`或者其他假值，列表[a]为真，因为它有一个元素。
### 试题：NO.07 
<strong>7.How do I iterate over a sequence in reverse order?</strong><br>
一种方法，如果是list，则直接调用其`reverse()`方法
>
<pre>
sequence = [1, 2, 3, 4, 5]
sequence.reverse()
print(sequence)
>>>
[5, 4, 3, 2, 1]
</pre>
<p>还有一种最通用但稍慢的解决方案是：</p>
<pre>
sequence = [1, 2, 3, 4, 5]
x = [sequence[i] for i in range(len(sequence)-1, -1, -1)]
    print(x)
>>>
[5, 4, 3, 2, 1]
</pre>
### 试题：NO.08
<strong>8.Python是如何进行类型转换的？</strong>

<pre>
>>>int('1234')                      # 将数字型字符串转为整形
1234 
>>>float(12)                        # 将整形或数字字符转为浮点型
12.0  
>>>str(98)                          # 将其他类型转为字符串型
'98'  
>>>list('abcd')                     # 将其他类型转为列表类型
['a', 'b', 'c', 'd']  
>>>dict.fromkeys(['name','age'])    # 将其他类型转为字典类型
{'age': None, 'name': None}  
>>>tuple([1, 2, 3, 4])              # 将其他类型转为元祖类型
(1, 2, 3, 4)  
</pre>
<pre>
     函数                                         描述
int(x [,base])                               将x转换为一个整数
long(x [,base] )                             将x转换为一个长整数
float(x)                                     将x转换到一个浮点数
complex(real [,imag])                        创建一个复数
str(x)                                       将对象 x 转换为字符串
repr(x)                                      将对象 x 转换为表达式字符串
eval(str)                                    用来计算在字符串中的有效Python表达式,并返回一个对象
tuple(s)                                     将序列 s 转换为一个元组
list(s)                                      将序列 s 转换为一个列表
set(s)                                       转换为可变集合
dict(d)                                      创建一个字典。d 必须是一个序列 (key,value)元组。
frozenset(s)                                 转换为不可变集合
chr(x)                                       将一个整数转换为一个字符
unichr(x)                                    将一个整数转换为Unicode字符
ord(x)                                       将一个字符转换为它的整数值
hex(x)                                       将一个整数转换为一个十六进制字符串
oct(x)                                       将一个整数转换为一个八进制字符串  </pre>
### 试题：NO.09
<strong>9.Python里面如何实现tuple和list的转换？</strong>
<pre>
# tuple转换成list
def to_list(t):
    return [i if not isinstance(i, tuple) else to_list(i) for i in t]

# 这个返回的生成器千万要注意，见下面打印结果
def to_generator(l):
    return (i if not isinstance(i, list) else to_tuple(i) for i in l)

# list
>>>a_list = [1, 2, [1, 2, 3], 3, 4, 5]
>>>print(tuple(a_list))
(1, 2, [1, 2, 3], 3, 4, 5)

>>>print(type(to_generator(a_list)))
< class 'generator'>

>>>print(to_generator(a_list))
< generator object to_generator.<locals>.<genexpr> at 0x000001F37F687150>

# tuple 转换成 list
>>>a_tuple = (1, 2, (1, 2, 3), 3, 4)
>>>print(list(a_tuple))
[1, 2, (1, 2, 3), 3, 4]

>>>print(type(to_list(a_tuple)))
< class 'list'>

>>>print(to_list(a_tuple))
[1, 2, [1, 2, 3], 3, 4]
</pre>
### 试题：NO.10
<strong>10.请写出一段Python代码实现删除一个list里面的重复元素？</strong>
>
<pre>
>>>L1 = [4, 1, 3, 2, 3, 5, 1]
>>>L2 = []
>>>[L2.append(i) for i in L1 if i not in L2]
>>>print(L2)
[4, 1, 3, 2, 5]
</pre>
### 试题：NO.11
<strong>11.Python里面如何实现单例模式？其他23中设计模式python如何实现？</strong>
<pre>

</pre>
### 试题：NO.12
<strong>12.Python里面如何copy一个对象？</strong>
>
<pre>
切片S[:]  # 注不能应用于字典  
深浅拷贝  # 能应用于所有序列和字典  
1. 浅拷贝S.copy()方法  
2. 深拷贝deepcopy(S)方法  
</pre>

例子 1:
>
<pre>
from copy import deepcopy
L1 = [1, [1, 2, 3], 2, 3]
print("before copy L1: ", L1)
L2 = L1.copy()
L2[1][2] = 1
print("after copy L2: ", L2)
print("after copy L1: ", L1)
</pre>

>运行结果:
<pre>
before copy L1:  [1, [1, 2, 3], 2, 3]
after copy L2:  [1, [1, 2, 1], 2, 3]
after copy L1:  [1, [1, 2, 1], 2, 3]
</pre>

例子 2:
<pre>
L1 = [3, [3, 4, 5], 4, 5]
print("before deepcopy L1: ", L1)
L2 = deepcopy(L1)
L2[1][2] = 1<br>
print("after deepcopy L2: ", L2)
print("after deepcopy L1: ", L1)
</pre>

>运行结果 :
<pre>
before deepcopy L1:  [3, [3, 4, 5], 4, 5]
after deepcopy L2:  [3, [3, 4, 1], 4, 5]
after deepcopy L1:  [3, [3, 4, 5], 4, 5]
</pre>
### 试题：NO.13
<strong>13.如何用Python来进行查询和替换一个文本字符串？</strong>
<pre>>>> words = 'Python is a very funny language!'  
>>> words.find('Python')                # 返回的为0或正数时，为其索引号  
0  
>>> words.find('is')  
7  
>>> words.find('hackfun')               # 返回-1表示查找失败  
-1  
>>> words.replace('Python', 'Java')     # replace()替换  
'Java is a very funny language!'  
</pre>
### 试题：NO.14
<strong>14.Python里面search()和match()的区别？</strong>
<pre>
>>> import re  
>>> re.match(r'python','Programing Python, should be pythonic')  
>>> obj1 = re.match(r'python','Programing Python, should be pythonic')  #返回None  
>>> obj2 = re.search(r'python','Programing Python, should be pythonic') #找到pythonic  
>>> obj2.group()  
'python'  
#re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；  
#re.search匹配整个字符串，直到找到一个匹配。  
</pre>
<pre>
>>> str = """
...         one line
...         two line
...         three line
...         """
>>> obj1 = re.match(r".*(three).*", str)
>>> print(obj1)
None
>>> obj2 = re.search(r"three", str)
>>> print(obj2.group())
three

# match要匹配全部行，需要加 re.DOTALL
>>> obj1 = re.match(r".*(three).*", str, re.DOTALL)
>>> print(obj1)
<_sre.SRE_Match object at 0x0000000002CAFBE8>
>>> print(obj1.group(1))
three
#re.match 默认只匹配一行，若要匹配全部需要加第三个参数: re.DOTALL；  
#re.search匹配整个字符串。
</pre>
### 试题：NO.15
<strong>15.有两个序列a,b，大小都为n，序列元素的值任意整型数，无序？</strong>
### 试题：NO.16
<strong>16.Python程序中文输出问题怎么解决？</strong>
<pre>
在Python3中，对中文进行了全面的支持，但在Python2.x中需要进行相关的设置才能使用中文。否则会出现乱码。  
Python默认采取的ASCII编码，字母、标点和其他字符只使用一个字节来表示，但对于中文字符来说，一个字节满足不了需求。  
为了能在计算机中表示所有的中文字符，中文编码采用两个字节表示。如果中文编码和ASCII混合使用的话，就会导致解码错误，从而才生乱码。  
解决办法:  
交互式命令中：一般不会出现乱码，无需做处理   
py脚本文件中：跨字符集必须做设置，否则乱码  
1. 首先在开头一句添加:  
# coding = utf-8    
# 或    
# coding = UTF-8    
# 或    
# -*- coding: utf-8 -*-   
2. 其次需将文件保存为UTF-8的格式！  
3. 最后: s.decode('utf-8').encode('gbk')
</pre>
### 试题：NO.17
<strong>17.Python如何捕捉异常？</strong>
<pre>

程序中出现异常情况时就需要异常处理。比如当你打开一个不存在的文件时。当你的程序中有  
一些无效的语句时，Python会提示你有错误存在。下面是一个拼写错误的例子，print写成了Print  
下面是异常最常见的几种角色  
1. 错误处理  
>>>可以在程序代码中捕捉和相应错误，或者忽略已发生的异常。  
>>>如果忽略错误，PYTHON默认的异常处理行为将启动:停止程序，打印错误信息。  
>>>如果不想启动这种默认行为，就用try语句来捕捉异常并从异常中恢复。  
2. 事件通知  
>>>异常也可用于发出有效状态的信号，而不需在程序间传递结果标志位。或者刻意对其进行测试  
3. 特殊情况处理  
>>>有时，发生了某种很罕见的情况，很难调整代码区处理。通常会在异常处理中处理，从而省去应对特殊情况的代码  
4. 终止行为  
>>>try/finally语句可确保一定会进行需要的结束运算，无论程序是否有异常  
5. 非常规控制流程 
</pre>
### 试题：NO.18
<strong>18.Python代码得到列表list的交集与差集？</strong>
<pre>
>>> list1 = [1, 2, 3, 4, 5]  
>>> list2 = [4, 5, 6, 7, 8, 9] 
>>> [i for i in list1 if i not in list2]  
[1, 2, 3]  
>>> [i for i in list1 if i in list2]  
[4, 5] 
</pre>
### 试题：NO.19
<strong>19.如何用Python来发送邮件？</strong>
### 试题：NO.20
<strong>20.介绍一下except</strong>
<pre>
try/except:          捕捉由PYTHON自身或写程序过程中引发的异常并恢复  
except:              捕捉所有其他异常  
except name:         只捕捉特定的异常  
except name, value:  捕捉异常及格外的数据(实例)  
except (name1,name2) 捕捉列出来的异常  
except (name1,name2),value: 捕捉任何列出的异常，并取得额外数据  
else:                如果没有引发异常就运行  
finally:             总是会运行此处代码  
 </pre>

### 试题：NO.21
<strong>21.src = "security/afafsff/?ip=123.4.56.78&id=45"，请写一段代码用正则匹配出IP</strong> 
<pre>
def get_ip():
    import re
    src = "security/afafsff/?ip=123.4.56.78&id=45"
    match_obj = re.match(".*\?ip=(\d{1,3}[.]\d{1,3}[.]\d{1,3}[.]\d{1,3})&.*", src)
    if match_obj:
        print(match_obj.group(1))

    
    search_obj = re.search("ip=(\d{1,3}.\d{1,3}.\d{1,3}.\d{1,3})", src, re.S) # re.S 改变'.'的行为
    # search_obj = re.search("ip=(\d{1,3}[.]\d{1,3}[.]\d{1,3}[.]\d{1,3})", src) # 这个正则就可以re.S
    if search_obj:
        print(search_obj.group(1))

>>>get_ip()
# 输出结果
>>>
123.4.56.78
123.4.56.78
</pre>