---
layout: post
title: 'XCODE - Debugging App When Launched '
date: 2015-09-30 17:04
author:     "Jesse"
catalog:    false
tags:
    - Xcode
---

有時候要追蹤`didFinishLaunchingWithOptions`但APP就直接run起來了
用下列方法就可以關閉自動run APP，我是在找推播問題時要用到

In XCode 6,7:
- Select the Scheme on the toolbar (just left of the device you're debugging with)
- Choose Edit Scheme
- Select Run in the left panel
- For the Launch option, select Wait for executable to be launched



referrence:
http://stackoverflow.com/questions/1239000/debugging-app-when-launched-by-push-notification