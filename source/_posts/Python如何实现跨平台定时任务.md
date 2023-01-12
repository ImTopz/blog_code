---
title: Python如何实现跨平台定时任务
date: 2023-1-12 18:23:00
tags: 学习
categories: python学习
---

前言：近日在学习funboost框架，在学习到其内置的定时任务时候，便顺便学习了一下APScheduler这款定时任务工具，不依赖于Linux的crontab工具。

参考链接:

```python
[https://blog.csdn.net/qq_40125653/article/details/103662019](https://blog.csdn.net/qq_40125653/article/details/103662019)
```

## APScheduler介绍

1.模块安装

```python
pip install apscheduler
```

2.具有四种组件

- 触发器(triggers) 指定定时任务的执行的时机
- 存储器(job stores) 可以定时持久化存储, 可以保存在数据库中或redis
    
    执行器(executors) 在定时任务执行时, 进程或者线程的方式执行任务
    
    调度器(schedulers)
    
    存储器代码示例：
    
    ```python
    # 存储在redis中
    from apscheduler.jobstores.redis import RedisJobStore
    # 存储在mongo中
    from apscheduler.jobstores.mongodb import MongoDBJobStore
    # 存储在数据库中
    from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
    ```
    
    调度器代码示例
    
    ```python
    # 以后台的方式运行
    from apscheduler.schedulers.background import BackgroundScheduler
    # 以阻塞的方式运行, 前台运行
    from apscheduler.schedulers.background import BlockingScheduler
    ```
    

3.详情

触发器：

