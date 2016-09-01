---
layout: post
title: 'Github Pages + Jekyll 快速架站'
date: 2016-08-30
author:     "Jesse"
catalog:    true
tags:
    - Github
    - Jekyll
---

這篇紀錄如何透過Github Pages + Jekyll 快速生成網站，並且使用Custom Domain。如果非前端專業，可以使用google搜尋[Jekyll Theme](http://jekyllthemes.org/)找到喜歡的主題修改使用，本站使用此 [Open Source](https://github.com/huxpro/huxpro.github.io/)。

### 基本概念
[Jekyll](https://jekyllrb.com/) 是一個Open Source的靜態網站產生框架，如官方說明。

> Jekyll is a simple, blog-aware, static site generator perfect for personal, project, or organization sites. Think of it like a file-based CMS, without all the complexity. Jekyll takes your content, renders Markdown and Liquid templates, and spits out a complete, static website ready to be served by Apache, Nginx or another web server. Jekyll is the engine behind GitHub Pages, which you can use to host sites right from your GitHub repositories

[Github Pages](https://pages.github.com/) 是 GitHub 提供的網站托管服務，他有流量與儲存空間無限制的優勢，速度也不會太差。

> You get one site per GitHub account and organization,
and unlimited project sites.

### Github

這邊直接使用現有的Theme來完成

* fork 您喜歡的Theme到您的repo並修改名稱，名稱必須符合：**(your account name)**.github.io，例如您的帳號為abc那您的repo名稱就必須為abc.github.io。
* 接下來就可以透過Git來新增修改文件，相關步驟這邊就不詳述。

### Jekyll

如果要透過git add / commit / push 在檢視修改後的網頁實在是太累了，所以我們要安裝 Jekyll 來直接使用本機來 Run 我們修改後的網頁。

* 修改 _config.yml 到您自身的設定，可 [參考](https://jekyllrb.com/docs/configuration/) 參數相關說明。
* 安裝：先切換到website repo的folder

```

gem install jekyll
bundle install
jekyll serve

```
* http://127.0.0.1:4000/

### Custom Domain

 這邊使用 Godaddy 的服務，設定應該差不多

##### Godaddy
* DNS設定如下：
<img src="/img/in-post/2016-08-30-create-website-with-github-and-jekyll/dns-a-domain.png " style="width: 450px;"/>
<img src="/img/in-post/2016-08-30-create-website-with-github-and-jekyll/dns-custom-domain.png " style="width: 450px;"/>
* [Apex Domain](https://help.github.com/articles/setting-up-an-apex-domain/)
* [Custom Domain](https://help.github.com/articles/quick-start-setting-up-a-custom-domain/)

##### Github

* setting -> custom domain
<img src="/img/in-post/2016-08-30-create-website-with-github-and-jekyll/github-custom-domain.png " style="width: 450px;"/>
* 在根目錄新增名為CNAME的file內容直接輸入 custom domain