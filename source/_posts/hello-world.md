---
title: hello world
date: 2022-07-04 14:32:19
tags:
---

## hello world

博客采用 hexo 和 GitHub Pages 结合，配置自己的个人博客页面，发布一些自己总结学习的内容。
同时本文也兼顾写一些我在博客部署时遇到的一些问题。

## hexo 部署到 GitHub Pages 的一些问题

首先要表明的是，hexo 的文档还是比较详实的，参照如下链接操作即可。
[hexo 的官方部署教程](https://hexo.io/docs/github-pages)
但是要提醒的是，主要要注意的是步骤二和步骤四。

``` yaml
on:
  push:
    branches:
      - master  # default branch
```

在修改 workflows 的 pages.yml 文件中，除了文档中专门提到的 node 的版本问题，还要修改如上面代码中写的分支，如果分支没有写对的话，这个 workflows 是不会执行的。执行不成功也就不会自动生成 gh-pages 分支来用于部署。

同时，在提交代码之后，GitHub Pages 的部署工作流会运行，因为代码中含有文件名为下划线开头的文件（如：_config.yml）。这将会导致文件被识别为 jekyll 博客，GitHub 就会自动按照 jekyll 博客的方式去处理，导致构造和部署的工作流失败。

针对这个工作流失败的问题描述和内容可以参考这个 [issue](https://github.com/hexojs/hexo/issues/3212)。解决方案同样可以参考这个issue中提到的[连接](https://github.blog/2009-12-29-bypassing-jekyll-on-github-pages/)。

然而，这并不是解决这个问题的“正确”方法。之所以会出现没有按照预想的方式进行部署和文档中提到的步骤六有关，因为还没有正确的设置 GitHub Pages 的资源位置信息。同时，如果去观察自动生成的 gh-pages 分支内容时，里面也是有和上面同样的处理方式的（添加一个名为 .nojekyll 的文件）。

当修改成功后，你应该看到你的工作流大致如下图：

{% asset_img workflows.jpg 工作流失败和成功的图片 %}

这时候就可以去访问 username.github.io 就可以看到自己的博客了。
