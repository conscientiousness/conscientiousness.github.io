---
layout: post
title: 'Rails issue解決筆記 - Migrations are pending'
date: 2015-04-12 11:44
author:     "Jesse"
catalog:    false
tags:
    - RoR
---
Model,View,Controller,Route大致瞭解後，開始來架環境，試著完成一個留言板的小專案

在這個步驟
`rails g model hotel room:string memo:text`
建立一個名為hotel的Model
且create資料表hotel中的兩個欄位

建立完之後記得`rake db:migrate`

少了這步就出現這個issue啦

ActiveRecord::PendingMigrationError
Migrations are pending. To resolve this issue, run: bin/rake db:migrate RAILS_ENV=development
![螢幕快照 2015-04-12 下午7.54.08.png](http://user-image.logdown.io/user/12628/blog/11894/post/260040/Mw4e5qQBRDefkp7yhCvH_%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%202015-04-12%20%E4%B8%8B%E5%8D%887.54.08.png)

db/migrate可看到檔案內容

延伸閱讀
[Active Record - 資料庫遷移(Migration)](https://ihower.tw/rails4/migrations.html)