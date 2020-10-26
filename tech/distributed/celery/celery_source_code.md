[Draft]Celery 源码阅读

- @task 构造类

[@task](https://github.com/hackfengJam/celery/blob/21baef53c39bc1909fd6eee9a2a20e6ce851e88c/celery/app/base.py#L362)


- 自动重试 
    autoretry_for celery/app/base.py _task_from_fun

[_task_from_fun](https://github.com/hackfengJam/celery/blob/21baef53c39bc1909fd6eee9a2a20e6ce851e88c/celery/app/base.py#L430)




* 自动重试 autoretry_for celery/app/base.py _task_from_fun
* task类 celery/app/base.py _task_from_fun
* 重试 celery/app/task.py retry(self 
* 任务执行 celery/app/trace.py build_tracer.trace_task
    * on_error
* 发送 
    * celery/app/consumer.py connect
    * celery/app/base.py connection_for_read 
* Request 
    * celery/celery/worker/request.py
* 持久化
    * 持久化 celery/backends/base.py mark_as_done 
* Worker 
    * celery/worker/worker.py 
        * self._conninfo = self.app.connection_for_read()
* TaskId 生成策略
    * uuid
