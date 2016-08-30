---
layout: post
title: 'object-C 學習筆記 - UIVIEW搖動效果'
date: 2015-09-25 10:56
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
http://stackoverflow.com/questions/10294451/animating-uitextfield-to-indicate-a-wrong-password

實測第一跟第二都可以的，但第二個如果有多個要同時動就要加其他寫法
以下為第一種寫法

```objc
-(void)shake:(UIView *)theOneYouWannaShake
{
    [UIView animateWithDuration:0.03 animations:^
     {
         theOneYouWannaShake.transform = CGAffineTransformMakeTranslation(5*direction, 0);
     }
                     completion:^(BOOL finished)
     {
         if(shakes >= 10)
         {
             theOneYouWannaShake.transform = CGAffineTransformIdentity;
             return;
         }
         shakes++;
         direction = direction * -1;
         [self shake:theOneYouWannaShake];
     }];
}
```

call
```objc
				direction = 1;
        shakes = 0;
        [self shake:_emailHintLabel];
        [self shake:_nameHintLabel];
        [self shake:_passHintLabel];
        [self shake:_confirmHintLabel];
```