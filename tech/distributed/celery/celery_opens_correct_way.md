## Celery 的正确打开方式

### 目录

[1. 环境](#1-环境)

[2. config_from_object 方式添加配置](#2-config_from_object-方式添加配置)

[3. 项目（Demo ）介绍](#3-项目demo-介绍)

--- 

[相关博文](#相关博文)


### 1. 环境
- Basic
    - linux/windows/Mac OS
    - python 3.6+
    - redis 2.8+

- Python 库
    - oyaml==0.9
    - celery==4.3.0 (pip install -U "celery[redis]")

### 2. config_from_object 方式添加配置

- 首先 Celery **app** 初始化操作，参考[Application](http://docs.celeryproject.org/en/latest/userguide/application.html?highlight=config#application)。

- 其次要知道 **app** 配置添加方式，可以使用 **app.config_from_object()**。

### 3. 项目（Demo ）介绍

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
<summary>点击查看 hackfun.__init__.py 源代码</summary>

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
    path = os.environ.get('CLOUDCARE_PROFWANG_CONFIG')
    if path is None:
        with open(base_path + "/config/config.yaml") as f:
            config = yaml.load(f)


init_config()

```

</details>


#### 3.3 实际项目应用参考，源码如下

#### 3.3.1 app 初始化

<details>
<summary>点击查看 hackfun.app.py 源代码</summary>

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
    host = celery_config['redis']['host']
    port = celery_config['redis']['port']
    password = celery_config['redis']['password']
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
<summary>点击查看 hackfun.celeryconfig.py 源代码</summary>

```python

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
    create_queue(task_default_queue),

    create_queue(get_worker_queue('webhook')),
    create_queue(get_worker_queue('internal')),
]

# Task
task_routes = {
    'webhook.*': {'queue': get_worker_queue('webhook')},
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

所有 config 相应释义 请查看官网 [Celery 文档 - Configuration and defaults](http://docs.celeryproject.org/en/latest/userguide/configuration.html#configuration-and-defaults)


### 相关博文

- [Celery 文档 - Configuration and defaults](http://docs.celeryproject.org/en/latest/userguide/configuration.html#configuration-and-defaults)

- [Celery 文档 - Configuration](http://docs.celeryproject.org/en/latest/userguide/application.html?highlight=config#configuration)

- [Celery 文档 - Routing Tasks](http://docs.celeryproject.org/en/latest/userguide/routing.html#routers)

### 感谢

[Celery 官方文档](http://docs.celeryproject.org/en/latest/index.html)


