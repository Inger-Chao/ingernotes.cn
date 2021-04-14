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
- `ctrl+b`: backword, 光标左移
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

#### 软连接与硬连接

- 软连接相当于 windows 下的快捷方式，使用绝对路径创建
- 硬连接共享同一块内存

#### 文件权限

- chmod 
  - 文字设定法
  - 数字设定法
    - r: 4
    - w: 2
    - x: 1
- Chown: 指定文件的拥有者改为制定的用户或组
- chgrp：改变文件或目录的所属群组，需要管理员权限。

#### 文件查找

- find：依据文件属性查找
  - -name: 依据文件名查找
  - -size: 大小
  - -type
- grep：按文件内容查找，`grep -r  "content" ~`

#### 磁盘

**磁盘设备种类**

- sd: SCSI DEvice
- hd: Hard Disk 硬盘
- fd: Floppy Disk 软盘

命名规则：sda sdb sdc sdd 最多可以允许有4个硬盘，每个硬盘有分区，最多有4个主分区1,2,3,4，扩展分区标号从 5 开始。

```bash
[joki@localhost ~]$ sudo fdisk -l
[sudo] joki 的密码：

磁盘 /dev/sda：240.1 GB, 240057409536 字节，468862128 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
磁盘标签类型：dos
磁盘标识符：0x00071654

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200   468860927   233380864   8e  Linux LVM
```

### 压缩包管理

#### 原始压缩工具

- Gzip -- .gz 格式压缩包：无法压缩目录

```bash
[joki@localhost day2]$ ls
animal  backup  cat.txt  dog.txt  fish.txt  test
[joki@localhost day2]$ gzip *.txt # 使用 gzip 压缩后缀为.txt的文件
[joki@localhost day2]$ ls
animal  backup  cat.txt.gz  dog.txt.gz  fish.txt.gz  test
[joki@localhost day2]$ gunzip *.gz # 解压缩gzip 压缩的文件
[joki@localhost day2]$ ls
animal  backup  cat.txt  dog.txt  fish.txt  test
```

- bzip2 -- .bz2 格式压缩包：和gzip差不多

#### 进化

- tar：不使用 z/j 参数时，只能对文件或目录打包

  - 参数
    - c -- 创建：压缩时调用
    - x -- 释放：解压缩
    - v -- 显示提示信息（可以省略）
    - f -- 指定压缩文件的名字
    - z -- 使用 gzip 的方式压缩文件，压缩完后后缀为 .gz
    - j -- 使用 bzip2 的方式压缩文件，压缩完后后缀为 .bz2
  - 压缩：
    - `tar zcvf 生成的压缩包名字(xxx.tar.gz) 要压缩的文件或目录`
    - `tar jcvf 生成的压缩包名字(xxx.tar.bz2) 要压缩的文件或目录`

  ```bash
  [joki@localhost day2]$ tar zcvf alltxt.tar.gz *.txt # 压缩并打包
  cat.txt
  dog.txt
  fish.txt
  [joki@localhost day2]$ ls
  alltxt.tar.gz  animal  backup  cat.txt  dog.txt  fish.txt  test
  [joki@localhost day2]$ rm *.txt
  [joki@localhost day2]$ tar zxvf alltxt.tar.gz # 解压缩
  cat.txt
  dog.txt
  fish.txt
  [joki@localhost day2]$ ls
  alltxt.tar.gz  animal  backup  cat.txt  dog.txt  fish.txt  test
  [joki@localhost day2]$ tar zxvf alltxt.tar.gz test/ # 解压到指定路径要加 -C
  tar: test：归档中找不到
  tar: 由于前次错误，将以上次的错误状态退出
  [joki@localhost day2]$ tar zxvf alltxt.tar.gz -C test/
  cat.txt
  dog.txt
  fish.txt
  ```

  - 解压缩： zxvf 解压后缀为 gz 的包， jxvf解压缩后缀为 bz2 的包，解压到指定路径要加 -C。

- rar -- 必须手动安装软件，压缩完成后会自动生成后缀为 .rar 的压缩文件；

  - 压缩:a，`rar a 生成的压缩文件名字(无需指定后缀) 压缩的文件或目录` ；
  - 解压缩 x ，`rar x 压缩文件名（解压缩目录）；`

