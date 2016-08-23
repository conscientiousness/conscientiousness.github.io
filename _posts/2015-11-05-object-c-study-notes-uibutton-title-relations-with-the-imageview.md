---
layout: post
title: 'Object-C 學習筆記 - UIButton內title與imageView關係'
date: 2015-11-05 17:12
comments: true
categories: 
---
這相對位置，搞了我四小時，為什麼現在才讓我找到

大概的key word
```objc
CGFloat spacing = 10; // the amount of spacing to appear between image and title
tabBtn.imageEdgeInsets = UIEdgeInsetsMake(0, 0, 0, spacing);
tabBtn.titleEdgeInsets = UIEdgeInsetsMake(0, spacing, 0, 0);
```

http://stackoverflow.com/questions/4564621/aligning-text-and-image-on-uibutton-with-imageedgeinsets-and-titleedgeinsets