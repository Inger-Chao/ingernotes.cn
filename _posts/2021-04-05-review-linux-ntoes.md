---
title: "Linux 基础知识"
date: 2021-04-16
category: blog
tag: 
- Linux
author: ingerchao
---

[TOC]



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

**ftp -- vsftpd：文件的上传和下载**

服务器端允许用户登陆，并给予一些访问权限，所以服务器端需要 **修改配置文件**，然后 **重启服务**。

1. 安装 `sudo yum install vsftpd`
2. 修改 `/etc/vsftpd/vsftpd.conf` 配置文件
3. 重启服务 `sudo service vsftpd restart`

客户端：

- 实名用户登陆
  1. ftp + 服务器 ip
  2. 输入用户名和密码
  3. 文件上传：`put file`
  4. 文件下载：`get file `
  5. 退出：`bye`
- 匿名用户登陆：匿名用户不允许在目录间切换，只能在指定目录范围内工作，因此必须在服务器中创建一个匿名用户的根目录。
  1. ftp + server ip
  2. 用户名：anonymous, 密码：直接回车跳过；
- lftp: 一个 ftp 客户端工具，可以上传和下载目录。

**ssh username@ip**

scp: super copy, 使用该命令的前提就是主机成功安装 openssh-server，可以实现不同主机之间的文件复制。

```bash
scp -r 目标用户名@目标主机ip:/目标文件的绝对路径 保存到本机的绝对（相对）路径
```

### 用户管理

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

### VIM

垂直分屏：`/:vsp filename `

切换操作：`ctrl + ww`

退出所有: `:qall`

系统级配置文件目录：`/etc/vim/vimrc`

用户及配置文件目录：`~/.vim/vimrc`

### GCC

**参数：**

- -E: 预处理；
- -S: 编译；
- -c：汇编；
- 链接没参数；

- -o: 产生目标文件，指定生成的文件名字；

- -I: 指定头文件路径（相对路径/绝对路径）；
- -i: 指定头文件名字（一般不用，直接放在 #include<xxx.h>）；
- -L: 指定连接的动态库或静态库路径 -L libpath；
- -l（小写L）：指定需要连接的库的名字，在使用时需要库名去头去尾，去掉前置 lib 和后缀 .a , -l libname；
- -D: 后面直接跟宏命名，定义一个编译时期宏，默认宏的内容为1；
  - -D key = value，定义命为 key 的宏，内容为 value；
- -O: Optimize, 优化代码，等级从 0 - 3，O3为最高等级；
- -Wall: 在程序编译时输出警告信息；
- -g：程序中添加一些调试信息，gdb调试时必须添加；

#### 静态库

**命名规则：** `lib + name + .a`；

制作步骤：1. 生成对应的.o 文件（与位置相关的），2. 使用 ar 静态库打包工具将生成的.o文件打包  `ar rcs + 静态库（libname.a） + 生成的所有 .o`；

```bash
[inger@VM-0-4-centos src]$ tree ../../
../../
└── include
    ├── head.h
    └── src
        ├── add.c
        └── mul.c

2 directories, 3 files
[inger@VM-0-4-centos src]$ gcc *.c -c -I ../../include/
[inger@VM-0-4-centos src]$ ls
add.c  add.o  mul.c  mul.o
[inger@VM-0-4-centos src]$ ar rcs libCalc.a *.o # 将后缀为 .o 的目标文件打包成静态库
[inger@VM-0-4-centos src]$ ls
add.c  add.o  libCalc.a  mul.c  mul.o
[inger@VM-0-4-centos day3]$ gcc main.c lib/libCalc.a -o main -Iinclude # 调用静态库
[inger@VM-0-4-centos day3]$ ./main
48
# 调用静态库的第二种方式
gcc main.c -Iinclude -L lib -l Calc -o myapp #-L 库目录 -l：库名

[inger@VM-0-4-centos lib]$ nm libCalc.a # 查看静态库中的内容

add.o:
0000000000000000 T add  # T 表示代码区

mul.o:
0000000000000000 T mul
```

若main函数中引用静态库中的某个函数，则会在编译时将用到的 .o 文件一起打包到可执行文件中。也就是说，**静态库打包的最小单位是 .o 文件**。

静态库的**优点**：

