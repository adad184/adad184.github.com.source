---
layout: post
title: "NimbusKit 介绍与使用实践"
date: 2013-08-06 16:51:28 +0800
comments: true
categories: 经验介绍
---

介绍
========

[NimbusKit 官网](http://nimbuskit.info/)  
[NimbusKit 源码](https://github.com/jverkoey/nimbus)

NimbusKit是一组用于快速开发的iOS框架,是源自*Facebook*的著名框架`Three20`的替代者,包括下面几大类的功能

* Attributed Label    -   富文字Label
* Badge - 数字角标
* Interapp - 应用间交互
* Launcher - 类桌面启动器
* Network Image - 网络图片下载显示
* Photo Albums - 相册
* Web Controller - 浏览器
* Table Models - 表格数据模型
* Overview - 直观方便的调试分析内嵌图形工具
* 等...

NimbusKit的demo很直观,编译运行以后就可以体验其强大的功能了

![Nimbus Demo](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-06-at-nimbus-jie-shao-yu-shi-yong-shi-jian2.png)


使用
========

因为之前的项目的富文本设计要支持`iOS5`和`iOS6`,而`attributeString`特性只有iOS6支持,所以使用了*NIAttributedLabel*

![富文本](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-06-at-nimbus-jie-shao-yu-shi-yong-shi-jian1.PNG)


这次新的项目由于有大量的表单页面,考虑到完全自己实现太耗时间,所以又重新考虑了`NimbusKit`的`Table Models`

NimbusKit提供的表单组件如下

|| 组件 								|| 功能 			||
||:--------------------------------:||:------------:||
|| NIRadioGroup 					|| 多选一 		||
|| NITextInputFormElement 			|| 文本输入 		||
|| NISwitchFormElement 				|| 开关 			||
|| NISliderFormElement 				|| 滑动条 		||
|| NISegmentedControlFormElement 	|| 分段 			||
|| NIDatePickerFormElement 			|| 时间选择		|| 
|| NITitleCellObject 				|| 单行文本 		|| 
|| NISubtitleCellObject 			|| 主副文本 		||
|| ... 								|| 等其他 		||

![基础表单组件](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-06-at-nimbus-jie-shao-yu-shi-yong-shi-jian4.png)

这里首先要介绍一下NimbusKit的Table-Modal-Action模型,先上一段简单代码

``` objc

@interface GGJSZController ()
<
UITableViewDelegate,
UITextFieldDelegate
>

@property (nonatomic, strong) UITableView *tableView;

@property (nonatomic, strong) NITableViewModel* model;
@property (nonatomic, strong) NITableViewActions *action;

@property (nonatomic, strong) NIActionBlock queryBlock;

@end



@implementation GGJSZController

- (id)init
{
    self = [super init];
    if (self) {
        
        self.tableView = [[UITableView alloc] initWithFrame:self.view.bounds style:UITableViewStyleGrouped];
        ...
        ...
        self.tableView.delegate = self;
        [self.view addSubview:self.tableView];
        
        [self configureBlock];
        [self configureForm];
        
    }
    return self;
}

- (void)configureBlock
{
    __weak GGUserController *weakSelf = self;
    
    self.queryBlock = ^BOOL(id object, UIViewController *controller, NSIndexPath* indexPath) {
        
    	...
    	...
    	...

        return YES;
    };
    
    
}

- (void)configureForm
{
    self.action = [[NITableViewActions alloc] initWithTarget:self];
    
    NSArray* tableContents =
    [NSArray arrayWithObjects:
     @"请输入驾驶证号",
     [NITextFieldFormElement textFieldElementWithID:0 labelText:@"驾驶证号" placeholderText:@"请输入驾驶证号" value:TESTID delegate:self],
     [self.action attachToObject:[NITapCellObject objectWithTitle:@"查询" color:COLOR_TAP]
                 navigationBlock:self.queryBlock],
     nil];
    
    self.model = [[NITableViewModel alloc] initWithSectionedArray:tableContents
                                                         delegate:(id)[NICellFactory class]];
    
    self.tableView.dataSource = self.model;
    self.tableView.delegate = [self.action forwardingTo:self];
    
}

@end


```

这段代码的效果如图

![表单效果](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-06-at-nimbus-jie-shao-yu-shi-yong-shi-jian3.png)

其关键的地方就是

``` objc

@property (nonatomic, strong) NITableViewModel* model;
@property (nonatomic, strong) NITableViewActions *action;

```

`NITableViewModel` 接管了`UITableView`的`dataSource`,并用一种更简单直观的方式创建表格内容,只需要创建对应的tableContents即可生成表单

`NITableViewActions` 接管了`UITableView`的`delegate`,提供了`NITableViewModel`中已连接对象(attachToObject)对点击事件以*block*的方式进行响应

`NITableViewModel`与`NITableViewActions`的恰当配合则能够生成所需的表单

自定义表单
========

在日常的开发中,仅仅依靠NimbusKit自带的表单组件肯定是无法完全满足我们的需求的,所以自定义表单组件则是非常必要的功能,而在NimbusKit中自定义表单组件也是非常容易的,比如自带的表单组件中只有独立的Label或者Input组件,并没有像上图那样左边为Label右边为Input的组件,而上图中所见的输入身份证的组件,即为我自定义的表单组件之一,其代码如下

``` objc

@interface NITextFieldFormElement : NIFormElement

+ (id)textFieldElementWithID:(NSInteger)elementID labelText:(NSString*)labelText placeholderText:(NSString *)placeholderText value:(NSString *)value delegate:(id<UITextFieldDelegate>)delegate;


@property (nonatomic, copy) NSString* labelText;
@property (nonatomic, copy) NSString* placeholderText;
@property (nonatomic, copy) NSString* value;
@property (nonatomic, assign) id<UITextFieldDelegate> delegate;

@end


@interface NITextFieldFormElementCell : NIFormElementCell <UITextFieldDelegate>
@property (nonatomic, readonly, NI_STRONG) GGTextField* textField;
@end





@implementation NITextFieldFormElement

+ (id)textFieldElementWithID:(NSInteger)elementID labelText:(NSString *)labelText placeholderText:(NSString *)placeholderText value:(NSString *)value delegate:(id<UITextFieldDelegate>)delegate
{
    
    NITextFieldFormElement* element = [super elementWithID:elementID];
    element.labelText = labelText;
    element.placeholderText = placeholderText;
    element.value = value;
    element.delegate = delegate;
    return element;
}

- (Class)cellClass {
    return [NITextFieldFormElementCell class];
}

@end



@implementation NITextFieldFormElementCell

@synthesize textField = _textField;

///////////////////////////////////////////////////////////////////////////////////////////////////
- (id)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier {
    if ((self = [super initWithStyle:style reuseIdentifier:reuseIdentifier])) {
        self.selectionStyle = UITableViewCellSelectionStyleNone;
        
        _textField = [[GGTextField alloc] init];
        [_textField setTag:self.element.elementID];
        [_textField setAdjustsFontSizeToFitWidth:YES];
        [_textField setMinimumFontSize:10.0f];
        [_textField setTextAlignment:NSTextAlignmentRight];
        [_textField addTarget:self action:@selector(textFieldDidChangeValue) forControlEvents:UIControlEventAllEditingEvents];
        [self.contentView addSubview:_textField];
    }
    return self;
}


///////////////////////////////////////////////////////////////////////////////////////////////////
- (void)layoutSubviews {
    [super layoutSubviews];
    
    CGFloat labelWidth = 80;
    CGRect frame = UIEdgeInsetsInsetRect(self.contentView.bounds, UIEdgeInsetsMake(5, 10, 5, 10));
    frame = CGRectMake(frame.origin.x+labelWidth, frame.origin.y, frame.size.width-labelWidth, frame.size.height);
    _textField.frame = frame;
}


///////////////////////////////////////////////////////////////////////////////////////////////////
- (void)prepareForReuse {
    [super prepareForReuse];
    
    self.textLabel.text = nil;
    
    _textField.placeholder = nil;
    _textField.text = nil;
}


///////////////////////////////////////////////////////////////////////////////////////////////////
- (BOOL)shouldUpdateCellWithObject:(NITextFieldFormElement *)textfieldElement {
    if ([super shouldUpdateCellWithObject:textfieldElement]) {
        
        self.textLabel.text = textfieldElement.labelText;
        
        _textField.placeholder = textfieldElement.placeholderText;
        _textField.text = textfieldElement.value;
        _textField.delegate = textfieldElement.delegate;
        
        _textField.tag = self.tag;
        
        [self setNeedsLayout];
        return YES;
    }
    return NO;
}


///////////////////////////////////////////////////////////////////////////////////////////////////
- (void)textFieldDidChangeValue {
    NITextFieldFormElement* textInputElement = (NITextFieldFormElement *)self.element;
    textInputElement.value = _textField.text;
}

@end

```

上述代码给出了如何自定义NimbusKit中的表单组件,可见表单组件几乎都是`NIFormElement`的子类,只要基于此类,定义显示所需的控件到`NIFormElement`内部,并调整对应位置即可

至此我们已经知道如何自定义组件了,但是如果有稍微复杂的需求的话,要如何实现呢? 

![复杂的自定义组件](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-06-at-nimbus-jie-shao-yu-shi-yong-shi-jian5.png)

如图,输入***出发城市***和***目的城市***的组件也是一个自定义组件,除了有很多Label和Input之外,其高度也跟普通的组件不一样.

其实自定义高度很简单,只要重载`NICell`(比如`NITextFieldFormElementCell`)的此方法并返回所需的高度

``` objc
+ (CGFloat)heightForObject:(id)object atIndexPath:(NSIndexPath *)indexPath tableView:(UITableView *)tableView;
```

然后在使用的ViewController中调用UITableView的方法

``` objc
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    CGFloat height = tableView.rowHeight;
    id object = [(NITableViewModel *)tableView.dataSource objectAtIndexPath:indexPath];
    id class = [object cellClass];
    if ([class respondsToSelector:@selector(heightForObject:atIndexPath:tableView:)]) {
        height = [class heightForObject:object atIndexPath:indexPath tableView:tableView];
    }
    return height;
}
```

即可自定义此组件的高度了(其实直接在上述方法中直接返回对应的高度即可,但是上述代码则有很高的通用性)


小结
========

至此,我们已经了解了`NimbusKit`的基本功能和如何对其表单模型进行自定义.

除了此次着重介绍的Table-Action模型之外,其实`NimbusKit`还有很多值得使用的功能,如**Launcher**功能,**Overview**功能,实用且集成起来很简单,在这就不一一介绍了.

最后,简单介绍一下NimbusKit的作者[Jeff Verkoeyen](http://jeffverkoeyen.com/),其主要作品有:

* Facebook for iPad ( June 2010 - June 2011 )
* Google Maps for iPhone ( June 2012 - April 2013 )




