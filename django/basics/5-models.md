---
layout: default
title: Django模型
parent: Django基础教程
nav_order: 5
---

# Django模型
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Model (模型) 简而言之即数据模型，是一个Django应用的核心。模型不是数据本身（比如数据表里的数据), 而是抽象的描述数据的构成和逻辑关系。
{: .fs-6 .fw-300 }

每个Django的模型(model)实际上是个类，继承了`models.Model`。每个Model应该包括属性(字段)，关系（比如单对单，单对多和多对多)和方法。当你定义好Model模型后，Django的接口会自动帮你在数据库生成相应的数据表(table)。这样你就不用自己用SQL语言创建表格或在数据库里操作创建表格了，是不是很省心？

## 模型定义小案例
假设你要开发一个名叫`bookstore`的应用，专门来管理书店里的书籍。我们首先要为书本和出版社创建模型。出版社有名字和地址。书有名字，描述和添加日期。我们还需要利用ForeignKey定义了出版社与书本之间单对多的关系，因为一个出版社可以出版很多书，每本书都有对应的出版社。我们定义了`Publisher`和`Book`模型，它们都继承了`models.Model`。你能看出代码有什么问题吗?

```python
# models.py
from django.db import models
 
class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField()
 
    def __str__(self):
        return self.name
		
class Book(models.Model):
    name = models.CharField(max_length=30)
    description = models.TextField(blank=True, null=True)
    publisher = ForeignKey(Publisher)
    add_date = models.DateField()
 
    def __str__(self):
        return self.name
```

模型创建好后，当你运行`python manage.py migrate` 命令创建数据表的时候你会遇到错误，错误原因如下：

- `CharField`里的`max_length`选项没有定义

- `ForeignKey(Publisher)`里的`on_delete`选项有没有定义

所以当你定义Django模型Model的时候，你一定要十分清楚2件事:

- 这个Field是否有必选项, 比如`CharField`的`max_length`和`ForeignKey`的`on_delete`选项是必须要设置的。

- 这个Field是否必需(blank = True or False)，是否可以为空 (null = True or False)。这关系到数据的完整性。

下面是订正错误后的Django模型：

```python
# models.py
from django.db import models
 
class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=60)
 
    def __str__(self):
        return self.name
		
class Book(models.Model):
    name = models.CharField(max_length=30)
    description = models.TextField(blank=True, default='')
    publisher = ForeignKey(Publisher,on_delete=models.CASCADE)
    add_date = models.DateField()
 
    def __str__(self):
        return self.name
```

修改模型后，你需要连续运行`python manage.py makemigrations`和`python manage.py migrate`这两个命令，前者检查模型有无变化，后者将变化迁移至数据表。如果一切顺利，Django会在数据库(默认sqlite)中生成或变更由`appname_modelname`组成的数据表，本例两张数据表分别为`bookstore_publisher`和`bookstore_book`。

## 模型的组成

一个标准的Django模型分别由模型字段、META选项和方法三部分组成。我们接下来对各部分进行详细介绍。Django官方编码规范建议按如下方式排列：

- 定义的模型字段：包括基础字段和关系字段
- 自定义的Manager方法：改变模型
- `class Meta选项`: 包括排序、索引等等(可选)。
- `def __str__()`：定义单个模型实例对象的名字(可选)。
- `def save()`：重写save方法(可选)。
- `def get_absolute_url()`：为单个模型实例对象生成独一无二的url(可选)
- 其它自定义的方法。

## 模型的字段

`models.Model`提供的常用模型字段包括基础字段和关系字段。

### 基础字段

**CharField() 字符字段**

一般需要通过max_length = xxx 设置最大长度。如不是必填项，可设置blank = True和default = ''。如果用于username, 想使其唯一，可以设置unique = True。如果有choice选项，可以设置 choices = XXX_CHOICES

**TextField() 文本字段**

适合大量文本，max_length = xxx选项可选。

**DateField() 和DateTimeField() 日期与日期时间字段**

可通过default=xx选项设置默认日期和时间。

- 对于DateField: default=date.today - 先要`from datetime import date`

- 对于DateTimeField: default=timezone.now - 先要`from django.utils import timezone`
- 如果希望自动记录一次修改日期(modified)，可以设置: `auto_now=True`
- 如果希望自动记录创建日期(created),可以设置`auto_now_on=True`

**EmailField() 邮件字段 **

如不是必填项，可设置blank = True和default = '。一般Email用于用户名应该是唯一的，建议设置unique = True

**IntegerField(), SlugField(), URLField()，BooleanField()**

可以设置blank = True or null = True。对于BooleanField一般建议设置`defaut = True or False`

**FileField(upload_to=None, max_length=100) - 文件字段 **

- upload_to = "/some folder/"：上传文件夹路径
- max_length = xxxx：文件最大长度

**ImageField (upload_to=None, max_length=100,)- 图片字段 **

- upload_to = "/some folder/": 指定上传图片路径

### 关系字段

**OneToOneField(to, on_delete=xxx, options) - 单对单关系**