1. 发布程序的时候，不需要提供对应的库；
2. 由于库已经被打包到了可执行文件中，所以**库的加载速度会很快**；

静态库的**缺点**：

1. 库很多的情况下，导致库的体积很大，从而可执行文件也会变大；
2. 库发生改变时，需要重新编译程序；当程序很大的时候，有可能编译一次，一天就过去了。

#### 动态库（共享库）

Linux 动态库对应 windows 中的 DLL 文件。

**命名规则：** `lib + name + .so`

制作步骤：1. 生成与位置无关的代码 （.o文件）；2. 使用 gcc 将 .o 打包成动态库。

```bash
[inger@VM-0-4-centos src]$ ls
add.c  mul.c
[inger@VM-0-4-centos src]$ gcc -fPIC -c *.c -I../include # 生成与位置无关的代码
[inger@VM-0-4-centos src]$ ls
add.c  add.o  mul.c  mul.o
[inger@VM-0-4-centos src]$ gcc -shared -o libCalc.so *.o -Iinclude # 打包动态库
[inger@VM-0-4-centos src]$ ls
add.c  add.o  libCalc.so  mul.c  mul.o
[inger@VM-0-4-centos day3]$ gcc main.c lib/libCalc.so -o maind -Iinclude # 使用动态库
[inger@VM-0-4-centos day3]$ ./maind
48
[inger@VM-0-4-centos day3]$ gcc main.c -Iinclude -L./lib -lCalc -o maind2 # 第二种使用动态库的方式，链接错误
[inger@VM-0-4-centos day3]$ ./maind2
./maind2: error while loading shared libraries: libCalc.so: cannot open shared object file: No such file or directory
#ldd 查看可执行程序运行时用到的所有动态库
[inger@VM-0-4-centos day3]$ ldd maind2
	linux-vdso.so.1 =>  (0x00007ffd38dd4000)
	libCalc.so => not found
	libc.so.6 => /lib64/libc.so.6 (0x00007fa843620000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fa8439ee000) # 通过这个ld链接器链接的
[inger@VM-0-4-centos day3]$ echo $LD_LIBRARY_PATH
:/home/inger/.VimForCpp/vim/bundle/YCM.so/el7.x86_64
[inger@VM-0-4-centos day3]$ vim ~/.bashrc # 修改 LD_LIBRARY_PATH 环境变量
[inger@VM-0-4-centos day3]$ source ~/.bashrc
[inger@VM-0-4-centos day3]$ ldd maind2
	linux-vdso.so.1 =>  (0x00007fff5b1fb000)
	libCalc.so => /home/inger/linux/day3/lib/libCalc.so (0x00007f2e8d9e6000) # 此时就找到了动态库目录
	libc.so.6 => /lib64/libc.so.6 (0x00007f2e8d618000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f2e8dbe8000)
[inger@VM-0-4-centos day3]$ echo $LD_LIBRARY_PATH
:/home/inger/.VimForCpp/vim/bundle/YCM.so/el7.x86_64:/home/inger/.VimForCpp/vim/bundle/YCM.so/el7.x86_64:/home/inger/linux/day3/lib
```

**动态库链接错误解决方案**

动态库通过动态链接器被调用，动态连接器的本质就是一个动态库。动态链接器的调用规则是根据环境变量调用的，`/lib`中存放所有文件执行时用到的动态库，所以该问题的解决方式 1 是将 .so 文件放到 /lib 目录中，但这种方式在实际情况中不能这样做，因为有可能用户创建的动态库与系统动态库产生重名问题，从而造成严重错误。

第二种解决方式是配置 `LD_LIBRARY_PATH` 环境变量（不常用），将动态库路径指定给该环境变量后，在调用时会先于系统动态库查找相应路径。

第三种临时测试时使用 LIB_LIBRARY_PATH = 动态库路径，然后 exprot LD_LIBRARY_PATH ，或者 `export LD_LIBRARY_PATH=./lib`, 终端关闭时失效。

第四种解决方式（推荐）：找到动态链接器的配置文件，将动态库的路径写到配置文件`/etc/ld.so.conf`中并更新`sudo ldconfig -v`。

动态库的**优点**：

1. 执行程序体积小；
2. 动态库函数接口不变的情况下更新不需要重新编译程序；

