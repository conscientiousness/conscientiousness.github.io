---
layout: post
title: 'Object-C 學習筆記 - UIView(3):Table View'
date: 2015-06-23 03:03
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
XCODE : 6.3.2
github : https://github.com/conscientiousness/demoUIView.git

##### <font color="blue">1.StoryBoard設定Static Content簡略介紹 : </font>
(1) 點在Table View:attribute inspector => content 設定動態

(2) 點在Cell:attribute inspector => accessory cell右方會顯示箭頭等圖示

(3) 點在Cell右鍵:show 可指定點擊cell時跳轉到指定的View Controller

##### <font color="blue">2.Dynamic prototypes 實作新增,移動,刪除Cell: </font>
(1) 先建立一個繼承於`UITableViewController`的Class

(2) 別忘了StoryBoard的Table View Controller中Custom Class要指定到自己建立的Class

(3) 就會預設`<UITableViewDelegate, UITableViewDataSource>`這些protocal，就可以用所屬的method來操作Table View,(UITableViewDelegate操作行為相關,UITableViewDataSource顯示內容相關)

(4) 開始程式碼的部分

<img class="" src="http://i.imgur.com/1nmtCVX.png" width="30%">

[4.1] 用Array來存儲資料,先宣告`NSMutableArray *datas`,顯示的row為array個數,只用1個section
```Objective-C Nav3TableViewController.m
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    // Return the number of sections.
    return 1;
}
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return datas.count ;
}
@end
```
[4.2] viewDidLoad時，建立相關按鈕
```java
- (void)viewDidLoad {
    [super viewDidLoad];
    self.navigationItem.rightBarButtonItem = self.editButtonItem;//右上方建立內建edit按鈕
    datas = [NSMutableArray new];
    //Add Button Support
    UIBarButtonItem *addButton = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemAdd target:self action:@selector(addBtnPressed)];
    self.navigationItem.leftBarButtonItem = addButton;//左上方建立新增按鈕，點擊會執行自定義的Method
}
```
[4.3] 建立點擊"+"會呼叫的Method
- 可參考:http://blog.changyy.org/2012/08/ios-uitableviewcontroller-datasource.html
- [注意] 要先更新`datas`在更新UI=>[self.tableView insertRowsAtIndexPaths...
```java
- (void) addBtnPressed {
    //將每次新增的資料放在array第一個位置
    [datas insertObject:[NSDate date] atIndex:0];
    //Insert into TableView 的位置
    NSIndexPath *insertIdexPath = [NSIndexPath indexPathForRow:0 inSection:0];
    //插入項目到指定位置(insertIdexPath)
    [self.tableView insertRowsAtIndexPaths:@[insertIdexPath] withRowAnimation:UITableViewRowAnimationAutomatic];
}
```
[4.4] 設定cellForRowAtIndexPath
```java
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    //storyboard tableView 要設定 Identifier的名稱 = myCell
    //dequeueReusableCellWithIdentifier 負責處理cell資源運用
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"myCell" forIndexPath:indexPath];

    // Configure the cell...
    NSData *targetItem = datas[indexPath.row];
    cell.textLabel.text = targetItem.description;
    return cell;
}
```
[4.5] 設定是否可編輯的權限，目前設定都可編輯return YES
```java
- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath {
    // Return NO if you do not want the specified item to be editable.
    //設定第三個cell不可編輯，點選edit時cell最前面不會出現編輯圖示
    //if (indexPath.row == 2)
        //return NO;
    //else
        return YES;
}
```
[4.6] 點選刪除前會到以下Method
```java
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath {
    if (editingStyle == UITableViewCellEditingStyleDelete) {
        // Delete the row from the data source
        /刪除array裡的資料
        [datas removeObjectAtIndex:indexPath.row];
        //更新UI
        [tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
    } else if (editingStyle == UITableViewCellEditingStyleInsert) {
        // Create a new instance of the appropriate class, insert it into the array, and add a new row to the table view
    }
}
```
[4.7] 支援移動Cell，也要更新array內的資料
```java
- (void)tableView:(UITableView *)tableView moveRowAtIndexPath:(NSIndexPath *)fromIndexPath toIndexPath:(NSIndexPath *)toIndexPath {
    //更新陣列內容
    //取移動cell row位置的資料=[datas objectAtIndex:fromIndexPath.row]
    id tmpItem = datas[fromIndexPath.row];
    //刪除該位置
    [datas removeObjectAtIndex:fromIndexPath.row];
    //插入到移動後的位置
    [datas insertObject:tmpItem atIndex:toIndexPath.row];
}
// Override to support conditional rearranging of the table view.
- (BOOL)tableView:(UITableView *)tableView canMoveRowAtIndexPath:(NSIndexPath *)indexPath {
    // Return NO if you do not want the item to be re-orderable.
    return YES;
}
```

##### <font color="blue">3.Custom Cell: </font>
(1)建立繼承於UITableViewCell的class,並且在stroyboard設定custom class,這邊命名為Nav4CustomTableViewCell
(2)在Table View Controller的class import剛才建立的.h檔
(3)定義屬於Nav4CustomTableViewCell的Cell物件，這樣就可以控制自定義的Cell
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    Nav4CustomTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"nav4CustomCell" forIndexPath:indexPath];

    // Configure the cell...
    ...
    return cell;
}

```