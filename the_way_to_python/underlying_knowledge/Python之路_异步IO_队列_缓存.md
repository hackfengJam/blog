## 协程

协程，又称微线程，纤程。英文名Coroutine。一句话说明什么是线程：协程是一种用户态的轻量级线程。

协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此：

协程能保留上一次调用时的状态（即所有局部状态的一个特定组合），每次过程重入时，就相当于进入上一次调用的状态，换种说法：进入上一次离开时所处逻辑流的位置。

 

### 1.协程的好处：

无需线程上下文切换的开销
无需原子操作锁定及同步的开销
　　"原子操作(atomic operation)是不需要synchronized"，所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。原子操作可以是一个步骤，也可以是多个操作步骤，但是其顺序是不可以被打乱，或者切割掉只执行部分。视作整体是原子性的核心。
方便切换控制流，简化编程模型
高并发+高扩展性+低成本：一个CPU支持上万的协程都不是问题。所以很适合用于高并发处理。
 

### 2.缺点：

无法利用多核资源：协程的本质是个单线程,它不能同时将 单个CPU 的多个核用上,协程需要和进程配合才能运行在多CPU上.当然我们日常所编写的绝大部分应用都没有这个必要，除非是cpu密集型应用。
进行阻塞（Blocking）操作（如IO时）会阻塞掉整个程序

#### 3.yield实现简单的协程
<pre>

def consumer(name):
    print("--->starting eating baozi...")
    while True:
        new_baozi = yield
        print("[%s] is eating baozi %s" % (name, new_baozi))
        # time.sleep(1)


def producer():
    r = con.next()  # python3下是__next__()
    r = con2.next()
    n = 0
    while n < 5:
        n += 1
        con.send(n)  # 
        con2.send(n)
        print("\033[32;1m[producer]\033[0m is making baozi %s" % n)

if __name__ == '__main__':
    con = consumer("c1")
    con2 = consumer("c2")
    p = producer()

结果
>>>
--->starting eating baozi...
--->starting eating baozi...
[c1] is eating baozi 1
[c2] is eating baozi 1
[producer] is making baozi 1
[c1] is eating baozi 2
[c2] is eating baozi 2
[producer] is making baozi 2
[c1] is eating baozi 3
[c2] is eating baozi 3
[producer] is making baozi 3
[c1] is eating baozi 4
[c2] is eating baozi 4
[producer] is making baozi 4
[c1] is eating baozi 5
[c2] is eating baozi 5
[producer] is making baozi 5
</pre>

看楼上的例子，我问你这算不算做是协程呢？你说，我他妈哪知道呀，你前面说了一堆废话，但是并没告诉我协程的标准形态呀，我腚眼一想，觉得你说也对，那好，我们先给协程一个标准定义，即符合什么条件就能称之为协程：

1. 必须在只有一个单线程里实现并发
2. 修改共享数据不需加锁
3. 用户程序里自己保存多个控制流的上下文栈
4. 一个协程遇到IO操作自动切换到其它协程

基于上面这4点定义，我们刚才用yield实现的程并不能算是合格的线程，因为它有一点功能没实现，哪一点呢？


### 4.Greenlet

pip install gevent 

greenlet是一个用C实现的协程模块，相比与python自带的yield，它可以使你在任意函数之间随意切换，而不需把这个函数先声明为generator

<pre>
# greenlet（手动切换）
from greenlet import greenlet


def xiecheng_test():
    print(12)
    gr2.switch()
    print(34)
    gr2.switch()


def xiecheng_test2():
    print(56)
    gr1.switch()
    print(78)


gr1 = greenlet(xiecheng_test)
gr2 = greenlet(xiecheng_test2)
gr1.switch()

结果
>>>
12
56
34
78
</pre>

### Gevent 

Gevent 是一个第三方库，可以轻松通过gevent实现并发同步或异步编程，在gevent中用到的主要模式是Greenlet, 它是以C扩展模块形式接入Python的轻量级协程。 Greenlet全部运行在主程序操作系统进程的内部，但它们被协作式地调度。

<pre>
import gevent

def foo():
    print('Running in foo')
    gevent.sleep(2)
    print('Explicit context switch to foo again')


def bar():
    print('Explicit context to bar')
    gevent.sleep(1)
    print('Implicit context switch back to bar')

gevent.joinall([
    gevent.spawn(foo),  # spawn:产生
    gevent.spawn(bar),
])

结果
>>>
Running in foo
Explicit context to bar
Implicit context switch back to bar
Explicit context switch to foo again
</pre>

<pre>
import gevent


def foo():
    print('Running in foo')
    gevent.sleep(2)
    print('Explicit context switch to foo again')


def bar():
    print('Explicit context to bar')
    gevent.sleep(1)
    print('Implicit context switch back to bar')


def func3():
    print('running func3')
    gevent.sleep(0)
    print('running func3 again')


gevent.joinall([
    gevent.spawn(foo),  # spawn:产生
    gevent.spawn(bar),
    gevent.spawn(func3),
])

结果
>>>
Running in foo
Explicit context to bar
running func3
running func3 again
Implicit context switch back to bar
Explicit context switch to foo again
</pre>

## 参考
[Python之路,Day9 - 异步IO\数据库\队列\缓存](http://www.cnblogs.com/alex3714/articles/5248247.html)

[Python之路,Day9, 进程、线程、协程篇](http://www.cnblogs.com/alex3714/articles/5230609.html)