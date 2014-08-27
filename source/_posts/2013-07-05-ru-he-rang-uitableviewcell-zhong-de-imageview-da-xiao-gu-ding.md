---
layout: post
title: "如何让 UITableViewCell 中的 imageView 大小固定"
date: 2013-07-05 16:41:39 +0800
comments: true
categories: 技巧心得
---

`UITableView`可以算是使用频率最高的组件之一的,在开发过程中经常需要展示一些简单的信息列表

![常见列表布局](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-06%2013.33.25.png)

如图,很多页面其实就是这种展示结果,通常需要`imageView`,`textLabel`,`detailTextlabel`,而`UITableViewCell`本身提供了方便的自动布局(当有图片和没图片时,textLabel和detailLabel的位置会左右自动调整). 但是图片的大小却是没有办法固定的(直接设置`imageView.frame`是无法固定`imageView`的大小的),那么一般来说解决这个问题的办法有两种:

* 固定显示图片的大小(包括PlaceHolder)
* 自定义tableViewCell,添加自定义的`imageView`,`textLabel`和`detailTextLabel`

这两种方式都可以解决这个问题,但是这两种方式其实都挺麻烦的,能否直接固定imageView的大小呢? 方法是有的,只需要重载`layoutSubviews`即可

``` objc 派生UITableViewCell

//自定义一个Cell
@interface MMCell : UITableViewCell

@end


@implementation MMCell

//重载layoutSubviews
- (void)layoutSubviews
{
    UIImage *img = self.imageView.image;
    self.imageView.image = [UIImage imageName:@"res/PlaceHolder.png"];
    [super layoutSubviews];
    self.imageView.image = img;
}

@end
```
	

这样,我们只要使用`MMCell`就可以固定`imageView`的大小了,且大小为`PlaceHolder.png`的大小(一般来说这种页面都会使用一个`PlaceHolder.png`来显示默认图片).

原理是在`UItableVeiw`的`layoutSubviews`调用时,会根据`imageView.image`的大小来调整`imageView`,`textLabel`,`detailTextLabel`的位置,在此之前我们先将`imageView.image`设置为`PlaceHolder.png`图片,等待重新布局完后再将原本的图片设置回去就可以了