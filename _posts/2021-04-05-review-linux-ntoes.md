---
title: "Linux 基础知识"
date: 2021-04-05
category: blog
tag: 
- Linux
author: ingerchao
---



### 快捷键

- `Ctrl+p`: previous, 上一条命令
- `ctrl+n`: next, 下一条命令
- `ctrl+b`: back, 光标左移
- `ctrl+f`: floor, 光标右移
- `ctrl+d`: delete, 删除光标所在字符
- `ctrl+h`: 删除光标前一个字符
- `ctrl+u`: 剪切光标前的所有字符
- `ctrl+k`: 剪切光标后的所有字符
- `ctrl+y`: 粘贴
- `ctrl+a`: 光标移至命令行首
- `ctrl+e`: end, 光标移至最后一个字符
- `ctrl+l`: clear, 清屏

- `history`: 查看历史命令
- `date`： 日期

### 目录

- `/` : 根目录
- `/bin`:  binary, 存放经常用到的命令可执行程序；
- `/boot`: 存放开机启动项；
- `/dev`: device, 存放外部设备的文件，Linux 中访问设备的方式与访问文件的方式一致；
- `/etc`: 存放所有系统管理所需要的配置文件和子目录;
- `/home`: 用户的主目录
- `/lib`: library, 存放系统最基本的动态连接共享库，几乎所有的应用程序都要用到这些共享库；
- `/lost-found`: 一般情况下是空的，系统非法关机后，这里存放一些碎片文件；
- `/media` && `/mnt`: 用于挂载外设，比如 u盘、光驱等，media 是自动挂载的目录，mnt 是让用户临时挂载的目录；
- `/root`: root 用户的主目录
-  `/usr`: user sofeware resource, 用户软件资源目录，用户的很多应用程序和文件都放在这个目录下面，类似windows 中的 program files。

```bash
[joki@localhost ~]$
```

- 用户名@主机名 当前路径
- $: 普通用户, # 超级用户

### 文件操作

-  白色：普通文件
- 蓝色：目录
- 绿色：可执行文件
- 红色：压缩文件
- 青色：链接文件
- 黄色：设备文件
  - block：快， 磁盘
  - char：字符设备，键盘
  - fifo：管道
- 灰色：其他文件

#### 命令

```bash
[joki@localhost ~]$ ls ~
cpp  install.sh
[joki@localhost ~]$ ls -a
.   .bash_history  .bash_profile  .cache   cpp      install.sh  .local    .pki  .VimForCpp  .ycm_extra_conf.py
..  .bash_logout   .bashrc        .config  .cquery  .LfCache    .mozilla  .vim  .vimrc
```

- `ls -a` : all;
- `ls -l`: list;
- `ls -la` : list and all;

```bash
[joki@localhost ~]$ ls -l
total 4
drwxrwxr-x. 3 joki joki  17 Apr  2 13:06 cpp
-rw-rw-r--. 1 joki joki 827 Apr  2 11:50 install.sh
```

- 文件类型
  - `-`: 普通文件
  - d：目录
  - l：链接符号
  - b：块设备
  - c：字符设备
  - s：socket 文件
  - p：管道
- 文件所有者权限：read write execute
- 同组用户权限
- 其他人权限
- 文件的硬链接数
- 该文件或目录的所有者
- 该文件所有的组
- 占用的存储空间
  - 文件：文件大小
  - 目录：目录占用 4096
- 文件最后创建或修改的时间
- 文件名

```bash
mkdir aa/bb/cc -p
mkdir -p bb/cc/dd
rm -r aa #递归删除目录下的所有内容
rm -ri aa #询问是否确认进入、删除目录
touch fileA # 创建文件, 若已有filaA，修改文件时间
cp source dest #若已有同名目标文件，会直接覆盖
```

查看文件内容：cat是一次性显示整个文件的内容，还可以将多个文件连接起来显示，它常与重定向符号配合使用，适用于文件内容少的情况；
more和less一般用于显示文件内容超过一屏的内容，并且提供翻页的功能。more比cat强大，提供分页显示的功能，less比more更强大，提供翻页，跳转，查找等命令。而且more和less都支持：用空格显示下一页，按键b显示上一页。

```bash
cat filename #显示文件所有内容
more filename # 先显示一屏幕，然后回车一行一行显示文件内容，空格下一屏，b上一页，q退出
less filename
head -n 5 filename #显示文件的前 n 行
tail -n 5 filename #显示文件的后 n 行
```



---

视频：[哔哩哔哩黑马](https://b23.tv/Jomal8)