[1.date](http://1.date): 特定的日期执行

```python
from datetime import date
from apscheduler.schedulers.blocking import BlockingScheduler #阻塞的方式在前台运行
sched = BlockingScheduler() #生成对象
def my_job(text):
    print(text)

# 在2023年1月12日00:00:00执行
sched.add_job(my_job, 'date', run_date=date(2023, 1, 12))
# 在2019年11月6日16:30:05, 可以指定运行的详细时间
sched.add_job(my_job, 'date', run_date=datetime(2023, 1, 12, 16, 30, 5))
# 运行时间也可以是字符串的形式
sched.add_job(my_job, 'date', run_date='2023-01-12 16:30:05', args=['text])
# 立即执行
sched.add_job(my_job, 'date')  
sched.start()
```

2. interval：以固定的时间间隔运行作业时使用

weeks (int) – 间隔的周数
days (int) – 间隔的天数
hours (int) – 间隔的小时
minutes (int) –间隔的分钟
seconds (int) – 间隔的秒
start_date (datetime|str) – 间隔时间的起点
end_date (datetime|str) – 间隔时间的结束点
timezone (datetime.tzinfo|str) – 时区
jitter (int|None) – 将作业执行 延迟的时间

```python
from datetime import datetime
# 每两小时执行一次
sched.add_job(job_function, 'interval', hours=2)
# 在2018年10月10日09:30:00 到2019年6月15日11:00:00的时间内，每两小时执行一次
sched.add_job(job_function, 'interval', hours=2, start_date='2018-10-10 09:30:00', end_date='2019-06-15 11:00:00')
# interval 可以指定在程序启动的时候, 触发运行 next_run_time 参数
sched.add_job(main, 'interval', hours=2, next_run_time=datetime.datetime.now())

```

3.cron：在一天中的特定时间定期运行作业时使用常见的参数

year (int|str) – 4位数的年份
month (int|str) – month (1-12)
day (int|str) – day (1-31)
week (int|str) – ISO week (1-53)
day_of_week (int|str) –工作日的编号或名称（0-6或周一，周二，周三，周四，周五，周六，周日）
hour (int|str) – 小时(0-23)
minute (int|str) – 分钟 (0-59)
second (int|str) – 秒 (0-59)
start_date (datetime|str) –最早触发的日期/时间（包括）
end_date (datetime|str) – 结束触发的日期/时间（包括）
timezone (datetime.tzinfo|str) – 时区
jitter (int|None) – 将执行作业延迟几秒执行
常见的表达式类型

```python
# 在6、7、8、11、12月的第三个周五的00:00, 01:00, 02:00和03:00 执行
sched.add_job(job_function, 'cron', month='6-8,11-12', day='3rd fri', hour='0-3')
# 在2014年5月30日前的周一到周五的5:30执行
sched.add_job(job_function, 'cron', day_of_week='mon-fri', hour=5, minute=30, end_date='2014-05-30')
# 执行的方式 用装饰器的形式, 每个月的最后一个星期日执行
@sched.scheduled_job('cron', id='my_job_id', day='last sun')
def some_decorated_task():
    print("I am printed at 00:00:00 on the last Sunday of every month!")
# 可以使用标准的crontab表达式执行
sched.add_job(job_function, CronTrigger.from_crontab('0 0 1-15 may-aug *'))
# 延迟120秒执行
sched.add_job(job_function, 'cron', hour='*', jitter=120)
```

对触发器学习就说这些，因为满足了我目前的开发需求

存储器:

个人习惯用redis

```python
REDIS_CONF = {
    "password": "xxxxx",
    "host": "127.0.0.1",
    "port": 6379,
    "db": 0}
from apscheduler.jobstores.redis import RedisJobStore
from apscheduler.jobstores.mongodb import MongoDBJobStore
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
# 存储器
job_stores = {
	# 使用redis存储
    'redis': RedisJobStore(jobs_key=jobs_key, run_times_key=run_times_key, **REDIS_CONF),
    # 使用mongo存储
    'mongo': MongoDBJobStore(),
    # 数据库存储
    'default': SQLAlchemyJobStore(url='sqlite:///jobs.sqlite')
    }
# 执行器
executors = {
    'default': ThreadPoolExecutor(20),  # 20个线程
    'processpool': ProcessPoolExecutor(5)  # 5个进程
}
job_defaults = {
    'coalesce': False,  # 相同任务触发多次
    'max_instances': 3  # 每个任务最多同时触发三次
}   
# 使用配置, 启动
scheduler = BackgroundScheduler(jobstores=jobstores, executors=executors, job_defaults=job_defaults, timezone=utc)
```

执行器:以进程或者线程的方式执行程序

```python
# 线程的方式执行
from apscheduler.executors.pool import ThreadPoolExecutor
executors = {
      'default': ThreadPoolExecutor(20) # 最多20个线程同时执行
  }
scheduler = BackgroundScheduler(executors=executors) 
# 进程的方式
executors = {
      'default': ProcessPoolExecutor(5) # 最多5个进程同时执行
  }
```

****调度器：****

• BlockingScheduler: 作为独立进程时使用

```python
from apscheduler.schedulers.blocking import BlockingScheduler
scheduler = BlockingScheduler()
scheduler.start()  
# 此处程序会发生阻塞复制代码
```

• BackgroundScheduler 后台运行, 在框架中使用

```python
from apscheduler.schedulers.background import BackgroundScheduler
scheduler = BackgroundScheduler()
scheduler.start()  
# 此处程序不会发生阻塞复制代码
```

AsyncIOScheduler : 当你的程序使用了asyncio的时候使用。
GeventScheduler : 当你的程序使用了gevent的时候使用。
TornadoScheduler : 当你的程序基于Tornado的时候使用。
TwistedScheduler : 当你的程序使用了Twisted的时候使用
QtScheduler : 如果你的应用是一个Qt应用的时候可以使用。

## 配置的三个方法:

### 1.

```python
from pytz import utc
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.jobstores.mongodb import MongoDBJobStore
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor
jobstores = {
    'mongo': MongoDBJobStore(),
    'default': SQLAlchemyJobStore(url='sqlite:///jobs.sqlite')
}
executors = {
    'default': ThreadPoolExecutor(20),  # 最大线程数
    'processpool': ProcessPoolExecutor(5)  # 最大进程数
}
job_defaults = {
    'coalesce': False,  
    'max_instances': 3  # 同一个任务启动实例的最大个数
}
# 配置的使用方式
scheduler = BackgroundScheduler(jobstores=jobstores, executors=executors, job_defaults=job_defaults, timezone=utc)
```

### 2.

```python
from apscheduler.schedulers.background import BackgroundScheduler

# 使用字典的形式添加配置
scheduler = BackgroundScheduler({
    'apscheduler.jobstores.mongo': {
         'type': 'mongodb'
    },
    'apscheduler.jobstores.default': {
        'type': 'sqlalchemy',
        'url': 'sqlite:///jobs.sqlite'
    },
    'apscheduler.executors.default': {
        'class': 'apscheduler.executors.pool:ThreadPoolExecutor',
        'max_workers': '20'
    },
    'apscheduler.executors.processpool': {
        'type': 'processpool',
        'max_workers': '5'
    },
    'apscheduler.job_defaults.coalesce': 'false',
    'apscheduler.job_defaults.max_instances': '3',
    'apscheduler.timezone': 'UTC',
})
```

### 3.

```python
from pytz import utc

from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.executors.pool import ProcessPoolExecutor

jobstores = {
    'mongo': {'type': 'mongodb'},
    'default': SQLAlchemyJobStore(url='sqlite:///jobs.sqlite')
}
executors = {
    'default': {'type': 'threadpool', 'max_workers': 20},
    'processpool': ProcessPoolExecutor(max_workers=5)
}
job_defaults = {
    'coalesce': False,
    'max_instances': 3
}
scheduler = BackgroundScheduler()
# 使用调度器对象的 configure属性增加 存储器, 执行器 存储器 的配置
scheduler.configure(jobstores=jobstores, executors=executors, job_defaults=job_defaults, timezone=utc)
```

### **8. 定时任务启动**

`scheduler.start()`

- 对于BlockingScheduler ，程序会阻塞在这，防止退出，作为独立进程时使用。
- 对于BackgroundScheduler，可以在应用程序中使用。不再以单独的进程使用。