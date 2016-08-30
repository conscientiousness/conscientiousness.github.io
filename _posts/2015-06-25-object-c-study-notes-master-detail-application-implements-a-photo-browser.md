---
layout: post
title: 'Object-C 學習筆記 - 用Master-Detail Application實作照片瀏覽功能[未完]'
date: 2015-06-25 08:51
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
XCODE : 6.3.2

<img class="" src="http://i.imgur.com/0PEuOzU.png" width="80%">

##### <font color="blue">1.StoryBoard : </font>
(1)先把Master Navigation後接的View改成collection View Controller
(2)設定Custom class
(3)Master-Collection View Controller中放上Image View與Label
(4)Detail-View Controller放入UIImage View, Slider, Bar Button Item
##### <font color="blue">2.Collection View Controller(Master): </font>

(1)Detail中顯示照片流程

- Collection View中把照片檔名存到名為datas的array
- 點選照片取得所選取位置，如(2)，然後把值丟到detail(記得detail的.h要設定變數才可存取)

(2)
```java
- (void) prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    UINavigationController *detailNavigationContoller = segue.destinationViewController;
    DetailViewController *detailViewController = (DetailViewController*)[detailNavigationContoller topViewController];
    NSArray *selectedItem = self.collectionView.indexPathsForSelectedItems;
    NSIndexPath *targetIndexPath = selectedItem[0];
    detailViewController.targetIndex = targetIndexPath.row;
    detailViewController.datas = datas;
    //handle splitViewController for iPad
    //防止crash 橫直轉換
    //左上角master按鈕設定
    detailViewController.navigationItem.leftBarButtonItem = self.splitViewController.displayModeButtonItem;
    //與自己定義的按鈕可以共存
    detailViewController.navigationItem.leftItemsSupplementBackButton = YES;
}
```

[說明]

- `segue.destinationViewController`只會抓到Detail的Navgation Controller，所以還要在寫(DetailViewController*)[detailNavigationContoller `topViewController`];才可以得到我們想要Detail View Controller。

- collectionView可以做多選，本練習只作單選可以用`indexPathsForSelectedItems`，這個方法會把每個選取到的index path放到array中，再從array取出第一個物件就可以得到我們選到的index paths。

- 因為是Master-Detail Split View Controller的架構，然後因為替換成Collection View Controller需要加上倒數第二行code，iPad螢幕橫直切換時才不會crash(可以看一開始建立好的MasterViewController.m裡面的prepareForSegue)

##### <font color="blue">3.Detail View Controller: </font>
「未完」