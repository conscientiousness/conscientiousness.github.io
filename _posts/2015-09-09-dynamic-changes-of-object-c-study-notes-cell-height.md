---
layout: post
title: 'Object-C 學習筆記 - 動態變更cell高度'
date: 2015-09-09 08:45
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
當輸入的字串過長時，超過Label長度就會顯示...
當然就要動態改變Label高度，cell的高度也要隨之改變

```objc
//Label需定義為0才會換行
cell.label.numberOfLines = 0;

//heightForRowAtIndexPath
//給予字型，大小
tmpStr = @"xxxxxxxxxxxxxxx";
NSDictionary *attributes = @{NSFontAttributeName: [UIFont fontWithName:@"HelveticaNeue" size:13]};
NSAttributedString * attrString = [[NSAttributedString alloc] initWithString:tmpStr attributes:attributes];

//寬度固定計算行高(NSStringDrawingUsesLineFragmentOrigin|NSStringDrawingUsesFontLeading)
CGRect rect = [attrString boundingRectWithSize:CGSizeMake(WIDTH_FOR_MEMO_LABEL, CGFLOAT_MAX) options:NSStringDrawingUsesLineFragmentOrigin context:nil];

//可以得到估算出的Label高度
rect.size.height
```