---
layout: post
title: "开源项目:MMPlaceHolder"
date: 2014-08-21 12:17:14 +0800
comments: true
categories: 技巧心得 
keywords: 
description: 
---


前言
========

最近在做一个新的项目 免不了又要跟UI打交道 之前N多的经验告诉我 为了达到设计图上的100%Copy 避免不了在切图的`尺寸`和`位置`上花大量的时间 (为什么不按美术切给你的尺寸来? 这... 只能说美术很难跟我达到天人合一... ( ͡° ͜ʖ ͡°)

那么能不能以实际的效果告诉设计师尺寸和位置呢?  如下面这样就皆大欢喜啦

![效果](https://dl.dropboxusercontent.com/u/433937/Blog/2014-08-21-kai-yuan-xiang-mu-mmplaceholder-1.png)

当然 很早之前就已经有前人做过这种事情了 比如

[PAPlaceholder](https://github.com/dhennessy/PAPlaceholder) 
[Masu](https://github.com/midnightSuyama/Masu)  

但是呢 共同的特点就是`难用`(怎么难用? 可以自己看一下他们的文档)

所以呢 处于好玩的目的 就自己写了一个简单易用的版本(怎么简单? 一行代码就搞定啦)

介绍
=======

[MMPlaceHolder](https://github.com/adad184/MMPlaceHolder) 

* 一行代码解决显示问题 简单易用
* 搭建码农和设计之间的沟通桥梁 减少沟通成本(Talk is cheap. Show me the code.)
* 显示大小自适应(最小支持30*30哦)

使用
=======

``` objc

- (void)showPlaceHolder;
- (void)showPlaceHolderWithLineColor:(UIColor*)lineColor;
- (void)showPlaceHolderWithLineColor:(UIColor*)lineColor backColor:(UIColor*)backColor;
- (void)showPlaceHolderWithLineColor:(UIColor*)lineColor backColor:(UIColor*)backColor arrowSize:(CGFloat)arrowSize;

- (void)hidePlaceHolder;
- (MMPlaceHolder *)getPlaceHolder;

```




