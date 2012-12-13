---

layout: post
title: "iOS6以下版本操作用户相册需要允许定位服务"
description: ""
category: 
tags: iOS兼容性

---

最近应用添加了一个新功能：将用户保存的图片放置到单独的相册。上线后发现很多用户反馈保存图片提示失败。查了一下，发现是在操作相册的时候出了问题：

在新建相册的时候，需要先遍历收集相册，看看同名相册是否存在。

~~~~
[self enumerateGroupsWithTypes:ALAssetsGroupAlbum
                        usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
                        }];
~~~~

同名相册不存在则新建相册：

~~~~
[self addAssetsGroupAlbumWithName:albumName
                      resultBlock:^(ALAssetsGroup *group) {
                      } failureBlock: completionBlock];
~~~~

而在iOS6以下系统，如果用户没有授权应用访问定位信息，调用这些相册操作会提示错误：

~~~~
Error Domain=ALAssetsLibraryErrorDomain Code=-3300 "Write failed" UserInfo=0xdfb4b80 {NSLocalizedFailureReason=There was a problem writing this asset because the write failed., NSUnderlyingError=0xdfc8960 "The operation couldn’t be completed. (PersistentURLTranslator error 13.)", NSLocalizedDescription=Write failed}
~~~~

###总结####
开始收到反馈时候，怀疑是用户容量满了，或者图片有问题。怎么也不会怀疑和定位服务有关，苹果这个api权限要求也不合理，新建相册为什么一定要用定位服务，相册的地理信息不能为空吗？可能苹果后来也觉得没有必要，才在iOS6取消了这个限制。

{% include JB/setup %}