动态库的**缺点**：

1. 发布程序的时候，需要将动态库提供给用户；
2. 动态库没有打包到应用程序中，所以在加载速度会稍微慢一点。

### GDB

```bash
[inger@VM-0-4-centos mysort]$ gdb app # 1. 启动 gdb
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-120.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/inger/linux/day3/mysort/app...done.
(gdb) l # l 查看程序源码, 每次只会列出来 10 行，按回车默认执行上一次命令可以继续看
1	#include <stdio.h>
2	#include "sort.h"
3
4	int main(){
5	  int i;
6	  int arr1[] = {9, 10 , 3, 4, 6, 29, 8, 17, 55, 80, 98, 50, 20, 115, 79, 34};
7	  int arr2[] = {9, 10 , 3, 4, 6, 29, 8, 17, 55, 80, 98, 50, 20, 115, 79, 34};
8
9	  int len = sizeof(arr1) / sizeof(arr1[0]);
10
(gdb) l main.c:main
1	#include <stdio.h>
2	#include "sort.h"
3
4	int main(){
5	  int i;
6	  int arr1[] = {9, 10 , 3, 4, 6, 29, 8, 17, 55, 80, 98, 50, 20, 115, 79, 34};
7	  int arr2[] = {9, 10 , 3, 4, 6, 29, 8, 17, 55, 80, 98, 50, 20, 115, 79, 34};
8
9	  int len = sizeof(arr1) / sizeof(arr1[0]);
10
(gdb)
11	  printf("Sort Array: \n");
12	  for(i = 0; i < len; ++i){
13	    printf("%d\t", arr1[i]);
14	  }
15	  printf("\n");
16
17	  selectionSort(arr1, len);
18	  printf("Selection Sort: \n");
19	  for(i = 0; i < len; ++i){
20	    printf("%d  ", arr1[i]);
(gdb) break 17 # 在17行打断点
Breakpoint 1 at 0x4007c1: file main.c, line 17.
(gdb) break 13 if i == 10 # 循环当 i == 10 的时候打一个断点
Breakpoint 2 at 0x400791: file main.c, line 13.
(gdb) i b # 查看断点信息
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004007c1 in main at main.c:17
2       breakpoint     keep y   0x0000000000400791 in main at main.c:13
	stop only if i == 10
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004007c1 in main at main.c:17
2       breakpoint     keep y   0x0000000000400791 in main at main.c:13
	stop only if i == 10
```

- 启动 gdb ： gdb appname
  - start -- 只执行一步
    - n: next, 直接跳过函数；
    - s: step, 进入函数内部；finish: 从当前进入的函数中退出；
    - c: continue, 直接停在断点位置；
  - run -- 直接运行完，需要在 run 之前打好断点；
- l: 查看源码，每次显示十行，回车继续显示；
  - list 10
  - l filename: 行号（也可以是函数名）
- 设置断点：break
  - 设置当前文件断点：b line number；
  - 设置指定文件断点：b filename : line number;
  - 设置条件断点：b line number  if value == 10;
  - 删除断点：delete break number;
  - 查看断点信息: info break;
- p(rint) var name ： 查看变量值；
- ptype var name: 查看变量类型；
- set var 变量名 = 值：设置变量值；
- display varname: 追踪变量的值，每次循环的时候都显示该变量的值；
  - undisplay varname: 取消追踪该变量的值；
  - 获取变量编号：info display

### Makefile

当项目很大的时候，gcc 编译指令会非常复杂，所以我们使用代码管理工具 makefile，将所有的代码编译命令都写到 makefile 中，就不需要我们手动再去敲很复杂的 gcc 了。

Makefile 文件中记录了编译程序的一些步骤。公司中的 makefile 都是架构师或者大牛写的，我们只需要编写指定模块就好了。

如果是我们自己的项目，写一个比较菜的 makefile 指令就够用了。

makefile 有两种命名规则 首字母大写或全部小写的 *makefile*. 

