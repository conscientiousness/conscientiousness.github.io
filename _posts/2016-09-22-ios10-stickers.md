---
layout:     post
title:      "iOS 10 iMessage貼圖上架教學"
date:       2016-09-22
author:     "Jesse"
header-img: "img/in-post/2016-09-22-ios10-stickers/header-image.jpg"
catalog:    true
tags:
    - iOS 10
    - iMessage
    - sticker
---

iOS 10 其中一個新的feature就是可以下載貼圖for iMessage, 此篇寫給非iOS開發者參考, 順便在複習一下上架流程, 首先您必須先獲得下面所列裝備.

> - Xcode
> - 開發者帳號

所以您需要有一台MacBook或含有macOS的相關產品來開啟Xcode, 且要繳[99 USD](https://developer.apple.com/support/compare-memberships/)給Apple當上架保護費/per year, 如果沒有認識的開發者或公司帳號, 對純粹只想上架貼圖的設計師是一筆不小開銷, 看到這邊應該就想左轉離開了吧xd

### 帳號設定與憑證申請

上架貼圖就跟開發一款APP上架的流程幾乎是一樣的，所以您必須經過XCODE build APP 之後上傳到itunes connect送審, 可以完全不需要寫code, 但在這之前比需先申請id, 憑證等等, 才能使用XCODE完成這些動作.

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
1. 步驟如上.
2. 記得點選檔案兩次, 會閃一下, 就會加到keychain中了.

<img src="/img/in-post/2016-09-22-ios10-stickers/keychain-cer-list.jpg" style="width: 850px;"/>






