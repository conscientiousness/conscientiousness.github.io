---
layout: post
title: 'Object-C 學習筆記 - app送審出現swift相關錯誤解法'
date: 2015-09-21 12:29
author:     "Jesse"
catalog:    false
tags:
    - Xcode
---
如果專案有用到Swift 之後要上架APP打包送到itunes connect如沒設定會寄問題信給你
使用下列設定就可以了

Target->'My project'->Build Settings->Embedded Content Contains Swift Code to YES

詳解：
https://developer.apple.com/library/ios/qa/qa1881/_index.html