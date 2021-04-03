---
layout: default
title:  Django权限详解
parent: 大江狗的Django进阶教程
nav_order: 8
---

# Django权限详解
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

如果你只是利用Django开发个人博客，大部分用户只是阅读你的文章而已，你可能根本用不到本文内容。但是如果你想开发一个内容管理系统或用户管理系统，你必需对用户的权限进行管理和控制。Django自带的权限机制(permissions)与用户组(group)可以让我们很方便地对用户权限进行管理。小编我今天就尝试以浅显的语言来讲解下如何使用Django自带的权限管理机制。

## 什么是权限?

权限是能够约束用户行为和控制页面显示内容的一种机制。一个完整的权限应该包含3个要素: 用户，对象和权限，即什么用户对什么对象有什么样的权限。

假设我们有一个应用叫blog，其包含一个叫Article(文章)的模型。那么一个超级用户一般会有如下4种权限，而一个普通用户可能只有1种或某几种权限，比如只能查看文章，或者能查看和创建文章但是不能修改和删除。

- 查看文章(view)

- 创建文章(add)

- 更改文章(change)
- 删除文章(delete) 

我们在Django的管理后台(admin)中是可以很轻易地给用户分配权限的。



Django Admin中的权限分配

Django中的用户权限分配，主要通过Django自带的Admin界面进行维护的。当你编辑某个user信息时, 你可以很轻易地在User permissions栏为其设置对某些模型查看, 增加、更改和删除的权限(如下图所示)。



Django的权限permission本质是djang.contrib.auth中的一个模型, 其与User的user_permissions字段是多对多的关系。当我们在INSTALLED_APP里添加好auth应用之后，Django就会为每一个你安装的app中的模型(Model)自动创建4个可选的权限：view, add,change和delete。(注: Django 2.0前没有view权限)。随后你可以通过admin将这些权限分配给不同用户。

 

查看用户的权限

 

权限名一般有app名(app_label)，权限动作和模型名组成。以blog应用为例，Django为Article模型自动创建的4个可选权限名分别为:

查看文章(view): blog.view_article

创建文章(add): blog.add_article

更改文章(change): blog.change_article

删除文章(delete): blog.delete_article

 

在前例中，我们已经通过Admin给用户A(user_A)分配了创建文章和修改文章的权限。我们现在可以使用user.has_perm()方法来判断用户是否已经拥有相应权限。下例中应该返回True。

user_A.has_perm('blog.add_article')

user_A.has_perm('blog.change_article')

 

如果我们要查看某个用户所在用户组的权限或某个用户的所有权限(包括从用户组获得的权限)，我们可以使用get_group_permissions()和get_all_permissions()方法。

user_A.get_group_permissions()

user_A.get_all_permissions()

 

手动定义和分配权限(Permissions)

 

有时django创建的4种可选权限满足不了我们的要求，这时我们需要自定义权限。实现方法主要有两种。下面我们将分别使用2种方法给Article模型新增了两个权限，一个是publish_article, 一个是comment_article。

 

方法1. 在Model的meta属性中添加permissions。

class Article(models.Model):
    ...
    class Meta:
        permissions = (
            ("publish_article", "Can publish article"),
            ("comment_article", "Can comment article"),
        )


方法2. 使用ContentType程序化创建permissions。

from blog.models import Article
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(article)
permission1 = Permission.objects.create(
    codename='publish_article',
    name='Can publish articles',
    content_type=content_type,
)

permission2 = Permission.objects.create(
    codename='comment_article',
    name='Can comment articles',
    content_type=content_type,
)
当你使用python manage.py migrate命令后，你会发现Django admin的user permissions栏又多了两个可选权限。

 

如果你不希望总是通过admin来给用户设置权限，你还可以在代码里手动给用户分配权限。这里也有两种实现方法。

 

方法1. 使用user.user_permissions.add()方法

myuser.user_permissions.add(permission1, permission2, ...)


方法2. 通过user所在的用户组(group)给用户增加权限

mygroup.permissions.add(permission1, permission2, ...)


如果你希望在代码中移除一个用户的权限，你可以使用remove或clear方法。

myuser.user_permissions.remove(permission, permission, ...)
myuser.user_permissions.clear()


注意权限的缓存机制

 

Django会缓存每个用户对象，包括其权限user_permissions。当你在代码中手动改变一个用户的权限后，你必须重新获取该用户对象，才能获取最新的权限。

 

比如下例在代码中给用户手动增加了change_blogpost的权限，如果不重新载入用户，那么将显示用户还是没有change_blogpost的权限。

from django.contrib.auth.models import Permission, User
from django.contrib.contenttypes.models import ContentType
from django.shortcuts import get_object_or_404

