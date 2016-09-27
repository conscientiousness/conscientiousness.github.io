---
layout:     post
title:      "Core Bluetooth學習筆記"
subtitle:   "Practical CoreBluetooth for Peripherals"
date:       2016-09-22
author:     "Jesse"
header-img: "img/in-post/2016-09-22-ios10-stickers/header-image.jpg"
catalog:    true
tags:
    - iOS
    - BLE
    - Objc
    - Swift
---

### 基本觀念

#### 1. 什麼是BLE - Bluetooth low energy?

> It’s sort of like Bluetooth. We all know Bluetooth, we all use it in speakers and so on, but the difference is this is a very low powered protocol. Typically, one battery can last a month or even years depending on how it is used. That allows us to do things we normally couldn’t do with Bluetooth. The standard is called Bluetooth 4.0, it started with something called “Smart Bluetooth” and evolved to this one, there’s a 200 page manual you can read before you go to sleep, it’s a gripping read. BLE is very cheap in terms of power use and the protocol is not too complex

#### 2. Core Bluetooth 架構

- App透過Core Bluetooth去控制BLE底層

<img src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBTechnologyFramework_2x.png" style="width: 250px;"/>

- 周邊與裝置傳輸對應示意圖

<img src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBDevices1_2x.png" style="width: 550px;"/>

- Centrals Discover and Connect to Peripherals That Are Advertising
1. Peripherals broadcast some of the data they have in the form of advertising packets. An advertising packet is a relatively small bundle of data that may contain useful information about what a peripheral has to offer.
2. A central can ask to connect to any peripheral that it has discovered advertising.

<img src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/AdvertisingAndDiscovery_2x.png" style="width: 500px;"/>



#### 1. 基本裝備

> - Xcode
> - 開發者帳號

