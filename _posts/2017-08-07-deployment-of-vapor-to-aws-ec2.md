---
layout:     post
title:      "部署Vapor2 App到AWS EC2"
subtitle:   "Deployment of a Vapor2 App to the AWS EC2"
date:       2017-08-07
author:     "Jesse"
header-img: "img/in-post/vapor/cover.jpg"
catalog:    true
tags:
    - Swift
    - Vapor
    - AWS EC2
    - Nginx
    - Supervisor
---

不久之前vapor改版升級至Vapor2, 效能也大幅提升([官方文章](https://medium.com/@qutheory/vapor-2-less-code-more-power-af1e1ec5affa)), 之後Swift4也不會像2轉3這麼恐怖的大改, 就決定用Vapor提供API給我自己的Side Project App用, 因為不論是Parse還是Firebase如果遇到資料結構稍微複雜真的很難用, 不過網路上相關文章也是很少, 踩了無數地雷, [Slack](http://vapor.team)問問題與尋找答案, 花了大概幾天才把後端的環境建置起來, 至於為什麼不用發展成熟的Nodejs, Rails, PHP...試過Nodejs大概兩個禮拜, 雖然有強大豐富的NPM資源, 教學資源也豐富, 身為iOS開發者還是寫Swift比較爽.

建置環境前, 要了解基本的Linux權限控管與語法, 才不會遇到一些問題, 例如一開始我沒把權限分好, 用Supervisor執行程式, 一直出現permission的錯誤, 以下只記錄我的建置步驟, 如有Linux的問題可參考[鳥哥的Linux 私房菜](http://linux.vbird.org/)

## 環境
---
1. Ubuntu 16.04 as operating system
2. AWS EC2 cloud instances
3. Swift 3.3.1 as programming language
4. Vapor 2.0 as web framework
5. PostgreSQL as Database

## 目標
---
1. Proxying Vapor HTTP servers behind Nginx.
2. Using Supervisor for start, stop, and restart your Vapor app.

## Step1 - 安裝與更新基本套件
---

Update the server, install auto-upgrades and a time service and finally initiate a reboot.

<script src="https://gist.github.com/conscientiousness/c5da7886660c899cfdda260dc85e0df8.js"></script>

## Step2 - Install Swift and Vapor
---

依照[官網](https://docs.vapor.codes/2.0/getting-started/install-on-ubuntu/)說明快速安裝,
因為有用到PostgreSQL, 如果沒安裝libpq-devcd會有編譯錯誤.

<script src="https://gist.github.com/conscientiousness/a98e1ca3ea2d72fe78186f6371c84f87.js"></script>

Verify Installation

```bash
eval "$(curl -sL check.vapor.sh)"
```

## Step3 - Install and setup Nginx
---

基本安裝

```bash
sudo apt-get update
sudo apt-get install nginx
```

Add a rule to firewall to allow TCP connections on port 80. (Only needs to be done if firewall is enabled)

```bash
sudo ufw allow 'Nginx HTTP'
sudo ufw status
```

Check if Nginx is running `systemctl status nginx`

![](/img/in-post/2017-08-07-deployment-of-vapor-to-aws-ec2/nginx-running.png)

Change the default configuration file in /etc/nignx/sites-available so that it looks like the following snippet.

```bash
vim /etc/nginx/sites-available/default
```
<script src="https://gist.github.com/conscientiousness/4606eb934b66eff1b9aa9b09e993cdf7.js"></script>

## Step4 - Install and Setup Supervisor
---

基本安裝

```bash
sudo apt-get update
sudo apt-get install supervisor
```

Create a new config file for your app /etc/supervisor/conf.d/APP_NAME.conf`

```bash
sudo vim /etc/supervisor/conf.d/APP_NAME.conf
```

Don’t forget to replace “USER_NAME”, "APP_NAME" and directory with your setting and also update the project directory path.

<script src="https://gist.github.com/conscientiousness/8acbd8881e2aa87514961c42dd69f71b.js"></script>

Load the configuration file and run "APP_NAME" in background using supervisor.

```bash
sudo supervisorctl reread
sudo supervisorctl add APP_NAME
sudo supervisorctl start APP_NAME
sudo supervisorctl status
```

## Reference
---
1. [Vapor Documentation](https://docs.vapor.codes/2.0/)
2. [Deployment of a Swift Vapor App to the AWS EC2 Cloud](http://bit.ly/2qan5Sj)
3. [Deploy a basic Vapor app with Nginx and Supervisor](http://bit.ly/2si1riz)
4. [Deploy Vapor 2 app With Nginx and Supervisor on AWS or any cloud platform running Ubuntu 16.04](http://bit.ly/2tV0uej)
