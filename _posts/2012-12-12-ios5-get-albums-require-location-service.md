---
layout: post
title: "iOS6以下版本获取用户相册需要允许定位服务"
description: ""
category: 
tags: []
---
最近应用添加了一个新功能：将用户保存的图片放置到单独的文件夹。上线后发现很多用户反馈保存图片提示失败。查一下这个问题，发现保存图片分了2步：

1. 保存图片到相册
2. 将图片转移到自定义相册（如果没有自定义相册则新建）

```
-(void)saveImage:(UIImage*)image
         toAlbum:(NSString*)albumName
withCompletionBlock:(SaveImageCompletion)completionBlock
{
    //write the image data to the assets library (camera roll)
    [self writeImageToSavedPhotosAlbum:image.CGImage
                           orientation:(ALAssetOrientation)image.imageOrientation
                       completionBlock:^(NSURL* assetURL, NSError* error) {
                           
                           //error handling
                           if (error!=nil) {
                               completionBlock(error);
                               return;
                           }
                           
                           //add the asset to the custom photo album
                           [self addAssetURL: assetURL
                                     toAlbum:albumName
                         withCompletionBlock:completionBlock];
                           
                       }];
}
```


{% include JB/setup %}
