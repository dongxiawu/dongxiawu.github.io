---
title: butterknife详解之源码结构分析
date: 2018-02-14 21:12:34
tags:
- butterknife
- 开源库
layout:
updated:
comments: true
categories:
- butterknife
permalink:
---

{% asset_img logo.png %}

> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

学习完 butterknife 的基本使用之后，本文主要讲解 butterknife 源码的组织结构。

<!--more-->

# 源码构成 #
首先我们来看一下 butterknife 的源码构成。

{% asset_img butterknife源码构成.png %}

其中
* butterknife ;主模块，提供android使用的API
* butterknife-annotations; 注解模块，提供了所需的注解
* butterknife-compiler；注解编译模块，提供了编译时用到的注解的处理器
* butterknife-gradle-plugin；插件模块，自定义的gradle插件，辅助生成有关代码
* butterknife-integration-test；该项目的测试用例模块
* butterknife-lint；该项目的静态代码检查模块
* sample；demo

从源码组成结构我们可以看出，库的结构非常清晰，基本上按功能分成了7个模块。

# butterknife模块结构 #

{% asset_img butterknife模块结构.png %}

可以看到，butterknife模块一共只有5个类，而且供客户端使用的只有 Butterknife 和 Unbinder 两个类，可以说将设计原则都应用得淋漓精致。

* DebouncingOnClickListener:按钮去抖，其原理时每次按钮按下时禁止按钮短时间内重复按下，处理按钮事件同时另起一个线程重新使能按钮。核心代码为：

      @Override public final void onClick(View v) {
          if (enabled) {
            enabled = false;
            v.post(ENABLE_AGAIN);
            doClick(v);
          }
        }

* ImmutableList 将视图数组转换成不可变链表，该方法比系统自带的方法 Collections.unmodifiableList(); 轻量

* Utils 实用方法，主要供生成的代码使用，如获取系统资源等。

* Unbinder 控件解绑接口，当控件不再使用时应该解绑，避免内存泄露。

* Butterknife

{% asset_img butterknife方法.png %}

从 Butterknife 的方法列表中可以看出 Butterknife类主要做了两件事

1. 绑定(bind)：

  其中，绑定的对象可以是 View、Activity、Dialog

  其步骤及原理是：
  1. 将 Activity、Dialog、View 等提供控件信息的对象等作 target 参数。
  2. 从 Activity、Dialog、View 等中找到根视图作为 source 参数。
  3. 生成一个类，类名为 target_ViewBinding，该类持有  target 引用，并实现 Unbinder 接口。
  4. 从 target 引用中获取控件名和对应的布局id，从 source 中找到对应视图，进行绑定。

2. 应用(apply)
  将 Action、Setter、Property 等操作应用在对应的控件上。其原理是实现 Action、Setter、Property 等接口，并通过ButterKnife.apply()方法应用在对应的控件上。

到这里 butterknife模块结构 就全部介绍完毕了。下一节介绍 butterknife 的注解系统。
