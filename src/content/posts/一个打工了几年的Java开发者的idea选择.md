---
title: 一个打工了几年的Java开发者打算换掉idea了
published: 2025-04-04
tags: ['idea','vscode','eclipse','netbeans','杂谈']
description: '为什么java开发者使用ide对钱包这么不友好'
category: '杂谈'
---

## 介绍下我自己

我自己一直订阅 JetBrains 的全家桶，也长期用 VS Code。平时写 Java 和 Kotlin 我用 IDEA，数据库管理用 DataGrip。但写 Go 和 JS 的时候，我更习惯用 VS Code。

不过，现在情况有点变了。随着年龄增长，我也到了婚嫁的时候，再加上物价一直在涨，工资却没什么变化，养家糊口压力确实挺大的。所以，我打算不再续订 JetBrains 的全家桶了，正在找一些开源的替代品，等订阅到期后永久回退版本也更不上时代的时候我也会被迫做出抉择`开源`or`破解版`。

## 为什么java开发者的ide选择如此少

Java 作为一门历史悠久的编程语言，其发展历程中涌现出了众多 IDE（集成开发环境），这些工具也随着技术的进步经历了多轮更迭。早期的 Java IDE，如 JBuilder、JCreator 等，曾一度风靡，但随着技术的发展，逐渐被新的工具所取代。

现在，开源免费的Java IDE（集成开发环境）有不少，但真正好用的却不多。比如曾经的“一哥”Eclipse，如今几乎没人用了，可能只有学校里的老师还会用MyEclipse来教学吧。Eclipse本身这么多年变化不大，还有不温不火的NetBeans。这些与IntelliJ IDEA一比，就显得不够便捷了。

这时候肯定有人会说：“那不是还有VS Code吗？”我也是VS Code的长期用户，知道VMware给它做了官方支持的Spring插件，Red Hat也开发了Java插件和Quarkus插件。但VS Code在开发Java时，其实并不那么顺手。首先，它的资源占用有点高，打开同一个项目，内存占用比IDEA大概高出1个G左右。而且，手动删除build目录后，它还会崩溃，得重新刷新加载构建缓存。还有，它对Kotlin的支持太差了。另外，不同项目需要不同版本的JDK，每次都要手动修改设置，用起来真是很麻烦。

后来，我换回了IntelliJ IDEA的社区版，但发现它缺少很多插件，用起来也不方便。于是，我发现了“毛子”的GigaIDE社区版，虽然比IDEA社区版好用一些，但依然很麻烦，安装了一些替代插件后，勉强能用。

最近，JetBrains开始逐步开放WebStorm、RustRover、Rider的非商用免费版。不过，IntelliJ IDEA在Java IDE市场里一直占据垄断地位，哪怕它不免费，大家也还是会继续选择它。

## 最后

Java 开发者们早就苦 IntelliJ IDEA 久矣。我期待微软、VMware 和 Red Hat 能够继续努力，进一步优化和打磨 VS Code，使其在 Java 开发领域更加完善。同时，也希望 Oracle 作为 Java 的“亲爹”，能够加大投入，为开发者提供更经济实惠且功能强大的开发工具。毕竟，一个更便宜、更好用的开发环境，对于广大 Java 开发者来说，无疑是极具吸引力的。面对市场被其他语言瓜分的压力，以及idea成本高昂，对于个人开发者与小企业来说用成本确实有点高了。
