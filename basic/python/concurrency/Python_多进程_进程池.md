## 先了解多线程使用的场景
1. io操作不占用cpu
2. 计算占用cpu
3. python多线程 不合适cpu密集操作型的任务，适合io操作密集型的任务

### 进程之间通信

#### 1.mutilprocess.Queue
使用方法跟threading里的queue差不多，我们来看一下区别
<pre>
# threading
import threading
import Queue

def f():
    q.put([42, None, 'hello'])

if __name__ == '__main__':
    q = Queue.Queue()
    p = threading.Thread(target=f,)
    p.start()
    print (q.get())
    p.join()

结果
>>>
[42, None, 'hello']
</pre>

<pre>
# Process
from multiprocessing import Process
import Queue

def f():
    q.put([42, None, 'hello'])

if __name__ == '__main__':
    q = Queue.Queue()
    p = Process(target=f,)
    p.start()
    print (q.get())
    p.join()
结果
>>>
NameError: global name 'q' is not defined
</pre>

<pre>
# 把线程Queue传给一个进程会出现什么结果呢？
from multiprocessing import Process
import Queue


def f(qq):
    qq.put([42, None, 'hello'])

if __name__ == '__main__':
    q = Queue.Queue()
    p = Process(target=f, args=(q, ))
    p.start()
    print (q.get())
    p.join()

结果
>>>
TypeError: can't pickle thread.lock objects
</pre>

<pre>
# 进程之间数据交互
import multiprocessing
from multiprocessing import Process

def f(qq):
    qq.put([42, None, 'hello'])

if __name__ == '__main__':
    q = multiprocessing.Queue()
    p = Process(target=f, args=(q, ))  # 这个q是克隆的，其实是两个queue，有一个中间翻译；
    # 子进程的queue放入数据后需要序列化之后通过中间翻译反序列化放进父进程的queue，看上去是共享queue，其实是两个queue
    p.start()
    print (q.get())
    p.join()

结果
>>>
[42, None, 'hello']



import multiprocessing
from multiprocessing import Process

def f(qq):
    qq.put([42, None, 'hello'])
    print (qq.__repr__)

if __name__ == '__main__':
    q = multiprocessing.Queue()
    p = Process(target=f, args=(q, ))  # 这个q是克隆的，其实是两个queue，有一个中间翻译，
    # 子进程的queue需要序列化之后反序列化放进父进程的queue，看上去是共享queue，其实是两个queue
    p.start()
    print (q.get())
    print (q.__repr__)
    p.join()

结果
>>>
<method-wrapper '__repr__' of Queue object at 0x00000000037D6A20>
[42, None, 'hello']
<method-wrapper '__repr__' of Queue object at 0x00000000031210B8>
</pre>


#### 2.mutilprocess.Pipe
The Pipe() function returns a pair of connection objects connected by a pipe which by default is duplex (two-way). For example:
<pre>
# 两个进程之间数据的传递
from multiprocessing import Process, Pipe

def f(conn):
    conn.send([42, None, 'hello from child'])
    conn.send([42, None, 'hello from child2'])
    conn.close()


if __name__ == '__main__':
    parent_conn, child_conn = Pipe()
    p = Process(target=f, args=(child_conn,))
    p.start()
    print(parent_conn.recv())  # prints "[42, None, 'hello']"
    print(parent_conn.recv())  # prints "[42, None, 'hello']"
    # print(parent_conn.recv())  # prints "[42, None, 'hello']"
    
    p.join()

结果
>>>
[42, None, 'hello from child']
[42, None, 'hello from child2']
</pre>

#### 3.Manager
A manager object returned by Manager() controls a server process which holds Python objects and allows other processes to manipulate them using proxies.

A manager returned by Manager() will support types list, dict, Namespace, Lock, RLock, Semaphore, BoundedSemaphore, Condition, Event, Barrier, Queue, Value and Array. For example,

<pre>
# 进程直接数据的共享
# manager已经帮你加锁了（其实是克隆，之后在合并，看似合并）

from multiprocessing import Process, Manager
import os

def f(d, l):
    d[os.getpid()] = os.getpid()

    l.append(os.getpid())
    print(l)


if __name__ == '__main__':
    with Manager() as manager:
        d = manager.dict()  # 生成一个字典，可在多个进程间共享和传递

        l = manager.list(range(5))  # 生成一个列表，可在多个进程间共享和传递
        p_list = []
        for i in range(10):
            p = Process(target=f, args=(d, l))
            p.start()
            p_list.append(p)
        for res in p_list:  # 等待结果
            res.join()

        print(d)
        print(l)


