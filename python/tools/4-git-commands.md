---
layout: default
title: Git命令大全
parent: Python Web开发工具
nav_order: 4
---

# Git命令大全
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---
Git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。Git已成为大多数IT企业首选的代码版本控制工具，也是Python Web开发者必需熟练使用的工具。本文整理了Git命令大全，可以作为开发者参考。
{: .fs-6 .fw-300 }

## 基本配置
```bash
# 查看当前生效的配置信息
$ git config -l

# 配置提交用户信息，--global代表全局
$ git config --global user.name <用户名>
$ git config --global user.email <邮箱地址>

# 更改Git缓存区的大小
# 缓存大小单位：B，例如：524288000（500MB）
$ git config --global http.postBuffer <缓存大小>

# 配置可以缓存密码，默认缓存时间15分钟
# 缓存时间单位：秒
$ git config --global credential.helper 'cache --timeout=<缓存时间>'

# 配置长期存储密码
$ git config --global credential.helper store
```
## 基础操作

```bash
# 初始化本地仓库，在当前目录下生成 .git 文件夹
$ git init

# 查看本地仓库的状态
$ git status

# 把指定1个或多个文件添加到暂存区中
$ git add <文件路径>

# 将工作区所有变化提交到暂存区，包括修改、新文件和删除文件。
$ git add . 
$ git add -A <文件路径>
$ git add --all <文件路径>

# 添加所有修改、已删除的文件到暂存区中，不包含新增文件
$ git add -u [<文件路径>]

# 移除跟踪指定的文件，并从本地仓库的文件夹中删除
$ git rm <文件路径>

# 移除跟踪指定的文件夹，并从本地仓库的文件夹中删除
$ git rm -r <文件夹路径>

# 移除跟踪指定的文件，在本地仓库的文件夹中保留该文件
$ git rm --cached

# 撤消工作区对文件的修改, 慎用
$ git checkout -- <文件名>

# 把暂存区中的文件提交到本地仓库中并添加描述信息
$ git commit -m "<提交的描述信息>"

# 修改上次提交的描述信息
$ git commit --amend

# 打印所有的提交记录，使用--oneline选项可以简化输出
$ git log 

# 使用--author选项可以打印出某个用户的提交记录
$ git log --author="大江狗"
```
## 版本回退
```bash
# 可查看所有分支的操作记录（包括已被删除的 commit 记录和 reset 的操作）
$ git reflog 

# 将 HEAD 的指向改变，回退到commit前的状态，文件未改变
$ git reset --soft <commit ID>

# 将 HEAD 的指向改变，文件被修改，均回到之前版本
$ git reset --hard <commit ID>

# 回退到上个版本
$ git reset --hard HEAD^

# 回退到前2次提交之前，以此类推，回退到n次提交之前
$ git reset --hard HEAD~3 

# 按提交id，可以回退，也可以前进
$ git reset --hard <commit ID>

# 生成一个新的提交来撤销某次提交
$ git revert <commit ID>
```

## 远程操作
```bash
# 列出已经存在的远程仓库
$ git remote

# 列出远程仓库的详细信息，在别名后面列出URL地址
$ git remote -v

# 添加远程仓库，别名一般默认origin
$ git remote add <远程仓库的别名> <远程仓库的URL地址>

# 修改远程仓库的别名
$ git remote rename <原远程仓库的别名> <新的别名>

# 删除指定名称的远程仓库
$ git remote remove <远程仓库的别名>

# 修改远程仓库的 URL 地址
$ git remote set-url <远程仓库的别名> <新的远程仓库URL地址>

# 将远程仓库所有分支的最新版本全部取回到本地
$ git fetch <远程仓库的别名>

# 将远程仓库指定分支的最新版本取回到本地
$ git fetch <远程主机名> <分支名>

# 把指定的分支合并到当前所在的分支下
$ git merge <分支名称>

# 从远程仓库获取最新版本，并合并，等于fetch + merge
$ git pull

# 把本地仓库的分支推送到远程仓库的指定分支
$ git push <远程仓库的别名> <本地分支名>:<远程分支名>

# 首次使用u推送后，下次无需设置别名和本地分支可直接git push。
$ git push -u origin main

# 删除指定的远程仓库的分支
$ git push <远程仓库的别名> :<远程分支名>
$ git push <远程仓库的别名> --delete <远程分支名>

# 删除指定的远程仓库的分支
$ git push <远程仓库的别名> :<远程分支名>
$ git push <远程仓库的别名> --delete <远程分支名>

# 将远程仓库代码复制一份到本地当前目录
$ git clone <远程仓库的网址>

# 指定本地仓库的目录
$ git clone <远程仓库的网址> <本地目录>

# -b 指定要克隆的分支，默认是main分支
$ git clone <远程仓库的网址> -b <分支名称> <本地目录>
```
## 分支操作
```bash
# 列出本地的所有分支，当前所在分支以 "*" 标出
$ git branch

# 同时列出本地和远程的所有分支
$ git branch -a

# 列出本地的所有分支并显示最后一次提交，当前所在分支以 "*" 标出
$ git branch -v

# 创建新分支，新的分支基于上一次提交建立
$ git branch <分支名>

# 修改分支名称
# 如果不指定原分支名称则为当前所在分支
$ git branch -m [<原分支名称>] <新的分支名称>

# 强制修改分支名称
$ git branch -M [<原分支名称>] <新的分支名称>

# 删除指定的本地分支
$ git branch -d <分支名称>

# 强制删除指定的本地分支
$ git branch -D <分支名称>

# 删除远程分支
$ git push origin --delete <分支名称>

# 切换到已存在的指定分支，如git checkout main
$ git checkout <分支名称>

# 与指定分支合并，比如将刚切换的main分支与dev分支合并
# 合并前建议先拉取git pull origin main，然后再合并
$ git merge dev

# 把已经提交的记录合并到当前分支
$ git cherry-pick <commit ID>

# 创建+切换到指定的分支，保留所有的提交记录
# 等同于 "git branch" 和 "git checkout" 两个命令合并
$ git checkout -b <分支名称>

# 创建+切换到指定的分支，删除所有的提交记录
$ git checkout --orphan <分支名称>
```
## 标签管理

```bash
# 打印所有的标签
$ git tag

# 添加轻量标签，指向提交对象的引用，可以指定之前的提交记录
$ git tag <标签名称> [<commit ID>]

# 添加带有描述信息的附注标签，可以指定之前的提交记录
$ git tag -a <标签名称> -m <标签描述信息> [<commit ID>]

# 切换到指定的标签
$ git checkout <标签名称>

# 查看标签的信息
$ git show <标签名称>

# 删除指定的标签
$ git tag -d <标签名称>

# 将指定的标签提交到远程仓库
$ git push <远程仓库的别名> <标签名称>

# 将本地所有的标签全部提交到远程仓库
$ git push <远程仓库的别名> –tags
```

## 参考资料

- https://git-scm.com/

- https://www.jianshu.com/p/93318220cdce

我是大江狗，一名Python Web开发与Django技术开发爱好者。您可以通过搜索【<a href="https://blog.csdn.net/weixin_42134789">CSDN大江狗</a>】、【<a href="https://www.zhihu.com/people/shi-yun-bo-53">知乎大江狗</a>】和搜索微信公众号【Python Web与Django开发】关注我！

![Python Web与Django开发](../../assets/images/django.png)
