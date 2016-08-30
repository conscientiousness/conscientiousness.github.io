---
layout: post
title: 'viewController切換由程式碼產生三種方法'
date: 2015-07-09 15:30
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---


iOS 使用者最常看到兩種切換 ViewControllers 的 Animations：

由下至上：例如輸入 Wi-Fi 密碼時，會有一個由下至上的動畫，出現一個輸入密碼的畫面
由右至左：例如設定 => 一般，會有一個由右至左的動畫，出現一些屬於一般的設定，並且在左上角有回去的按鈕
基本上，前者是用 presentViewController: 實作，後者是用 pushViewController: 實作
這兩個 methods 到底有何差異？
最大差異在於語義上的不同
presentViewController: 暗示使用者需要完成某件事情，輸入密碼、增加資料等等，使用者必須完成或者取消，才能做其他事
pushViewController: 則是讓使用者瀏覽資料，使用者可以決定要前進或回到哪個頁面

了解了這兩個 methods，那 addChildViewController: 又為何物？
addChildViewController: 是 iOS SDK 5+ 之後有的 method
透過它，開發者可以寫出客製化的 NavigationController，
更精準地說，讓 ViewController 變成一個 containner，維持 view hierarchy，使得 app 的呈現更加多樣化


主要分為三種切換 ViewControllers 的 Animations：

1.presentViewController
由下至上：例如輸入 Wi-Fi 密碼時，會有一個由下至上的動畫，出現一個輸入密碼的畫面
PresentViewController *firstviewController = [self.storyboard instantiateViewControllerWithIdentifier:@"PresentViewController"];
[self presentViewController:firstviewController animated:YES completion:nil];

2.pushViewController(Navigation Controller)：可前進或回到哪個頁面
由右至左：例如設定 -> 一般，會有一個由右至左的動畫，出現一些屬於一般的設定，並且在左上角有回去的按鈕，類似巢狀的感覺
PushViewController * firstviewController = [self.storyboard instantiateViewControllerWithIdentifier:@"PushViewController"];
[self.navigationController pushViewController: firstviewController animated:YES];


3.self.view addSubView:view 、self.window addSubView
特別要注意的是，這個方法只是把新頁面加掛上當前頁面
如果再用 self.navigationControler，pushViewController或 presentViewController 會無效。