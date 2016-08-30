---
layout: post
title: 'Object-C 學習筆記 - UIWebView'
date: 2015-06-14 13:51
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
XCODE : 6.3.2
github : https://github.com/conscientiousness/demoUIWebView


# 目標 :
APP內網頁瀏覽及操作

# 畫面 :
![螢幕快照 2015-06-14 下午10.13.06.png](http://user-image.logdown.io/user/12628/blog/11894/post/280325/OpG7cHDSBOzJ8S9RqHDQ_%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%202015-06-14%20%E4%B8%8B%E5%8D%8810.13.06.png)

# 說明 :
- 完成UI關聯
```java
@interface ViewController () <UITextFieldDelegate,UIWebViewDelegate>
@property (weak, nonatomic) IBOutlet UITextField *urlTextField;
@property (weak, nonatomic) IBOutlet UIButton *backbutton;
@property (weak, nonatomic) IBOutlet UIButton *nextButton;
@property (weak, nonatomic) IBOutlet UIButton *stopButton;
@property (weak, nonatomic) IBOutlet UIButton *refreshButton;
@property (weak, nonatomic) IBOutlet UIActivityIndicatorView *loadindView;
@property (weak, nonatomic) IBOutlet UIWebView *webView;
@end
```

- 取得UITextField內的網址，點選GO後執行

```java
- (IBAction)goButtonPress:(id)sender {
    NSString *urlString = _urlTextField.text;
    //轉換為NSURL格式:有可能的格式為http,https,file,telnet,mailto...
    NSURL *url = [NSURL URLWithString:urlString];
    //把url丟給request : HTTPGet的動作
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [_webView loadRequest:request];
}
```
- 如要預設直接載入網址，可以在viewdidload加上
```java
//在view執行時就觸發goButtonPress
[self goButtonPress:nil];
```

- UITextField對於鍵盤的顯示修改

![螢幕快照 2015-06-14 下午10.39.08.png](http://user-image.logdown.io/user/12628/blog/11894/post/280325/Bt0B2t9RkCR2IiWqlapa_%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%202015-06-14%20%E4%B8%8B%E5%8D%8810.39.08.png)

- 在storyboard中設定UITextField的delegate指向ViewController

![螢幕快照 2015-06-14 下午11.12.14.png](http://user-image.logdown.io/user/12628/blog/11894/post/280325/IO5nB4U6TFKd1LLg2SID_%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%202015-06-14%20%E4%B8%8B%E5%8D%8811.12.14.png)


- 宣告在@interface ViewController ()後面 `<UITextFieldDelegate,UIWebViewDelegate>`
此UITextFieldDelegate,UIWebViewDelegate為內建的protocol方法，內容大致如下。
```java
@protocol UITextFieldDelegate <NSObject>
@optional
- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField;        // return NO to disallow editing.
- (void)textFieldDidBeginEditing:(UITextField *)textField;           // became first responder
- (BOOL)textFieldShouldEndEditing:(UITextField *)textField;          // return YES to allow editing to stop and to resign first responder status. NO to disallow the editing session to end
- (void)textFieldDidEndEditing:(UITextField *)textField;             // may be called if forced even if shouldEndEditing returns NO (e.g. view removed from window) or endEditing:YES called
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string;   // return NO to not change text
- (BOOL)textFieldShouldClear:(UITextField *)textField;               // called when clear button pressed. return NO to ignore (no notifications)
- (BOOL)textFieldShouldReturn:(UITextField *)textField;              // called when 'return' key pressed. return NO to ignore.
@end
```

- Objective-C 只有`單一繼承`，對於繼承自不同`父類別`的`類別`(class)來說，如果要使用共同的`方法`(methos)，就需要使用 `Protocol`，但 Protocol 本身並不會實作，只會作方法的宣告。實作的部份，由類別自己來作。

- 接下來，實作此UITextFieldDelegate的方法，再按下Return時會執行，前面已改成顯示`go`

```java
- (BOOL) textFieldShouldReturn:(UITextField *)textField {
    //關閉鍵盤
    [textField resignFirstResponder];
    //呼叫goButtonPress method
    [self goButtonPress:nil];
    return NO;
}
```

- storyboard上設定讓瀏覽器達到上一頁,下一頁,停止讀取,重新整理的目標

![螢幕快照 2015-06-14 下午11.28.43.png](http://user-image.logdown.io/user/12628/blog/11894/post/280325/cakxhILGQGScu0oGcg30_%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%202015-06-14%20%E4%B8%8B%E5%8D%8811.28.43.png)

- 然後實作UIWebViewDelegate內的方法，ViewController可以得到瀏覽器讀取運作情形，藉此控制indicator此取名為loadindView

```java
//回報資訊給viewControler
- (void) webViewDidStartLoad:(UIWebView *)webView {
    [_loadindView startAnimating];
}
- (void) webViewDidFinishLoad:(UIWebView *)webView {
    [_loadindView stopAnimating];
}
- (void) webview:(UIWebView *)webView didFailLoadWithError:(NSError *)error{
    [_loadindView stopAnimating];
}
```

- 最後這個method可以知道，我們URL跳轉的流程，可以藉此來控管是否要跳轉網址

```java
- (BOOL) webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    NSString *urlString = [request.URL absoluteString];
    NSLog(@"URL String: %@",urlString);
    return YES;
}
```

- 來測試一下，執行http://google.com/會出現以下
```
URL String: http://google.com/
URL String: http://www.google.com.tw/?gfe_rd=cr&ei=nZ99VcnTJJKT9QWU54HABw
URL String: https://www.google.com.tw/?gfe_rd=cr&ei=nZ99VcnTJJKT9QWU54HABw&gws_rd=ssl
```

