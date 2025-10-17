---
title: devcontainer让我免于今日的加班
published: 2025-02-18
tags: ['devcontainer','java','docker','vscode']
description: '开发过程最怕环境boom，这时就需要devcontainer来拯救你了'
category: '杂谈'
---

## 被Windows与jdk1.8坑了
原本工作计划中，进度已经快了非常多了。但是跑单测的时候出现了与预期不符的结果，由于不知道何种魔法导致在Windwos11 24h2 中得到了与预期不符的结果，于是一直浪费时间排查问题。之后脑袋灵光一闪会不会是jdk1.8太老了Windows11太新了。   
于是找用mac的同事帮我跑了下，结果他居然正常了。然后我就开始折腾wsl，在wsl中结果也正常麻了(╬▔皿▔)╯。然后就把开发环境迁移到了wsl中又废了大半天时间。

## 又被openvpn2与wsl还有seatunnel给一起坑了
原先我开发过程中使用的是docker中的seatunnel镜像，但是又没有老版本可以用于是自己下了1.8的jdk与2.3.4的seatunnel。原本以为很快就能结束，但是问题来了，wsl中的代理和openvpn与clash打架了。又浪费了大半天时间搞环境终于在配置wsl的网络为镜像的时候终于可以了。   
好了原本以为一切顺利，但是又连不上docker desktop中运行的pg了。又浪费了大半天时间

## 忍无可忍容器化
没办法最后还是祭出了大杀器`docker`，在构建了docker镜像后写了devcontainer.json在vscode中跑起来了，在里头模拟了运行环境的目录结构与配置。终于跑通了单测，也顺利完成了开发。   
镜像构建脚本如下
```Dockerfile
FROM eclipse-temurin:8-ubi9-minimal

RUN microdnf install dnf git -y && \
	microdnf clean all 

RUN mkdir -p /opt/seatunnel

RUN wget https://mirrors.tuna.tsinghua.edu.cn/apache/seatunnel/2.3.4/apache-seatunnel-2.3.4-bin.tar.gz && \
	tar -zxvf apache-seatunnel-2.3.4-bin.tar.gz -C /opt/seatunnel && \
	rm -rf apache-seatunnel-2.3.4-bin.tar.gz

RUN cd /opt/seatunnel/apache-seatunnel-2.3.4 && sh bin/install-plugin.sh 2.3.4

RUN cd /opt/seatunnel/apache-seatunnel-2.3.4/lib && \
	wget https://repo1.maven.org/maven2/org/postgresql/postgresql/42.7.5/postgresql-42.7.5.jar && \
	wget https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/9.2.0/mysql-connector-j-9.2.0.jar
```
devcontainer.json配置文件如下
```Dockerfile
{
  "image": "seatunnel:2.3.4",

  "customizations": {
    "vscode": {
      "extensions": ["vscjava.vscode-java-pack","vmware.vscode-boot-dev-pack","mhutchie.git-graph","donjayamanne.githistory","huizhou.githd","redhat.vscode-xml","redhat.vscode-yaml"]
    }
  }
}
```
感兴趣的可以试试

## 吐槽
已经5202年了怎么还有新项目用jdk1.8，打工嘛╮(╯▽╰)╭ ，上司说啥就是啥。