from myapp.models import BlogPost

def user_gains_perms(request, user_id):
    user = get_object_or_404(User, pk=user_id)
    # any permission check will cache the current set of permissions
    user.has_perm('myapp.change_blogpost') 

    content_type = ContentType.objects.get_for_model(BlogPost)
    permission = Permission.objects.get(
        codename='change_blogpost',
        content_type=content_type,
    )
    user.user_permissions.add(permission)
     
    # Checking the cached permission set
    user.has_perm('myapp.change_blogpost')  # False
     
    # Request new instance of User
    # Be aware that user.refresh_from_db() won't clear the cache.
    user = get_object_or_404(User, pk=user_id)
     
    # Permission cache is repopulated from the database
    user.has_perm('myapp.change_blogpost')  # True


用户权限的验证(Validation)

 

我们前面讲解了用户权限的创建和设置，现在我们将进入关键一环，用户权限的验证。我们在分配好权限后，我们还需要在视图views.py和模板里验证用户是否具有相应的权限，否则前面设置的权限形同虚设。这就是为什么我们前面很多django实战案例里，没有给用户分配某个模型的add和change权限，用户还是还能创建和编辑对象的原因。

 

1. 视图中验证

在视图中你当然可以使用user.has_perm方法对一个用户的权限进行直接验证。当然一个更好的方法是使用@permission_required这个装饰器。

permission_required(perm, login_url=None, raise_exception=False)

你如果指定了login_url, 用户会被要求先登录。如果你设置了raise_exception=True, 会直接返回403无权限的错误，而不会跳转到登录页面。使用方法如下所示:

from django.contrib.auth.decorators import permission_required

@permission_required('polls.can_vote')
def my_view(request):
    ...
如果你使用基于类的视图(Class Based View), 而不是函数视图，你需要继承PermissionRequiredMixin这个类，如下所示:

from django.contrib.auth.mixins import PermissionRequiredMixin

class MyView(PermissionRequiredMixin, View):
    permission_required = 'polls.can_vote'
    # Or multiple of permissions:
    permission_required = ('polls.can_open', 'polls.can_edit')


2. 模板中验证

在模板中验证用户权限主要需要学会使用perms这个全局变量。perms对当前用户的user.has_module_perms和user.has_perm方法进行了封装。当我们需要判断当前用户是否拥有blog应用下的所有权限时，我们可以使用:

{{ perms.blog }}


我们如果判断当前用户是否拥有blog应用下发表文章讨论的权限，则使用:

{{ perms.blog.comment_article }}
这样结合template的if标签，我们可以通过判断当前用户所具有的权限，显示不同的内容了.

{% if blog.article %}
    <p>You have permission to do something in this blog app.</p>
    {% if perms.blog.add_article %}
        <p>You can add articles.</p>
    {% endif %}
    {% if perms.blog.comment_article %}
        <p>You can comment articles!</p>
    {% endif %}
{% else %}
    <p>You don't have permission to do anything in the blog app.</p>
{% endif %}


用户组(Group)

 

用户组(Group)和User模型是多对多的关系。其作用在权限控制时可以批量对用户的权限进行管理和分配，而不用一个一个用户分配，节省工作量。将一个用户加入到一个Group中后，该用户就拥有了该Group所分配的所有权限。例如，如果一个用户组editors有权限change_article, 那么所有属于editors组的用户都会有这个权限。

 

将用户添加到用户组或者给用户组(group)添加权限，一般建议直接通过django admin进行。如果你希望手动给group添加或删除权限，你可以使用如下方法。

mygroup.permissions = [permission_list]
mygroup.permissions.add(permission, permission, ...)
mygroup.permissions.remove(permission, permission, ...)
mygroup.permissions.clear()
如果你要将某个用户移除某个用户组，可以使用如下方法。

myuser.groups.remove(group, group, ...) #
myuser.groups.clear()


Django自带权限机制的不足

 

Django自带的权限机制是针对模型的，这就意味着一个用户如果对Article模型有change的权限，那么该用户获得对所有文章对象进行修改的权限。如果我们希望实现对单个文章对象的权限管理，我们需要借助于第三方库比如django guardian。至于该库的使用，我们以后再详细介绍。




原创不易，转载请注明来源。我是大江狗，一名Django技术开发爱好者。您可以通过搜索【<a href="https://blog.csdn.net/weixin_42134789">CSDN大江狗</a>】、【<a href="https://www.zhihu.com/people/shi-yun-bo-53">知乎大江狗</a>】和搜索微信公众号【Python Web与Django开发】关注我！

![Python Web与Django开发](../../assets/images/django.png)