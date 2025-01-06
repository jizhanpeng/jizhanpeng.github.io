---
layout:     post
title:      "Github pages 的 workflow 突然运行失败"
subtitle:   "Github pages 的 workflow 运行失败解决办法"
date:       2025-1-06 16:42:10
author:     "JIzp"
header-img: "img/bg-material.jpg"
tags:
    - problem
---

# Github pages 的 workflow 突然运行失败

------

今天想发一篇博客，突然发现我使用 GitHub pages 搭建的博客网页突然不能运行工作流了。

## 提示：

```
The current runner (ubuntu-24.04-x64) was detected as self-hosted because the platform does not match a GitHub-hosted runner image (or that image is deprecated and no longer supported).
In such a case, you should install Ruby in the $RUNNER_TOOL_CACHE yourself, for example using https://github.com/rbenv/ruby-build
You can take inspiration from this workflow for more details: https://github.com/ruby/ruby-builder/blob/master/.github/workflows/build.yml
$ ruby-build 3.1.4 /opt/hostedtoolcache/Ruby/3.1.4/x64
Once that completes successfully, mark it as complete with:
$ touch /opt/hostedtoolcache/Ruby/3.1.4/x64.complete
It is your responsibility to ensure installing Ruby like that is not done in parallel.
```

## 原因：

在运行失败的 workflow 里可以看到警告：ubuntu-latest pipelines will use ubuntu-24.04 soon. For more details, see https://github.com/actions/runner-images/issues/10636

大概意思就是 ubuntu-latest 将被替换为 ubuntu-24.04 。而如果 jekyll 和 ruby 版本与 ubuntu-24.04 不兼容就会 build 失败。

## 解决：

把.github\workflows 下的 jekyll.yml 文件中的 runs-on 部分修改成如下配置

```
jobs:
  # Build job
  build:
    runs-on: ubuntu-22.04
```

成功解决。
