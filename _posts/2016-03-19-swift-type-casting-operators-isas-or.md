---
layout: post
title: 'Swift - Type-Casting Operators「is,as,?和!」 '
date: 2016-03-19 14:23
author:     "Jesse"
catalog:    false
tags:
    - Swift
    - iOS
---
expression is type
expression as type
expression is? type  expression as! type

(1) as 運算子會把目標表達式轉換成指定的型別（specified type），過程如下：

- 如果型別轉換成功， 那麼目標表達式就會回傳指定型別的實例（instance）. 例如：把子類別（subclass）變成父類別（superclass）時.
- 如果轉換失敗，則會拋出編譯錯誤（ compile-time error）。
- 如果上述兩個情況都不是（也就是說，編譯器在編譯時期無法確定轉換能否成功，） 那麼目標表達式就會變成指定的型別的optional. （is an optional of the specified type ） 然後在執行時，如果轉換成功， 目標表達式就會作為 optional的一部分來回傳， 否則，目標表達式回傳nil. 對應的範例是： 把一個 superclass 轉換成一個 subclass.

```swift
class SomeSuperType {}
class SomeType: SomeSuperType {}
class SomeChildType: SomeType {}
let s = SomeType()

let x = s as SomeSuperType  // known to succeed; type is SomeSuperType
let y = s as Int            // known to fail; compile-time error
let z = s as SomeChildType  // might fail at runtime; type is SomeChildType?
```

使用'as'做型別轉換跟正常的型別宣告，對於編譯器來說是一樣的。例如：
```swift
let y1 = x as SomeType  // Type information from 'as'
let y2: SomeType = x    // Type information from an annotation
```

(2) 'is' 運算子在“執行時（runtime）”會做檢查。 成功會回傳true, 否則 false


Ref:
https://tommy60703.gitbooks.io/swift-language-traditional-chinese/content/chapter3/04_Expressions.html#type-casting_operators