```bash
[inger@VM-0-4-centos makefile]$ ls
add.c  head.h  main.c  mul.c
[inger@VM-0-4-centos makefile]$ touch makefile
[inger@VM-0-4-centos makefile]$ vim makefile
[inger@VM-0-4-centos makefile]$ cat makefile # 查看 makefile 中的内容
app:main.c add.c mul.c # 目标【最终要生成的文件名字】: 依赖【生成目标所需要的原材料】
		gcc main.c add.c mul.c -o app # tab【必须有！】 命令
[inger@VM-0-4-centos makefile]$ make # 使用 make 指令生成 app
gcc main.c add.c mul.c -o app
[inger@VM-0-4-centos makefile]$ ls
add.c  app  head.h  main.c  makefile  mul.c
```

**makefile 规则**：目标，依赖，命令。

```makefile
app:main.c add.c mul.c 
		gcc main.c add.c mul.c -o app 
```

第一个版本的 makefile 存在缺陷，文件很多的情况下，编译效率是非常低的。我们想要达到的效果是，修改一次就编译一次，不修改就不编译的。那么就需要生成 .o。第二个版本的makefile如下：

```makefile
app:main.o add.o mul.o
  gcc main.o add.o mul.o -o app
main.o:main.c
  gcc -c main.c
add.o:add.c
  gcc -c add.c
mul.o:mul.c
  gcc -c mul.c
```

**第一条规则用来生成最终目标**app，后面三条规则用于生成子目标，这些子目标是用来生成最终目标的依赖。当调用第一个规则时会发现不存在依赖，所以会往下查找有没有生成依赖的规则，找到了就会先执行下面的规则。在v2版本的 makefile 中，第一条规则是最后被执行的。使用 v2 版本的 makefile，当文件修改时，无需调用其他文件的编译命令。第一条规则中的命令，不管什么时候都会被最后执行。

makefile 向下检索，构建出一棵关系依赖树，从下向上执行命令，生成最终目标文件。目标是通过依赖生成的，makefile **通过比较目标和依赖的时间**，当依赖的时间大于之前的目标时间时，就会调用编译命令来进行更新；如果依赖的时间比之前的目标时间短，就不需要调用编译命令了。

makefile 可以用自定义变量、模式规则中的公式等进行替换：

```makefile
obj=main.o add.o mul.o
target=app
$(target):$(obj)
  gcc $(obj) -o $(target)
%.o:%.c
  gcc -c $< -o $@
```

- `$<` : 规则中的第一个依赖项；
- `$@`: 规则中的目标；
- `$^`: 规则中的所有依赖项；

> :warning: 注： 以上三个符号的规则只能在命令中使用。

makefile 中也有一些自己维护的变量，系统维护的变量有些是有默认值，有些没有，可以由用户设置。

- CC=cc
- CPPFLAGS = -I, 预处理器需要的选项；
- CFLAGS：编译时使用的参数，-Wall -g -c；
- LDFLAGS：链接库使用的选项，-L -l；

用户可以修改变量的默认值，比如 `CC=gcc` 。

```makefile
obj=main.o add.o mul.o
target=app
# makefile 中自己维护的变量
CC=gcc
CPPFLAGS = -I
$(target):$(obj)
	 $(CC) $(obj) -o $(target)
%.o:%.c
	$(CC) -c $< -o $@
```

这样makefile看起来就更高大上了。

#### Makefile 中的函数

v4 版本的 makefile 对带有变量的 v3 版 makefile 进行改进，makefile 有自带的函数可以完成一些工作，且 makefile 中的所有函数都是有返回值的，如：

- `src=$(wildcard ./*.c) `: 查找当前目录下的所有 .c 文件;

- `obj=$(patsubst ./%.c, ./%.o, $(src))`: patsubst 是用于 **匹配替换** 的函数，将src中，指定目录下的所有后缀为 %.c 的文件都替换为 %.o；

```makefile
#obj=main.o add.o mul.o
target=app
# makefile 中自己维护的变量
src=$(wildcard ./*.c) # 查找当前目录下的所有 .c 文件
obj=$(patsubst ./%.c, ./%.o, $(src))
CC=gcc
CPPFLAGS = -I
# 直接 make 时生成顶级目标
$(target):$(obj)
	 $(CC) $(obj) -o $(target)
%.o:%.c
	$(CC) -c $< -o $@

# make clean: 直接执行下面的命令，不生成顶级目标
clean:
	# 命令前加 - 表示该命令执行失败时忽略该命令；
	-rm $(obj) $(target)
# make hello: 直接执行下面命令，不生成顶级目标
hello:
	echo "hello makefile!\n"
```

