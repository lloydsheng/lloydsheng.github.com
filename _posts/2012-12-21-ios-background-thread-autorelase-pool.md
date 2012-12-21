---
layout: post
title: "iOS4.3下后台线程执行autolease警告"
description: ""
category: 
tags: autorelase iOS
---
{% include JB/setup %}

最近在兼容iOS4.3的时候发现Console疯狂打印警告，5.0以上的系统没有出现过：

~~~~
2012-12-21 16:36:46.933 Sauron[499:790f] *** __NSAutoreleaseNoPool(): Object 0x33e0d0 of class NSPathStore2 autoreleased with no pool in place - just leaking
2012-12-21 16:36:46.989 Sauron[499:790f] *** __NSAutoreleaseNoPool(): Object 0x3d2c50 of class NSFileAttributes autoreleased with no pool in place - just leaking
2012-12-21 16:36:47.001 Sauron[499:790f] *** __NSAutoreleaseNoPool(): Object 0x3adac0 of class NSPathStore2 autoreleased with no pool in place - just leaking
...
~~~~

Debug发现是在一段清理缓存的代码导致的，这段每次调用autolease都会导致一次警告。

~~~~
[NSThread detachNewThreadSelector:@selector(oncheckAndClearDisk) toTarget:self withObject:nil];

// 检测缓存大小然后清除
- (void)checkAndClearDisk
{
    [NSThread detachNewThreadSelector:@selector(oncheckAndClearDisk) toTarget:self withObject:nil];
}

- (void)oncheckAndClearDisk
{
    NSDate *start = [NSDate date];
    unsigned long long int totalSize = [self folderSize:diskCachePath];
    DLog(@"get cache size time = %f", [[NSDate date] timeIntervalSince1970] - [start timeIntervalSince1970]);
    DLog(@"cache size = %lli", totalSize/1024/1024);
    if (totalSize/1024/1024 >= kMaxCacheSize) {
        NSDate *start = [NSDate date];
        [self clearDisk];
        DLog(@"clear cache size time = %f", [[NSDate date] timeIntervalSinceDate:start]);
    }
}
~~~~

从苹果的官方文档了解到：每个线程都要自己维护autorlease pool，也就是说你创建的线程中如果用了autorelease则需要维护一个autorelase pool。方法很简单用@autoreleasepool { }包裹线程要执行代码就可以了。上面的代码加上@autoreleasepool后问题解决。

~~~~
- (void)oncheckAndClearDisk
{
	@autoreleasepool {
		NSDate *start = [NSDate date];
		...
	}
}
~~~~

###相关参考####
* [https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html)
* [http://stackoverflow.com/questions/4313122/nsautoreleasenopool-object-0x753c2f0-of-class-general-autoreleased-with-no](http://stackoverflow.com/questions/4313122/nsautoreleasenopool-object-0x753c2f0-of-class-general-autoreleased-with-no)
