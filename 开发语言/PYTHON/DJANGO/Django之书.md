---
title: Django之书
tags: python,django
grammar_cjkRuby: true
---

# 模型
模型<-->数据库表

 - django.db.models.Model子类
 - 模型类属性相当于数据库字段
 - django提供自动访问数据库的api

## 快速上手

示例代码：
```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
	
"""
将自动创建id字段
"""
```
## 使用模型

 - step1.配置应用至setting.INSTALLED_APP列表中。
 - step2.使用`python manage.py makemigrations APPNAME`,生成表结构
 - step3.使用`python manage.py migrate`,执行数据迁移

## 字段
**注意：避免使用与模型API冲突**
### 字段类型
每个字段对应于数据库表中的列，字段类型对应该列数据类型，其中每个字段都是Filed类的实例。
可自定义字段
### 字段选项
#### null
True，当字段为空时，数据库中字段设置为Null。默认False
#### blank
True，该字段允许为空
#### choices
接收一个可迭代list或者tuple，基本单位二元tuple
 ```python
from django.db import models
class Person(models.Model):
	SHIRT_SIZES = (
		('S', 'Small'),
		('M', 'Medium'),
		('L', 'Large'),
	)
	name = models.CharField(max_length=60)
	shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```
 tuple中第二个元素的访问可以使用get_FOO_display()，在本例中就是get_shirt_size_display()
#### default
为字段设定默认值
#### help_text
可现实于Form中，对于文档很有帮助，即使字段不显示在admin的Form中
#### primary_key
设置主键，Django会自动添加自增主键
#### unique
设置为True，该字段全表不可重复
#### 备注名（verbose name）
可选参数，除了 ForeignKey ， ManyToManyField 和 OneToOneField ，这些类第一个参数为类名，后面可用verbose_name参数

## 关联关系：
### 多对一：
使用django.db.models.ForeignKey类
 *==建议，外键字段名，设置为想要关联的表名==*
### 多对多：
使用 django.db.models.ManyToManyField 类
*==建议，该属性名设置为复数名词，关联想要关联的表名==*
多对多关系一般需要使用第三方表格，以实现特殊信息的添加，可以使用ManyToManyField的through属性，指定第三方表格
```python
from django.db import models

class Person(models.Model):
	name = models.CharField(max_length=128)

	def __str__(self):
		return self.name

class Group(models.Model):
	name = models.CharField(max_length=128)
	members = models.ManyToManyField(Person, through='Membership')

	def __str__(self):
		return self.name

class Membership(models.Model):
	person = models.ForeignKey(Person, on_delete=models.CASCADE)
	group = models.ForeignKey(Group, on_delete=models.CASCADE)
	date_joined = models.DateField()
	invite_reason = models.CharField(max_length=64)
```
中间表(多对多属性)不能使用add，create，set，remove等方法，因为无法确认二元关系组的唯一性（member1,member2），但是可以使用clear()方法，清除所有多对多的关系。
### 一对一：
使用OneToOneField定义一对一关系，需要一个位置参数：与模型相关的类
可选参数：parent_link
通常自动将该属性设置为主键
## 跨文件模型
## 字段命名限制

 1. 不能是Python保留字
 2. 一个字段名称不能包含连续多个下划线，因为django查询语法会用到双下划线
 3. 字段名称结尾不能使用下划线
## 支持自定义的字段类型
## Meta选项：
使用内部Meta类：
```python
from django.db import models

class Ox(models.Model):
	horn_length = models.IntegerField()

	class Meta:
		ordering = ["horn_length"]
		verbose_name_plural = "oxen"    
```
## 模型属性objects
模型当中最重要的属性是Manager，默认名为objects。Manager只能通过模型类访问，不能通过实例访问。
## 模型方法
方法提供的是“行级”操作，即操作一条数据记录    

**==__str__()==**
一个 Python 的“魔法方法”，返回值友好地展示了一个对象。Python 和 Django 在要将模型实例展示为纯文本时调用。最有可能的应用场景是交互式控制台或后台。你将会经常定义此方法；默认提供的不是很好用。

*==注意：重写父类方法时super()的使用==*

可以执行原生的SQL

## 模型继承
Django 有三种可用的集成风格：

 1. 常见情况下，你仅将父类用于子类公共信息的载体，因为你不会想在每个子类中把这些代码都敲一遍。这样的父类永远都不会单独使用，所以 抽象基类 是你需要的。
 2.  若你继承了一个模型（可能来源其它应用），且想要每个模型都有对应的数据表，客官这边请 多表继承。 
 3.  最后，若你只想修改模型的 Python 级行为，而不是以任何形式修改模型字段， 代理模型 会是你的菜。
   
   