> 更多内容可以详细查看 [GNU make](https://www.gnu.org/software/make/manual/make.html) 官方文档。

### 虚拟地址空间

为了防止不同进程同一时刻在物理内存中运行而对物理内存的争夺和践踏，采用了虚拟内存<sup>[1](https://blog.csdn.net/qq_34170700/article/details/106996143)</sup>。

**Linux 每一个运行的程序操作系统都会为其分配 0 - 4G 的虚拟地址空间。** 虚拟内存技术使得不同进程在运行过程中，它所看到的是自己独自占有了当前系统的4G内存。所有进程共享同一物理内存，每个进程只把自己目前需要的虚拟内存空间映射并存储到物理内存上。 

事实上，在每个进程创建加载时，内核只是为进程“创建”了虚拟内存的布局，具体就是初始化进程控制表中内存相关的链表，实际上并不立即就把虚拟内存对应位置的程序数据和代码（比如.text .data段）拷贝到物理内存中，只是建立好虚拟内存和磁盘文件之间的映射就好（叫做存储器映射），等到运行到对应的程序时，才会通过 缺页异常 来拷贝数据。还有进程运行过程中，要动态分配内存，比如malloc时，也只是分配了虚拟内存，即为这块虚拟内存对应的页表项做相应设置，当进程真正访问到此数据时，才引发缺页异常。

>  请求分页系统、请求分段系统和请求段页式系统都是针对虚拟内存的，通过请求实现内存与外存的信息置换。

虚拟地址空间中 1G 内核空间中的PCB 进程控制块，PCB中会维持一张文件描述符表，每打开一个新文件会占用一个文件描述符，而且使用的是空闲的最小的一个文件描述符。

用户地址空间（从 0 地址到 3G，从下往上）分别是：

- 0-4K 受保护的地址：拒绝用户访问，比如 `#define NULL (void*) 0`; 
- .text: 代码段，二进制机器指令；
- .data: 已初始化的全局变量；
- .bss: 未初始化的全局变量；

> 从 .text 到 .bss 段合起来称之为 ELF（Linux 下可执行文件格式）；

- 堆空间：运行时的堆，在程序运行中使用new 或 malloc申请的内存区域。该区域的起始地址受start_brk影响，和BSS段之间会有一个随机值的空洞；该区域的内存增长方式是由低地址向高地址增长。
- 共享库：调用 C 标准库或 Linux 系统 IO 函数；
- 栈空间：用户进程栈；该区域起始地址也存在一个随机值，通过PF_RANDOMIZE来设置。栈的增长方向是由高地址向低地址增长，并且若**设置了RLIMIT_STACK即规定了栈的大小**。
- 命令行参数；
- 环境变量区：当前程序用到的所有环境变量；

程序执行时，从代码段开始执行，然后开始初始化全局变量。程序执行过程中，每当遇到1个局部变量，就在栈空间中从上往下开始分配空间，也就是从高地址向低地址空间开始分配；每当遇到 new 或 malloc 的变量， 从堆空间中分配内存；每当调用动态库时，将其加载到共享区中的空闲位置，所以，动态库的生成的时候必须生成与位置无关的代码，也是因为库程序的执行必须相对位置；静态库打包后，程序代码段是在代码段的，所以每次在内存中存放的位置都是固定的。

**CPU 为什么要使用虚拟地址空间与物理地址空间映射？解决了什么问题**

1. 方便编译器和操作系统安排程序的地址分布。程序可以使用一系列相邻的虚拟地址来访问物理内存中不相邻的大内存缓冲区；
2. 方便进程之间隔离。不同进程使用的虚拟地址彼此隔离，一个进程中的代码无法更改正在由另一进程使用的物理内存；
3. 方便操作系统使用较小的内存空间 。程序可以使用一系列虚拟地址来访问大于可用物理内存的内存缓冲区。当物理内存的供应量变小时，内存管理器会将物理内存页（通常大小为 4KB ）保存到磁盘文件。数据或代码页会根据需要在物理内存与磁盘之间移动。

---

视频：[哔哩哔哩黑马](https://b23.tv/Jomal8)

资料：

- [GNU make 官方文档](https://www.gnu.org/software/make/manual/make.html)