结果
>>>
[0, 1, 2, 3, 4, 3228]
[0, 1, 2, 3, 4, 3228, 15168]
[0, 1, 2, 3, 4, 3228, 15168, 5576]
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628]
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628, 5288]
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628, 5288, 4392]
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628, 5288, 4392, 14832]
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628, 5288, 4392, 14832, 9916]
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628, 5288, 4392, 14832, 9916, 6740]
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628, 5288, 4392, 14832, 9916, 6740, 6532]
{15168: 15168, 6628: 6628, 5576: 5576, 9916: 9916, 14832: 14832, 5288: 5288, 6740: 6740, 6532: 6532, 3228: 3228, 4392: 4392}
[0, 1, 2, 3, 4, 3228, 15168, 5576, 6628, 5288, 4392, 14832, 9916, 6740, 6532]

</pre>


#### 4.进程同步

Without using the lock output from the different processes is liable to get all mixed up.
lock
<pre>
from multiprocessing import Process, Lock


def f(l, i):
    l.acquire()
    try:
        print('hello world', i)
    finally:
        l.release()


if __name__ == '__main__':
    lock = Lock()

    for num in range(10):
        Process(target=f, args=(lock, num)).start()

结果
>>>
('hello world', 1)
('hello world', 0)
('hello world', 2)
('hello world', 3)
('hello world', 4)
('hello world', 6)
('hello world', 7)
('hello world', 5)
('hello world', 8)
('hello world', 9)

# 既然进程互相独立，为什么还要锁呢？其实他们在共享一块屏幕，防止打印的时候，数据不会乱。
# 不加锁，可能会出现下面的状况
('hello world', 7)
('hello wo('hello world', 8)
rld', 9)
</pre>

#### 5.进程池　　

进程池内部维护一个进程序列，当使用时，则去进程池中获取一个进程，如果进程池序列中没有可供使用的进进程，那么程序就会等待，直到进程池中有可用进程为止。

进程池中有两个方法：

- apply # 串行
- apply_async # 并行，但必须加pool.join(),

<pre>
#!/usr/bin/env python
# -*- coding:utf-8 -*-

__Author__ = "HackFun"
__Date__ = '2017/9/15 22:49'

from  multiprocessing import Process, Pool, freeze_support
import time
import os


def Foo(i):
    time.sleep(2)
    print ("in process", os.getpid())
    return i + 100


def Bar(arg):
    print('-->exec done:', arg)

if __name__ == '__main__':
    # freeze_support()

    pool = Pool(processes=5)  # processes=5 允许进程池里同时放入5个进程
	
	print ("主进程", os.getpid())

    for i in range(10):
        pool.apply_async(func=Foo, args=(i,), callback=Bar)
        # callback=回调,比如备份到数据库，log日志可以用这个 

        # pool.apply_async(func=Foo, args=(i,))
        # 10个进程都启动了，只是类似挂起不执行，只有五个cpu处理,
        # 在linux下可以ps看一下，此时是有10个进程的，只不过，只有五个执行。

        # pool.apply(func=Foo, args=(i,))

    print('end')
    pool.close()
    pool.join()  # 进程池中进程执行完毕后再关闭，如果注释，那么程序直接关闭。一定要线关闭进程池再join，自己试试区别。


结果
>>>
('\xe4\xb8\xbb\xe8\xbf\x9b\xe7\xa8\x8b', 2480)
end
('in process', 4624)
('in process', 16100)
('-->exec done:', 100, 2480)
('in process', 11156)
('in process', 14836)
('-->exec done:', 101, 2480)
('in process', 15840)
('in process', 4624)
('-->exec done:', 102, 2480)
('in process', 16100)
('in process', 11156)
('-->exec done:', 103, 2480)
('in process', 14836)
('in process', 15840)
('-->exec done:', 104, 2480)
('in process', 4624)
('in process', 16100)
('-->exec done:', 105, 2480)
('in process', 11156)
('in process', 14836)
('-->exec done:', 106, 2480)
('in process', 15840)
('in process', 4624)
('-->exec done:', 107, 2480)
('in process', 16100)
('in process', 11156)
('-->exec done:', 108, 2480)
('in process', 14836)
('in process', 15840)
('-->exec done:', 109, 2480)


由上面结果可以看出callback回调函数是由父进程来做的。
这样的好处：例如，连接数据库备份，父进程只需要连接一次，而子进程要连接多次。
</pre>


## 参考
[Python之路,Day9 - 异步IO\数据库\队列\缓存](http://www.cnblogs.com/alex3714/articles/5248247.html)

[Python之路,Day9, 进程、线程、协程篇](http://www.cnblogs.com/alex3714/articles/5230609.html)
