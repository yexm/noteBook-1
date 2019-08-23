---
title: Celery With Django
tags: django,celery,async,task,event,cronjob in django
grammar_cjkRuby: true
---

>**Note:**
>Celery 3.1版本之后不再需要第三方库与Django结合

Django项目目录结构如下：

```bash
- proj/
  - manage.py
  - proj/
    - __init__.py
    - settings.py
    - urls.py
```
需要创建proj/proj/celery.py，***该结构适用于大型项目***

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'proj.settings')

app = Celery('proj')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

接下来需要将`app`导入proj/proj/\_\_init__.py模块，以确保django启动时可以正确加载`app`，@shared_task装饰器会用到它。

proj/proj/\_\_init__.py:
```python
from __future__ import absolute_import, unicode_literals

# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app

__all__ = ('celery_app',)
```

~~您编写的任务可能存在于可重用的应用程序中，而可重用的应用程序不能依赖于项目本身，因此您也不能直接导入应用程序实例。~~ **此处不是很理解，未测试**

引入@shared_task装饰器，用以定义异步方法
```python
from __future__ import absolute_import, unicode_literals
from celery import shared_task


@shared_task
def add(x, y):
    return x + y


@shared_task
def mul(x, y):
    return x * y


@shared_task
def xsum(numbers):
    return sum(numbers)
```

其余使用celery的方法与celery无差异，使用django-celery可以实现使用django默认数据库作为broker，当未尝试新版本是否支持。

但是对于backend，是可以使用django默认数据库的。

1.首先安装`django-celery-results`库：
```bash
$ pip install django-celery-results
```

2.安装`django-celery-results`

```python
INSTALLED_APPS = (
    ...,
    'django_celery_results',
)
```
3.生成结果后端数据库

```bash
$ python manage.py migrate celery_results
```
4.配置结果后端

```python
CELERY_RESULT_BACKEND = 'django-db'
CELERY_CACHE_BACKEND = 'django-cache'
```

启动celery worker

```bash
$ celery -A proj worker -l info
```