---
layout: post
title: 'Object-C 學習筆記 - UIView(2):Tab Bar,Navigation View'
date: 2015-06-22 05:49
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
XCODE : 6.3.2
github : https://github.com/conscientiousness/demoUIView.git

## Tab Bar
##### <font color="blue">1.Tab Bar Item圖示規格 : </font>
iOS Devloper Library -> iOS Human Interface Guidelines ->UI Elements -> Icon and Image Sizes  (http://ppt.cc/JTnrs)

##### <font color="blue">2.設定badge :</font>
(1)由StoryBoard
<img class="" src="http://i.imgur.com/TlwPh9b.png" width="50%">
(2)由程式碼

```objc
self.tabBarItem.badgeValue = @"早安";
```

##### <font color="blue">3.分頁切換控制</font>

(1)View Controller加入protocol
`<UITabBarControllerDelegate>`

(2)加入此method

```objc
- (BOOL)tabBarController:(UITabBarController *)tabBarController
shouldSelectViewController:(UIViewController *)viewController {
	//此View不允許切換
  return NO;
}
```
## Navigation
##### <font color="blue">1.Navigation Item設定 : </font>
(1)由Storyboard:
放UI元件到Navigation Bar上方左右兩邊，需先把Navigation Item放上去，且元件會包含在Bar Button Item內。
<img class="left" src="http://i.imgur.com/sZUMeCn.png" width="40%">
<img class="" src="http://i.imgur.com/WVREn61.png" width="50%">
(2)由程式碼生成UIBarButtonItem:
```objc
UIBarButtonItem *left1=[[UIBarButtonItem alloc] initWithTitle:@"left1" style:UIBarButtonItemStylePlain target:self action:@selector(barButtonPressed:)];

UIBarButtonItem *left2=[[UIBarButtonItem alloc] initWithTitle:@"left2" style:UIBarButtonItemStylePlain target:self action:@selector(barButtonPressed:)];

//多個
self.navigationItem.leftBarButtonItems = @[left1,left2];
```
[註記]
- style中有兩個方法，在iOS7 UI扁平化後樣式都一樣
- @selector:把某個方法(method)包裝成變數，以上為例，點下bar button item後會觸發本身(target:self)View Controller的barButtonPressed:這個方法

##### <font color="blue">2.Title View設定 : </font>
(1)由程式碼改變Title View內容:這邊把titleViewf內容取代為Indicator為例
```java
//Add Indicator to TitleView
UIActivityIndicatorView *indicatorView = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleWhiteLarge];
//顏色
indicatorView.color = [UIColor redColor];
//讓Indicator轉動
[indicatorView startAnimating];
//取代navigation titleView位置
self.navigationItem.titleView = indicatorView;
```
[補充]自定義UI元件在View Controller內位置
```java
CGRect frame = indicatorView.frame;
//frame.size.width=取得indicatorView原本的寬
indicatorView.frame = CGRectMake(100, 100, frame.size.width, frame.size.height);
[self.view addSubview:indicatorView];
```

##### <font color="blue">3.Tool Bar設定 : </font>
```objc
//show ToolBar
[self.navigationController setToolbarHidden:NO animated:YES];
UIBarButtonItem *tool1 = [[UIBarButtonItem alloc] initWithBarButtonSystem:UIBarButtonSystemCamera target:self action:@selector(toolBarButtonPressed:)];
[self.navigationController setToolbarHidden:NO animated:YES];
UIBarButtonItem *tool2 = [[UIBarButtonItem alloc] initWithBarButtonSystem:UIBarButtonSystemTrash target:self action:@selector(toolBarButtonPressed:)];
[self.navigationController setToolbarHidden:NO animated:YES];
UIBarButtonItem *tool3 = [[UIBarButtonItem alloc] initWithBarButtonSystem:UIBarButtonSystemSearch target:self action:@selector(toolBarButtonPressed:)];
[self setToolbarItems:@[tool1,tool2,tool3] animated:YES];
```

[說明]
- 會在畫面下方`貼齊靠左`顯示三個系統自帶的icon
- 如果要顯示如下方所示，可以在tool2左右插入`UIBarButtonSystemItemFlexibleSpace`

<img class="" src="http://i.imgur.com/rPFPWug.png" width="50%">

```objc
UIBarButtonItem *flexiableSpace = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace target:nil action:nil];
[self setToolbarItems:@[tool1,flexiableSpace,tool2,flexiableSpace,tool3] animated:YES];
```

[補充]
- 用UIBarButtonSystemItemfixedSpace可自訂間距

##### <font color="blue">4.畫面跳轉 : </font>
(1)跳至另一個View Controller(XIB)...舊
```objc
ChildViewController *viewController = [[childViewontroller alloc] initWithNibName:@"ChildViewController" bundle:nil];
//指定下層View Controller標題文字
viewController.title=@"Assigned Child Title";
[self.navigationController pushViewController.viewController animated:YES];
```
(2)使用stroyboard，顯示一個View Controller，需指定Segue identifier
```java
[self performSegueWithIdentifier:@"identifier..." sender:nil]
```
(3)返回Root View Controller(最前面)
```java
[self.navigationController popToRootViewControllerAnimated:YES];
```
##### <font color="blue">5.其他補充 : </font>
(1)顯示/隱藏 Navigation Bar
```objc
[self.navigationController setNavigationBarHidden:YES animated:YES];
```
(2)XCODE中Navigation Bar說明:By default, UINavigationBar displays a back button on the left and a title in the center,back button文字會預設為前一個View Controller的title,假如A->B，在A ViewController由下方code可修改文字顯示,
```objc
self.navigationItem.backBarButtonItem =[[UIBarButtonItem alloc] initWithTitle:@"我要換這個" style:UIBarButtonItemStylePlain target:nil action:nil];
```
(3)承(2)如要在左上方多一UIBarButtonItem,可透過`leftItemsSupplementBackButton`來設定，或者於左上角或右上角都可加入超過一個UIBarButtonItem,透過`leftBarButtonItems`或`rightBarButtonItems`設定。

