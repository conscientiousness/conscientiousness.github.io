---
layout: post
title: 'Object-C 學習筆記 - Status bar 動態變更顏色'
date: 2016-04-20 08:41
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---

info.plist中的`View controller-based status bar appearance`要設為`YES`。

如要意隱藏NavigationBar要使用`self.navigationController.navigationBarHidden = YES`才可以。

```objc
- (UIStatusBarStyle)preferredStatusBarStyle {
		return (whiteStatusBar)?UIStatusBarStyleLightContent:UIStatusBarStyleDefault;
}
```

`setNeedsStatusBarAppearanceUpdate`會觸發`preferredStatusBarStyle`，也可以這樣實現特效。

```objc
[UIView animateWithDuration:0.3 animations:^{
            [self setNeedsStatusBarAppearanceUpdate];
}];
```