---
layout: post
title: 'Object-C 學習筆記 - 圖片縮圖'
date: 2015-09-07 06:00
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
此為學習筆記，如有錯誤煩請指正，感恩:)

參考Parse open source範例修改而成，可以傳入縮圖模式(UIViewContentModeScaleAspectFill、UIViewContentModeScaleAspectFit)和圖片大小，可得到1:1照片或等比例長寬，效能跟壓縮品質還不錯。

(1)
- 設定長寬、縮圖模式(UIViewContentModeScaleAspectFill、UIViewContentModeScaleAspectFit)、JEPG壓縮強度
```objc
+ (NSData*)resizeImage:(UIImage *)anImage width:(float)width height:(float)height contentMode:(UIViewContentMode)contentMode{

    UIImage *resizedImage = [anImage resizedImageWithContentMode:contentMode
    bounds:CGSizeMake(width, height) interpolationQuality:kCGInterpolationHigh];

    NSData *imageData = UIImageJPEGRepresentation(resizedImage, 0.7f);

    return imageData;
}
```

(2)category UIImage
- Resizes the image according to the given content mode, taking into account the image's orientation
```objc UIImage+Resize.m
- (UIImage *)resizedImageWithContentMode:(UIViewContentMode)contentMode
                                  bounds:(CGSize)bounds
                    interpolationQuality:(CGInterpolationQuality)quality {
    CGFloat horizontalRatio = bounds.width / self.size.width;
    CGFloat verticalRatio = bounds.height / self.size.height;
    CGFloat ratio;

    switch (contentMode) {
        case UIViewContentModeScaleAspectFill:
            ratio = MAX(horizontalRatio, verticalRatio);
            break;

        case UIViewContentModeScaleAspectFit:
            ratio = MIN(horizontalRatio, verticalRatio);
            break;

    }

    CGSize newSize = CGSizeMake(self.size.width * ratio, self.size.height * ratio);
    return [self resizedImage:newSize interpolationQuality:quality];
}
```

(3)
- Returns a rescaled copy of the image, taking into account its orientation
- The image will be scaled disproportionately if necessary to fit the bounds specified by the parameter
```objc
- (UIImage *)resizedImage:(CGSize)newSize interpolationQuality:(CGInterpolationQuality)quality {
    BOOL drawTransposed;

    switch (self.imageOrientation) {
        case UIImageOrientationLeft:
        case UIImageOrientationLeftMirrored:
        case UIImageOrientationRight:
        case UIImageOrientationRightMirrored:
            drawTransposed = YES;
            break;

        default:
            drawTransposed = NO;
    }

    return [self resizedImage:newSize
                    transform:[self transformForOrientation:newSize]
               drawTransposed:drawTransposed
         interpolationQuality:quality];
}
```
(4)
- Returns a copy of the image that has been transformed using the given affine transform and scaled to the new size
- The new image's orientation will be UIImageOrientationUp, regardless of the current image's orientation
- If the new size is not integral, it will be rounded up
```objc
- (UIImage *)resizedImage:(CGSize)newSize
                transform:(CGAffineTransform)transform
           drawTransposed:(BOOL)transpose
     interpolationQuality:(CGInterpolationQuality)quality {
    CGRect newRect = CGRectIntegral(CGRectMake(0, 0, newSize.width, newSize.height));
    CGRect transposedRect = CGRectMake(0, 0, newRect.size.height, newRect.size.width);
    CGImageRef imageRef = self.CGImage;

    // Build a context that's the same dimensions as the new size
    CGContextRef bitmap = CGBitmapContextCreate(NULL,
                                                newRect.size.width,
                                                newRect.size.height,
                                                CGImageGetBitsPerComponent(imageRef),
                                                0,
                                                CGImageGetColorSpace(imageRef),
                                                CGImageGetBitmapInfo(imageRef));

    // Rotate and/or flip the image if required by its orientation
    CGContextConcatCTM(bitmap, transform);

    // Set the quality level to use when rescaling
    CGContextSetInterpolationQuality(bitmap, quality);

    // Draw into the context; this scales the image
    CGContextDrawImage(bitmap, transpose ? transposedRect : newRect, imageRef);

    // Get the resized image from the context and a UIImage
    CGImageRef newImageRef = CGBitmapContextCreateImage(bitmap);
    UIImage *newImage = [UIImage imageWithCGImage:newImageRef];

    // Clean up
    CGContextRelease(bitmap);
    CGImageRelease(newImageRef);

    return newImage;
}
```


Ref:
- [Parse - Anypic](https://www.parse.com/tutorials/anypic)
- [CGRectIntegral what is the usage of it?](http://stackoverflow.com/questions/9975219/cgrectintegral-what-is-the-usage-of-it)
- [UIImage的属性imageOrientation](http://www.ganlvji.com/uiimage-image-orientation-clip-image/)
