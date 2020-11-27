---
title: conda的使用
date: 2020-11-23 21:00:20
categories:
- 环境搭建
tags:
- anaconda
---

### 安装anaconda
<!--more-->
首先需要安装anaconda，在不同的电脑上下载使用很多次了，还是挺方便的。正好开通了博客，就记录下conda的使用。

### conda创建虚拟环境
anaconda安装成功之后，如果不成功，网上很多安装的博客可以查看。应该有conda命令了。

```conda -V```

返回```conda 4.7.12```

##### 1.创建虚拟环境

命令：
```conda create --name pytorch python=3.6.5```

安装总是失败
```
Collecting package metadata (current_repodata.json): ...working... done
Solving environment: ...working... failed with repodata from current_repodata.json, will retry with next repodata source.
Collecting package metadata (repodata.json): ...working...
```

我添加了conda换源，可是还是不行
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```

修改命令
```conda create --name pytorch python=3.6```


2.安装成功之后，可以查看确认下```conda env list```
