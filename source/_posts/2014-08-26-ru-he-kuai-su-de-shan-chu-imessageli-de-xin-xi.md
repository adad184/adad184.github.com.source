---
layout: post
title: "如何快速的删除mac上imessage里的(垃圾)信息"
date: 2014-08-26 08:51:34 +0800
comments: true
categories: 技巧心得
keywords: 
description: 
---

前言
========

imessage现在处于一个非常鸡肋的状态 开了无用 关了可惜 而且貌似跟中国的运营商还有点点冲突(经常各种群发短信的时候夹杂着imessage 对方就收不到) 

当然有垃圾就要清除 手机上还好 看到了顺手左划就删掉了 但是倒霉的是在mac不会同步被删除 而且万一你从没没用过mac上的`Messages`这个服务 赶紧去点开 有惊喜

![imessage](https://dl.dropboxusercontent.com/u/433937/Blog/2014-08-26-ru-he-kuai-su-de-shan-chu-imessageli-de-xin-xi2.png)

恩 是的 你肯定发现这个东东已经成了各种小广告的垃圾场

![imessage](https://dl.dropboxusercontent.com/u/433937/Blog/2014-08-26-ru-he-kuai-su-de-shan-chu-imessageli-de-xin-xi.png)

ok 是时候准备删除了(强迫症你别理我) 于是 点X - 确认删除 - 点X - 确认删除 ...... 当重复了十几遍以后 你会觉得这样太煞笔了 这几百条的垃圾 删除到什么时候去?  shift批量选择? 你试试支持么(这个特性不支持确实有点不合理)

那么如何快速的删除imessage里的信息呢?


方法
========

* 方法一 快速删除单条记录

``` objc
	组合键 option+command+delete(大的那个 小键盘不管用)
```

这样可以快速的删除单挑 好处是可以选择要保留的

* 方法二 快速清空所有记录

```objc
    退出messages应用
    rm -r ~/Library/Messages/chat.*
    重启messages
```
    
据说这个方法可以清空所有信息 但是由于我测试的时候没有推出messages就直接删除文件了 导致我重启了以后所有信息都还在(汗 这是为什么 明明文件删除了) 所以我不确定这个方法是否在10.9上面还有效
    