- to必需指向其他模型，比如 Book or 'self' .
- 必需指定`on_delete`选项(删除选项): i.e, "`on_delete = models.CASCADE`" or "`on_delete = models.SET_NULL`" .
- 可以设置 "`related_name = xxx`" 便于反向查询。

**ForeignKey(to, on_delete=xxx, options) - 单对多关系**

- to必需指向其他模型，比如 Book or 'self' .
- 必需指定`on_delete`选项(删除选项): i.e, "`on_delete = models.CASCADE`" or "`on_delete = models.SET_NULL`" .
- 可以设置"default = xxx" or "null = True" ;
- 如果有必要，可以设置 "`limit_choices_to =` ",
- 可以设置 "`related_name = xxx`" 便于反向查询。

**ManyToManyField(to, options) - 多对多关系**

- to 必需指向其他模型，比如 User or 'self' .
- 设置 "`symmetrical = False` " 表示多对多关系不是对称的，比如A关注B不代表B关注A
- 设置 "`through = 'intermediary model'` " 如果需要建立中间模型来搜集更多信息。
- 可以设置 "`related_name = xxx`" 便于反向查询。

举例：一个人加入多个组，一个组包含多个人，我们需要额外的中间模型记录加入日期和理由。

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

## 模型的META选项

- `abstract=True`:  指定该模型为抽象模型
- `proxy=True`: 指定该模型为代理模型
- `verbose_name=xxx`和`verbose_name_plural=xxx`: 为模型设置便于人类阅读的别名
- `db_table= xxx`: 自定义数据表名
- `odering=['-pub-date']`: 自定义按哪个字段排序，`-`代表逆序
- `permissions=[]`: 为模型自定义权限
- `managed=False`: 默认为True，如果为False，Django不会为这个模型生成数据表
- `indexes=[]`: 为数据表设置索引，对于频繁查询的字段，建议设置索引
- `constraints=`: 给数据库中的数据表增加约束。

## 模型的方法

### 标准方法

以下三个方法是Django模型自带的三个标准方法：

- `def __str__()`：给单个模型对象实例设置人为可读的名字(可选)。
- `def save()`：重写save方法(可选)。
- `def get_absolute_url()`：为单个模型实例对象生成独一无二的url(可选)

除此以外，我们经常自定义方法或Manager方法

### 示例一：自定义方法 

```python
# 为每篇文章生成独一无二的url
def get_absolute_url(self):
    return reverse('blog:article_detail', args=[str(self.id)])

# 计数器
def viewed(self):
    self.views += 1
    self.save(update_fields=['views'])
```

### 示例二：自定义Manager方法

```python
# First, define the Manager subclass.
class DahlBookManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(author='Roald Dahl')

# Then hook it into the Book model explicitly.
class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)

    objects = models.Manager() # The default manager.
    dahl_objects = DahlBookManager() # The Dahl-specific manager.
```

## 完美的高级Django模型示例

一个完美的django高级模型结构如下所示，可以满足绝大部分应用场景，希望对你有所帮助。

```python
from django.db import models
from django.urls import reverse
 
 
# 自定义Manager方法
class HighRatingManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(rating='1')
    
 
class Product(models.Model):
    # CHOICES选项
    RATING_CHOICES = (
        ("1", 'Very good'),
        ("2", 'Good'),
        ("3", 'Bad'),
    )
 
    # 数据表字段
    name = models.CharField('name', max_length=30)
    rating = models.CharField(max_length=1, choices=RATING_CHOICES)
 
    # MANAGERS方法
    objects = models.Manager()
     high_rating_products =HighRatingManager()
 
    # META类选项
    class Meta:
        verbose_name = 'product'
        verbose_name_plural = 'products'
 
    # __str__方法
    def __str__(self):
        return self.name
 
    # 重写save方法
    def save(self, *args, **kwargs):
        do_something()
        super().save(*args, **kwargs) 
        do_something_else()
 
    # 定义绝对路径
    def get_absolute_url(self):
        return reverse('product_details', kwargs={'pk': self.id})
 
    # 定义其它方法
    def do_something(self):
```

## 小结

本章我们介绍了Django模型的组成: 字段(基础字段和关系字段), META选项和方法。使用Django模型的好处是你无需使用SQL语句创建复杂的数据表，Django会自动根据模型为你生成数据表。同样在Django中如果你希望对数据库中的数据进行增删查改，你也无需使用原生的SQL语句。反之你可以使用Django框架自带的ORM(Object-Relational Mapper, 对象关系映射)提供的API进行相关操作。下章我们将重点介绍如何使用这些API语句操作我们的模型，对数据表里的数据进行增删查改。

原创不易，转载请注明来源。我是大江狗，一名Django技术开发爱好者。您可以通过搜索【<a href="https://blog.csdn.net/weixin_42134789">CSDN大江狗</a>】、【<a href="https://www.zhihu.com/people/shi-yun-bo-53">知乎大江狗</a>】和搜索微信公众号【Python Web与Django开发】关注我！

![Python Web与Django开发](../../assets/images/django.png)
