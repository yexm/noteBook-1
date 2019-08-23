# Django文档整理

## 3.2 Models和数据库

模型就是数据库表的一个抽象。

### 3.2.1 Models

- 每一个model都是一个django.db.models.model的一个子类
- 每一个model属性都表示数据库中的一各域
- 基于以上，Django自动生成了访问数据库的API

#### 快速实例

这个实例model定义了一个带有first_name和last_name的Person类

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

first_name 和 last_name被称为model的fields，每个field对应一个类的属性，每个属性对应着数据库中的一行。

上面的Person model会像这样创建一个数据库表：

```sql
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
```

技术说明：

- 表的名称myapp_person会自动从一些模型元数据中派生出来，但是可以被重写。
- id字段是自动添加的，但是可以重写此行为。
- 本例中的CREATE TABLE SQL使用PostgreSQL语法进行格式化，但值得注意的是Django使用了针对设置文件中指定的数据库后端定制的SQL。

#### 使用model

在INSTALLED_APPS中添加APPNAME就不翻译了，如果不会就不要玩了。

使用 *manage.py makemigrations*，并执行*manage.py migrate*， 创建数据库表。

#### Field

模型中最重要的部分——也是模型中唯一需要的部分——是它定义的数据库字段列表。

字段由类属性指定。注意不要选择与模型API冲突的字段名，比如clean、save或delete。

```python
from django.db import models
class Musician(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    instrument = models.CharField(max_length=100)
class Album(models.Model):
    artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    release_date = models.DateField()
    num_stars = models.IntegerField()
```

#### Field types

模型中的每个字段都应该是适当字段类的实例。Django使用字段类类型来确定一些事情:

- 列类型，它告诉数据库要存储什么类型的数据(例如INTEGER、VARCHAR、TEXT)
- 呈现表单字段时使用的默认HTML小部件
- 用于Django管理和自动生成表单的最小验证需求

除了Django自带的字段外，还可以自定义用户字段。

#### Field options

常用参数总结：

- *null*， 如果True，Django将使用NULL填充数据库中的空值。默认是False
- *blank*， 如果为真，field运行设置为空，默认是False

  注意，这与null不同。null纯粹与数据库相关，而blank与验证相关。如果字段为blank=True，表单验证将允许输入空值。如果字段为blank=False，则需要该字段。

- *choices*，可迭代二元元组，用于此字段的选择。如果给定这个选项，默认的表单小部件将是一个选择框，而不是标准文本字段，并将选择限制为给定的选项。

  choices定义如下：

```python
    YEAR_IN_SCHOOL_CHOICES = (
        ('FR', 'Freshman'),
        ('SO', 'Sophomore'),
        ('JR', 'Junior'),
        ('SR', 'Senior'),
        ('GR', 'Graduate'),
    )

```

  每个元组中的第一个元素是将存储在数据库中的值。第二个元素由字段的表单小部件显示。
  
  实例：

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

```python
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```

- *default*, 设置field默认值。
- *help_text*, 表单小部件将显示额外的“帮助”文本。即使您的字段没有在表单上使用，它对文档也很有用。
- *primary_key*, 如果为True，该field将作为主键

    如果没有为模型中的任何字段指定primary_key=True, Django就会自动添加一个IntegerField作为主键，所以除非想重写默认的主键，否则不需要在任何字段上设置primary_key=True。

    主键字段是只读的。如果你更改现有对象上的主键值，然后保存它，就会在旧对象旁边创建一个新对象。例如:

```python
from django.db import models
class Fruit(models.Model):
    name = models.CharField(max_length=100, primary_key=True)
```

```python
>>> fruit = Fruit.objects.create(name='Apple')
>>> fruit.name = 'Pear'
>>> fruit.save()
>>> Fruit.objects.values_list('name', flat=True)
<QuerySet ['Apple', 'Pear']>
```

- *unique*, 如果为True该field全局唯一

#### Automatic primary key fields

不翻译了很简单

#### Verbose field names

每个Field第一个参数可以设置 Verbose name，概述字段用途。

ForeignKey, ManyToManyField和OneToOneField要求第一个参数是模型类，所以使用verbose_name关键字参数，所以需要使用verbos_name="*string*"指定。

### 关系

显然，关系数据库的强大之处在于表之间的关联。Django提供了一些方法来定义三种最常见的数据库关系类型:多对一、多对多和一对一。

#### 多对一关系

要定义多对一关系，请使用django.db.models.ForeignKey，与其他field一样它将作为一个model中的属性。

如下例中的关系：

```python
from django.db import models
class Manufacturer(models.Model):
    # ...
    pass
class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    # ...
```

建议将外键字段名称设置为关联的Model的小写名字。

#### 多对多关系

实例：

```python
from django.db import models
class Topping(models.Model):
    # ...
    pass
class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```

建议使用关联模型小写复数形式定义属性名

ManyToManyField在哪个类中不重要，但是不能两个都定义。

#### Extra fields on many-to-many relationships

啥意思？

多对多关系，形成的中间表，有的时候也需要带属性，如果使用ManyToManyField，无法为中间关联表定义属性，通过through选项，自定义中间表，添加需要的必要属性。

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

使用中间模型有一些限制：

- 使用中间模型时，两个原模型有且只有一个可以使用through属性
- 使用中间模型， 两个外键允许是相同的模型，但同样需要使用through_fields
- 使用中间模型，定义一个到自己的many-to-many关系，需要设定参数symmetrical=False

使用中间模型，不能使用add()，create()或set()来创建关系。remove()同样不可用，可用clear()删除所有关系。

#### 一对一关系

与多对一没有实际差异。

#### 跨文件Model引用

可通过模块导入方式，引入其他文件的models

#### Field name restrictions

告诉你不要用一些关键字等为Field定义名字

### Modle方法

插入个应该知道的知识点：
super().methodName(),调用父类方法，可用于重写model方法中。

### 3.2.2 查询

#### 创建对象

实例：

```python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

#### 保存对象变更

实例：

```python
>>> b5.name = 'New name'
>>> b5.save()
```

#### 存储外键域和多对多域

外键域与对象变更存储一致。多对多使用add()方法，添加记录。

实例：

```python
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

#### 对象检索（Retrieving objects）













## 问题

- Proxy Model能解决什么样的问题？
- 递归引用的Model需要尝试一下。


