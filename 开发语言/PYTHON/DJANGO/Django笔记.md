---
title: Django笔记
tags: django
grammar_cjkRuby: true
---

# Django
## Models
### Field Options
#### null
当unique=True， blank=True，的条件下，null应当被设置为true。原因：避免存储多个带有空值的对象时违反唯一性约束（unique constraint violations）。避免在CharField使用，这会造成该域存在两种空值表示形式NULL和""。


#### blank【搞定】
#### choices
限定值映射，格式为：
```python
SOME_CHOICES = [
	(A, B),
	(A, B),
	...
]
```
#### db_clolumn
自定义field名
#### db_index
True，将创建索引
#### db_tablespace【暂时搞不定】
#### default【搞定】
#### editable
默认True，设为False在admin中将不予显示，模型验证也将被跳过。
#### error_messages【搞定】
#### help_text
#### primary_key
主键意味着，null=False， unique=True
#### unique_for_date、unique_for_month、unique_for_year
可作用于DateField和DateTimeField，某领域设定了以上选项，则该领域和指定的时间领域组成的组合键，全局唯一。
#### verbose_name
人类可读名，自定义，就是让你起个名，有用的能提醒你这是干什么用的。
#### validators

### Field Type

#### CharField
目前CharField.max_length，是强制选项。
#### DateField
自有属性
**DateField.auto_now**
可以用以模拟latest modify

**DateField.auto_now_add**
可用用以生成时间戳



### Relationship Fields
#### ForeignKey
ForeignKey.on_delete

 - CASCADE，级联删除
 - PROTECT，通过抛出 ProtectedError异常，来保护要被删除的对象

#### ManyToManyField
ManyToManyField.through，指定中间表
ManyToManyField.through_field，指定创建组成多对多关系的fields
#### OneToOneField

#### 关于related_name和related_qurey_name的理解
模型定义：
```python
# -*- coding: UTF-8 -*-
from __future__ import unicode_literals
from django.db import models

# Create your models here.

class Author(models.Model):
    name = models.CharField(verbose_name='姓名', max_length=50)
    age = models.IntegerField(verbose_name='年龄')

class Book(models.Model):
    name = models.CharField(verbose_name='书名', max_length=100)
    author = models.ForeignKey(Author, verbose_name='作者', related_name='bs', related_query_name='b')
```

查询方法：
```python
>>> Author.objects.filter(b__name='learn java')
[<Author: jim>]
>>> author = Author.objects.get(pk=1)
>>> author.bs.all()
[<Book: learn java>, <Book: learn python>]
```

### 多表继承
多表继承父类元属性，已经应用于父类，子类一般无需定义。但是抽象类继承是有意义的。
子类无法访问父类的MetaClass， 但是有一些限制项是可以从父类继承的，ordering或者get_latest_by

### 代理模型

用以保护父模型的一些设置或方法。
例如，在代理模型中修改默认排序，将不会变更父类的排序方式，但是数据仍然可以反映到，代理模型中。
实际上，代理模型与父模型在同一个表上。
代理模型的父类必须是一个非抽象类， 不能多继承，不提供数据库表之间的链接；代理模型可以继承任意数量的抽象模型类，前提是它们不定义任何模型字段；代理模型还可以从共享非抽象父类的任意数量的代理模型继承。
**一般原则**

 1. 如果你要镜像一个已存在的模型或者数据库，并且不需要所有的原始数据库表项。Meta.managed=False，该选项通常用于建模不受Django控制的数据库视图和表。
 2. 若果你想要改变一个模型的python行为，但是保持fields与原始数据库一致，使用 Meta.proxy=True。作用如上所述。


### 多重继承
就像普通的多重继承一样，但是多个继承类的第一个MetaClass将决定子类的Meta属性。

使用隐式的主键定义的模型，多重继承会出现异常，要显示定义AutoField。
或者使用公共祖先类定义默认主键，但是所有父类都需要定义与祖先类的 OneToOneField
如下：
```python
class Piece(models.Model):
    pass

class Article(Piece):
    article_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class Book(Piece):
    book_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class BookReview(Book, Article):
    pass
```

Django的模型类，子类不可以重写父类（非抽象）属性。如果尝试覆盖父类模型中的字段，Django将抛出**FieldError**

