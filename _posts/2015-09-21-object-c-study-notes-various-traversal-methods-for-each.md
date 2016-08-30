---
layout: post
title: 'Object-C 學習筆記 - 多種遍歷方法(for-each)與比較'
date: 2015-09-21 02:02
comments: true
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
###C Loops (for/while)
```objc
for (NSUInteger i = 0; i < [array count]; i++) {
  id object = array[i];
  NSLog(@"%@", object)
}
```

###List Comprehension (for/in)
[1]
```objc
for (id object in array) {
    NSLog(@"%@", object);
}
```
[2]
<NSFastEnumeration>
NSFastEnumerationState
[3]
NSEnumerator

###Enumerate With Blocks
[1]
```objc
[array enumerateObjectsUsingBlock:^(id object, NSUInteger idx, BOOL *stop) {
    NSLog(@"%@", object);
}];
```
[2]
```objc
- (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts
                         usingBlock:(void (^)(id obj, NSUInteger idx, BOOL *stop))block
```

NSEnumerationOptions
```objc
enum {
   NSEnumerationConcurrent = (1UL << 0),
   NSEnumerationReverse = (1UL << 1),
};
typedef NSUInteger NSEnumerationOptions;
```



###Reference
[更詳細用法](http://nshipster.com/enumerators/)
[速度比較](http://blog.sunnyxx.com/2014/04/30/ios_iterator/)