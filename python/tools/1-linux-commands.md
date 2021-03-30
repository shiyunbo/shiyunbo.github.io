---
layout: default
title: Linux常用命令大全
parent: Python Web开发工具
nav_order: 1
---

# Linux常用命令大全
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---
实际工作中Python Web项目基本上都是部署在Linux系统上的(比如Ubuntu, Centos)，因此熟悉掌握Linux命令对于Python开发和运维工作者是最基本的要求。本文根据网络内容整理了Python Web开发项目经常用到的Linux命令。
{: .fs-6 .fw-300 }

## 文件和目录操作

```bash
# 切换目录
cd /home 进入/home目录
cd .. 返回上一级目录
cd ../.. 返回上两级目录
cd ~user1 进入user1的主目录
cd - 返回上次所在的目录
pwd 显示当前工作目录

# 查看文件
ls 查看目录中的文件
ls -F 查看目录中的文件
ls -l 显示文件和目录的详细资料
ls -a 显示隐藏文件
ls *［0-9］* 显示包含数字的文件名和目录名
cat 查看文件内容
more、less  分页显示文本文件内容
head、tail   显示文件头、尾内容
basename 显示文件名或目录名
dirname 显示文件或目录路径。
tree 显示文件和目录由根目录开始的树形结构

# 文件及目录的创建与删除
touch 创建空文件
echo 创建带有内容的文件
mkdir dir1 创建dir1目录
mkdir dir1 dir2 同时创建两个目录
mkdir -p /tmp/dir1/dir2 逐级创建多个目录
rm -f file1 删除file1文件，-f强制删除
rmdir dir1 删除dir1空目录
rm -rf dir1 强制删除dir1目录及其子目录及文件，-r表示递归删除

# 文件与目录的移动及复制
mv dir1 new_dir 重命名/移动一个目录
cp file1 file2 复制一个文件
cp dir/* . 复制一个目录下的所有文件到当前工作目录
cp -a /tmp/dir1. 复制一个目录到当前工作目录, -a表示所有
ln -s file1 lnk1 创建一个指向文件或目录的软链接
ln file1 lnk1 创建一个指向文件或目录的物理链接
```

## 文件查找与统计

```bash
# 创建一个属于 “admin” 用户组的用户
find / -name file1 

# 搜索属于用户 ‘user1’ 的文件和目录
find / -user user1 

# 在目录‘/ home/user1’ 中搜索带有‘.bin’ 结尾的文件
find /home/user1 -name \*.bin 

# 搜索在过去100天内未被使用过的执行文件
find /usr/bin -type f -atime +100 

# 搜索在10天内被创建或者修改过的文件
find /usr/bin -type f -mtime -10 

# 搜索以 ‘.rpm’ 结尾的文件并定义其权限
find / -name \*.rpm -exec chmod 755 ‘{}’ \; 

# 在所有txt文件中查找包含有python的文件。-l表示以列表显示
grep -l python *.txt  

# 查找etc及子目录包含python的文件，-i表示不区分大小写，-R表示递归
grep -iR python /etc/* 

# 寻找以 ‘.ps’ 结尾的文件 
locate \*.ps 

# 显示一个二进制文件、源码或man的位置,比如bash命令
whereis bash 

# 显示一个二进制文件或可执行文件(比如bash)的完整路径
which bash 

# 统计text.txt中行数、字数、字符数
wc test.txt 
```

## 用户和群组

```bash
# 群组的管理
groupadd group_name 创建一个新用户组
groupdel group_name 删除一个用户组
groupmod -n new_group old_group 重命名一个用户组
newgrp group_name 登陆进一个新的群组以改变新创建文件的预设群组

# 新用户的创建
useradd user1 创建一个新用户，可结合如下选项使用：
    -u 指定用户的UID
    -g 指定用户所属的群组
    -d 指定用户的home目录
    -c 指定用户的备注信息
    -s 指定用户所用的shell

# 修改用户属性
usermod -c “FTP User” -g system -d /ftp/user1 -s /bin/nologin user1 

# 删除一个用户 （ ’-r‘ 排除主目录）
userdel -r user1 

# 密码管理
passwd 修改口令
passwd user1 修改一个用户的口令 （只允许root执行）
change -E 2020-12-31 user1 设置用户口令的失效期限

# 用户切换
id 查看用户的uid,gid及归属的用户组。
su 切换用户身份。
visudo 编辑/etc/sudoers文件的专属命令。
sudo 以另外一个用户身份（默认root用户）执行事先在sudoers文件允许的命令。
```

