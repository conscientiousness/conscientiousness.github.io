---
layout:     post
title:      "Core Bluetooth學習筆記"
subtitle:   "Practical CoreBluetooth for Peripherals"
date:       2016-09-22
author:     "Jesse"
header-img: "img/in-post/2016-09-27-ios-corebluetooth/header-bg-image.jpg"
catalog:    true
tags:
    - iOS
    - BLE
    - Objc
    - Swift
---

### 前言

最近有空來複習一下Core Bluetooth, Demo Code:

- [Objective-C](https://github.com/conscientiousness/BLEPractice)
- <del>Swift 3.0</del> 尚未完成
- Using Xcode 8

### 基本觀念

#### 1. 什麼是BLE - Bluetooth low energy?

> It’s sort of like Bluetooth. We all know Bluetooth, we all use it in speakers and so on, but the difference is this is a very low powered protocol. Typically, one battery can last a month or even years depending on how it is used. That allows us to do things we normally couldn’t do with Bluetooth. The standard is called Bluetooth 4.0, it started with something called “Smart Bluetooth” and evolved to this one, there’s a 200 page manual you can read before you go to sleep, it’s a gripping read. BLE is very cheap in terms of power use and the protocol is not too complex

#### 2. 架構

###### <2.1 App透過Core Bluetooth去控制BLE底層>

<img src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBTechnologyFramework_2x.png" style="width: 250px;"/>

###### <2.2 周邊與裝置傳輸對應示意圖>

<img src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBDevices1_2x.png" style="width: 550px;"/>

###### <2.3 Peripherals Advertising>

1. Peripherals broadcast some of the data they have in the form of advertising packets. An advertising packet is a relatively small bundle of data that may contain useful information about what a peripheral has to offer.
2. A central can ask to connect to any peripheral that it has discovered advertising.

<img src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/AdvertisingAndDiscovery_2x.png" style="width: 500px;"/>

###### <2.4 Peripheral>

Each peripheral has services, it can have as many services as you want, and each one of these services has characteristics.

<img src="/img/in-post/2016-09-27-ios-corebluetooth/peripheral-structure.png" style="width: 300px;"/>

###### <2.5 Peripheral - Server>

In general you have a service, then this service holds many characteristics, and each characteristic holds a type, value, properties and so on. You don’t have to know the entire thing to work with CoreBluetooth, the most important thing is the reading, this is what we’re actually trying to get and manipulate and change, whatever you need to do, we want this reading and we want to know what we can do with it.

<img src="/img/in-post/2016-09-27-ios-corebluetooth/peripheral-structure-detail.png" style="width: 450px;"/>

###### <2.5 Background>

1. 可被BLE硬體喚醒.
2. 可於App在背景時繼續掃描, 資料傳輸.
3. Apple對於背景執行的省電有額外的規範要求.
4. Background Mode Setting:

> Uses Bluetooth LE accessories - Central Mode

> Acts as a Bluetooth LE accessories - Peripheral Mode

<img src="/img/in-post/2016-09-27-ios-corebluetooth/background-mode-setting.png" style="width: 450px;"/>

**參考**: [Core Bluetooth Background Processing for iOS Apps](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html)

### Build a Bluetooth Chat App

> **目標:**
> 1. Peripheral Mode: 成為Chat Room Server, 提供其他裝置連入聊天.
> 2. Central Mode: 掃描Peripheral並尋找提供聊天室功能的Peripheral, 連入聊天.

#### 1. Central Mode

- 建立`CentralModeTableViewController`.
- 宣告Protocal `CBCentralManagerDelegate, CBPeripheralDelegate`.
- We initialize by calling a new manager, with delegate, queue and options from `[[CBCentralManager alloc] initWithDelegate:self queue:nil]`.

```objc
- (void)centralManagerDidUpdateState:(CBCentralManager *)central {

    CBManagerState state = central.state;

    if (state != CBManagerStatePoweredOn) {
        NSString *message = [NSString stringWithFormat:@"BLE is not available.(error: %ld)",state];
        NSLog(@"%@",message);
    }

}
```
- ↑ 實作centralManagerDidUpdateState, 用來監聽Central狀態, 除了poweredOn, 其餘都是error.


```objc
- (void) startToScan {

    // 指定特定Service UUID, 沒有也可傳nil
    NSArray *servies = @[];

    // Scan時, 是否允許相同UUID裝置同時出現
    NSDictionary *options = @{CBCentralManagerScanOptionAllowDuplicatesKey:@YES};
    [centerManger scanForPeripheralsWithServices:servies options:options];
}
```
- ↑ 在TableviewController上, 新增一個Switch, 用來控制掃描開關, 並實作開始與停止控制, 當執行`scanForPeripheralsWithServices`將會call`didDiscoverPeripheral`delegate method.


```objc
@interface DiscoveredPeripheral : NSObject

@property (nonatomic, strong) CBPeripheral *peripheral;
@property (nonatomic, assign) NSInteger lastRSSI;
@property (nonatomic, strong) NSDate *lastSeenDateTime;

@end
```
- ↑ 建立處理Peripheral的Model.

```objc
- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *,id> *)advertisementData RSSI:(NSNumber *)RSSI {

    NSString *uuid = peripheral.identifier.UUIDString;
    BOOL isExist = allDiscovered[uuid] != nil;

    if(isExist == NO) {
        NSLog(@"Found: %@(%@), RSSI: %ld",peripheral.identifier,peripheral.name, (long)[RSSI integerValue]);
    }

    DiscoveredPeripheral *item = [DiscoveredPeripheral new];
    item.peripheral = peripheral;
    item.lastRSSI = [RSSI integerValue];
    item.lastSeenDateTime = [NSDate date];
    [allDiscovered setObject:item forKey:uuid];

    [self.tableView reloadData];
}
```
- ↑ 將scan到的peripheral, 存到dictionary中, 並將資料顯示在tableview上, peripheral的UUID是Central manager自動分配.

```objc
- (void) tableView:(UITableView *)tableView accessoryButtonTappedForRowWithIndexPath:(NSIndexPath *)indexPath {

    [self connectWithIndexPath:indexPath];
}

- (void) connectWithIndexPath:(NSIndexPath *)indexPath {

    NSArray *allkeys = allDiscovered.allKeys;
    DiscoveredPeripheral *target = allkeys[indexPath.row];
    [centerManger connectPeripheral:target.peripheral options:nil];
    detailInfoString = [NSMutableString new];
}
```
- ↑ 點選cell accessory btn時, 連接peripheral並顯示資訊, 透過實作這兩個delegate: didConnectPeripheral,didFailToConnectPeripheral.

```objc
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {

    NSLog(@"Peripheral Connect: %@",peripheral.name);

    [self stopScanning];

    peripheral.delegate = self;
    [resetServices removeAllObjects];
    // 尋找Serevices
    [peripheral discoverServices:nil];
}

- (void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(nullable NSError *)error {

    NSLog(@"Fail to Connect Peripheral: %@",error.description);
}

- (void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(nonnull CBPeripheral *)peripheral error:(nullable NSError *)error {

    NSLog(@"Disconnected: %@",peripheral.name);
    [self startToScan];
}
```
- ↑ 實作處理連上Peripheral, Error handler, 斷線處理的delegate

```objc
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(nullable NSError *)error {

    if(error) {
        NSLog(@"didDiscoverServices error: %@",error.description);
        [centerManger cancelPeripheralConnection:peripheral];
        [self startToScan];
        return;
    }

    NSArray *allServices = peripheral.services;
    [resetServices addObjectsFromArray:allServices];

    // Pick the first one to discover characteristic
    // discoverCharacteristics 觸發 didDiscoverCharacteristicsForService
    [peripheral discoverCharacteristics:nil forService:[resetServices firstObject]];
    [resetServices removeObjectAtIndex:0];
}
```
- ↑ 尋找到Peripheral的Services, 再去找Service中的Characteristics

```objc
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(nullable NSError *)error {

    if(error) {
        NSLog(@"didDiscoverCharacteristics error: %@",error.description);
        [centerManger cancelPeripheralConnection:peripheral];
        [self startToScan];
        return;
    }

    [detailInfoString appendFormat:@"*** Peripheral: %@ (%ld services)\n",peripheral.name,peripheral.services.count];
    [detailInfoString appendFormat:@"** Service: %@ (%ld characteristic)\n",service.UUID.UUIDString,service.characteristics.count];

    for(CBCharacteristic *cbc in service.characteristics) {
        [detailInfoString appendFormat:@"* Characteristic: %@\n",cbc.UUID.UUIDString];
    }

    if(resetServices.count == 0) {
        // show result
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Result" message:detailInfoString preferredStyle:UIAlertControllerStyleAlert];
        UIAlertAction *ok = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil];
        [alert addAction:ok];
        [self presentViewController:alert animated:YES completion:^{
            [centerManger cancelPeripheralConnection:peripheral];
            [self startToScan];
            detailInfoString = nil;
        }];

    } else {
        [detailInfoString appendString:@"\n"];
        [peripheral discoverCharacteristics:nil forService:[resetServices firstObject]];
        [resetServices removeObjectAtIndex:0];
    }
}
```
- ↑ 尋找到Service中的Characteristics並把結果顯示出來, 原則就是連線時停止掃描, 有error就`cancelPeripheralConnection`且重新開啟掃描.

> **流程:** 連接Peripheral -> 取得所有service(用array存service) -> 利用service取得Characteristic(利用service的array)

- 第一階段完成DEMO

<img src="/img/in-post/2016-09-27-ios-corebluetooth/show-info.png" style="width: 400px;"/>

#### 2. Chat Room in Central Mode

- 建立TalkingViewController, 可以輸入與觀看訊息.

```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    _talkingPeripheral.delegate = self;
    // 開啟Characteristic的NotifyValue, Central將可以收到通知, 觸發didUpdateValueForDescriptor
    [_talkingPeripheral setNotifyValue:YES forCharacteristic:_talkingCharacteristic];
}
```
- ↑ 指派目前Peripheral的delegate並且開啟Notify

```objc
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(nonnull CBCharacteristic *)characteristic error:(nullable NSError *)error {

    // MARK: - Step 20
    NSString *content = [[NSString alloc] initWithData:characteristic.value encoding:NSUTF8StringEncoding];
    NSLog(@"Receive from Peripheral: %@",content);
    // End Step 20
}
```
- ↑ NotifyValue

```objc
- (BOOL) textFieldShouldReturn:(UITextField *)textField {

    [textField resignFirstResponder];

    if(textField.text.length > 0) {
        NSString *context = [NSString stringWithFormat:@"[iPhone6] %@\n",textField.text];
        NSData *data = [context dataUsingEncoding:NSUTF8StringEncoding];

        [_talkingPeripheral writeValue:data forCharacteristic:_talkingCharacteristic type:CBCharacteristicWriteWithResponse];
    }
    return NO;
}
```
- ↑ 送出

> CBCharacteristicWriteWithResponse/CBCharacteristicWriteWithoutResponse: 設定Peripheral收到是否有Response

```objc
- (void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(nullable NSError *)error {
    if(error != nil) {
        NSLog(@"didWriteValueForCharacteristic error: %@",error.description);
    }
}
```
- ↑ Response

#### 3. Chat Room in Peripheral Mode

> **注意**: Peripheral Mode下iOS10要在plist新增權限:
NSBluetoothPeripheralUsageDescription


- 使裝置提供Chat Room Service
- 一樣先拉出聊天室的layout
- 初始化PeripheralManager, `[[CBPeripheralManager alloc] initWithDelegate:self queue:nil]`

```objc
- (IBAction)switchValueChanged:(id)sender {

    if([sender isOn]) {
        CBUUID *uuid = [CBUUID UUIDWithString:kServiceUUID];
        NSArray *uuids = @[uuid];

        // CBAdvertisementDataServiceDataKey的value必須要用array
        NSDictionary *info = @{CBAdvertisementDataServiceDataKey: uuids,
                               CBAdvertisementDataLocalNameKey: kPeripheralName};
        [peripheralManager startAdvertising:info];

    } else {
        [peripheralManager stopAdvertising];
    }
}
```
- ↑ 用switch控制peripheral開關Advertising

```objc
- (void)peripheralManagerDidUpdateState:(CBPeripheralManager *)peripheral {

    CBManagerState state = peripheral.state;

    if(state != CBManagerStatePoweredOn) {
        NSLog(@"peripheralManagerDidUpdateState error = %ld",state);
    } else {
        CBUUID *uuidService = [CBUUID UUIDWithString:kServiceUUID];
        CBUUID *uuidCharacteristic = [CBUUID UUIDWithString:kCharacteristicUUID];

        CBCharacteristicProperties porperties = CBCharacteristicPropertyWrite | CBCharacteristicPropertyRead | CBCharacteristicPropertyNotify;
        CBAttributePermissions permissions = CBAttributePermissionsReadable | CBAttributePermissionsWriteable;

        chatCharacteristic = [[CBMutableCharacteristic alloc] initWithType:uuidCharacteristic properties:porperties value:nil permissions:permissions];

        CBMutableService *chatService = [[CBMutableService alloc] initWithType:uuidService primary:YES];
        chatService.characteristics = @[chatCharacteristic];

        [peripheralManager addService:chatService];
    }
}
```
- ↑ 如狀態為poweredOn時新增Service到peripheralManager

```objc
- (void)peripheralManager:(CBPeripheralManager *)peripheral central:(CBCentral *)central didSubscribeToCharacteristic:(CBCharacteristic *)characteristic {

    NSString *hello = [NSString stringWithFormat:@"[%@] Welcome ! Here is %@, (Total:%ld, Max Length:%ld)",kPeripheralName,kPeripheralName,chatCharacteristic.subscribedCentrals.count,central.maximumUpdateValueLength];

    [self sendText:hello central:central];

    _logTextView.text = [NSString stringWithFormat:@"%@%@",hello,_logTextView.text];
}
```
- ↑ 當Central notify這個Characteristic, 此方法會觸發, 當有central連入, 發送歡迎訊息

```objc
- (void)sendText:(NSString *)text central:(CBCentral *)central {
    NSData *data = [text dataUsingEncoding:NSUTF8StringEncoding];

    if(central == nil) {
        [peripheralManager updateValue:data forCharacteristic:chatCharacteristic onSubscribedCentrals:nil];
    } else {
        [peripheralManager updateValue:data forCharacteristic:chatCharacteristic onSubscribedCentrals:@[central]];
    }
}
```
- ↑ 發送訊息共用Method, 無傳入Central就發給所有訂閱者

- (BOOL)textFieldShouldReturn:(UITextField *)textField {

    [textField resignFirstResponder];

    if(textField.text.length > 0) {
        NSString *content = [NSString stringWithFormat:@"[%@] %@\n",kPeripheralName,textField.text];
        [self sendText:content central:nil];

        _logTextView.text = [NSString stringWithFormat:@"%@%@",content,_logTextView.text];
    }

    return NO;
}
- ↑ 發送訊息

```objc
- (void) peripheralManager:(CBPeripheralManager *)peripheral didReceiveWriteRequests:(nonnull NSArray<CBATTRequest *> *)requests {

    // MARK: - Step 29
    for(CBATTRequest *tmp in requests) {

        // Tell Central it is received.
        [peripheralManager respondToRequest:tmp withResult:CBATTErrorSuccess];

        // Show on UI and forward to all Centrals
        NSString *content = [[NSString alloc] initWithData:tmp.value encoding:NSUTF8StringEncoding];

        if(content != nil) {

            // 發給其他user
            [self sendText:content central:nil];
            _logTextView.text = [NSString stringWithFormat:@"%@%@",content,_logTextView];
        }
    }
    // End Step 29
}
```
- ↑ （1) 當Central writeValue to Characteristic:`[_talkingPeripheral writeValue:data forCharacteristic:_talkingCharacteristic type:CBCharacteristicWriteWithResponse]` 會觸發`didReceiveWriteRequests`方法.
<br>
(2) 回應requset並將訊息發送給所有訂閱者.

### Demo

<iframe width="560" height="315" src="https://www.youtube.com/embed/a2EWzPlwx20" frameborder="0" allowfullscreen></iframe>

- 00:00 - 00:21 : Central Mode
- 00:22 - 00:50 : Enter Chat Room in Central Mode
- 00:51 : Create Chat Room in Peripheral Mode

### Reference

- [Realm News - corebluetooth](https://realm.io/news/yoav-schwartz-corebluetooth-peripherals/)
- [Apple Core Bluetooth Programming Guide](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1)




