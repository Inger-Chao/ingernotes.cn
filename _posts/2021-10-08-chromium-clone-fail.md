---
title: "Failed to connect to chromium.googlesource.com port 443: Operation timed out"
layout: post
date: 2021-10-08
tag:
- extra
category: blog
author: ingerchao
---

git clone googlesource 的各种 timeout：`Failed to connect to chromium.googlesource.com port 443: Operation timed out`

GFW 挡不住上学人的求知欲。

一通改代理也没用。

后面通过灵机一动将 `chromium.googlesource.com/` 替换为`mirrors.tuna.tsinghua.edu.cn/git/chromiumos`!! 果然成了！

```bash
# invalid
git clone https://chromium.googlesource.com/chromium/tools/depot_tools
```

```bash
# avaliable
git clone https://mirrors.tuna.tsinghua.edu.cn/git/chromiumos/chromium/tools/depot_tools
```

---

参考: https://www.wenjiangs.com/doc/jxbhwbxz