## 查询模型
创建对象，使用对象的save()方法

### 检索对象
通过QuerySet和Manager实现
#### 检索全部对象

``` python
all_entries = Entry.objects.all()
```
#### 过滤检索指定对象

- filter(\*\*kwargs)
- exclude(\*\*kwargs)

```python
Entry.objects.filter(pub_date__year=2006)

Entry.objects.all().filter(pub_date__year=2006)
```

支持链式过滤

``` python
Entry.objects.filter(...).exclude(...).filter(...)
```
*QuerySet唯一性：*
每个QuerySet都是唯一的，如上的链式查找，实际上创建了三个QuerySet。

``` python
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```
这种方式将访问三次数据库

``` python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```
这种方式后两次访问将使用缓存

#### 使用get()获取唯一对象
#### 查询结果支持分片操作

#### Field查询
语法
```
field__lookuptype=value
```

lookup type:

 - exact
 - iexcact
 - contains
 - icontains
 - startswith
 - endswith

连表查询
例子展示了，查询所有blog name带有“Beatles Blog”的Entry对象
``` python
>>> Entry.objects.filter(blog__name='Beatles Blog')
```
连表查询仍然可以使用lookuptype

***Spanning multi-valued relationships***
这一小节没有明白

##### 过滤器引用模型上的字段
F 表达式， 如下所示：

``` python
>>> from django.db.models import F
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
```
可对F表达式，进行加、减、乘、除、幂运算、模运算。
F表达式可以使用双下划线，调用跨关系调用。
F表达式支持位操作 .bitand(), .bitor(), .bitrightshift()

##### 主键查询快捷方式
例如

``` python
>>> Blog.objects.get(pk=14) # pk implies id__exact
```

#### 缓存和QuerySet

每个QS包含一个缓存。
一个QS对数据进行了evaluated，那么下次调用QS，将直接从缓存中返回。
例如

``` python
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```
将两次访问数据库

``` python
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # Evaluate the query set.
>>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
```
对数据库仅进行了一次访问。

##### 何时不进行缓存
调用切片将不使用缓存。

``` python
>>> queryset = Entry.objects.all()
>>> print(queryset[5]) # Queries the database
>>> print(queryset[5]) # Queries the database again
```

但是当QS已经被计算过，那么使用切片仍将使用缓存。

``` python
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # Queries the database
>>> print(queryset[5]) # Uses cache
>>> print(queryset[5]) # Uses cache
```
#### Q对象的复杂查找
Q对象可以进行与或非的集合运算
Q对象可以普通的属性同事使用，Q要定义在前面。
#### 使用delete()方法，进行删除操作
#### 更新QS内所有对象
使用update()方法。
更新无法跨表
#### 关联对象
select_related()会预先缓存所有一对多的关系。

related_name选项值可以替代FOO__set的模式



### 聚合计算
aggregate（）和annotate（）不同在于annotate（）返回的是QS，而aggregate返回的是字典计算结果。

使用annotate（）组合多个聚合，会出现错误，因为聚合使用的是连接而不是子查询，对于大多数的聚合函数都是这样的，但是Count聚合函数可以通过**distinct**避免这个问题

实例
```python
>>> book = Book.objects.first()
>>> book.authors.count()
2
>>> book.store_set.count()
3
>>> q = Book.objects.annotate(Count('authors'), Count('store'))
>>> q[0].authors__count
6
>>> q[0].store__count
6

>>> q = Book.objects.annotate(Count('authors', distinct=True), Count('store', distinct=True))
>>> q[0].authors__count
2
>>> q[0].store__count
3
```

#### 连接和聚合
连表聚合

``` python
>>> from django.db.models import Max, Min
>>> Store.objects.annotate(min_price=Min('books__price'), max_price=Max('books__price'))
```

可以使用过滤器对annotate结果集进行操作
实际使用中需要认真考虑annotate和过滤器的先后顺序
可以在注释的结果上生成聚合结果。


简单说：聚合是计算结果，注释是对每一个元数据的附加属性。

### Manager（没有太多难点）
### 事务管理（暂时不涉及）
### 支持多数据库和数据库路由