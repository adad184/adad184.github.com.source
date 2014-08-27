---
layout: post
title: "解决表单被键盘遮住的问题(续)"
date: 2013-12-19 16:31:34 +0800
comments: true
categories: 
keywords: 
description: 
---

刚才检查代码的时候 发现了之前代码的一些问题 这里做一下修正 为此我为UIView和UITableView各新增了一个Category方法

``` objc UIView的Category
- (BOOL) haveSubview:(UIView*)subView
{
    UIView *v = subView;
    
    while (v)
    {
        if ( self == v )
        {
            return YES;
        }
        
        v = v.superview;
    }
    
    return NO;
}
```


``` objc UITableVIew的Category方法
- (BOOL) haveSubview:(UIView*)subView
{
    if ( v && [self haveSubview:v] )
    {
        while ( v && ![[v class] isSubclassOfClass:[UITableViewCell class]]) {
            v = v.superview;
        }
        
        if ( v )
        {
            NSLog(@"%@",NSStringFromClass(v.class));
            
            UITableViewCell *cell = (UITableViewCell*)v;
            
            NSLog(@"%@",[self indexPathForCell:cell]);
            
            [self scrollToRowAtIndexPath:[self indexPathForRowAtPoint:cell.center] atScrollPosition:UITableViewScrollPositionBottom animated:YES];
        }
    }
}
```

这样 当我们需要处理弹出键盘时 就只需要如此调用就可以了

``` objc 调用方法
UIView *v = [UIResponder currentFirstResponder];
[self.tableView scrollToView:v];
```


