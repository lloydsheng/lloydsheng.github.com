---
layout: post
title: "iOS导航自定义"
description: ""
category: 
tags: [导航]
---

相信很多iOS开发者都会遇到导航自定义的问题，大部分应用都会做自定义，比如改背景颜色，贴个背景图片，改高度等（虽然苹果不建议改），我们的应用在这个问题上先后使用过2种方案。

第一种方案：对UINavigationBar做扩展，使用Object-C的Category特性。覆盖以下方法:

~~~~
// 自定导航高度
- (void)layoutSubviews
{
    [super layoutSubviews];
    if (_barHeight) {
        self.height = _barHeight;
        for (UIView *view in self.subviews) {
            //            DLog(@"DTNavigationBar height = %f", self.height);
            view.center = CGPointMake( view.left + view.width/2.0f, _barHeight/2.0f);
        }
    }
}

// 自定义导航背景
- (void)drawRect:(CGRect)rect
{
    [super drawRect:rect];
    if (_barBackImage) {
        [_barBackImage drawInRect:rect];
    }
}
~~~~

这种方法挖了一个很大的坑，带来了很多问题：

1. 由于是全局覆盖，导致很多系统提供的controller的导航样式也被修改了，从而导致发邮件，发短信的界面各种难看。
2. 兼容性不是很好，在系统升级iOS6后，导航按钮彻底乱了。

后来采取了第2种方案，将系统的导航隐藏，在基类BaseViewController里面添加添加一个自己的导航：

~~~~
if (!_navigationBar) {
    _navigationBar = [[DTNavigationBar alloc] initWithFrame:CGRectMake(0, 0, self.view.width, 44)];
    _navigationBar.backgroudImage = [UIImage imageNamed:@"navigationbar_bg.png"];
    [self.view addSubview:_navigationBar];
}
~~~~

然后在每次添加其他subview的时候对其height和top重新计算。

~~~~
- (void)viewWillLayoutSubviews
{
    [super viewWillLayoutSubviews];
    if (self.navigationBar != nil) {
        [self.view bringSubviewToFront:self.navigationBar];
        for (UIView *view in self.view.subviews) {
            if (![view isKindOfClass:[DTNavigationBar class]]) {
                // 避免重复布局
                if (![_layoutedViews objectForKey:[NSValue valueWithNonretainedObject:view]]) {
                    [_layoutedViews setObject:[NSNumber numberWithBool:YES]
                                       forKey:[NSValue valueWithNonretainedObject:view]];
                    if (view.top < self.navigationBar.height) {
                        view.top += self.navigationBar.height;
                        view.height -= self.navigationBar.height;
                    }
                }
            }
        }
    }
}
~~~~

这种方案，完美解决了之前的问题，就是需要自己重新实现一个navigationbar的功能，但问题不大，这都是一次性的工作。


{% include JB/setup %}
