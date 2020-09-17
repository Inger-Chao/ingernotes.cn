---
title: "GaitSet 复现"
layout: post
category: blog
tag:
- Deep Learning
- Paper
date: 2020-09-17 16:46
author: ingerchao
---



### 准备

- 数据集：申请 [CASIA-B](http://www.cbsr.ia.ac.cn/english/Gait%20Databases.asp) 数据集和 [OU-MVLP](http://www.am.sanken.osaka-u.ac.jp/BiometricDB/GaitMVLP.html) 两个数据库的解压密码
- 克隆[源代码 git 仓库](https://github.com/AbnerHqC/GaitSet)到本地
- [pytorch 环境配置](./2020-09-17-linux-nvidia-cuda)

### 安装程序所需的依赖



```bash
# scipy
pip3 install scipy

# skbuild
pip3 install scikit-build

# cmake
pip3 install cmake

# opencv
pip3 install opencv-python
```





