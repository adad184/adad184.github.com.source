---
layout: post
title: "个人总结的一些 APP 的代码实践"
date: 2013-12-10 14:56:46 +0800
comments: true
categories: 技巧心得
---

建立一个辅助的APP类,减少对AppDelegate的修改
---------------------------------------

最开始接触iOS开发的时候,如果需要一些全局变量或者全局函数的时候,总是直接在`AppDelegate`中添加,因为`AppDelegate`可以直接获取

``` objc
[UIApplication sharedApplication].delegate
```

但是时间长了还是觉得这样不太好,`AppDelegate`本身有其自己的作用(对于App本身的一些事件进行处理,如启动,切换,推送),这样做感觉怪怪的,所以还是自己弄一个专门处理我们所需的全局变亮或者全局函数的对象会更好一些


``` objc

//APPHelper.h
@interface APPHelper

+ (APPHelper*)call;

- (void) configureWindow:(UIWindow*)window;

@property (nonatomic, readonly) AppDelegate *delegate;
@property (strong, readonly) UIWindow *window;

@end


//APPHelper.m

@interface APPHelper ()


@end


@implementation APPHelper

- (id)init
{
    self = [super init];

    if (self) {
        
        _delegate = (GGAppDelegate*)[UIApplication sharedApplication].delegate;
    }

    return self;
}


+ (APPHelper *)call
{
    static dispatch_once_t  onceQueue;
    static APPHelper *appInstance;

    dispatch_once(&onceQueue, ^{
        appInstance = [[APPHelper alloc] init];
    });
    return appInstance;
}

- (UIWindow *)window
{
    return self.delegate.window;
}



- (void)configureWindow:(UIWindow*)window
{
    
    UINavigationController *nav = [[UINavigationController alloc] init];

    ...
    ...
    ...
    
    window.rootViewController = nav;
    
}

@end

```

然后 在预编译头`*.pch`中加入

``` objc
#import "AppHelper.h"


#define APP ([APPHelper call])
```

就可以直接在代码的任意一个地方直接使用此类了,如

``` objc

    //设置APP为圆角
    APP.window.layer.cornerRadius = 5.0f;
    APP.window.layer.masksToBounds = YES;

```




简单的Autoresizing的宏
---------------------------------------
一开始我就喜欢代码布局,从来没使用过IB或者SB开发,所以如何在代码中用**Autoresizing**就显得很重要了(那个时候还没有*AutoLayout*).

