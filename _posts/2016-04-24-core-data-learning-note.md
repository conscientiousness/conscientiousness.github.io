---
layout: post
title: 'Swift 學習筆記 - Core Data'
date: 2016-04-24 14:26
author:     "Jesse"
catalog:    false
tags:
    - swift
    - iOS
---

## 一、組成如下：
[img]http://i.imgur.com/iGjOl3R.png[/img]

1. entities - data, ie.Person (NSManaged Object)
2. context (NSManagedObjectContext)
3. store (NSPersistent)

## 二、實作新增修改刪除的APP

#### 1、建立Entity

幾個Attributes的定義與使用時機 :
- Binary Data :
> If you need to store photos, audio, or some other contiguous BLOB of zeros and ones, you should use the Binary Data attribute type. When an NSManagedObject subclass is created from an entity containing a binary data attribute, the resulting property type is NSData. Depending on the data you’re storing, the approach in converting your data to and from NSData will differ. One common scenario is storing photos. To store a photo, you convert a UIImage to NSData using UIImagePNGRepresentation() or UIImageJPEGRepresentation(). To retrieve a photo, you convert the NSData to UIImage using the UIImage class method imageWithData. Binary Data is a good choice for large files because they can be seamlessly stored outside the database using the Allows External Storage attribute setting. With this setting enabled, Core Data will decide whether it is more efficient to store the file inside or outside of the database.

- Transformable :
> The Transformable attribute data type is used to store an Objective-C object. This attribute type is a flexible option that allows you to store an instance of any class. An example of a transformable attribute is an instance of UIColor. When an NSManagedObject subclass is created from an entity containing a transformable attribute, the resulting property type is id. For the id object to make it into the store (and back again), you’ll need to use an instance of NSValueTransformer or an instance of a subclass of NSValueTransformer. This class helps transparently convert the attribute to NSData and back again. This is a reasonably simple process, especially when the class you want to store implements the NSCoding protocol. If it does, the system provides a default transformer that already knows how to archive and un-archive specific objects.

#### Relationship關係

Item(Relationships:store)<<->Store(Relationships:items)
表示one `Store` can have many `Items`

[img]http://i.imgur.com/7amqM1b.png?1[/img]

#### 2、建立NSManagedObject Subclass

(1) select xxxx.xcdatamodeld -> entities -> editor -> Create NSManagedObject Subclass

(2) unselect "Use scalar properties for primitive data types".

All the number object are going to use NSNumber :

>
String maps to String
Integer 16/32/64, Float, Double and Boolean map to NSNumber
Decimal maps to NSDecimalNumber
Date maps to NSDate
Binary data maps to NSData
Transformable maps to AnyObject

(*) If you want to work directly with primitive data types such as Double and Int32, you could have checked the box next to "Use scalar properties for primitive data types" in the last dialog when you were generating the managed object subclass

參考 : http://goo.gl/sVWFn4 , http://goo.gl/TLPCE1

#### 3、取得Core Data資料


Ref:
- https://www.objc.io/issues/4-core-data/core-data-overview/
- http://www.iphonelife.com/blog/31369/unleash-your-inner-app-developer-tuning-core-data
- http://code.tutsplus.com/tutorials/core-data-and-swift-subclassing-nsmanagedobject--cms-25116
- Tim Roadley. 「Learning Core Data for iOS: A Hands-On Guide to Building Core Data Applications (Jason Arnold's Library)」 iBooks.