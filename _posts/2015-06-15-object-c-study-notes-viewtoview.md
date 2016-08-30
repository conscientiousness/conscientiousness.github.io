---
layout: post
title: 'Object-C 學習筆記 - UIView(1):Modal'
date: 2015-06-15 05:48
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
XCODE : 6.3.2
github : https://github.com/conscientiousness/demoUIView.git

# 畫面 :

![螢幕快照 2015-06-15 下午1.52.27.png](http://user-image.logdown.io/user/12628/blog/11894/post/280400/1ESDUwRTvqNn7i0luiu2_%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%202015-06-15%20%E4%B8%8B%E5%8D%881.52.27.png)

# 說明 :

### <font color="blue">快速建立view之間前進與返回</font>

1.前進到黃色view，設定trigger segue的action為show

![](http://i.imgur.com/UKomTol.png)

2.返回到白色view，輸入以下

(1)

```java viewController.m
- (IBAction)backToWhite:(UIStoryboardSegue*)segue {
    NSLog(@"backToWhite");
}
```

(2)在storyboard上，黃色view的exit即可選擇

![](http://i.imgur.com/AuuPpzU.png)

⚠ 不可在黃色建立segue到白色view當作返回，這樣view不會destory會一直疊加造成記憶體不足!

### <font color="blue">由程式碼建立黃view返回白view</font>

(1)首先建立一個繼承 UIViewController 的 class 名為 YellowViewController (.m / .h)
(2)storyboard設定黃色view的class為YellowViewController

![](http://i.imgur.com/UqY6MZ3.png)

(3)建立按鈕Touch up inside，輸入以下method即可實現返回白色view

```java YellowViewController.m
- (IBAction)back2BtnPressed:(id)sender {
    [self dismissViewControllerAnimated:YES completion:^{
        NSLog(@"back2BtnPressed");
    }];
}
```

(4)解構子(Destructor):在物件導向程式設計裡是一個方法，當物件的生命週期結束時，它會自動地被呼叫執行。
由此可以觀察到黃色返回到白色view時會進到這個method。

```java YellowViewController.m
 - (void) dealloc {
    NSLog(@"YellowViewController dealloc.");
}
```

(5)先在storyboard設定segue的identifier

![](http://i.imgur.com/Rhb1Q3v.png)

表示會前進identifier為"goGreen"的segue

```java viewController.m
[self performSegueWithIdentifier:@"goGreen" sender:nil];
```

(6)由storyboard設定的segue可由下列method做一些控制(要先設定好segue的identifier)，例如是否有無wifi才可跳轉頁面等等

```java viewController.m
- (BOOL) shouldPerformSegueWithIdentifier:(NSString *)identifier sender:(id)sender {
    //identifier 會是從此view要前往其他view的Segue名稱
    if([identifier isEqualToString:@"goYellow"]){
        return NO;
    }else{
        return YES;
    }
}
```


### <font color="blue">view之間資料傳遞</font>

(1)目標把白色view的程式產生的時間顯示在黃色view的Label
(2)在YellowViewController.m建立Lable關聯，YellowViewController.h要放infoString物件可供其他class存取，然後藉由他要改變Label得值

```java YellowViewController.m
@interface YellowViewController ()
@property (weak, nonatomic) IBOutlet UILabel *infoLabel;
@end
...
- (void)viewDidLoad {
    [super viewDidLoad];
    //白色view會改變該_infoString
    _infoLabel.text = _infoString;
}
```

```java YellowViewController.h
@interface YellowViewController : UIViewController
@property (nonatomic,strong) NSString* infoString;
@end
```

(3)到白色view的viewController.m import YellowViewController.h然後輸入以下method

```java viewController.m
- (void) prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender{
    //判斷是否為goYellow這條segue才執行，避免crash
    if([segue.identifier isEqualToString:@"goYellow"]){
         //第一種寫法
         //取得目標viewController
         YellowViewController *target =segue.destinationViewController;
         target.infoString = [[NSDate date] description];
        //第二種寫法
        ///id target = segue.destinationViewController;
        //target setValue:[[NSDate date] description]
                  //forKey:@"infoString"];
    }
}
```

簡略說明 :
- 當prepareForSegue做完之後才會跳到黃色的view
- 時間如要顯示台灣時間須作轉換(+8)
- 第二種寫法的key相當於property的意思，所以infoString的value就是[[NSDate date] description]，不import YellowViewController.h 也可以work
