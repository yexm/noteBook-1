## Tasks
Tasks是Celery应用的组成部分（building blocks）.

一个Task从任意可调用类中创建出来的。他扮演了定义task被调用时发生了什么，以及一个worker接收到消息后做了什么的双重角色。

每一个task类有一个队列名字（queue name），这个名字与message相关联，以便worker能够找到正确的函数执行。

一个task message在message被worker确认前是不会被删除的。一个worker能够提前预定许多消息，即使是因为电源故障或其他原因，worker进程被杀掉了，消息仍然可以重新投递给另一个worker。

理想情况下（ideally），task函数应该是幂等的（idempotent）：这意味着函数不会因为使用同样的变量多次调用造成未预期的影响。因为worker无法检验你的函数是否是幂等的，默认的方式在执行之前是提前确认消息，这样已经开始的事务调用就不会被再次执行了。

如果你的函数是幂等的，你可以设置acks_late选项在事务返回之后再获取消息确认信息。参阅FAQ条目[should I use retry or acks_late?](http://docs.celeryproject.org/en/stable/faq.html#faq-acks-late-vs-retry)。

注意如果子进程执行的task被终结，即使acks_late被设置为enable，worker仍将确认message（该消息从队列中被删除）。这样的行为基于以下原因：

- 我们不想重新运行被内核强制发送SIGSEGV或类似信号的进程对应task
- 我们任务系统管理员刻意杀死该task不是为了重新自动运行它
- 一个task分配太多内存是有触发OOM kill风险的，类似的情况可能还会发生
- 一个task当被重新投递时总是失败，或许会导致一个高频率的消息环，从而拖死系统

如果你真的在上面这些场景中重新投递消息，那么建议你考虑使用task_reject_on_worker_lost选项。

> ## Warning
> 一个无限期阻塞的task可能最终将停止正在进行其他工作的worker实例
> 如果你的task执行了一个I/O，请确认你对这些操作添加了超时设置，就像为使用requests库发起的web请求添加超时设置如下：
> ```python
> connect_timeout, read_timeout = 5.0, 30.0
> response = requests.get(URL, timeout=(connect_timeout, read_timeout))
> ```
> 时间限制很方便的能够确保所有事务及时返回，但是一个时间限制事件会强制杀死进程，所以只用他们检测还没有触发自定义超时的事务。
> 默认prefork的子进程对于长时间运行的事务并不友好，所以如果使用分钟/小时级的任务请确认已经在celery worker命令中加入了-0fair参数。参见Prefork pool prefetch setting获取更多信息。最佳实践是将长时间运行与短时间运行的tasks自动路由到专用的（dedicated）workers上
>如果你的worker挂起，那么在提交问题前请先调查有哪些tasks在运行，多数情况下挂起都是由于一个或几个tasks正在进行网络操作。

本章你将学习关于定义tasks的全部内容

### Basics

你可以轻易的从一个被task()装饰器修饰的可调用的对象中创建一个task
```python
from .models import User

@app.task
def create_user(username, password):
    User.objects.create(username=username, password=password)
```
有许多选项可以作为参数传递给装饰器：
```python
@app.task(serializer='json')
def create_user(username, password):
    User.objects.create(username=username, password=password)
```
>>> 多装饰器
当使用多个装饰器的时候确保task在最外层

### Bound tasks
一个tasks被绑定意味着task函数的第一个参数永远是实例自身（self），就像Python绑定方法：
```python
logger = get_task_logger(__name__)

@task(bind=True)
def add(self, x, y):
    logger.info(self.request.id)
```
*重试、访问当前事务请求信息和添加到自定义事务基类的附加功能要求绑定事务。*

### Task ingeritance
task装饰器的base参数指定了task的基类：
```python
import celery

class MyTask(celery.Task):

    def on_failure(self, exc, task_id, args, kwargs, einfo):
        print('{0!r} failed: {1!r}'.format(task_id, exc))

@task(base=MyTask)
def add(x, y):
    raise KeyError()
```

### Names
每个task必须有一个唯一的name
如果没有一个明确的name，task装饰器将自动生成一个，这个名字将基于1）task被定义的模块；2）task函数的名字。

设置明确name的例子：
```python
>>> @app.task(name='sum-of-two-numbers')
>>> def add(x, y):
...     return x + y

>>> add.name
'sum-of-two-numbers'
```

使用模块名作为名称空间是一个最佳实践，这种命名方式即使其他模块有相同的方法名也不会造成冲突。
```python
>>> @app.task(name='tasks.add')
>>> def add(x, y):
...     return x + y
```

可以使用`.name`属性检查task的名字

```python
>>> add.name
'tasks.add'
```
这里我们定义的tasks.add与自动生成的是一样的。
tasks.py:
```python
@app.task
def add(x, y):
    return x + y
```

```python
>>> from tasks import add
>>> add.name
'tasks.add'
```

### Automatic naming and relative imports

相对导入和自动名称生成不能很好地结合在一起，所以如果使用相对导入，应该显式地设置名称。

例如，如果客户机导入模块“.tasks”来表示“myapp.tasks”，worker使用“myapp.tasks”导入模块，生成的名称将不匹配，worker将抛出一个NotRegistered错误。

这就是使用Django和使用project.myapp风格时在INSTALLED_APPS命名的关键:
```python
INSTALLED_APPS = ['project.myapp']
```
If you install the app under the name project.myapp then the tasks module will be imported as project.myapp.tasks, so you must make sure you always import the tasks using the same name:
