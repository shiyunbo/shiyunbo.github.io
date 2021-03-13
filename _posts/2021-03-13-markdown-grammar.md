```yaml
layout: post
title: Markdown基本语法
description: "Markdown的基本语法，如标题、引用、列表、图片、超链接等等。"
author: 大江狗
category: Notes
```

# markdown基本语法

## 标题
###  ##二级标题
### ###三级标题

## 字体
正常文本
**加粗** 
*斜体* 
***斜体加粗*** 
~~删除线~~ 
下脚标：H~2~O
上脚标：x^2^


## 引用
> 一级引用, 爱因斯坦说该国
>
> > 二级应用，他真的说过吗

## 代码块
### 句中代码，使用反引号`
这是句中代码示例，可以输入`enter`键查`左一`看
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
姓名|年龄|性别
---|:--:|---:
张三|20|男
李四|18|女

## 脚注
这是一个[^footnote1]


## 待办事宜

- [ ] **Cmd Markdown 开发**
    - [x] 改进 Cmd 渲染算法，使用局部渲染技术提高渲染效率
    - [x] 支持以 PDF 格式导出文稿
    - [x] 新增Todo列表功能 [语法参考](https://github.com/blog/1375-task-lists-in-gfm-issues-pulls-comments)
    - [ ] 改进 LaTex 功能
        - [ ] 修复 LaTex 公式渲染问题
        - [ ] 新增 LaTex 公式编号功能
- [ ] **自定义待办事项**
    - [x] 已完成
    - [ ] 未完成

- [ ] **新的任务**

## 反斜杠转义
Markdown 使用反斜杠`\`转义特殊字符，比如`*`
双*号正常显示: **加粗文本**
转义字符: \*\*加粗文本\*\*

## 分割线
使用`****`或则`----`分割

## 流程图
```bash
st=>start: Start
io=>inputoutput: Input and Output
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
```

示例：
```flow
st=>start: Start
io=>inputoutput: Input and Output
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
```

更多语法参考：[流程图语法参考](http://adrai.github.io/flowchart.js/)

## 序列图

#### 示例 1

```seq
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

#### 示例 2

```seq
Title: Here is a title
A->B: Normal line
B-->C: Dashed line
C->>D: Open arrow
D-->>A: Dashed open arrow
```

更多语法参考：[序列图语法参考](http://bramp.github.io/js-sequence-diagrams/)

## 甘特图
甘特图内在思想简单。基本是一条线条图，横轴表示时间，纵轴表示活动（项目），线条表示在整个期间上计划和实际的活动完成情况。它直观地表明任务计划在什么时候进行，及实际进展与计划要求的对比。

```gantt
    title 项目开发流程
    section 项目确定
        需求分析       :a1, 2016-06-22, 3d
        可行性报告     :after a1, 5d
        概念验证       : 5d
    section 项目实施
        概要设计      :2016-07-05  , 5d
        详细设计      :2016-07-08, 10d
        编码          :2016-07-15, 10d
        测试          :2016-07-22, 5d
    section 发布验收
        发布: 2d
        验收: 3d
```

更多语法参考：[甘特图语法参考](https://knsv.github.io/mermaid/#gant-diagrams)



## 数学公式
当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。

$ 表示行内公式： 
```
$E=mc^2$ 
```

质能守恒方程可以用一个很简洁的方程式 $E=mc^2$ 来表达。

$$ 表示整行公式：
```
$$\sum_{i=1}^n a_i=0$$

$$f(x_1,x_x,\ldots,x_n) = x_1^2 + x_2^2 + \cdots + x_n^2 $$

$$\sum^{j-1}_{k=0}{\widehat{\gamma}_{kj} z_k}$$
```

示例
$$\sum_{i=1}^n a_i=0$$

$$f(x_1,x_x,\ldots,x_n) = x_1^2 + x_2^2 + \cdots + x_n^2 $$

$$\sum^{j-1}_{k=0}{\widehat{\gamma}_{kj} z_k}$$


更多参考：[MathJax](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference) 

向量矩阵
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
${$tep1}{\style{visibility:hidden}{(x+1)(x+1)}}
$$
方程
$$
x^2+y^2=z^2 \tag{1$'$}
$$
$$
x^3+y^3=z^3  
$$
$$
x^4+y^4=z^4 \tag{*} 
$$
$$
x^5+y^5=z^5 \tag*{*}
$$
$$
x^6+y^6=z^6 \tag{1-1} 
$$

复数
$$
z = \overbrace{
   \underbrace{x}_\text{real} + i
   \underbrace{y}_\text{imaginary}
  }^\text{complex number}
$$

分段函数
$$
f(x) = \left\{
  \begin{array}{lr}
    x^2 & : x < 0\\
    x^3 & : x \ge 0
  \end{array}
\right.
$$

$$
u(x) = 
  \begin{cases} 
   \exp{x} & \text{if } x \geq 0 \\
   1       & \text{if } x < 0
  \end{cases}
$$

方程组
$$
\left\{ 
\begin{array}{c}
    a_1x+b_1y+c_1z=d_1 \\ 
    a_2x+b_2y+c_2z=d_2 \\ 
    a_3x+b_3y+c_3z=d_3
\end{array}
\right. 
$$



线性模型
$$
h(\theta) = \sum_{j = 0} ^n \theta_j x_j
$$



均方误差
$$
J(\theta) = \frac{1}{2m}\sum_{i = 0} ^m(y^i - h_\theta (x^i))^2
$$



批量梯度下降
$$
\frac{\partial J(\theta)}{\partial\theta_j}=-\frac1m\sum_{i=0}^m(y^i-h_\theta(x^i))x^i_j 
$$



推导过程
$$
\begin{aligned}
\frac{\partial J(\theta)}{\partial\theta_j}
& = -\frac1m\sum_{i=0}^m(y^i-h_\theta(x^i)) \frac{\partial}{\partial\theta_j}(y^i-h_\theta(x^i)) \\
& = -\frac1m\sum_{i=0}^m(y^i-h_\theta(x^i)) \frac{\partial}{\partial\theta_j}(\sum_{j=0}^n\theta_jx_j^i-y^i) \\
& = -\frac1m\sum_{i=0}^m(y^i-h_\theta(x^i))x^i_j
\end{aligned}
$$



case环境的使用
$$
a =
   \begin{cases}
     \int x\, \mathrm{d} x\\
     b^2
   \end{cases}
$$
​	


带方框的等式
$$
\begin{aligned}
 \boxed{x^2+y^2 = z^2}
\end{aligned}
$$



最大（最小）操作符
$$
\begin{gathered}
\operatorname{arg\,max}_a f(a) 
 = \operatorname*{arg\,max}_b f(b) \\
 \operatorname{arg\,min}_c f(c) 
 = \operatorname*{arg\,min}_d f(d)
\end{gathered}
$$



求极限
$$
\begin{aligned}
  \lim_{a\to \infty} \tfrac{1}{a}
\end{aligned}
$$
$$
\begin{aligned}
   \lim\nolimits_{a\to \infty} \tfrac{1}{a}
\end{aligned}
$$



求积分
$$
\begin{aligned}
   \int_a^b x^2  \mathrm{d} x
\end{aligned}
$$
$$
\begin{aligned}
   \int\limits_a^b x^2  \mathrm{d} x
\end{aligned}
$$




求累加
$$
\begin{aligned}
   \sum\nolimits' C_n
\end{equation}
$$

$$
\begin{aligned}
   \sum_{n=1}\nolimits' C_n
\end{aligned}
$$

$$
\begin{aligned}
   \sideset{}{'}\sum_{n=1}C_n
\end{aligned}
$$

$$
\begin{aligned}
   \sideset{_a^b}{_c^d}\sum
\end{aligned}
$$



省略号的使用
$$
{1+2+3+\ldots+n}
$$

[^footnote1]: *注脚* 在本文结尾部分。