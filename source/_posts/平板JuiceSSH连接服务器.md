---
title: 平板JuiceSSH连接服务器
date: 2023-11-12
---

# 为什么使用JuiceSSH

原本使用termux连接服务器，但是termux不支持中文输入，找了很多网站，发现都是安卓键盘Layout的问题，没有中文布局。但是由于我只需要连服务器（平板能有啥性能，而且架构还是arm），因此ssh工具也是备选的，因此就看中了JuiceSSH。体验下来确实不错，支持xterm-256 color，稳定性也不错，可以后台运行而不会像termux一样一段时间ssh进程就退出了。

<!-- more -->

# SpaceVim编辑总是出现q键

问题类似于[neovim - vim over ssh: character q popping up - Ask Ubuntu](https://askubuntu.com/questions/1001401/vim-over-ssh-character-q-popping-up)

解决方法见[FAQ · neovim/neovim Wiki](https://github.com/neovim/neovim/wiki/FAQ#nvim-shows-weird-symbols-2-q-when-changing-modes)

最好是在`.vim/init.vim`中添加`set guicursor=`。这样对于其他ssh终端的问题也能解决。显然在JuiceSSH里面用除了xterm-256 color的终端模拟器是会降低体验的。

# JuiceSSH平板外接键盘无法输入中文

## 起因

使用JuiceSSH在荣耀v7pro中连接服务器，同时使用外接键盘，突然发现无法输入中文。

## 解决方案

在平板设置>系统和更新>语言和输入法 中将“安全输入”关闭即可。推测平板是将JuiceSSH终端作为一个需要安全保护措施的输入框了。
