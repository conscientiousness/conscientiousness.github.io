---
layout: post
title: 'swift 學習筆記 - 多種GCD用法'
date: 2016-05-13 14:39
comments: true
categories: 
---
1. 背景執行
let defaultPriority = DISPATCH_QUEUE_PRIORITY_DEFAULT
let backgroundQueue = dispatch_get_global_queue(defaultPriority, 0)
dispatch_async(backgroundQueue, {
     let result = doSomeExpensiveWork()
     dispatch_async(dispatch_get_main_queue(), {
          //use `result` somehow 
     })
})
2. Singleton
+ (instancetype) sharedInstance {
     static dispatch_once_t onceToken;
     static id sharedInstance;
     dispatch_once(&amp;onceToken, ^{
          sharedInstance = [[self alloc] init];
     });
     return sharedInstance;
}
3. with blocks

 typealias DoneBlock = () -> ()
typealias WorkBlock = (DoneBlock) -> ()

class AsyncSerialWorker {
     private let serialQueue = dispatch_queue_create("ios.taipei.queue", DISPATCH_QUEUE_SERIAL)

     func enqueueWork(work: WorkBlock) {
          dispatch_async(serialQueue) {
               let semaphore = dispatch_semaphore_create(0)
               work({
                    dispatch_semaphore_signal(semaphore)
               })
               dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER)
          }
     }
}
4. multi-task

 dispatch_group_t group = dispatch_group_create()
     for item in someArray {
          dispatch_group_async(group, backgroundQueue, {
               performExpensiveWork(item: item)
          })
     }
     dispatch_group_notify(group, dispatch_get_main_queue(), {
     // all the work is complete
}

ref:
http://ios.taipei/?p=20