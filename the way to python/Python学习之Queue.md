
## Python3 queue
>Python2 中为 Queue

##### 1.Python queue 模块有三种队列：
<pre>
1.python queue模块得FIFO队列先进先出。
2.LIFO类似于堆，即先进后出。
3.还有一种是优先级队列级别越低越先出来。
</pre>

##### 2.针对这三种队列分别有三个构造函数:
<pre>
1.class queue.Queue(maxsize) FIFO
2.class queue.LifoQueue(maxsize) LIFO
3.class queue.PriorityQueue(maxsize) 优先级队列
</pre>
##### 3.介绍一下此包中的常用方法
<pre>
queue.qsize() 返回队列的大小
queue.empty() 如果队列为空，返回True,反之False
queue.full() 如果队列满了，返回True,反之False
queue.full 与 maxsize 大小对应
queue.get([block[, timeout]]) 获取队列，timeout等待时间
queue.get_nowait() 相当Queue.get(False)
非阻塞 queue.put(item) 写入队列，timeout等待时间
queue.put_nowait(item) 相当Queue.put(item, False) 
queue.task_done() 在完成一项工作之后，queue.task_done() 函数向任务已经完成的队列发送一个信号
queue.join() 实际上意味着等到队列为空，再执行别的操作
</pre>

**task_done()**<br>
意味着之前入队的一个任务已经完成。由队列的消费者线程调用。每一个```get()```调用得到一个任务，接下来的```task_done()```调用告诉队列该任务已经处理完毕。
如果当前一个join()正在阻塞，它将在队列中的所有任务都处理完时恢复执行（即每一个由```put()```调用入队的任务都有一个对应的```task_done()```调用）。

***join()***<br>
阻塞调用线程，直到队列中的所有任务被处理掉。
只要有数据被加入队列，未完成的任务数就会增加。当消费者线程调用```task_done()```（意味着有消费者取得任务并完成任务），未完成的任务数就会减少。当未完成的任务数降到0，```join()```解除阻塞。

***put(item[, block[, timeout]])***<br>
将item放入队列中。<br>
    1. 如果可选的参数block为True且timeout为空对象（默认的情况，阻塞调用，无超时）。<br>
	2. 如果timeout是个正整数，阻塞调用进程最多timeout秒，如果一直无空空间可用，抛出Full异常（带超时的阻塞调用）。<br>
	3. 如果block为False，如果有空闲空间可用将数据放入队列，否则立即抛出Full异常其非阻塞版本为```put_nowait```等同于```put(item, False)```

***get([block[, timeout]])***<br>
从队列中移除并返回一个数据。block跟timeout参数同put方法
其非阻塞方法为```get_nowait()```相当与```get(False)```

***empty()***<br>
如果队列为空，返回```True```,反之返回```False```


