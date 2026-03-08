---
layout: default
title:  Django事务操作和锁
parent: 大江狗的Django进阶教程
nav_order: 16
---

# Django事务操作和锁
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

事务处理(transaction)对于Web应用开发至关重要, 它可以维护数据库的完整性, 使整个系统更加安全。比如用户A通过网络转账给用户B，数据库里A账户中的钱已经扣掉，而B账户在接收过程中服务器突然发生了宕机，这时数据库里的数据就不完整了。加入事务处理机制后，如果在一连续交易过程中发生任何意外, 程序将回滚，从而保证数据的完整性。本文将总结事务的四大特性以及Django项目开发中如何操作事务，并以实际代码实现悲观锁和乐观锁。

{: .fs-6 .fw-300 }

## 事务的四大特性(ACID)

如果想要说明一个数据库或者一个框架支持事务性操作，则必须要满足下面的四大特性：

- 原子性（Atomicity）：整个事务中的所有操作，要么全部完成，要么全部不完成。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没发生过一样。

- 一致性 （Consistency）：事务开始之前和事务结束后，数据库的完整性约束没有被破坏。

- 隔离性（Isolation）：隔离性是指当多个用户并发访问数据库时，比如同时访问一张表，数据库每一个用户开启的事务，不能被其他事务所做的操作干扰，多个并发事务之间，应当相互隔离。

- 持久性（Durability）：事务执行成功后，该事务对数据库的更改是持久保存在数据库中的，不会被回滚。

注意：

- 并不是所有的数据库或框架支持事务操作。比如在MySQL中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。

以下是关于事务的一些常用术语，我们在接下来文章中会用到。

- 开启事务：Start Transaction
- 事务结束：End Transaction
- 提交事务：Commit Transaction
- 回滚事务：Rollback Transaction

## Django默认事务行为

Django是支持事务操作的，它的默认事务行为是自动提交，具体表现形式为：每次数据库操作(比如调用save()方法)会立即被提交到数据库中。但是如果你希望把连续的SQL操作包裹在一个事务里，就需要手动开启事务。

## 全局开启事务

在Web应用中，常用的事务处理方式是将每次请求都包裹在一个事务中。全局开启事务只需要将数据库的配置项`ATOMIC_REQUESTS`设置为True，如下所示：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db1',
        'HOST': 'dbhost',
        'PORT': '3306',
        'USER': 'dbuser‘,
        'PASSWORD': 'password',
        "ATOMIC_REQUESTS": True,  #全局开启事务，绑定的是http请求响应整个过程
    }
```

它的工作原理是这样的：每当有请求过来时，Django会在调用视图方法前开启一个事务。如果完成了请求处理并正确返回了结果，Django就会提交该事务。否则，Django会回滚该事务。

如果你全局开启了事务，你仍然可以使用`non_atomic_requests`装饰器让某些视图方法不受事务控制，如下所示：

```python
from django.db import transaction

@transaction.non_atomic_requests
def my_view(request):
    do_stuff()

# 如有多个数据库，让使用otherdb的视图不受事务控制
@transaction.non_atomic_requests(using='otherdb')
def my_other_view(request):
    do_stuff_on_the_other_database()
    
```

虽然全局开启事务很简单，但Django并不推荐开启全局事务。因为一但将事务跟 HTTP 请求绑定到一起时，每一个请求都会开启事务，当访问量增长到一定的时候会造成很大的性能损耗。在实际开发过程中，很多GET请求根本不涉及到事务操作，一个更好的方式是局部开启事务按需使用。

## 局部开启事务

Django项目中局部开启事务，可以借助于`transaction.atomic`方法。使用它我们就可以创建一个具备原子性的代码块，一旦代码块正常运行完毕，所有的修改会被提交到数据库。反之，如果有异常，更改会被回滚。

`atomic`经常被当做[`装饰器`](https://docs.python.org/3/glossary.html#term-decorator)来使用，如下所示：

```python
# 案例一：函数视图
from django.db import transaction

@transaction.atomic
def viewfunc(request):
    # This code executes inside a transaction.
    do_stuff()

# 案例二： 基于类的视图
from django.db import transaction
from rest_framework.views import APIView

class OrderAPIView(APIView):
        # 开启事务，当方法执行完以后，自动提交事务
        @transaction.atomic  
        def post(self, request):
             pass  #
```

使用了`atomic`装饰器，整个视图方法里的代码块都会包裹着一个事务中运行。有时我们希望只对视图方法里一小段代码使用事务，这时可以使用`transaction.atomic()`显式地开启事务，如下所示：

```python
from django.db import transaction

def viewfunc(request):
    # 默认自动提交
    do_stuff()
    
    # 显式地开启事务
    with transaction.atomic():
        # 下面这段代码在事务中执行
        do_more_stuff()
```

## Savepoint回滚

在事务操作中，我们还会经常显式地设置保存点(savepoint)。一旦发生异常或错误，我们使用`savepoint_rollback`方法让程序回滚到指定的保存点。如果没有问题，就使用`savepoint_commit`方法提交事务。示例代码如下：

```python
from django.db import transaction

def viewfunc(request):
    # 默认自动提交
    do_stuff()
    
    # 显式地开启事务
    with transaction.atomic():
        # 创建事务保存点
        sid = transaction.savepoint()
        
        try:
            do_more_stuff()
        except Exception as e:
            # 如发生异常，回滚到指定地方。
            transaction.savepoint_rollback(sid)
            
        # 如果没有异常，显式地提交一次事务
        transaction.savepoint_commit(sid)
    
    return HttpResponse("Success")
