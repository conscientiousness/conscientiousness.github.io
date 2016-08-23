---
layout: post
title: 'Object-C 學習筆記 - Status bar 顏色變換'
date: 2016-04-20 08:41
comments: true
categories: 
---
1. info.plist中的`View controller-based status bar appearance`要設為`YES`
2. 如要意隱藏NavigationBar要使用`self.navigationController.navigationBarHidden = YES`才可以
3. ViewController中
```objc
- (UIStatusBarStyle)preferredStatusBarStyle
{
		return (whiteStatusBar)?UIStatusBarStyleLightContent:UIStatusBarStyleDefault;
}
```
4. `setNeedsStatusBarAppearanceUpdate`會觸發`preferredStatusBarStyle`，也可以這樣實現特效
```objc
[UIView animateWithDuration:0.3 animations:^{
            [self setNeedsStatusBarAppearanceUpdate];
}];
```