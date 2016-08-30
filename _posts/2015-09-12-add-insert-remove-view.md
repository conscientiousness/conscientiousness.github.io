---
layout: post
title: 'Object-C 學習筆記 - 幾種add/remove/insert View的用法'
date: 2015-09-12 15:25
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---

最常用就addsubview
但還有很多種方法可以新增在指定位置

####Managing the View Hierarchy
```objc
addSubview:
bringSubviewToFront:
sendSubviewToBack:
removeFromSuperview
insertSubview:atIndex:
insertSubview:aboveSubview:
insertSubview:belowSubview:
exchangeSubviewAtIndex:withSubviewAtIndex:
```

範例：將我們想新增的View/Layer新增在指定的View/Layer下一層
```objc
[self.view insertSubview:blurView belowSubview:_infoView];
[self.view.layer insertSublayer:avPlayerLayer below:self.videoView.layer];
```


Reference:
http://stackoverflow.com/questions/10051514/how-can-i-insert-a-subview-below-the-other-subviews
https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html

各種view的玩法，想要玩一些有的沒的變化，應該要把他看完，在找時間把UIView Class Reference啃完 Orzzz