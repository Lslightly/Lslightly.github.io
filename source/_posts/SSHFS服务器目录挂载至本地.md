---
title: SSHFS服务器目录挂载至Windows本地
date: 2023-10-28T21:28:54+08:00
tags:
    - ssh
---

## 概述

本篇文章主要是说明如何在win11环境下使用sshfs将服务器目录挂载到本地磁盘，方便在本地创建和编辑服务器文件。

<!-- more -->

## 想法起因

在服务器上编写markdown文档，由于需要结果截图，所以就需要在windows下进行截图，然后保存到windows下的某个目录，然后再从这个目录将图片通过vscode拷贝到服务器下。显然这里有两次写操作。除此之外，还需要考虑在windows下将文件保存在哪个目录下。最后还可能需要手动删除这个文件。这种多出来的不必要的小文件就很令人心烦。

对于wsl中，可以通过windows的资源管理器直接访问wsl中的目录，十分的方便，因此希望能够通过某种方式能够在windows的资源管理器中直接访问服务器目录。回到最开始的场景，直接将截图保存到服务器目录下即可。

这个想法另外解决的场景是：当需要下载某个东西到服务器中时，我们一般来说需要获取被下载文件的链接，然后通过`curl`或者`wget`下载。但是存在某些情况我们无法简单获取链接（比如通过按钮响应方式返回一个url然后再进行下载的情况），此时我们就只能下载到本地然后再转移到服务器中。对于这种情况，在sshfs的解决方案下，我们就可以直接将被下载文件保存到服务器中。不过一个坏处是下载文件走的链路仍然是被下载文件的服务器->本机->服务器，仍然没有解决速度（由于文件不是直达）的问题，但是它解决了我们需要等待下载完成然后再拷贝的时间浪费。

## 解决方案

1. 下载[winfsp安装包](https://github.com/winfsp/winfsp/releases)和[sshfs-win安装包](https://github.com/winfsp/sshfs-win/releases)。可以使用`winget`下载

```powershell
winget install -h -e --id "WinFsp.WinFsp" && winget install -h -e --id "SSHFS-Win.SSHFS-Win"
```

2. 下载[sirikali GUI前端](https://github.com/mhogomchungu/sirikali/releases/)
3. 下载GUI前端并完成安装后，可以在右下角Menu中设置语言为中文。
4. 然后创建卷，选择sshfs
5. 将服务器地址，端口号和私钥（如果有）进行设置，选择自动挂载卷。然后点击添加
6. 在我的收藏中选中刚刚添加的路径，连接即可。

## 探寻过程

在[Windows 下使用 SSHFS 通过 SSH 协议挂载远程服务器目录 - 知乎](https://zhuanlan.zhihu.com/p/314245985)中提到使用[evsar3/sshfs-win-manager: A GUI for SSHFS-Win (https://github.com/billziss-gh/sshfs-win)](https://github.com/evsar3/sshfs-win-manager)，但是这个项目太老了，导致其使用的sshfs和ssh版本较老，会出现如下情况，在此记录。

```
Bad owner or permissions on /cygdrive/c/Users/Administrator/.ssh/config
```
