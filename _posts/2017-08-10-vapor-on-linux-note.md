---
layout:     post
title:      "Vapor除雷備忘"
date:       2017-08-07
author:     "Jesse"
header-img: "img/in-post/vapor/cover.jpg"
catalog:    true
tags:
    - Swift
    - Vapor
    - AWS EC2
    - Ubuntu
    - Linux
---

這篇記錄開發Vpaor遇到的問題與解決方法

## 環境
---

1. Ubuntu 16.04 as operating system
2. AWS EC2 cloud instances
3. Swift 3.3.1 as programming language
4. Vapor 2.0 as web framework
5. PostgreSQL as Database

## URLRequest and URLSession
---

#### 問題

使用第三方的API來資料交換，macOS執行正常，但在Linux上發現Header未送出, 導致無法驗證Token

#### 解決

追蹤發現是在Linux上的Bug，目前已Fix但應該下個版本才會Release.

1. [SR-2617](https://bugs.swift.org/browse/SR-2617)
2. [SR-3463](https://bugs.swift.org/browse/SR-3463)

送出多加`Content-Type`可以解決，即使Value亂打

```swift
request.addValue("application/json", forHTTPHeaderField: "Content-Type")
```

## arc4random_uniform
---

#### 問題

[Can't use arc4random on Linux](https://bugs.swift.org/browse/SR-685)

#### 解決

用Vapor提供的Method解決(亂數1-10)

```swift
import Random

Int.random(min: 1, max: 10)
```

其他方式實作

[Swift arc4random_uniform(max) in Linux](https://docs.vapor.codes/2.0/getting-started/install-on-ubuntu/)

## Linux with GCD
---

#### 問題

用到GCD的地方發生編譯錯誤

#### 解決

要`import Dispatch`, 原因如下:

As of Swift 3.0 release version GCD is available on Linux with simple import Dispatch, which is slightly confusing as it's not needed on iOS/macOS, where Dispatch module is imported by default.
