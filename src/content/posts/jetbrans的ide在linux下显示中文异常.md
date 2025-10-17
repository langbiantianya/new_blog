---
title: jetbrans的ide在linux下显示中文异常
published: 2025-04-27
tags: ['linux','fedora','idea','jetbrans']
description: 'jetbrans的ide在linux下显示中文异常'
category: '技术'
---

## 原因

jetbrans的ide在linux下使用的默认中文字体是微软雅黑导致了中文全部变成方块。

## 解决办法

### 1. 拷贝windows下的字体到linux

把字体拷贝到`/user/share/fonts/`目录下再执行`sudo fc-cache -fv`

### 2. 修改ide设置

打开设置->外观和行为->外观->字体->使用自定义字体
![显示配置](../../assets/jetbrans-IDE-displays-Chinese-exception-on-Linux/QQ20250427-164144.png)
