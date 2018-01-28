---
title: 软件分析工具-understand
date: 2018-01-28 16:32:58
tags:
- 软件分析工具
- understand
layout:
updated:
comments: true
categories:
- 开发工具
permalink:
---

{% blockquote Linus Benedict Torvalds %}
Read The Fucking Source Code.
{% endblockquote %}

{% asset_img 轻轻松松看源码.jpg %}

> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

<!--more-->

不知道各位平时有没有阅读系统源码或者项目的习惯，毕竟连 Linux 的创造者 —— Linus 老人家都鼓励我们多去阅读源码。但是一想到一些源码动辄上万行的代码量，真的有耐心一直坚持阅读的人又有多少呢？而且源码毕竟是别人写的，有时候只看源码想要理解作者的意图还是有点困难的。那么有没有什么办法能够提高我们阅读源码的速度，或者能够帮助我们理解作者的意图呢？

我们平时阅读源码的时候，一行一行地读肯定不是最有效的方式，总结一下我们阅读源码的目的主要有以下几点：
1. 了解功能实现方式
2. 学习代码组织形式
3. 在原有基础上增加功能

再来看一下我们阅读源码的时候最希望知道的东西有什么
1. 这个类是干什么的，哪些类使用了这个类
2. 这个方法是干什么的，它修改了那些变量
3. 这个变量有什么用，哪个方法里修改了这个变量

所以到这里我们可以推导出如何快速有效地阅读源码了
1. 阅读文档。读源码时阅读文档可以帮助我们了解这个库，类，方法等的功能和使用场景，能够是我们对源码有大致的了解。
2. 阅读注释。优秀的源码必定带有大量的注释，通过注释我们可以了解类的实现原理、方法的功能等。
3. 看类名，方法名，变量名等。好的源码可以用过变量名等就可以知道大概的作用，这也叫做“注释式编程”。
4. 学习设计模式。绝大部分源码有运用了设计模式，设计模式也被称为开发人员的共同语言。了解设计模式能够让你快速地知道源码作者的意图和代码组织结构。

以上说的都是一些我们阅读源码需要修炼的“硬功夫”，需要一步一个脚印地去学习，去进步。但是有一些工具却能够加快我们学习地脚步。比如一个叫做 Understand 的代码分析工具。

我们先来看看这个工具的效果。比如当我想看看这个类有哪些变量和方法时，它是这样的：
{% asset_img UMLClassDiagram-okhttp3-OkHttpClient.png %}

当我想看看某个方法被哪些方法调用了，它是这样的：
{% asset_img ClusterCallButterfly-OkHttpClient-OkHttpClient.png %}

当我想看看某个方法的工作流程，它是这样的：
{% asset_img ControlFlow-OkHttpClient-OkHttpClient.png %}

当我想看看某个方法的时序图时，它是这样的：
{% asset_img UMLSequenceDiagram-OkHttpClient-OkHttpClient.png %}

而且，我们可以通过点击图上的任意方法和类实现跳转，简直就是把代码变成流程图有木有。难怪 Understand 的宣传语就是“让你看代码一目十行”。

到这里，你应该知道 Understand 的作用了，Understand集成了代码编辑器，代码跟踪器和代码分析器，提供了很强大的界面，将分析结果以各种形式（图形、图表、架构图等）呈现给用户，能很大程度的方便程序员进行开发，维护，调试其代码。

Understand 有一下特点
1. 支持多语言：Ada, C, C++, C#, Java, FORTRAN, Delphi, Jovial, and PL/M ，混合语言的project也支持
2. 多平台： Windows/Linux/Solaris/HP-UX/IRIX/MAC OS X
3. 代码语法高亮、代码折迭、交叉跳转、书签等基本阅读功能。
4. 可以对整个project的architecture、metrics进行分析并输出报表。
5. 可以对代码生成多种图（butterfly graph、call graph、called by graph、control flow graph、UML class graph等），在图上点击节点可以跳转到对应的源代码位置。
6. 提供Perl API便于扩展。作图全部是用Perl插件实现的，直接读取分析好的数据库作图。
7. 内置的目录和文件比较器。
8. 支持project的snapshot，并能和自家的TrackBack集成便于监视project的变化。

看到这里，你是不是迫不及待地想试一下了呢？不过这个软件目前是收费的，而且安装包不能在官网下载到。必须发邮件去申请才能获取下载连接，而且回复时间也不固定，反正我发了一周还没有收到邮件，不过不用怕，在公众号后台回复 ”understand” 可以获取 Understand 的安装包、注册码和使用教程哦。
