---
title: torch.cuda.is_available()结果为false
layout: post
date: 2020-09-07
tag: 
- Deep Learning
- Linux
category: blog
author: ingerchao
---





- 已安装 python3.6.4 并更改 `python -> python3`, `python2 -> python2`
- 使用 pip3 安装

按照[这篇博客](https://blog.csdn.net/weixin_45250844/article/details/101854871)在 `222` 实例安装 pytorch 环境时，CUDA 和 GPU 驱动不可用。

**检查 CUDA 版本：**

```bash
[root@222 inger]# cat /usr/local/cuda/version.txt
CUDA Version 10.0.130
[root@222 ~]# cd /usr/local/cuda/bin
[root@222 bin]# ./nvcc -V
nvcc: NVIDIA (R) Cuda compiler vdriver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130
```

- https://blog.csdn.net/ljp1919/article/details/102640512
- https://blog.csdn.net/daydaydreamer/article/details/102877064

**检查 GPU 驱动版本**：

```bash
[root@222 inger]# cat /proc/driver/nvidia/version
NVRM version: NVIDIA UNIX x86_64 Kernel Module  418.88  Mon Jul 22 20:45:26 CDT 2019
GCC version:  gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)
```

按照 [这篇博客](https://blog.csdn.net/qq_36005129/article/details/107414409?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param) 执行 2.2 代码，报错，说 Nvidia 驱动太旧，让更新。

```bash
[root@222 ~]# python
Python 3.6.4 (default, Sep  7 2020, 08:00:22)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.zeros(1).cuda()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/python3/lib/python3.6/site-packages/torch/cuda/__init__.py", line 186, in _lazy_init
    _check_driver()
  File "/usr/local/python3/lib/python3.6/site-packages/torch/cuda/__init__.py", line 77, in _check_driver
    of the CUDA driver.""".format(str(torch._C._cuda_getDriverVersion())))
AssertionError:
The NVIDIA driver on your system is too old (found version 10010).
Please update your GPU driver by downloading and installing a new
version from the URL: http://www.nvidia.com/Download/index.aspx
Alternatively, go to: https://pytorch.org to install
a PyTorch version that has been compiled with your version
of the CUDA driver.
```

我在解决的过程中，使用`nvidia-smi` 查看 GPU 驱动版本型号和 CUDA Driver API 的型号报了如下错误：

```bash
[root@222 ~]# nvidia-smi
Unable to determine the device handle for GPU 0000:00:05.0: Unknown Error
```