为此我还专门研究了一下IB(那个时候还没有SB),并把生成的nib用[nib2objc](https://github.com/akosma/nib2objc/)转换成了代码来学习.

使用下面的宏,可以很轻松的实现**Autoresizing**.

``` objc
#define FlexibleT                   UIViewAutoresizingFlexibleTopMargin
#define FlexibleB                   UIViewAutoresizingFlexibleBottomMargin
#define FlexibleL                   UIViewAutoresizingFlexibleLeftMargin
#define FlexibleR                   UIViewAutoresizingFlexibleRightMargin
#define FlexibleH                   UIViewAutoresizingFlexibleHeight
#define FlexibleW                   UIViewAutoresizingFlexibleWidth

#define FixedMarginT                FlexibleW | FlexibleB
#define FixedMarginB                FlexibleW | FlexibleT
#define FixedMarginL                FlexibleH | FlexibleR
#define FixedMarginR                FlexibleH | FlexibleL
#define FixedHorizental             FlexibleW | FlexibleT | FlexibleB
#define FixedVertical               FlexibleH | FlexibleL | FlexibleR
#define FixedALL                    FlexibleW | FlexibleH
#define FixedCenter                 FlexibleL | FlexibleR | FlexibleT | FlexibleB
```

使用上述的宏时,最好对`UIView`扩展一下,添加下列方法

``` objc
- (void) autoResize:(UIViewAutoresizing) type
{
    self.autoresizingMask = mask;
    self.autoresizesSubviews = YES;
}
```

使用方法如下

``` objc

@implementation XXXViewController

- (id)init
{
    self = [super init];
    if (self) {
        // Custom initialization
        
        self.tableView = [[UITableView alloc] initWithFrame:self.view.bounds style:UITableViewStyleGrouped];
        **[self.tableView autoResize:FixedALL];**    
        self.tableView.delegate = self;
        [self.view addSubview:self.tableView];
        
        ...
        ...
    }
    return self;
}

```

关于各个宏的作用如下

||              宏             ||          含义        ||
||:--------------------------:||:--------------------:||
|| FixedMarginB               ||      下侧距离固定      ||
|| FixedMarginL               ||      左侧距离固定      ||
|| FixedMarginR               ||      右侧距离固定      ||
|| FixedHorizental            ||      左右距离固定      ||
|| FixedVertical              ||      上下距离固定      ||
|| FixedALL                   ||      四周距离固定      ||
|| FixedCenter                ||      居中             ||

从这张图上我们可以看到对于各个值的含义,出自[stackoverflow](http://stackoverflow.com/questions/7754851/autoresizing-masks-programmatically-vs-interfact-builder-xib-nib)

![值定义](https://dl.dropboxusercontent.com/u/433937/Blog/2013-12-10-ge-ren-zong-jie-de-%5B%3F%5D-xie-app-de-dai-ma-shi-jian1.png)


至今为止这套宏在我的开发过程中还使用得很好,所以我也没有去研究新的`AutoLayout`(好像也比较的复杂,),不过在`Github`上有个对`AutoLayout`封装得很好的库[Masonry](https://github.com/cloudkite/Masonry),有空的时候可以研究一下




一些常用的Category
---------------------------------------


``` objc


///////////////////////////////////////
//  UIColor
///////////////////////////////////////

//通过RGBA值(比如红色FF0000FF)生成UIColor
+ (UIColor* ) colorWithHex:(int)color {
    
    float red = (color & 0xff000000) >> 24;
    float green = (color & 0x00ff0000) >> 16;
    float blue = (color & 0x0000ff00) >> 8;
    float alpha = (color & 0x000000ff);
    
    return [UIColor colorWithRed:red/255.0 green:green/255.0 blue:blue/255.0 alpha:alpha/255.0];
}




///////////////////////////////////////
//  UIImage
///////////////////////////////////////

//复制当前图片
- (UIImage *)duplicate
{
    CGImageRef newCgIm = CGImageCreateCopy(self.CGImage);
    UIImage *newImage = [UIImage imageWithCGImage:newCgIm scale:self.scale orientation:self.imageOrientation];
    CGImageRelease(newCgIm);
    
    return newImage;
}

//使当前图片可拉伸
- (UIImage *)stretched
{
    CGSize size = self.size;
    
    UIEdgeInsets insets = UIEdgeInsetsMake(truncf(size.height-1)/2, truncf(size.width-1)/2, truncf(size.height-1)/2, truncf(size.width-1)/2);
    
    return [self resizableImageWithCapInsets:insets];
}

//使当前图片抗锯齿(当图片在旋转时有用, 原理就是在图片周围加1px的透明像素)
- (UIImage *)antiAlias
{
    int border = 1;
    CGRect rect = CGRectMake(border, border, self.size.width-2*border, self.size.height-2*border);
    
    UIImage *img = nil;
    
    UIGraphicsBeginImageContext(CGSizeMake(rect.size.width,rect.size.height));
    [self drawInRect:CGRectMake(-1, -1, self.size.width, self.size.height)];
    img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    UIGraphicsBeginImageContext(self.size);
    [img drawInRect:rect];
    UIImage* antiImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return antiImage;
}

//创建纯色的图片
+ (UIImage *)imageWithColor:(UIColor *)color Size:(CGSize)size
{
    CGRect rect = CGRectMake(0.0f, 0.0f, size.width, size.height);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return [image stretched];
}

//imageNamed的非缓存版
+ (UIImage *)imageName:(NSString *)name
{
    NSString *path = [[[NSBundle mainBundle] bundlePath] stringByAppendingString:[NSString stringWithFormat:@"/%@",name]];
    
    return [UIImage imageWithContentsOfFile:path];
}




///////////////////////////////////////
//  UIButton
///////////////////////////////////////

- (float)x
{
    return self.frame.origin.x;
}
- (float)y
{
    return self.frame.origin.y;
}
- (float)w
{
    return self.frame.size.width;
}
- (float)h
{
    return self.frame.size.height;
}

- (void)setTagName:(NSString*)name
{
    self.tag = [name hash];
}

- (UIView *)viewWithName:(NSString *)name
{
    return [self viewWithTag:[name hash]];
}

- (void) autoResize:(UIViewAutoresizing)mask
{
    self.autoresizingMask = mask;
    self.autoresizesSubviews = YES;
}

- (void)setPosition:(CGRect)position
{
    self.bounds = CGRectMake(0, 0, position.size.width, position.size.height);
    self.center = CGPointMake(position.origin.x,position.origin.y);
}




///////////////////////////////////////
//  UIButton
///////////////////////////////////////

+ (UIButton*) buttonWithTarget:(id)target action:(SEL)sel
{
    UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
    [btn addTarget:target action:sel forControlEvents:UIControlEventTouchUpInside];
    return btn;
}

- (void) setTextN:(NSString*)n H:(NSString*)h D:(NSString*)d S:(NSString *)s
{
    if ( n )
    {
        [self setTitle:n forState:UIControlStateNormal];
    }
    
    if ( h )
    {
        [self setTitle:h forState:UIControlStateHighlighted];
    }
    
    if ( d )
    {
        [self setTitle:d forState:UIControlStateDisabled];
    }
    
    if ( s )
    {
        [self setTitle:s forState:UIControlStateSelected];
    }
}

- (void) setImageN:(NSString*)n H:(NSString*)h D:(NSString*)d S:(NSString *)s
{
    if ( n )
    {
        [self setImage:[UIImage imageName:n] forState:UIControlStateNormal];
    }
    
    if ( h )
    {
        [self setImage:[UIImage imageName:h] forState:UIControlStateHighlighted];
    }
    
    if ( d )
    {
        [self setImage:[UIImage imageName:d] forState:UIControlStateDisabled];
    }
    
    if ( s )
    {
        [self setImage:[UIImage imageName:s] forState:UIControlStateSelected];
        
        if ( h )
        {
            [self setImage:[UIImage imageName:h] forState:UIControlStateHighlighted | UIControlStateSelected];
        }
        
    }
}




///////////////////////////////////////
//  NSString
///////////////////////////////////////

-(NSString *) trimHead
{
    NSInteger i;
    NSCharacterSet *cs = [NSCharacterSet whitespaceAndNewlineCharacterSet];
    for(i = 0; i < [self length]; i++)
    {
        if ( ![cs characterIsMember: [self characterAtIndex: i]] ) break;
    }
    return [self substringFromIndex: i];
}

-(NSString *) trimTail
{
    NSInteger i;
    NSCharacterSet *cs = [NSCharacterSet whitespaceAndNewlineCharacterSet];
    for(i = [self length] -1; i >= 0; i--)
    {
        if ( ![cs characterIsMember: [self characterAtIndex: i]] ) break;    
    }
    return [self substringToIndex: (i+1)];
}

- (NSString *) trimBoth
{
    return [[self trimHead] trimTail];
}

- (NSString*)MD5
{
    const char *ptr = [self UTF8String];
    
    unsigned char md5Buffer[CC_MD5_DIGEST_LENGTH];
    
    CC_MD5(ptr, strlen(ptr), md5Buffer);
    
    NSMutableString *output = [NSMutableString stringWithCapacity:CC_MD5_DIGEST_LENGTH * 2];
    for(int i = 0; i < CC_MD5_DIGEST_LENGTH; i++) 
        [output appendFormat:@"%02x",md5Buffer[i]];
    
    return output;
}

- (BOOL)equals:(NSString *)str
{
    return [self compare:str] == NSOrderedSame;
}

- (CGFloat)heightByFont:(UIFont *)font width:(CGFloat)width
{
    return [self sizeWithFont:font
            constrainedToSize:CGSizeMake(width, CGFLOAT_MAX)
                lineBreakMode:NSLineBreakByWordWrapping].height;
}





///////////////////////////////////////
//  NSDictionary
///////////////////////////////////////

//使取得的值不会使NSNull,在JSON解析的时候会有这种问题,有时候服务器返回了 {"test":null},但是被解析成了NSNull,这时APP这边处理起来会比较麻烦
- (id)objectForKeyNotNull:(id)key {
    id object = [self objectForKey:key];
    if (object == [NSNull null])
        return nil;
    
    return object;
}




///////////////////////////////////////
//  NSAttributedString
///////////////////////////////////////

- (CGFloat)heightByWidth:(CGFloat)width
{
    CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((__bridge CFAttributedStringRef)self);
    CGSize targetSize = CGSizeMake(width, CGFLOAT_MAX);
    CGSize fitSize = CTFramesetterSuggestFrameSizeWithConstraints(framesetter, CFRangeMake(0, [self length]), NULL, targetSize, NULL);
    CFRelease(framesetter);
    
    return fitSize.height;
}


```
