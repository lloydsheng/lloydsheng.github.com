---
layout: post
title: "QuartzCore内存泄露假象"
description: ""
category: 
tags: 内存泄露 QuartzCore MemoryLeak 性能优化
---
{% include JB/setup %}

最近一直在优化应用，用的是XCode自带的Instruments工具。在优化一个有很多UILabel的页面到时候发现没创建一次页面，内存就占用就增加几百K，这个在手机应用上是非常致命的内存泄露。从图上可以看出来，增加的内存几乎都被QuartzCore占用了。

[![图1](/images/Snip20130108_8.png)](/images/Snip20130108_8.png)

[![图2](/images/Snip20130108_4.png)](/images/Snip20130108_4.png)

这应该是内存泄露的假象，在内存非常充足的情况下，QuartzCore的缓存貌似不会及时释放，直到内存占用某个到阀值，模拟器下面大概是11M左右，不同的设备可能会有一定的差异。不了解QuartzCore内存机制的内部实现，网上的资料也很少，只能自己分析。

[![图2](/images/Snip20130108_10.png)](/images/Snip20130108_10.png)

另外，在做性能优化的时候，应该以真机为准，由于资源和环境有很大差异，模拟器很难模拟真实环境。


####相关参考####
* [http://stackoverflow.com/questions/5892551/iphone-objective-c-memory-leaks-in-quartzcore-library](http://stackoverflow.com/questions/5892551/iphone-objective-c-memory-leaks-in-quartzcore-library)

