---
layout: post
title: "如何从多重 modalViewController 中直接返回最底层"
date: 2013-12-10 14:59:58 +0800
comments: true
categories: 技巧心得
---

`ModalViewController`是经常会用到的展现`ViewController`的方式,而显示和收起`ModalViewController`也是很简单的

``` objc

- (void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion NS_AVAILABLE_IOS(5_0);
- (void)dismissViewControllerAnimated: (BOOL)flag completion: (void (^)(void))completion NS_AVAILABLE_IOS(5_0);

- (void)presentModalViewController:(UIViewController *)modalViewController animated:(BOOL)animated NS_DEPRECATED_IOS(2_0, 6_0);
- (void)dismissModalViewControllerAnimated:(BOOL)animated NS_DEPRECATED_IOS(2_0, 6_0);

```

但是有的时候我们的需求很特殊,比如在一个`ModalViewController`里要present另一个`ModalViewController`,甚至再present一个`ModalViewController`,**然后**可能在某个时候APP发出一条消息,需要一下子dismiss掉所有的`ModalViewController`(比如你在使用过程中,突然APP检测到你的登录状态异常,需要重新登录,这个时候所有的页面都需要消失),这时候该如何办呢?

正巧我现在正在做的项目遇到了这个问题,所以研究了一下,得到了以下的解决办法:

首先,必须知道现在整个APP最顶层的`ViewController`是哪个,我的做法是在每个`ViewController`的`viewWillAppear`中记录一下,当然这个操作是自动完成的,因为每个项目,我都会从`UIViewController`派生一个子类,然后再从这个子类派生所有的`ViewController`方便管理.

``` objc

@interface MMViewController : UIViewController

@end

@implementation MMViewController

- (void)viewWillAppear:(BOOL)animated
{
    APP.presentingController = self;
}
@end

```

得到了顶层的`ViewController`以后,事情就简单了,我们只要追根溯源,找到最底层的`ViewController`就行了

``` objc

if ( APP.presentingController )
{
    UIViewController *vc = self.presentingController;
    
    if ( !vc.presentingViewController )
    {
        return;
    }
    
    while (vc.presentingViewController)
    {
        vc = vc.presentingViewController;
    }
    
    [vc dismissViewControllerAnimated:YES completion:^{

    }];
    
}

```