```

注意：虽然SQLite支持保存点，但是[`sqlite3`](https://docs.python.org/3/library/sqlite3.html#module-sqlite3) 模块设计中的缺陷使它们很难使用。

## 事务提交后回调函数

有的时候我们希望当前事务提交后立即执行额外的任务，比如客户下订单后立即邮件通知卖家，这时可以使用Django提供的`on_commit`方法，如下所示：

```python
# 例1
from django.db import transaction

def do_something():
    pass  # send a mail, invalidate a cache, fire off a Celery task, etc.

transaction.on_commit(do_something)

# 例2：调用celery异步任务
transaction.on_commit(lambda: some_celery_task.delay('arg1'))
```

## 悲观锁与乐观锁

在电商秒杀等高并发场景中，仅仅开启事务还是无法避免数据冲突。比如用户A和用户B获取某一商品的库存并尝试对其修改，A, B查询的商品库存都为5件，结果A下单5件，B也下单5件，这就出现问题了。解决方案就是操作( 查询或修改)某个商品库存信息是对其加锁。

常见的锁有悲观锁和乐观锁，接下来我们来看下在Django项目中如何通过代码实现：

- 悲观锁就是在操作数据时，假定此操作会出现数据冲突，在整个数据处理过程中，使数据处于锁定状态。 悲观锁的实现，往往依靠数据库提供的锁机制实现的。
- 乐观锁是指操作数据库时想法很乐观，认为这次的操作不会导致冲突，在操作数据时，并不进行任何其他的特殊处理, 而在进行更新时，再去判断是否有冲突了。乐观锁不是数据库提供的锁，需要我们自己去实现。

### 悲观锁

Django中使用悲观锁锁定一个对象，需要使用`select_for_update()`方法。它本质是一个行级锁，能锁定所有匹配的行，直到事务结束。两个应用示例如下所示：

```python
# 案例1：类视图，锁定id=10的SKU对象
class OrderView(APIView):
    
    @transaction.atomic
    def post(self, request):
        # select_for_update表示锁,只有获取到锁才会执行查询,否则阻塞等待。
        sku = GoodsSKU.objects.select_for_update().get(id=10)
        
        # 等事务提交后，会自动释放锁。 
        return Response("xxx")

    
# 案例2：函数视图，锁定所有符合条件的文章对象列表。
from django.db import transaction

with transaction.atomic():
    entries = Entry.objects.select_for_update().filter(author=request.user)
    for entry in entries:
        ...
```

一般情况下如果其他事务锁定了相关行，那么本次查询将被阻塞，直到锁被释放。 如果不想要使查询阻塞的话，使用`select_for_update(nowait=True)`。

当你同时使用`select_for_update`与`select_related`方法时，`select_related`指定的相关对象也会被锁定。你可以通过` select_for_update(of=(...))`方法指定需要锁定的关联对象，如下所示：

```python
# 只会锁定entry(self)和category，不会锁定作者author
entries = Entry.objects.select_related('author', 'category'). select_for_update(of=('self', 'category'))
```

注意：

1. `select_for_update`方法必须与事务(transaction)同时使用。
2. MySQL版本要在8.0.1+ 以上才支持 `nowait`和 `of`选项。

### 乐观锁

乐观锁实现一般使用记录版本号，为数据表增加一个版本标识(version)字段，每次对数据的更新操作成功后都对版本号执行+1操作。每次执行更新操作时都去判断当前版本号是不是该条数据的最新版本号，如果不是说明数据已经同时被修改过了，则丢弃更新，需要重新获取目标对象再进行更新。

Django项目中实现乐观锁可以借助于`django-concurrency`这个第三方库, 它可以给模型增加一个`version`字段，每次执行save操作时会自动给版本号+1。

```python
from django.db import models
from concurrency.fields import IntegerVersionField

class ConcurrentModel( models.Model ):
    version = IntegerVersionField( )
    name = models.CharField(max_length=100)
```

下例中a和b同时获取了pk=1的模型对象信息，并尝试对其name字段进行修改。由于a.save()方法调用成功以后对象的版本号version已经加1，b再调用b.save()方法时将会报`RecordModifiedError`的错误，这样避免了a，b同时修改同一对象信息造成数据冲突。

```python
a = ConcurrentModel.objects.get(pk=1)
a.name = '1'

b = ConcurrentModel.objects.get(pk=1)
b.name = '2'

a.save()
b.save()
```

那么问题来了，什么时候该用悲观锁，什么时候该用乐观锁呢？这主要需要考虑4个因素：

- 并发量： 如果并发量不大且不允许脏读，可以使用悲观锁解决并发问题；但如果系统的并发非常大的话,悲观锁定会带来非常大的性能问题, 建议乐观锁。
- 响应速度：如果需要非常高的响应速度，建议采用乐观锁方案，成功就执行，不成功就失败，不需要等待其他并发去释放锁。乐观锁并未真正加锁，效率高。

- 冲突频率：如果冲突频率非常高，建议采用悲观锁，保证成功率。冲突频率大，选择乐观锁会需要多次重试才能成功，代价比较大。

- 重试代价：如果重试代价大，建议采用悲观锁。悲观锁依赖数据库锁，效率低。更新失败的概率比较低。

## 小结

本文总结了事务的四大特性以及Django项目开发中如何操作事务，并以实际代码演示了悲观锁和乐观锁。你都学会了吗?

原创不易，转载请注明来源。我是大江狗，一名Django技术开发爱好者。您可以通过搜索【<a href="https://blog.csdn.net/weixin_42134789">CSDN大江狗</a>】、【<a href="https://www.zhihu.com/people/shi-yun-bo-53">知乎大江狗</a>】和搜索微信公众号【Python Web与Django开发】关注我！

![Python Web与Django开发](../../assets/images/django.png)

