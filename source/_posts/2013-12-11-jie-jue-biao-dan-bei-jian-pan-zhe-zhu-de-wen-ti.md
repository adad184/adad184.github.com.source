---
layout: post
title: "解决表单被键盘遮住的问题"
date: 2013-12-11 11:01:59 +0800
comments: true
categories: 技巧心得
---

问题
=================

处理表单的时候,一定会碰到的就是输入控件被键盘遮住的问题,如图:

![实例](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-11-jie-jue-biao-dan-bei-jian-pan-zhe-zhu-de-wen-ti1.png)

左边是普通表单,中间是2B表单,右边是文艺表单.

分析
=================
处理这种问题无非就是2个步骤:

1. 键盘弹出时,缩小`UITableView`的`frame`
2. 滚动`UITableView`,让当前输入的控件可见

代码写出来就是这几步

1. 捕获键盘事件
2. 计算键盘高度并调整`UITableView`的`frame`
3. 获取当前正在输入的控件
4. 计算其在`UITableView`中的位置,并滚动到其位置让其可见

那么如何一步一步的来实现这些步骤呢?

捕获键盘事件
-----------------
``` objc 捕获键盘事件
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(actionKeyboardShow:)
                                             name:UIKeyboardDidShowNotification
                                           object:nil];
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(actionKeyboardHide:)
                                             name:UIKeyboardWillHideNotification
                                           object:nil];

- (void)actionKeyboardShow:(NSNotification *)notification
{
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidShowNotification object:nil];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidChangeFrameNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(actionKeyboardShow:)
                                                 name:UIKeyboardDidChangeFrameNotification
                                               object:nil];
    
}

- (void)actionKeyboardHide:(NSNotification *)notification
{
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidChangeFrameNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(actionKeyboardShow:)
                                                 name:UIKeyboardDidShowNotification
                                               object:nil];
}

```

计算键盘高度并调整`UITableView`的`frame`
-----------------
``` objc 计算键盘高度并调整UITableView的frame
- (void)actionKeyboardShow:(NSNotification *)notification
{
    CGSize keyboardSize = [[[notification userInfo] objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue].size
    self.tableView.frame = CGRectMake(0, 0, 320, self.view.h-keyboardSize.height);
    
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidShowNotification object:nil];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidChangeFrameNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(actionKeyboardShow:)
                                                 name:UIKeyboardDidChangeFrameNotification
                                               object:nil];
    
}
```

获取当前正在输入的控件
-----------------

这里得说一句,普通程序员一般是这样来获取的

``` objc UIView的Category
- (UIView *) getFirstResponder
{
    if (self.isFirstResponder) {
        return self;
    }
    
    for (UIView *subView in self.subviews) {
        UIView *firstResponder = [subView getFirstResponder];
        if (firstResponder != nil) {
            return firstResponder;
        }
    }
    
    return nil;
}
```

虽然没错,但是文艺程序员应该[这样来获取](http://stackoverflow.com/questions/5029267/is-there-any-way-of-asking-an-ios-view-which-of-its-children-has-first-responder/14135456#14135456)

``` objc UIResponder的Category
static __weak id currentFirstResponder;

+(id)currentFirstResponder {
    currentFirstResponder = nil;
    [[UIApplication sharedApplication] sendAction:@selector(findFirstResponder:) to:nil from:nil forEvent:nil];
    return currentFirstResponder;
}

-(void)findFirstResponder:(id)sender {
    currentFirstResponder = self;
}

```


同理,有时候我们需要让键盘消失,那么也有三种做法可以选择

``` objc
[someView resignFirstResponder];

[self.view endEditing:YES];

[[UIApplication sharedApplication] sendAction:@selector(resignFirstResponder) to:nil from:nil forEvent:nil];
```

如何选择呢? It's up to U.

计算其在`UITableView`中的位置,并滚动到其位置让其可见
-----------------
``` objc 计算其在UITableView中的位置,并滚动到其位置让其可见
- (void)actionKeyboardShow:(NSNotification *)notification
{
    CGSize keyboardSize = [[[notification userInfo] objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue].size;
    self.tableView.frame = CGRectMake(0, 0, 320, self.view.h-keyboardSize.height);
    
    UIView *v = [UIResponder currentFirstResponder];
    
    if ( v )
    {
        while ( ![v isKindOfClass:[UITableViewCell class]]) {
            v = v.superview;
        }
        
        UITableViewCell *cell = (UITableViewCell*)v;
        
        [self.tableView scrollToRowAtIndexPath:[self.tableView indexPathForRowAtPoint:cell.center] atScrollPosition:UITableViewScrollPositionBottom animated:YES];
    }
    
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidShowNotification object:nil];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidChangeFrameNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(actionKeyboardShow:)
                                                 name:UIKeyboardDidChangeFrameNotification
                                               object:nil];
    
}
```
