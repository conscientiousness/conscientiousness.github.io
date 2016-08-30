---
layout: post
title: 'Object-C 學習筆記 - 字串格式'
date: 2015-06-16 13:25
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
### 格式總覽

apple developer：
https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Strings/Articles/formatSpecifiers.html

### 用法

```java
NSString *engVoc=@"English";
NSLog(@"engVoc=%@",engVoc);

//long
NSInteger i = 42;
printf("%ld\n", (long)i);

//double
SKSpriteNode *ground = [SKSpriteNode spriteNodeWithColor:[UIColor greenColor] size:CGSizeMake(self.scene.frame.size.width, 100)];
NSLog(@"scene=%e,frame=%e",ground.scene.frame.size.width,ground.frame.size.width);
```



