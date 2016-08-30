---
layout: post
title: 'Object-C 學習筆記 - 基本UI動畫呈現:照片輪播、view移動'
date: 2015-06-25 06:07
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
XCODE : 6.3.2
github : https://github.com/conscientiousness/demoUIAnimation

## 實作目標

- 點擊Yellow,上方view以設定動畫向上移出
- 點擊Green,左方view以設定動畫向左移出
- 右下方Image View 輪播方式呈現

<img class="" src="http://i.imgur.com/GB72VKo.png" width="20%">

### <font color="blue">1.Image View輪播 : </font>

```java
NSArray *images = @[[UIImage imageNamed:@"cat1.jpg"],
                       [UIImage imageNamed:@"cat2.jpg"],
                       [UIImage imageNamed:@"cat3.jpg"]];


_theImageView.contentMode = UIViewContentModeScaleAspectFit;//設定照片縮放方式
_theImageView.animationImages = images;
_theImageView.animationRepeatCount = 0;

_theImageView.animationDuration = 3.0;//跑一輪所需時間(s)
[_theImageView startAnimating];
```

[注意]
- 放置圖檔大小所耗用的記憶體
- 無法判斷停下來是在第幾張照片
