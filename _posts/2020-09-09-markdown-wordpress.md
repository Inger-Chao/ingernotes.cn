---
title: 将 jekyll、hexo 博客的 .md 文件批量上传至基于 wordpress 的博客
layout: post
tag:
-  wordpress
date: 2020-09-09
url: markdown-wordpress
author: ingerchao
category: blog
---



在将基于 Git page 和 jekll 的博客文章转移到 wordpress 的过程中，将自己以往的 _post/*.md 文章向 wordpress 转移是一件比较费力的事情，了解到python-markdown自动发送wordpress文章

在网上找到了一篇文章，是使用 python 的几个包，自动将本地 `.md` 文件上传至 wordpress 博客。

所使用的 python 模块：

- **python-frontmatter**：通过python-frontmatter库获取文章信息，标题、分类、标签、正文内容等
- **markdown2**：通过markdown2库将正文内容转换成HTML格式
- **python-xmlrpc-wordpress**：最后将这些信息通过python-wordpress-xmlrpc库发布到网站上



### 使用方法

1. 安装三个 pip 包；

```bash
pip3 install python-frontmatter

pip3 install markdown2

# 有些博客是通过 git 地址安装的，我使用地址安装不能用，最终通过 pip3 install 成功的
pip3 install python-xmlrpc-wordpress
```

2. 创建 `uploadposts.py`

```bash
vi uploadposts.py
```

```python
#!python
# -*- coding:utf-8 -*-

# 1 导入模块 front matter
import sys
import frontmatter
# 2 导入模块 markdown2
import markdown2
from markdown2 import Markdown
# 3 导入模块  wordpress_xmlrpc
from wordpress_xmlrpc import Client, WordPressPost
from wordpress_xmlrpc.methods.posts import GetPosts,NewPost


# 1 front matter
# 获得md文章路径信息
dir_md = sys.argv[1]
# 通过front matter.load函数加载读取文档里的信息
# 这里关于Python-front matter模块的各种函数使用方式GitHub都有说明，下面直接贴可实现的代码
post = frontmatter.load(dir_md)
# 将获取到的信息赋值给变量
post_title = post.metadata['title']
post_tag = post.metadata['tag']
post_category = post.metadata['category']
# post_url = post.metadata['url']
# 通过print函数来看我们获取到信息状态，确定无误后这个步骤是不需要的
print(post_title)
print(post_tag)
print(post_category)
# print(post_url)
print(post.content)


# 2 markdown2
# post.content里面是我们md格式的正文内容，现在转换成HTML格式
markdowner = Markdown()
post_content_html = markdowner.convert(post.content)
post_content_html = post_content_html.encode("utf-8")
# 现在print post_content_html看看,是不是HTML标签了
print(post_content_html)


# 3 wordpress_xmlrpc
wp = Client('http://localhost:8080/inger.cool/xmlrpc.php', '用户名', '密码')
# 现在就很简单了，通过下面的函数，将刚才获取到数据赋给对应的位置
post = WordPressPost()
post.title = post_title
# post.slug文章别名
# 我网站使用%post name%这种固定链接不想一长串，这里是最初md文章URL的参数，英文连字符格式
# post.slug = post_url
post.content = post_content_html
# 分类和标签
post.terms_names = {
  'post_tag': post_tag,
  'category': post_category
    }
# post.post_status有publish发布、draft草稿、private隐私状态可选，默认草稿。如果是publish会直接发布
# post.post_status = 'publish'
# 推送文章到WordPress网站
wp.call(NewPost(post))


```

3. `uploadposts.py` 与 `posts` 文件夹处于同一目录，然后执行 bash 语句对 posts 目录下的所有 md 文件进行传输。

```bash
for file in ./posts/*
do
python3 uploadposts.py $file $file;
done
```



----

参考链接：

- [python-markdown自动发送wordpress文章（python-xmlrpc-wordpress）](http://www.95408.com/blog/3552.html)