- zip 

  - 压缩：`zip 压缩包名字（无需指定后缀） 压缩的文件或目录（-r）`
  - 解压缩: `unzip 压缩包名字 -d (指定解压目录) `

### 进程管理

```bash
[inger@VM-0-4-centos ~]$ who # 查看当前在线用户的情况
inger    pts/0        2021-04-14 15:51 (113.140.6.195)
```

登陆的用户名， 使用的设备终端，登陆到系统的时间

tty7 桌面终端？tty 1-6 文字终端

pts/0: 设备终端

```bash
[inger@VM-0-4-centos ~]$ ps a # 
  PID TTY      STAT   TIME COMMAND
 1283 tty1     Ss+    0:00 /sbin/agetty --noclear tty1 linux
 1284 ttyS0    Ss+    0:00 /sbin/agetty --keep-baud 115200,38400,9600 ttyS0 vt220
19450 pts/0    Ss     0:00 -bash
20563 pts/1    Ss+    0:00 -bash
20630 pts/0    R+     0:00 ps a
[inger@VM-0-4-centos ~]$ ps au # 列出更加丰富的信息
[inger@VM-0-4-centos ~]$ ps aux # 再加上没有终端的进程信息
```

ps aux 参数会给出一堆信息，可以通过管道（|）进行重定向。*command1 | command 2* 将 command 1 的输出作为command 2 的输入，不显示在屏幕上。

```bash
[inger@VM-0-4-centos ~]$ ps aux | grep bash
inger    19450  0.0  0.1 116728  3164 pts/0    Ss   15:51   0:00 -bash
inger    20563  0.0  0.1 116728  3156 pts/1    Ss+  15:58   0:00 -bash
inger    21840  0.0  0.0 112816   988 pts/0    R+   16:06   0:00 grep --color=auto bash # 最后一条并不是我们查找到的内容，而是grep在查找 bash 时占用的进程。
[inger@VM-0-4-centos ~]$ kill -l # 查看信号 使用 kill -9 PID 可以杀死一个进程
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX
[inger@VM-0-4-centos ~]$ kill -9 19450 
Connection to 121.4.61.226 closed. 
```

查看当前进程的环境变量:

```bash
[inger@VM-0-4-centos ~]$ env | grep PATH
LD_LIBRARY_PATH=:/home/inger/.VimForCpp/vim/bundle/YCM.so/el7.x86_64
PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/inger/.local/bin:/home/inger/bin
```

Linux 下环境变量的格式是一个 key-value 格式的键值对，格式为 `PATH = value1 : value2 : value3 : value4 : ... `。

Linux 下 `top` 指令打开类似于 windows 下的任务管理器，但是只能查看，不能做任何操作。

### 网络管理

```bash
[inger@VM-0-4-centos ~]$ ifconfig # 查看本机网络信息
[inger@VM-0-4-centos ~]$ ping baidu.com
PING baidu.com (220.181.38.148) 56(84) bytes of data.
64 bytes from 220.181.38.148 (220.181.38.148): icmp_seq=1 ttl=49 time=26.8 ms
[inger@VM-0-4-centos ~]$ nslookup baidu.com
Server:		183.60.83.19
Address:	183.60.83.19#53

Non-authoritative answer: # 百度的服务器地址
Name:	baidu.com
Address: 220.181.38.148
Name:	baidu.com
Address: 39.156.69.79
```

nslookup 获取域名所对应的 ip, 为什么服务器可以知道域名的服务器地址？之前连过会有本地缓存，系统有一些默认的域名对应的 ip 地址，每次连网的时候都会刷新。

### 服务器搭建

#### ftp -- vsftpd



###用户管理

创建用户： sudo adduser  username，adduser 本质上是一个脚本，所以用起来比较方便。用户名不可以有大写字母。 

- `sudo useradd -s /bin/bash -g itcast -d /home/itcast -m itcast`
  - -s 指定命令解析器类型
  - -g 用户所属组
  - -d 用户的目录
  - -m 如果用户 home 目录不存在就创建一个

修改当前用户密码：passwd

创建用户组: sudo groupadd groupname

删除用户：sudo deluser username , sudo userdel -r username 连同家目录一起删掉

```bash
[inger@VM-0-4-centos ~]$ vi /etc/passwd # 查看当前系统的所有用户
```



---

视频：[哔哩哔哩黑马](https://b23.tv/Jomal8)

