---
layout: post
title: Markdown基本语法
description: "markdown基本语法，如标题、字体、引用、列表、图片、超链接和表格等等。"
author: 大江狗
category: Notes
---

Markdown的语言浅显易懂，可以让作者专注于写作。本文总结了markdown基本语法，如标题、字体、引用、列表、图片、超链接和表格等等。

## 标题

```bash
# 一级标题
## 二级标题
### 三界标题
```

## 字体

```bash
正常文本
**加粗** 
*斜体* 
***斜体加粗*** 
~~删除线~~ 
下脚标：H~2~O
上脚标：x^2^
```

展示效果如下所示：

正常文本
**加粗** 
*斜体* 
***斜体加粗*** 
~~删除线~~ 


## 引用
```bash
> 这是一级引用，爱因斯坦说
>
> > 二级应用，他真的说过吗
```
展示效果如下所示：
> 一级引用, 爱因斯坦说
>
> > 二级应用，他真的说过吗

## 代码块

句中引用代码或代码高亮，使用一个单反引号开始。对于如下代码块，可以使用三个反引号开始。反引号(\`)一般位于键盘的左上角。

### shell代码块

```shell
pip install django-celery
```

### python代码块

```python
# List with filter - Login not required
def article_list_by_author(request, username):
    user = get_object_or_404(User, username=username)
    return render(request, 'blog/article_list_by_author.html', context)
```

## 列表

无序列表用*号, -号或+号开头

* 农药法规介绍
* 如何风险评估

有序列表直接用数字

1. 生活的苦恼
2. 我们的爱情

列表嵌套，只需要另起新行时留3个空格

1. 一级有序
   * 一级无序 
   * 二级无序
2. 二级有序

## 图片

```bash
![图片alt](图片地址 ''可选图片title'')
例子:
![阿里云logo](https://img.alicdn.com/tfs/TB1Ly5oS3HqK1RjSZFPXXcwapXa-238-54.png ''阿里云logo '')
```

![阿里云Logo](https://img.alicdn.com/tfs/TB1Ly5oS3HqK1RjSZFPXXcwapXa-238-54.png)

也支持直接插入或粘贴，粘贴后的图片位于在`Typora\typora-user-images\`文件夹。

## 超链接

```csharp
[超链接名](超链接地址 "超链接可选标题title")
title可加可不加。也可直接使用html原生语句<a href=""></a>
```

示例:
[百度](http://baidu.com)
<a href="https://www.jianshu.com/u/1f5ac0cf6a8b" target="_blank">简书</a>

电子邮箱：我的email是<john@gmail.com>，使用`<>`包括email地址

## 表格

```bash
表头|表头|表头
---|:--:|---:
内容|内容|内容
内容|内容|内容

第二行|分割表头和内容。
- 有一个就行，为了对齐，多加了几个
---表示文字默认居左
:--:表示文字居中
---：表示文字居右
注：原生的语法两边都要用 | 包起来。此处省略
```

示例:

| 姓名 | 年龄 | 性别 |
| ---- | :--: | ---: |
| 张三 |  20  |   男 |
| 李四 |  18  |   女 |

## 脚注
```bash
这是一个[^footnote1] 
# 注意还需要定义footnote1的链接
```
这是一个[^footnote1]

## 反斜杠转义

Markdown 使用反斜杠`\`转义特殊字符，比如`*`
双*号正常显示: **加粗文本**
转义字符: \*\*加粗文本\*\*

## 分割线

使用`****`或则`----`分割

## 待办事宜

```bash
- [x] **自定义待办事项**
   - [x] 已完成
   - [ ] 未完成
- [x] **新的任务**
```


- [x] **自定义待办事项**
   - [x] 吃饭
   - [ ] 睡觉
- [x] **新的任务**


[^footnote1]: *注脚* 在本文结尾部分。