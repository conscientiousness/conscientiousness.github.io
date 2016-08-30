---
layout: post
title: 'Object-C 學習筆記 - iOS推播Push '
date: 2015-09-04 08:21
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
此為學習筆記，如有錯誤煩請指正，感恩:)

## 流程

![](http://i.imgur.com/tFRftWP.png)
必須透過APNs發送通知

![](http://i.imgur.com/ybsUBR0.png)
透過每台裝置的deviceToken讓server跟APNs溝通

## 憑證
簡單紀錄一下，第一次用到肝火旺盛，但跑過一次流程就會了

[1]本機CSR檔案
鑰匙圈->從憑證授權要求憑證->輸出CSR

[2]iOS App IDs
Application Services:要開啟Push Notifications

[3]cer匯入本機
從鑰匙圈
Development SSL Certificate
Production SSL Certificate
轉出p12

在轉成pem
`openssl pkcs12 -in path.p12 -out newfile.crt.pem -clcerts -nokeys`
`openssl pkcs12 -in path.p12 -out newfile.key.pem -nocerts -nodes`

## 目標:建立一個有推播功能的即時聊天室

#### <font color="blue">(1) 建一個名為Communicator類別(Model)負責網路通訊傳輸</font>

[1.1]定義主機位置與處理不同狀況的php用字串相連方式處理
```objc
#define BASE_URL @"http://127.0.0.1/PushMessage"
//HTTP POST
//發訊息
#define SENDMESSAGE_URL [BASE_URL stringByAppendingPathComponent:@"sendMessage.php"]
//接收訊息
#define RETRIVEMESSAGES_URL [BASE_URL stringByAppendingPathComponent:@"retriveMessages.php"]
//更新EVICETOKEN
#define UPDATEDEVICETOKEN_URL [BASE_URL stringByAppendingPathComponent:@"updateDeviceToken.php"]

```

[1.2]建立一個Singleton物件，讓APP內可以共用此通訊模組
```objc
static Communicator *_SingletonCommunicator = nil;
+ (instancetype) shared {

    if(_SingletonCommunicator == nil)
    {
        _SingletonCommunicator = [Communicator new];
    }

    return _SingletonCommunicator;
}
```
另一種寫法
```objc
+(instancetype)sharedInstance
{
    static dispatch_once_t token;
    static Communicator *_SingletonCommunicator = nil;
    dispatch_once(&token, ^{
        _SingletonCommunicator = [Communicator new];
    });
    return _SingletonCommunicator;
}
```

[1.3]處理POST方法(以AFNetWorking實作)
```objc
- (void) doPost:(NSString*)url parameters:(NSDictionary*)parameters success:(void (^)(id result)) successBlock fail:(void (^)(NSError *error)) failBlock {

    //先建立AFNetWorking他的Singleton物件
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
    //對應php端 ContentTypes
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"text/html"];

    [manager POST:url parameters:parameters success:^(AFHTTPRequestOperation *operation, id responseObject) {

        NSLog(@"JSON: %@",responseObject);
        successBlock(responseObject);

    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {

        NSLog(@"Error: %@",error.description);
        failBlock(error);

    }];
}
```

[1.4]處理下載訊息的Method
```objc
- (void) downloadMessagesWithCompletion:(void (^)(NSError *error, id result))completion {
		//post 接收訊息的 api 得到json回傳值
    [self doPost:RETRIVEMESSAGES_URL parameters:nil success:^(id result) {
        completion(nil,result);
    } fail:^(NSError *error) {
        completion(error,nil);
    }];
}
```

[1.5]處理送出訊息的Method
```objc
- (void) sendMessage:(NSString*)message userName:(NSString*)userName completion:(void (^)(NSError *, id))completion {


    NSDictionary *jsonObj = @{USER_NAME_KEY:userName,MESSAGE_KEY:message};

    //將dictionary轉為json
    NSData *jsonData = [NSJSONSerialization dataWithJSONObject:jsonObj options:NSJSONWritingPrettyPrinted error:nil];

    //但回傳為 NSData不好操作,所以再轉為 NSString 以utf8作編碼
    NSString *jsonString = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];

		//AFNetWorking 會將dictionary內容輸出為 -> data={"Username":"tester","Message":"aaaa"}
    NSDictionary *parameters = @{DATA_KEY:jsonString};

    [self doPost:SENDMESSAGE_URL parameters:parameters success:^(id result) {

        completion(nil,result);

    } fail:^(NSError *error) {

        completion(error,nil);
    }];
}
```

#### <font color="blue">(2) ViewController</font>

[2.1]完成layout
由上而下,由左而右,姓名textField->發言內容textField->更新訊息button->發送訊息button->所有訊息textView
![](http://i.imgur.com/N7Zc5iP.png)

[2.2]宣告並實作Communicator
```objc
#import "Communicator.h"
@interface ViewController ()
{
    Communicator *communicator;
}
-(void)viewDidLoad {
    [super viewDidLoad];
    communicator = [Communicator shared];
}
```

備註:也可以直接用`[[Communicator shared] xxxx]`爽就好

[2.3]Refresh Button Pressed
```objc
[communicator downloadMessagesWithCompletion:^(NSError *error, id result) {

        if(error)
        {
            NSLog(@"Error: %@",error.description);
        }
        else
        {
            NSLog(@"message: %@",[result description]);

            NSArray *messages = result[MESSAGES_KEY];

            //Revert array 讓最新的在上
            messages = [[messages reverseObjectEnumerator]allObjects];

            for(NSDictionary *tmp in messages)
            {
                NSString *tmpID = tmp[ID_KEY];
                NSString *userName = tmp[USER_NAME_KEY];
                NSString *message = tmp[MESSAGE_KEY];

                //判斷是否有更新的資料(以資料庫累加的id作判斷)，有才會寫到對話紀錄裡
                if([tmpID integerValue]>lastMessageID){
                    NSString *fullString = [NSString stringWithFormat:@"(%@) %@: %@",tmpID
                    ,userName,message];
                    [self appendToLog:fullString];
                }
            }
            // Update lastMessageID
            NSDictionary *lastItem = messages.lastObject;
            lastMessageID = [lastItem[ID_KEY] integerValue];
        }
    }];
```

[2.4]Send Button Pressed
```objc
-(IBAction)sendBtnPressed:(id)sender {
    if(_messageTF.text.length == 0){
        return;
    }
    NSString *message = _messageTF.text;
    NSString *userName = _userNameTF.text;
    [communicator sendMessage:message userName:userName completion:
    ^(NSError *error, id result) {

        if(error)
            NSLog(@"Error: %@",error.description);
        else
            [self refreshBtnPressed:nil];
    }];
}
```

#### <font color="blue">(3) 推播</font>

[3.1]appDelegate didFinishLaunchingWithOptions註冊APNs
```objc
- (void) registerAPNS {
    UIApplication *app = [UIApplication sharedApplication];

    if ([app respondsToSelector:@selector(registerForRemoteNotifications)])
    {
        // iOS 8 and latter
      	//開啟alert,badge,sound
        UIUserNotificationType type = UIUserNotificationTypeAlert |UIUserNotificationTypeBadge | UIUserNotificationTypeSound;

        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:type categories:nil];
        [app registerUserNotificationSettings:settings];
        [app registerForRemoteNotifications];
    }
    else // before iOS8
    {
        UIRemoteNotificationType type = UIRemoteNotificationTypeAlert |UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound;

        [app registerForRemoteNotificationTypes:type];
    }
}
```

[3.2]取得裝置的DeviceToken
```objc appDelegate.m
- (void) application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {

    NSLog(@"APNS Register successfully: %@",deviceToken.description);

    //拆成之後要送給APNs的格式
    NSString *deviceTokenString = [deviceToken description];
    deviceTokenString = [deviceTokenString stringByReplacingOccurrencesOfString:@"<" withString:@""];
    deviceTokenString = [deviceTokenString stringByReplacingOccurrencesOfString:@">" withString:@""];
    deviceTokenString = [deviceTokenString stringByReplacingOccurrencesOfString:@" " withString:@""];

    NSLog(@"deviceTokenString: %@",deviceTokenString);

    //Notify ViewController
    [[NSNotificationCenter defaultCenter] postNotificationName:DEVICETOKEN_RECEIVED_NOTIFICATION object:deviceTokenString];
}
```

[3.3]建立更新deviceToken的Method
```objc Communicator.m
- (void) updateDeviceToken:(NSString*)message userName:(NSString*)userName completion:(void (^)(NSError *, id))completion {

    NSDictionary *jsonObj = @{USER_NAME_KEY:userName,DEVICETOKEN_KEY:message};

    NSData *jsonData = [NSJSONSerialization dataWithJSONObject:jsonObj options:NSJSONWritingPrettyPrinted error:nil];

    NSString *jsonString = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];

    NSDictionary *parameters = @{DATA_KEY:jsonString};

    [self doPost:UPDATEDEVICETOKEN_URL parameters:parameters success:^(id result) {

        completion(nil,result);

    } fail:^(NSError *error) {

        completion(error,nil);
    }];
}
```

[3.4]建立NotficationCenter接收通知，收到通之後更新deviceToken
```objc ViewController.m
[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(didDeviceTokenReceived:) name:DEVICETOKEN_RECEIVED_NOTIFICATION object:nil];

//避免換手機DeviceToken改變無差別更新
-(void) didDeviceTokenReceived:(NSNotification *)notify{
    deviceToken = notify.object;
    [communicator updateDeviceToken:deviceToken userName:_userNameTF.text
    completion:^(NSError *error, id result) {
        if(error)
        {
            NSLog(@"Error:%@",error.description);
        }
        else{
            NSString *content = [NSString stringWithFormat:@"Update deviceToken OK.(%@)"
            ,deviceToken];
            [self appendToLog:content];
        }
    }];
}
```

[3.5]自動更新機制，當有人發訊息自動更新
```objc appDelegate.m
 -(void) application:(UIApplication *)application
  didReceiveRemoteNotification:(NSDictionary *)userInfo {
    NSLog(@"didReceiveRemoteNotification: %@",userInfo);
    if(application.applicationState == UIApplicationStateActive)
    {
        [[NSNotificationCenter defaultCenter] postNotificationName:MESSAGES_SHOULD_REFRESH_NOTIFICATION object:nil];
    }
}
```

```objc ViewController.m
//viewDidLoad加上
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(refreshBtnPressed:) name:MESSAGES_SHOULD_REFRESH_NOTIFICATION object:nil];

-(IBAction)refreshBtnPressed:(id)sender {
    [communicator downloadMessagesWithCompletion:^(NSError *error, id result) {
        if(error)
        {
            NSLog(@"Error: %@",error.description);
        }
        else
        {
            NSLog(@"message: %@",[result description]);
            NSArray *messages = result[MESSAGES_KEY];
            //Revert array
            messages = [[messages reverseObjectEnumerator]allObjects];
            for(NSDictionary *tmp in messages)
            {
                NSString *tmpID = tmp[ID_KEY];
                NSString *userName = tmp[USER_NAME_KEY];
                NSString *message = tmp[MESSAGE_KEY];

                if([tmpID integerValue]>lastMessageID){
                    NSString *fullString = [NSString stringWithFormat:
                    @"(%@) %@: %@",tmpID,userName,message];
                    [self appendToLog:fullString];
                }
            }
            // Update lastMessageID
            NSDictionary *lastItem = messages.lastObject;
            lastMessageID = [lastItem[ID_KEY] integerValue];
        }
    }];
}
```

```objc appDelegate.m
- (void)applicationDidBecomeActive:(UIApplication *)application {
	//當app從背景執行回來時執行更新留言
    [[NSNotificationCenter defaultCenter] postNotificationName:MESSAGES_SHOULD_REFRESH_NOTIFICATION object:nil];
}
```

[3.6]註冊失敗時
```objc appDelegate.m
- (void) application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error {
    NSLog(@"APNS Register Fail: %@",error.description);
}
```

[3.7]可以控制BadgeNumber當更新時歸零
```objc
[UIApplication sharedApplication].applicationIconBadgeNumber = 0;
```