---
layout: post
title: "完美解决 interactivePopGestureRecognizer 卡住的问题"
date: 2013-12-12 16:19:10 +0800
comments: true
categories: 
keywords: 
description: 
published: false
---

`interactivePopGestureRecognizer`是iOS7推出的解决`VeiwController`滑动后退的新功能,虽然很实用,但是坑也很多啊,用过的同学肯定知道问题在哪里,所以具体问题我就不描述了,这里只给出如何完美解决`interactivePopGestureRecognizer`卡住的问题.

当然我们要自定义UINavigationController来解决这个问题:

``` objc

@interface GGNavigationController : UINavigationController
<
UINavigationControllerDelegate,
UIGestureRecognizerDelegate
>
@end

@implementation GGNavigationController

- (void)viewDidLoad
{
    NSFUNC;
    [super viewDidLoad];
    
    __weak GGNavigationController *weakSelf = self;
    
    if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)])
    {
        self.interactivePopGestureRecognizer.delegate = weakSelf;
        self.delegate = weakSelf;

        //初始时如果只有一个viewController,也要禁用此手势,不然在屏幕左划一下以后,就会卡住
        self.interactivePopGestureRecognizer.enabled = (self.viewControllers.count > 1);
    }
}
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    //在push的过程中一定要禁用此手势才不会卡住
    if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)])
    {
        self.interactivePopGestureRecognizer.enabled = NO;
    }
    
    [super pushViewController:viewController animated:animated];
}

#pragma mark UINavigationControllerDelegate

- (void)navigationController:(UINavigationController *)navigationController
       didShowViewController:(UIViewController *)viewController
                    animated:(BOOL)animate
{
    if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)])
    {
        //当Nav里只有一个viewController时 必须禁止手势 不然会卡住
        self.interactivePopGestureRecognizer.enabled = (self.viewControllers.count > 1);
    }
}

@end

```