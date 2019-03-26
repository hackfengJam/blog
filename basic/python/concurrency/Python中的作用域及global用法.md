## Python中的作用域及global用法

#### 1.作用域

> Python中，一个变量的作用域总是由代码中被赋值的地方所决定的
<pre>
函数定义了本地作用域，而模块定义的是全局作用域。
如果想要在函数内定义全局作用域，需要加上global修饰符。

变量名解析：LEGB原则
当在函数中使用未认证的变量名时，Python搜索４个作用域
[本地作用域(L)(函数内部声明但没有使用global的变量),
  之后是上一层结构中def或者lambda的本地作用域(E),
   之后是全局作用域(G)（函数中使用global声明的变量或在模块层声明的变量）,
    最后是内置作用域(B)（即python的内置类和函数等）］
并且在第一处能够找到这个变量名的地方停下来。如果变量名在整个的搜索过程中都没有找到，Python就会报错。

补：
上面的变量规则只适用于简单对象，当出现引用对象的属性时，则有另一套搜索规则:
属性引用搜索一个或多个对象，而不是作用域，并且有可能涉及到所谓的"继承"。
</pre>

#### 2.globle用法

> Python在模块层定义的变量（无需global修饰），如果在函数中没有再定义同名变量，可以在函数中当做全局变量使用，如下例子：

<pre>
hackfun = 11

def test_global():
    print('in_fun: ', hackfun)

test_global()
print('out_fun:　', hackfun)


结果
in_fun:  11
out_fun:　 11
</pre>

> 但如果在函数中使用此变量之后，再赋值/定义（因为Python是弱类型语言，赋值语句和其定义变量的语句一样），则会产生引用为定义变量的错误。如下例子：
<pre>
hackfun = 11

def test_global():
    print('in_fun: ', hackfun)
    hackfun = 2

test_global()
print('out_fun:　', hackfun)


结果
UnboundLocalError: local variable 'hackfun' referenced before assignment
</pre>

> 而如果在函数中的的再赋值/定义，在使用之前，则会正常运行，但需要注意的是，函数中的变量和模块中定义的全局变量不为同一个。如下例子：
<pre>
hackfun = 11
def test_global():
    hackfun = 2
    print('in_fun: ', hackfun)

test_global()
print('out_fun:　', hackfun)


结果
in_fun:  2
out_fun:　 11
</pre>

> 那么如果，我可能在函数使用某一变量后又对其进行修改（即再赋值），怎么让函数里面的变量是模块层定义的那个全局变量而不是函数内部的局部变量呢？这时候global修饰符就派上用场了。如下例子：
<pre>
hackfun = 11
def test_global():
    global hackfun
    print('in_fun: ', hackfun)
    hackfun = 2

test_global()
print('out_fun:　', hackfun)


结果
in_fun:  11
out_fun:　 2
</pre>

在用global修饰符声明hackfun后（注：global语句不允许同时进行赋值，如global hackfun = 2是不允许的）上述的到11 和 2，得到了我们想要的效果。

### 参考
[Python中的作用域及global用法](http://www.cnblogs.com/summer-cool/p/3884595.html)

[ Python容易混淆的地方](http://blog.csdn.net/carolzhang8406/article/details/6855525 " Python容易混淆的地方")