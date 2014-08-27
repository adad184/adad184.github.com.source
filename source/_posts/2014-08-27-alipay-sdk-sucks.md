title: 支付宝iOS SDK的那些坑(系统繁忙，请稍后再试)
date: 2014-08-27 16:39:45
tags:
categories: 技巧心得
---

前言
========
支付宝的iOS SDK真是个奇葩的存在 按理说这么重要的SDK 理应提供详尽的文档和技术支持(虽然说实际使用较简单) 
但是跑到[官方论坛](http://club.alipay.com/thread-htm-fid-703-page-6.html)一看 都是各种各样的问题 以及千篇一律的解答

这里不谈如何继承 只谈问题 如果你按照官方文档一步一步的调试发现任何问题没有 那么 恭喜你 都会抢答了

如果你跟我一样 遇到跳转支付时现实"系统繁忙,请稍后再试" 的问题(如图 没截到iOS的 找了一张android的图凑数)

![问题截图](https://dl.dropboxusercontent.com/u/433937/Blog/2014-08-27-alipay-sdk-sucks.png)


问题
========
官方示例中支付相关的代码

``` objc
//构造订单
NSString *appScheme = @"AlipaySdkDemo";
NSString* orderInfo = [self getOrderInfo:indexPath.row];
NSString* signedStr = [self doRsa:orderInfo];

//形成订单参数
NSString *orderString = [NSString stringWithFormat:@"%@&sign=\"%@\"&sign_type=\"%@\"",
                         orderInfo, signedStr, @"RSA"];

//调用支付借口
[AlixLibService payOrder:orderString AndScheme:appScheme seletor:_result target:self];

```

问题就出在`orderInfo`这里 那么`orderInfo`是什么呢?

``` objc

-(NSString*)getOrderInfo:(NSInteger)index
{
    /*
     *点击获取prodcut实例并初始化订单信息
     */
    Product *product = [_products objectAtIndex:index];    
    AlixPayOrder *order = [[AlixPayOrder alloc] init];
    order.partner = PartnerID;
    order.seller = SellerID;

    order.tradeNO = [self generateTradeNO]; //订单ID（由商家自行制定）
    order.productName = product.subject; //商品标题
    order.productDescription = product.body; //商品描述
    order.amount = [NSString stringWithFormat:@"%.2f",product.price]; //商品价格
    order.notifyURL =  @"http%3A%2F%2Fwwww.xxx.com"; //回调URL
    
    return [order description];
}

```

可以到看`orderInfo`就是`AlixPayOrder`的字符串化 再看看其description函数
``` objc

- (NSString *)description {
    NSMutableString * discription = [NSMutableString string];
    [discription appendFormat:@"partner=\"%@\"", self.partner ? self.partner : @""];
    [discription appendFormat:@"&seller_id=\"%@\"", self.seller ? self.seller : @""];
    [discription appendFormat:@"&out_trade_no=\"%@\"", self.tradeNO ? self.tradeNO : @""];
    [discription appendFormat:@"&subject=\"%@\"", self.productName ? self.productName : @""];
    [discription appendFormat:@"&body=\"%@\"", self.productDescription ? self.productDescription : @""];
    [discription appendFormat:@"&total_fee=\"%@\"", self.amount ? self.amount : @""];
    [discription appendFormat:@"&notify_url=\"%@\"", self.notifyURL ? self.notifyURL : @""];
    
    [discription appendFormat:@"&service=\"%@\"", self.serviceName ? self.serviceName : @"mobile.securitypay.pay"];
    [discription appendFormat:@"&_input_charset=\"%@\"", self.inputCharset ? self.inputCharset : @"utf-8"];
    [discription appendFormat:@"&payment_type=\"%@\"", self.paymentType ? self.paymentType : @"1"];

    //下面的这些参数，如果没有必要（value为空），则无需添加
    [discription appendFormat:@"&return_url=\"%@\"", self.returnUrl ? self.returnUrl : @"www.xxx.com"];
    [discription appendFormat:@"&it_b_pay=\"%@\"", self.itBPay ? self.itBPay : @"1d"];
    [discription appendFormat:@"&show_url=\"%@\"", self.showUrl ? self.showUrl : @"www.xxx.com"];

    
    for (NSString * key in [self.extraParams allKeys]) {
        [discription appendFormat:@"&%@=\"%@\"", key, [self.extraParams objectForKey:key]];
    }
    return discription;
}
```

我猜问题可能出现在这里
``` objc

    [discription appendFormat:@"&return_url=\"%@\"", self.returnUrl ? self.returnUrl : @"www.xxx.com"];
    [discription appendFormat:@"&it_b_pay=\"%@\"", self.itBPay ? self.itBPay : @"1d"];
    [discription appendFormat:@"&show_url=\"%@\"", self.showUrl ? self.showUrl : @"www.xxx.com"];

```

可能是这种莫名其妙的默认值导致了问题的出现(代码里注释了无需添加 可为啥官方demo是没有问题的?) 
`orderInfo`也不过是一堆参数的拼凑而已 同时 我求证了同事在android的sdk里 参数也都是手工拼凑的 那么将必填参数自行组织一下 应该就可以了

方案
========

经过实践 只需要填下下面出现的这些参数 就没有问题了

``` objc

-(NSString*)getOrderInfo
{
    
    NSMutableString * discription = [NSMutableString string] ;
    [discription appendFormat:@"partner=\"%@\"", PartnerID];
    [discription appendFormat:@"&seller_id=\"%@\"", SellerID];
    [discription appendFormat:@"&out_trade_no=\"%@\"", [self getOrderID]];
    [discription appendFormat:@"&subject=\"%@\"", yourSubject];
    [discription appendFormat:@"&body=\"%@\"", yourBody];
    [discription appendFormat:@"&total_fee=\"%@\"", yourPrice];
    [discription appendFormat:@"&notify_url=\"%@\"", yourHandleAction];
    [discription appendFormat:@"&service=\"%@\"", @"mobile.securitypay.pay"];
    
    return discription;
}

```

尾声
========

当然 回过头来看 问题有这么几个
* 官方demo写得不严谨 且注释太少
* 官方文档写没有指出相关问题 以及对相应参数的解释(对于参数,仅提到了一句"**支付参数提交时,需要组装订单信息 orderInfo,其中 参数以 key=”value”形式呈现,参数之间以“&”分割, 获取 Alipay 支付对象调用支付。**"")
* 官方论坛没有相应的技术支持

结论就是: 坑爹