## 权限管理

```bash
# 显示权限
ls -lh 

# 增加目录的所有人（u）、群组（g）以及其他人（o）读、写和执行的权限
# + 表示添加，r, w, x分别代表读、写和执行
chmod ugo+rwx directory1 

# 删除群组（g）与其他人（o）对目录的读写执行权限，-表示移除
chmod go-rwx directory1 

# 改变一个文件的所有人属性
chown user1 file1

# 改变一个目录的所有人属性并同时改变改目录下所有文件的属性
chown -R user1 directory1 

# 改变文件的群组
chgrp group1 file1 

# 改变一个文件的所有人和群组属性
chown user1:group1 file1 
```

## 常用系统管理命令
以下命令中最重要的是服务器性能监控命令和进程管理命令，尤其是监控CPU和当前负载信息的`uptime`, 监控内存使用情况的 `free`, 监控磁盘使用情况的 `df`, 以进程管理的`top`, `ps`和 `kill`命令。
```bash
# 基本命令
who   显示在线登陆用户
whoami   显示当前操作用户
hostname   显示主机名
uname   显示系统信息
clear   清屏
alias   对命令重命名 如：alias showmeit="ps -aux" ，另外解除使用unaliax showmeit

# 网络管理
ifconfig 查看网络情况
ping 测试网络连通

# 性能监控 - 非常重要
chkconfig 管理Linux系统开机启动项
uptime 获取CPU运行时间和查询Linux系统负载等信息
free 监控内存及交换分区的使用
sar 全面地获取系统的CPU、运行队列、磁盘 I/O、内存和网络等性能数据。
df 查看磁盘使用情况 df -h 带有单位显示磁盘信息
netstat 显示网络状态信息，netstat -t可查看tcp连接
vmstat 虚拟内存统计
mpstat 显示各个可用CPU的状态统计
du 查看目录大小 du -h /home带有单位显示目录信息
iostat 统计系统IO

# 进程管理 - 非常重要
top 动态显示当前耗费资源最多进程信息
  - P: CPU 占用率大小的顺序排列进程列表  
  - M: 以内存占用率大小的顺序排列进程列表  
ps 显示瞬间进程状态 
ps -aux 全格式显示进程信息,BSD风格
ps -ef 全格式显示进程信息,System V风格
kill 杀死进程，可以先用ps 或 top命令查看进程的id，然后再用kill命令杀死进程。
killall 通过进程名终止进程

# 开机、关机与注销
shutdown -h hours:minutes & 按预定时间关闭系统 
shutdown -c 取消按预定时间关闭系统 
shutdown -r now 重启
reboot	重启
poweroff 关闭电源。
logout 注销。
```

## 示例：Ubuntu系统更新Python版本
最后我们来展示下如何使用Linux命令更新Ubuntu系统上的Python版本。
```python
# 安装 python 3.9
sudo apt-get update
sudo apt-get install python3.9

# 建立软链接
ls -l /usr/bin/ | grep python
sudo rm /usr/bin/python
sudo ln -s /usr/bin/python3.9 /usr/bin/python
```

## 参考资料

- https://blog.csdn.net/yueyueniaolzp/article/details/81133122

本文由大江狗根据网络内容整理，全文转载请注明来源。我是大江狗，一名Django技术开发爱好者。您可以通过搜索【<a href="https://blog.csdn.net/weixin_42134789">CSDN大江狗</a>】、【<a href="https://www.zhihu.com/people/shi-yun-bo-53">知乎大江狗</a>】和搜索微信公众号【Python Web与Django开发】关注我！

![Python Web与Django开发](../../assets/images/django.png)
