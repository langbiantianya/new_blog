---
title: 或许windwos才是最好的linux发行版本
published: 2025-10-11
tags: ['windows','linux',"web开发","wsl"]
description: 'windwos还是如此适合开发者'
category: '杂谈'
---

## wsl是大杀器

随着微软完善了wsl，也借着wsl开源这个热点聊一聊为什么windwos是最好的linux发行版本。  
wsl从一开始的1.0升级到2.0使用体验的提升是非常明显，支持了 x11 与 Wayland 在windows上直接使用gui后是质的飞跃，不在局限于终端。而且wsl支持众多的linux发行版本如常用的 fedora、archlinux、debian、ubuntu、rhel衍生版本、openeulr等，总有一个适合你，使用时也不用太担心修改系统配置导致崩溃。  
wsl现在也支持在子系统中调用gpu来进行计算加速，对于普通的深度学习与ai模型微调来说那是相当方便。  
wsl中的linux与windows的文件系统交互也是相当方便，可以互相访问。对于io要求比较高的情况还是直接拷贝到wsl中吧。

## 优秀的第三方开发工具支持

还有丰富的开发工具的迭代支持，如自家的 vscode 以及 jetbrains 家的软件支持，使得开发体验上基本与linux上无异。尤其是非中文用户的体验，不用忍受jetbrains软件缺少微软雅黑导致乱码的问题。  
docker desktop 在wsl的利用上也是非常优秀，可以直接在windows的终端中无缝使用，也可以在设置中扩展支持到其他的wsl中的linux发行版本。vscode 与 jetbrains 也有适合docker的插件安装后使用更加方便快捷。

## 稳定易用的桌面环境

虽然大家都在吐槽最近的 windows10 与 windows11 bug多，但是不得不承认相对与 linux 上的大多数桌面环境使用起来更加简单且更符合直觉。而且对于小尺寸高分屏windows中的缩放也是相当好用。

## 娱乐

工作之余也是要在公司摸鱼的，这个时候就需要mumu或者崩铁启动在后台挂挂机，虽然现在linux在v社的努力下可以玩的游戏更多了，但还是没有windows下玩游戏更方便。
