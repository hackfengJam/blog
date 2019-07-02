## Celery 的正确打开方式

### 前言

#### 1. 以下来自[官网](http://www.celeryproject.org/)，对 Celery 介绍

Celery：分布式任务队列。

Celery 是基于分布式消息传递的异步任务队列/作业队列。它专注于实时操作，但也支持调度。 

执行单元（称为任务）使用多处理，**Eventlet** 或 **gevent** 在单个或多个工作服务器上并发执行。任务可以异步（在后台）或同步执行（等到准备就绪）。

Celery 用于生产系统，每天处理数百万个任务。

<details>
<summary>原文</summary>

Celery: Distributed Task Queue. 

Celery is an asynchronous task queue/job queue based on distributed message passing.	
It is focused on real-time operation, but supports scheduling as well.

The execution units, called tasks, are executed concurrently on a single or more worker servers using multiprocessing, Eventlet, or gevent.
Tasks can execute asynchronously (in the background) or synchronously (wait until ready).

Celery is used in production systems to process millions of tasks a day.
</details>

#### 2. 本文结合「官方文档」及「实际用例」全面了解，Python分布式任务队列模块：Celery，究竟该如何使用。



### 目录

[1. 环境](#1-环境)

[2. config_from_object 方式添加配置](#2-config_from_object-方式添加配置)

[3. 项目（Demo ）介绍 - APP 初始化](#3-项目demo-介绍---app-初始化)

[4. 项目（Demo ）介绍 - 任务调度](#4-项目demo-介绍---任务调度)

[5. 总结](#5-总结)

--- 

[相关博文](#相关博文)

### 1. 环境
- Basic
    - linux/windows/Mac OS
    - python 3.6+
    - redis 3.2+

- Python 库
    - oyaml==0.9
    - celery==4.3.0 (pip install -U "celery[redis]")

### 2. config_from_object 方式添加配置

- 首先 Celery **app** 初始化操作，参考[Application](http://docs.celeryproject.org/en/latest/userguide/application.html?highlight=config#application)。

- 其次要知道 **app** 配置添加方式，可以使用 **app.config_from_object()**。

### 3. 项目（Demo ）介绍 - APP 初始化

#### 3.1 项目目录，部分结构大致如下

<pre>
|
`-- hackfun
     |-- config
          ` -- config.yaml
     |-- core
          | -- __init__.py
          ` -- calc.py
     |-- worker
          | -- __init__.py
          ` -- task_handler.py
     |-- __init__.py
     |-- app.py
     `-- celeryconfig.py

 
</pre>

#### 3.2 config 配置文件以及加载 config 到内存

##### 3.2.1 config 配置文件
<details>
<summary>点击查看 hackfun/config/config.yaml </summary>

```yaml
project_namespace: hackfun
celery:
  app:
    name: "Celery"
    redis:
      host: "127.0.0.1"
      port: 6379
      password: "123456"
      celery_broker_db: 2
      celery_result_db: 3
logging:
    file: "/tmp/hackfun.log"
    level: "INFO"

```

</details>

##### 3.2.2 加载 config 到内存

<details>
<summary>点击查看 hackfun.__init__.py 源码</summary>

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

__Author__ = "HackFun"
__Date__ = "2019/6/20 4:37 PM"

# Build-in Modules
import os

# 3rd-party Modules
import yaml

# Project Modules

base_path = os.path.dirname(os.path.abspath(__file__))

config = {}


def init_config():
    global config
    path = os.environ.get('HACKFUN_CONFIG')
    if path is None:
        with open(base_path + "/config/config.yaml") as f:
            config = yaml.load(f)


init_config()

```

</details>


#### 3.3 实际项目应用参考，源码如下

#### 3.3.1 app 初始化

<details>
<summary>点击查看 hackfun.app.py 源码</summary>

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

__Author__ = "HackFun"
__Date__ = "2019/6/20 4:37 PM"

# Build-in Modules

# 3rd-party Modules
from celery import Celery

# Project Modules
from hackfun import config

def init_celery_app(celery_config):
    # redis
    host = celery_config['redis']['host']
    port = celery_config['redis']['port']
    password = celery_config['redis']['password']
    
    # celery
    celery_broker_db = celery_config['redis']['celery_broker_db']
    celery_result_db = celery_config['redis']['celery_result_db']
    
    broker_url = f'redis://:{password}@{host}:{port}/{celery_broker_db}'
    result_url = f'redis://:{password}@{host}:{port}/{celery_result_db}'
    
    _app_celery = Celery(celery_config["name"])
    _app_celery.conf.update(
        dict(
            broker_url=broker_url,
            result_backend=result_url,
        ))
    return _app_celery


app_celery = init_celery_app(config["celery"]["app"])
app_celery.config_from_object('hackfun.celeryconfig')
```

</details>

#### 3.3.2 celeryconfig 编写

<details>
<summary>点击查看 hackfun.celeryconfig.py 源码</summary>

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

# Build-in Modules
import re

# 3rd-party Modules
from celery.schedules import crontab
from kombu import Queue

# Project Modules
from hackfun import config


def create_queue(queue_name):
    return Queue(queue_name, routing_key=queue_name)


def get_worker_queue(name=None):
    name = name or 'default'
    queue_kw = dict(
        projectName=config.get('project_namespace', 'hackfun'),
        queueName=name,
    )
    # celery -A hackfun.app.app_celery worker -Q hackfun#workerQueue@default -c 1
    return '{projectName}#workerQueue@{queueName}'.format(**queue_kw)


'''
Some fixed Celery configs
'''

# Worker
worker_concurrency = 5
worker_prefetch_multiplier = 1
worker_max_tasks_per_child = 50

# Worker log
worker_hijack_root_logger = False
worker_log_color = False
worker_redirect_stdouts = False

# Queue
task_default_queue = get_worker_queue('default')
task_default_routing_key = task_default_queue
task_queues = [
    # 默认队列  
    create_queue(task_default_queue),
    
    # webhook 任务队列
    create_queue(get_worker_queue('webhook')),
    
    # 内部调用异步任务队列
    create_queue(get_worker_queue('internal')),
]

# Task
task_routes = {
    # webhook 调用系统，单独任务队列
    'webhook.*': {'queue': get_worker_queue('webhook')},
    
    # 内部调用异步任务，放入相应队列
    'internal.*': {'queue': get_worker_queue('internal')},
}

imports = [
]

# Beat
beat_schedule = {}

# Job serialization
task_serializer = 'json'
result_serializer = 'json'
accept_content = ['json']

# Time zone
enable_utc = True
timezone = 'Asia/Shanghai'

# ============= Content for YOUR project below =============
# Tasks
imports.append('hackfun.worker.task_handler')
imports.append('hackfun.worker.timed_task')

beat_schedule['issue_timed_task_issue_daily'] = {
    'daily_task': {
        'task': 'hackfun.worker.timed_task.issue_daily',
        'schedule': crontab(minute="0", hour='10')
    }
}

```

</details>

#### 3.4 注意

- 所有 config 相应释义，参考官方文档 [Configuration and defaults](http://docs.celeryproject.org/en/latest/userguide/configuration.html#configuration-and-defaults)

- 值得注意的是 celeryconfig 中 **imports** ，此项容易被开发者遗漏，所有异步任务 python 模块都需要被导入（各位熟悉python，应该都懂的=。=），**task_handler.py** 和 **timed_task.py** 将在后文介绍


### 4. 项目（Demo ）介绍 - 任务调度

#### 4.1 需要被调度的异步任务方法，应当如何编写？

<details>
<summary>点击查看 hackfun.worker.task_handler.py 部分源码</summary>

```python

# Project Modules
from hackfun.app import app_celery

# 将会消费 internal.* 对应的 queue，即 get_worker_queue('internal')
@app_celery.task(name="internal.print", bind=True, base=BrokerTask)
def internal_print(self, message_body, *args, **kwargs):
    """
    内部调用异步任务 handler
    
    异步任务 print 方法

    :param self:
    :param message_body:
    :param args:
    :param kwargs:
    :return:
    """
    result = "internal_print"
    print(message_body, *args, **kwargs)

    return result

# 将会消费 webhook.* 对应的 queue，即 get_worker_queue('webhook')
@app_celery.task(name="webhook.print", bind=True, base=BrokerTask)
def webhook_print(self, message_body, *args, **kwargs):
    """
    webhook 调用异步任务 handler

    异步任务 print 方法

    :param self:
    :param message_body:
    :param args:
    :param kwargs:
    :return:
    """
    result = "webhook_print"
    print(message_body, *args, **kwargs)

    return result

```

</details>

#### 4.2 异步调用方和执行方，调用举例

- 异步调用方

    - 调用方式
    
        - ```.apply_async(args[, kwargs[, …]])``` 例如：
        
            ```python
            internal_print.apply_async(
                args=("internal_print_haha", "internal_print other args"),
                kwargs=dict(a=1, b=2),
            )
            ```
        
        - ```.apply_async(args[, kwargs[, …]])``` 例如：
        
            ```python
            internal_print.delay(
                "internal_print_haha", "internal_print other args",
                a=1, b=2,
            )
            ```
        
        - 详情参考官方文档 [Calling Tasks](http://docs.celeryproject.org/en/latest/userguide/calling.html?highlight=apply_async#calling-tasks)

    - name="internal.print"，意味着调用者，异步调用此方法，将被投放到 internal.* 对应 Queue
    
- 异步执行方

    - worker启动命令：celery -A hackfun.app.app_celery worker -Q hackfun#workerQueue@internal -c 4
    
    - -Q 代表指定 worker 消费的队列，队列名 请看 celeryconfig.get_worker_queue 方法；
    
    - -c ：worker 并发进程数；
    
    - 启动命令后：将会消费 internal.* 对应的 queue，即 get_worker_queue('internal')
    
    - 将参数传递 internal_print(self, message_body, *args, **kwargs)
    
        - ```self```: base=BrokerTask 对应基类的实例，[BrokerTask源码](https://github.com/hackfengJam/MyBlog-py3/blob/master/utils/celery_test/worker/message_handler.py)
        
        - ```message_body, *args, **kwargs```: 其他参数全部由**调用方**与**执行方**开发人员协调，自定义。
        
        - 用例中：参数类型值将得到如下结果：
        
|   参数列表   |   类型  |   值  |
| ----------- | -------- | ----- |
| message_body| str   | "internal_print_haha"  |
| args        | tuple | ("internal_print other args", )  |
| kwargs      | dict  | dict(a=1, b=2)  |

   
#### 4.3 方法解释

##### 4.3.1 internal_print

```python
@app_celery.task(name="internal.print", bind=True, base=BrokerTask)
def internal_print(self, message_body, *args, **kwargs):
    pass
```
- 异步调用方

    - 调用方式
    
        - apply_async
        
        ```python
        internal_print.apply_async(
            args=("internal_print_haha", "internal_print other args"),
            kwargs=dict(a=1, b=2),
        )
        ```
        
        - delay
        
        ```python
        internal_print.delay(
            "internal_print_haha", "internal_print other args",
            a=1, b=2,
        )
        ```

    - name="internal.print"，意味着调用者，异步调用此方法，将被投放到 internal.* 对应 Queue
    
- 异步执行方

    - worker启动命令：celery -A hackfun.app.app_celery worker -Q hackfun#workerQueue@internal -c 4
    
    - 启动命令后：将会消费 internal.* 对应的 queue，即 get_worker_queue('internal')
    
    - 将参数传递 internal_print(self, message_body, *args, **kwargs)
    
        - ```self```: base=BrokerTask 对应基类的实例，[BrokerTask源码](https://github.com/hackfengJam/MyBlog-py3/blob/master/utils/celery_test/worker/message_handler.py)
        
        - ```message_body, *args, **kwargs```: 其他参数全部由**调用方**与**执行方**开发人员协调，自定义。
        
        - 用例中：参数类型值将得到如下结果：
        
|   参数列表   |   类型  |   值  |
| ----------- | -------- | ----- |
| message_body| str   | "internal_print_haha"  |
| args        | tuple | ("internal_print other args", )  |
| kwargs      | dict  | dict(a=1, b=2)  |

       

##### 4.3.2 webhook_print
```python
@app_celery.task(name="webhook.print", bind=True, base=BrokerTask)
def webhook_print(self, message_body, *args, **kwargs):
    pass
```
- 异步调用方

    - 调用方式
    
        - apply_async
        
        ```python
        webhook_print.apply_async(
            args=("webhook_print_haha", "webhook_print other args"),
            kwargs=dict(a=3, b=4),
        )
        ```
        
        - delay
        
        ```python
        webhook_print.delay(
            "webhook_print_haha", " webhook_print other args",
            a=3, b=4,
        )
        ```

    - name="webhook.print"，意味着调用者，异步调用此方法，将被投放到 webhook.* 对应 Queue

- 异步执行方

    - worker启动命令：celery -A hackfun.app.app_celery worker -Q hackfun#workerQueue@webhook -c 4
    
    - 启动命令后：将会消费 webhook.* 对应的 Queue，即 get_worker_queue('webhook')

    - 将参数传递 webhook_print(self, message_body, *args, **kwargs)
    
        - ```self```: base=BrokerTask 对应基类的实例，[BrokerTask源码](https://github.com/hackfengJam/MyBlog-py3/blob/master/utils/celery_test/worker/message_handler.py)
        
        - ```message_body, *args, **kwargs```: 其他参数全部由**调用方**与**执行方**开发人员协调，自定义。
        
        - 用例中：参数类型值将得到如下结果：
        
|   参数列表   |   类型  |   值  |
| ----------- | -------- | ----- |
| message_body| str   | "webhook_print_haha"  |
| args        | tuple | ("webhook_print other args", )  |
| kwargs      | dict  | dict(a=3, b=4)  |

       


#### 4.4 相应 worker 启动命令？

- worker启动命令：celery -A hackfun.app.app_celery worker -Q hackfun#workerQueue@webhook -c 4

- 更多 worker 启动命令参数，参考官方文档

    - [celery.bin.celery](http://docs.celeryproject.org/en/latest/reference/celery.bin.celery.html#celery-bin-celery)

    - [celery.bin.worker](http://docs.celeryproject.org/en/latest/reference/celery.bin.worker.html#celery-bin-worker)
    

#### 4.4 注意

- 对失败进行重试处理

    - 任务投放（调用方）
        - 如果连接失败，Celery 将自动重试发送消息，并且可以配置重试行为 - 例如重试频率或最大重试次数；- 或者一起禁用。
        
        - 可以在调用时指定，投放失败是否重试：```internal.apply_async((2, 2), retry=False)```
        
        - 详情参考官方文档 [Message Sending Retry](http://docs.celeryproject.org/en/latest/userguide/calling.html?highlight=retry#message-sending-retry)

    - 任务执行（执行方）

        - 可以在装饰器中指定 **重试次数** 以及 **重试任务间隔**：```@app_celery.task(name="internal.print", bind=True, base=BrokerTask, default_retry_delay=30, max_retries=3)```
        
        - ```max_retries = 3```放弃前的最大重试次数。如果设置为None，它将永远不会停止重试。
        
        - ```default_retry_delay = 180```应执行重试任务之前的默认时间（以秒为单位）。默认为3分钟。
        
        - 重试详情，参考官方文档 [Retrying](http://docs.celeryproject.org/en/latest/userguide/tasks.html?highlight=retry#retrying)
        
        - 更多 ```@app.task()``` 参数，参考官方文档 [celery.app.task](http://docs.celeryproject.org/en/latest/reference/celery.app.task.html#celery-app-task)
        
        - 同时 装饰器中 **base=BrokerTask** 为基类，```class BrokerTask(app_celery.Task): ```继承自 **app.Task**
        
        - Celery 官方提供四种handle。详情：[Handlers](http://docs.celeryproject.org/en/latest/userguide/tasks.html?highlight=retry#handlers)
        
            - ```after_return(self, status, retval, task_id, args, kwargs, einfo)```
            
            - ```on_failure(self, exc, task_id, args, kwargs, einfo)```
            
            - ```on_retry(self, exc, task_id, args, kwargs, einfo)```
            
            - ```on_success(self, retval, task_id, args, kwargs)```
        
        - BrokerTask 可以基于**app.Task** 的 [Handlers](http://docs.celeryproject.org/en/latest/userguide/tasks.html?highlight=retry#handlers)
         实现自定义功能。例如，任务执行：
         
            - 重试时，做一些日志打印
            
            - 错误时，是否可以考虑持久化错误任务，后续排查；或者放入一个类似 DLQ（之所以，说类似 DLQ；是因为，严格意义上 DLQ 可能不太恰当，篇幅有限不再赘述：） wiki解释：[Dead letter queue]((https://en.wikipedia.org/wiki/Dead_letter_queue))） 的地方等；
            
            - 成功，可以做一些结果持久化/事件通知等。 

- 内存泄漏问题

    - Github 上 [Celery](https://github.com/celery/celery) 项目，有不少 [Issue](https://github.com/celery/celery/issues?utf8=%E2%9C%93&q=memory+leak) 都提出，有出现内存泄露问题。
    
    - Celery 有可能发生内存泄露，可以像这样设置：```CELERYD_MAX_TASKS_PER_CHILD = 40 # 每个worker执行了多少任务就会死掉```
    
    - worker_max_tasks_per_child 配置项，往 celeryconfig.py 文件里，像其他选项一样，添加即可。
    
    - 相关配置项详情，参考官方文档：[worker_max_tasks_per_child](http://docs.celeryproject.org/en/latest/userguide/configuration.html?highlight=CELERYD_MAX_TASKS_PER_CHILD#worker-max-tasks-per-child)

### 5. 总结

- 对失败任务处理

- 内存泄漏问题，不可忽视

- Celery 是比较强大的异步任务框架，遇到问题时，多看官方文档，以及在开源社区的提问（相信大多数问题已经有人提问，且有合理的解答）；这真的非常重要。

- 定时任务忘记写了，后续有空补一下 -:)

    - timedelta
    
    - crontab
    
    - 相关文档
    
        - [Crontab schedules](http://docs.celeryproject.org/en/latest/userguide/periodic-tasks.html?#crontab-schedules) 
        
        - [celery.schedules](http://docs.celeryproject.org/en/latest/reference/celery.schedules.html#celery-schedules)
        
        - [celery.bin.beat](http://docs.celeryproject.org/en/latest/reference/celery.bin.beat.html?#celery-bin-beat)

- [Message Protocol](http://docs.celeryq.org/en/latest/internals/protocol.html)

### 相关博文

- [Celery 文档 - Configuration and defaults](http://docs.celeryproject.org/en/latest/userguide/configuration.html#configuration-and-defaults)

- [Celery 文档 - celery.bin.worker](http://docs.celeryproject.org/en/latest/reference/celery.bin.worker.html#celery-bin-worker)

- [Celery 文档 - celery.bin.celery](http://docs.celeryproject.org/en/latest/reference/celery.bin.celery.html#celery-bin-celery)

- [Celery 文档 - Calling Tasks](http://docs.celeryproject.org/en/latest/userguide/calling.html?highlight=apply_async#calling-tasks)

- [Celery 文档 - Configuration](http://docs.celeryproject.org/en/latest/userguide/application.html?highlight=config#configuration)

- [Celery 文档 - Routing Tasks](http://docs.celeryproject.org/en/latest/userguide/routing.html#routers)

### 感谢

- [官网](http://www.celeryproject.org/)

- [Celery 官方文档](http://docs.celeryproject.org/en/latest/index.html)