所以您需要有一台MacBook或含有macOS的相關產品來開啟Xcode, 且要繳[99 USD](https://developer.apple.com/support/compare-memberships/)給Apple當上架保護費/per year, 如果沒有認識的開發者或公司帳號, 對純粹只想上架貼圖的設計師是一筆不小開銷, 而且還要用開發工具來build貼圖App, 看到這邊應該就想左轉離開了吧xd

#### 2. 貼圖如何設計

- 請先詳讀[iOS Human Interface Guidelines - Messaging](https://developer.apple.com/ios/human-interface-guidelines/extensions/messaging/), 這邊詳細定義貼圖Size, 大小, format.
- Apple還順便推薦(賣)了Motion來製作貼圖, 如有購買請看這篇: [Creating Stickers with Motion](https://developer.apple.com/support/stickers/motion/).
- 最後，[Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)這邊定義審核機制, 我用關鍵字`sticker`搜尋, 條例：2.3.4, 4.4.3有相關, 基本上就是可以說明清楚你的貼圖用法,不要違法等.

### 先來個囉唆的設定

上架貼圖就跟開發一款APP上架的流程幾乎是一樣的，所以您必須經過Xcode build APP 之後上傳到itunes connect送審, 可以完全不需要寫code, 但在這之前需先申請id, 憑證等等, 才能使用Xcode完成這些動作.

#### 1. 開發者帳號申請

先到[開發者網站](https://developer.apple.com/)點選右上Account, 使用apple id登入後, 左邊Menu點選Membership可以看到帳戶的狀態, 應該會看到繳交App保護費給Apple的選項, 最下面勾選Auto-Renew Membership可以每年自動繳交保護費.
<img src="/img/in-post/2016-09-22-ios10-stickers/dev-membership-screen.jpg" style="width: 700px;"/>

#### 2. APP ID 申請

- 點選左側Certificates, IDs & Profiles, 在點選左測App IDs, 之後點選右上角`+`新增App id.
<img src="/img/in-post/2016-09-22-ios10-stickers/app-ids-screen.jpg" style="width: 700px;"/>
- App ID Description: Name是用來描述你的App Id 只要符合他的規定可以填你想要的, 之後會顯示在列表中.
- App ID Suffix: 選擇Explicit App ID, Bundle ID會跟Xcode中設定有關且在APP Store中不會與別人重複, 就用apple範例來取, 如果有重複他會提示, 像我的貼圖專案名稱叫blueSticker, 我就用com.jesse.blueSticker
- App Services: 不用管他, 貼圖App完全用不到.
- 點選Continue, 然後就會在list看到你建立的ID拉.
<img src="/img/in-post/2016-09-22-ios10-stickers/app-id-create-screen.jpg" style="width: 500px;"/>

#### 3. 建立憑證(Certificates)

###### <3.1 本機端>

- 打開keychain(鑰匙圈), 直接用spotlight搜尋最快, 就是個鑰匙圖案的App in Lunchpad > 其他
- 點選左上角, 如下圖, 輸出certSigningRequest(CSR)檔案
![](/img/in-post/2016-09-22-ios10-stickers/keychain-setting.png)
- 選擇您的email, 輸入名稱, CA email不用輸入, 選擇儲存到硬碟.
- 這邊的檔案是稍候在開發者網站要上傳的.

###### <3.2 Certificates>

- 回到[開發者網站](https://developer.apple.com/)點選Certificates > All, 點選右上角`+`新增Certificate
- 這邊有好多選項, 把會用到的列出來:

> **Development**
> - iOS App Development: iPhone測試使用, 可以透過Xcode安裝App到你的iPhone.

> **Production**
> - App Store and Ad Hoc: 就是上架要用的憑證拉, 而Ad Hoc憑證可以在上架之前, 讓Devices中已設定id的手機安裝.

所以我們要建立兩個憑證 for Development and Production, 步驟幾乎一樣.

<img src="/img/in-post/2016-09-22-ios10-stickers/certificate-kind.jpg" style="width: 500px;"/>

- iOS App Development
1. 選擇`iOS App Development`, 點選Continue這邊是教你怎麼輸入CSR, 剛剛已經建好了.
2. 再點選Continue會看到Choose File, 選擇剛剛輸出的CSR檔案, 之後點選Generate.
3. 點選Download存到電腦中, 點選檔案兩次, 會自動加到keychain中.

- App Store and Ad Hoc
1. 選擇`App Store and Ad Hoc`, 其餘步驟如上.
2. 記得點選檔案兩次, 會閃一下, 就會加到keychain中了.

<img src="/img/in-post/2016-09-22-ios10-stickers/keychain-cer-list.jpg" style="width: 850px;"/>

###### <3.3 Devices>

這邊設定哪些手機可以透過Xcode或Ad Hoc安裝App

- 如果之前已經用過Xcode
- 一樣點選Devices > All 後點選`+`
- Name依你喜好輸入, 之後會顯示在list
- UUID可以打開itunes查詢, 框選的地方點選一次就會顯示UUID

<img src="/img/in-post/2016-09-22-ios10-stickers/search-uuid-from-itunes.jpg" style="width: 700px;"/>


###### <3.4 Provisioning Profile>

這個的用途就是他會打包Certificates, Device IDs, App ID的描述文件, 所以只要上述有變更, Provisioning Profile也要記得更新呀, 這邊一樣也要作兩次囉 for 開發和上架, 步驟一樣, 只要選`iOS App Development` 和 `App Store`

- 點選Provisioning Profile > All, 點選右上角`+`, 選iOS App Development或App Store, 點選Continue.
- Select App ID: 選擇之前建好的App ID, 點選Continue.
- Select certificates: 選擇之前建好的憑證, 點選Continue.
- Select devices: 選擇可以實機測試的裝置.
- 最後可以不用download, Xcode可以自動下載, 或下載後點兩下也可.

### 開始製作貼圖App拉

#### 1. 建立專案

- 開啟Xcode8, 選擇Create a new Xcode project.
- 選擇Sticer Pack Application, click Next.

<img src="/img/in-post/2016-09-22-ios10-stickers/proj-kind.png" style="width: 500px;"/>

- Bundle Identifier這邊要跟剛剛APP ID的一樣, 修改Product Name and Orgnization Identifier, Bundle Identifier會跟著連動.
- 選擇儲存專案的位置.

#### 2. 配置圖檔

- 接下來是要準備不同大小的icon, 點選左邊Stickers.xcstickers > iMessage App Icon.
- 大小上面都有寫, 例如29x29要放2x,3x, 意思就是要放2倍=58x58, 3倍=87x87的圖, 如果有用Sketch就export選擇2x,3x囉, 之後有空在寫Sketch的script可以直接將所有大小輸出.
- 然後在Sticker Pack 放入你的貼圖, 順序跟顯示貼圖的順序是一樣的.

<img src="/img/in-post/2016-09-22-ios10-stickers/import-sticker-icon.png" style="width: 600px;"/>

#### 3. 測試看看貼圖效果

- 點選Xcode上方Product > Build (  <kbd>⌘+B</kbd>  ), 看有沒有錯誤.
- 點選Generic iOS Device選擇你要Run的Decice

<img src="/img/in-post/2016-09-22-ios10-stickers/choose-device.png" style="width: 300px;"/>
<img src="/img/in-post/2016-09-22-ios10-stickers/device-list.png" style="width: 300px;"/>

- 選完之後點選Xcode上方Product > Run (  <kbd>⌘+R</kbd>  ), Simulator就會開啟iMessage和你貼圖清單, 有兩個帳號可以互相傳送測試囉.

<img src="/img/in-post/2016-09-22-ios10-stickers/ipad-screen-shot.png" style="width: 500px;"/>

### 準備上架囉

#### 1. 建立新App

- [iTunes Connect](https://itunesconnect.apple.com): 提供開發處理上架App的後台
- 登入後, 點選左上角`+`, 新的App

> 1. 選擇平台: iOS
> 2. 名稱: 會顯示在App Store上, 之後可以作多國語系
> 3. 語言: 看你喜好
> 4. 套裝組ID: 最一開始憑證那邊設定好的APP ID, 如果找不到就代表設定有誤
> 5. SKU: 隨便打, 我通常就把Bundle ID倒過來打, com.jesse.sticker -> sticker.jesse.com

<img src="/img/in-post/2016-09-22-ios10-stickers/itunes-new-app.png" style="width: 400px;"/>

#### 2. 上傳

- 接下來, 回到Xcode, 我們要來把App上傳到iTunes Connect.
- 點上左上角Xcode -> Prefences.. ( <kbd>⌘+,</kbd> )
- 點選你的開發者帳號(Agent) -> View Detail

<img src="/img/in-post/2016-09-22-ios10-stickers/prefences.jpg" style="width: 600px;"/>

- 點選Download All Profiles將設定資料同步.
- 再來, 將剛剛選擇的Device改回Generic iOS Device

<img src="/img/in-post/2016-09-22-ios10-stickers/choose-device.png" style="width: 300px;"/>

- 接下來, 打包App, 點選Xcode上方, Product -> Archive, 等他跑完會跳出這個視窗, 如果不小心關掉, 可以從Xcode上方 Window -> Organizer 開啟

<img src="/img/in-post/2016-09-22-ios10-stickers/archive-screen.png" style="width: 600px;"/>

- 點選Upload to App Store..., 選擇你的開發者帳號, next到底, 就會開始上傳了.

#### 3. 資料設定

- 接下來把App資訊, 定價與供應狀況, 填一填.
- 在點選1.0準備提交, 這裡要上傳螢幕截圖, 現在只需要準備最大尺寸的截圖就可以了, iPhone: 5.5, iPad: 12.9, 以前所有尺寸都需要, 根本浪費生命🌝

<img src="/img/in-post/2016-09-22-ios10-stickers/prepare-for-submit.png" style="width: 700px;"/>

- 如何截圖: 打開模擬器, copy screen

<img src="/img/in-post/2016-09-22-ios10-stickers/copy-screen.png" style="width: 400px;"/>

- 要符合截圖大小,[看這裡](https://developer.apple.com/library/content/documentation/LanguagesUtilities/Conceptual/iTunesConnect_Guide/Appendices/Properties.html), 以下簡單列出直立的大小

> 1. iPhone 5.5 : 1242 × 2208
> 2. iPad 12.9 : 2048 × 2732

> **注意** : 圖片不能有透明度, 也就是alpha = 100%

- 準備一張1024x1024的App Store icon.

<img src="/img/in-post/2016-09-22-ios10-stickers/app-store-icon.png" style="width: 200px;"/>

- 這邊把勾選拿掉, 不需要展示.

<img src="/img/in-post/2016-09-22-ios10-stickers/no-need-display.png" style="width: 400px;"/>

- 選擇剛剛上傳的App.

<img src="/img/in-post/2016-09-22-ios10-stickers/select-app.png" style="width: 700px;"/>

- 如果找不到可以點選活動, 看是不是還在處理中.

<img src="/img/in-post/2016-09-22-ios10-stickers/app-status.png" style="width: 700px;"/>

- 接下來就儲存, 送審啦！

### Reference

- [Realm News - corebluetooth](https://realm.io/news/yoav-schwartz-corebluetooth-peripherals/)
- [Apple Core Bluetooth Programming Guide](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1)




