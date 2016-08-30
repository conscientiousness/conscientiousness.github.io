---
layout: post
title: 'Object-C 學習筆記 - @property'
date: 2015-05-24 16:06
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
# property 說明與用法

#### 版本差異
1. Xcode 2.0 中開始提供的一種功能，簡化一個實體變數設定的過程。
2. Xcode 4.5 後，不需在 @implementation區段加上 @synthesize，Compiler時會自動加上。

#### 用法
1. 宣告在類別的 ＠interface 區段。
2. 用來指定 getter 與 setter 行為特性。
3. 以`.`運算子來做存取。

```java
@interface cat : NSObject
@property int count;
@end
```

**[getter] 得到cat類別(class)裡property物件count值，幾種寫法**

- NSLog(@"%i",cat.count)
- 用在同一個類別裡，self.count
- 用在同一個類別裡，_count

**[setter] 設定cat類別裡count值，幾種寫法**

- cat.cnt=100
- 用在同一個類別裡，self.cnt=100
- 用在同一個類別裡，_cnt=100

####參數
1.set方法內存管理相關
 - retain :
 - assign :
 - copy   :

 2.是否要建立set方法
 - readwrite :
 - readonly  :

 3.多執行緒管理
 - nonatomic : 沒有使用多個(thread;線程;執行緒)時，建議使用。
 - atomic    : 有多執行緒時在用。





