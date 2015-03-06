---
layout: post
title: "如何根据设备选择不同尺寸的图片"
date: 2015-03-04 00:03:20 +0800
comments: true
categories: 技术
---

在项目中，可以通过设置不同尺寸设备的LaunchImage，来使得App适配这些设备，项目中其他地方使用的图片，只能设置为@1x、@2x、@3x，并且这些图都是成比例的。要是在不同不同尺寸设备上使用不同大小的图片，则需要在代码中一一判断，然后加载。

<!--more-->

现有一个案例是，App第一次启动的时候需要加载引导页，引导页一般都是全屏的图片，这个时候对于不同尺寸的设备来说，就需要不同大小的引导图片了。方法肯定是通过判断设备尺寸来选择加载哪种大小的图片，在[StackOverFlow](http://stackoverflow.com/questions/25892207/how-to-specify-size-for-iphone-6-customised-edge-to-edge-image)上看到利用扩展UIImage来实现该效果，有方便也有不方便之处：

* 方便之处在于只要添加该UIImage类别，代码中使用类别中的类方法即可
* 不方便之处在于，对于图片以及命名来说，有严格要求。图片格式必须要是PNG格式的；图片命名必须按照指定规则

当然，总体来说，代码量少了，图片的限制都是一些机械操作，实际上还是方便许多的。下面具体来学学如何根据设备不同选择不同尺寸的图片。

##1 封装UIImage类别

代码可以在[GitHub](https://github.com/victorjiang/UIImage-VJDeviceSpecificMedia/)上下载。

###1.1 设备类型枚举`VJDeviceClass`

首先看`VJDeviceClass`这个枚举，根据设备尺寸不同分为以下几种类型：

```
// VJDeviceClass enum
typedef NS_ENUM(NSInteger, VJDeviceClass) {
    // iPhone
    VJDeviceClass_iPhone,
    VJDeviceClass_iPhoneRetina,
    VJDeviceClass_iPhone5,
    VJDeviceClass_iPhone6,
    VJDeviceClass_iPhone6plus,
    
    // iPad
    VJDeviceClass_iPad,
    VJDeviceClass_iPadRetina,
    
    // unKnown
    VJDeviceClass_unKnown
};
```
设备类型包括iPhone、iPad、以及unKnown，这里发现根据尺寸不同这种说法其实是错误的，严格来说应该是分辨率不同，比如iPhone3GS之前的机型和iPhone4，其实他们尺寸是一样的，但是他们的分辨率不同，iPhone3GS之前的机型是`VJDeviceClass_iPhone`类型，而iPhone4是`VJDeviceClass_iPhoneRetina`类型。习惯了说尺寸，在这里解释以下，应该不影响大家的理解。

###1.2 获取当前设备类型

定义一个函数来获取当前设备的类型：

```
VJDeviceClass VJ_CurrentDeviceClass()
{
    CGFloat greaterPixelDimension = fmaxf([[UIScreen mainScreen] bounds].size.height, [[UIScreen mainScreen] bounds].size.width);
    
    switch ((NSInteger)greaterPixelDimension) {
        case 480:
            return ([[UIScreen mainScreen] scale] > 1.0) ? VJDeviceClass_iPhoneRetina : VJDeviceClass_iPhone;
            break;
        case 568:
            return VJDeviceClass_iPhone5;
            break;
        case 667:
            return VJDeviceClass_iPhone6;
            break;
        case 736:
            return VJDeviceClass_iPhone6plus;
            break;
        case 1024:
            return ([[UIScreen mainScreen] scale] > 1.0) ? VJDeviceClass_iPadRetina : VJDeviceClass_iPad;
            break;
        default:
            return VJDeviceClass_unKnown;
            break;
    }
}
```
通过获取设备屏幕的尺寸来判断设备的类型，注意，这里就涉及到上面说的iPhone3GS&iPhone4的问题，还需要再根据`scale`属性来进一步判断，iPad也同样存在这个问题。

###1.3 获取图片对象

给UIImage的类别添加两个类方法，模仿`imageNamed:`方法：

```
// imageName without suffix, image's type default is PNG
+ (instancetype)vj_imageForDeviceWithName:(NSString *)imageName
{
    return [UIImage vj_imageForDeviceWithName:imageName type:@"png"];
}

// specify the image's type
+ (instancetype)vj_imageForDeviceWithName:(NSString *)imageName type:(NSString *)type
{
    NSString *suffixString;
    switch (VJ_CurrentDeviceClass()) {
        case VJDeviceClass_iPhone:
            suffixString = @"";
            break;
        case VJDeviceClass_iPhoneRetina:
            suffixString = @"@2x";
            break;
        case VJDeviceClass_iPhone5:
            suffixString = @"-568h@2x";
            break;
        case VJDeviceClass_iPhone6:
            suffixString = @"-667h@2x";
            break;
        case VJDeviceClass_iPhone6plus:
            suffixString = @"-736h@3x";
            break;
        case VJDeviceClass_iPad:
            suffixString = @"~ipad";
            break;
        case VJDeviceClass_iPadRetina:
            suffixString = @"~ipad@2x";
            break;
        case VJDeviceClass_unKnown:
        default:
            suffixString = @"";
            break;
    }
    
    UIImage *image = nil;
    NSString *imageFullName = [imageName stringByAppendingString:suffixString];
    
    // if type is not png, imageFullName & imageName will append the suffix of type
    if (![[type lowercaseString] isEqualToString:@"png"]) {
        imageFullName = [[imageFullName stringByAppendingString:@"."] stringByAppendingString:type];
        imageName = [[imageName stringByAppendingString:@"."] stringByAppendingString:type];
    }
    
    image = [UIImage imageNamed:imageFullName];
    
    if (!image) {
        image = [UIImage imageNamed:imageName];
    }
    
    return image;
}
```
参考的[StackOverFlow](http://stackoverflow.com/questions/25892207/how-to-specify-size-for-iphone-6-customised-edge-to-edge-image)其实只给了第一个方法，而且方法里并没有判断图片类型，这就存在一个问题，说这个问题之前先说一个需要注意的地方：

* 注：`imageName`这个参数是不带图片扩展的图片名，如图片`image.png`，`imageName`的值应该是`image`，而不是`image.png`，看上面方法的实现，其实是在`imageName`后面拼接字符，所以如果`imageName`参数带图片扩展的话，最终图片名就不对了。

下面再来说这个问题，[StackOverFlow](http://stackoverflow.com/questions/25892207/how-to-specify-size-for-iphone-6-customised-edge-to-edge-image)上的代码其实要求图片必须是PNG格式的。UIImage的类方法`imageNamed:`参数如果是不带扩展的图片名，则默认加载的是PNG格式的图片，如果图片格式为JPEG，获取到的UIImage为nil，加载不出来图片。

所以我在原作者的基础上修改了这个方法，并又增加了一个方法，可以指定图片扩展。`vj_imageForDeviceWithName:`方法默认加载PNG格式的图片。

##2 修改图片名称

根据代码可以看出，不同尺寸设备，其实是加载不同的图片，通过给imageName后拼接不同的后缀实现。

* iPhoneRetina	--->		"@2x"			--->	960 × 640
* iPhone5			--->		"-568h@2x"	--->	1136 × 640
* iPhone6			--->		"-667h@2x"	--->	1334 × 750
* iPhone6plus		--->		"-736h@3x"		--->	2208 × 1242
* iPad				--->		"~ipad"		--->	1024 × 768
* iPadRetina		--->		"~ipad@2x"		--->	2048 × 1536

iPhone和unKnown类型不用拼接后缀。

修改图片名称，我是用的Automator的Service，通过添加Service能够快速的设置，还可以批量设置。

<!--添加图片1-->
->![apns_prac_1](http://victorjiang.github.io/images/2015/uiimage_device_1.png)<-

打开Automator，选择Service

<!--添加图片2-->
->![apns_prac_1](http://victorjiang.github.io/images/2015/uiimage_device_2.png)<-

1. 选择**Files&Folders**
2.  双击**Rename Finder Items**
3.  添加

接下来就是设置该Service了，如图：

<!--添加图片3-->
->![apns_prac_1](http://victorjiang.github.io/images/2015/uiimage_device_3.png)<-

1. 选择Finder中的图片文件
2. 选择添加文本
3.  输入要添加的字符，以及设置字符添加的位置，这里设置在原文件名后面添加
4. 保存为`-568h@2x`（Command + s）

这样就完成了一个Service的添加，接下来就可以快速修改图片名称了，在Finder中找到要修改的图，选中右击，在选项框中选择Services，就会出现刚刚添加的Service`-568h@2x`。

<!--添加图片4-->
->![apns_prac_1](http://victorjiang.github.io/images/2015/uiimage_device_4.png)<-
 
类似，可以添加其他Services，如图中的`-736h@3x`、`~ipad@2x`等等，其实还可以添加一些别的Services，如图中本人添加的`Compress_50%`，美工给的图基本是高清的，需要自己压缩一下；还有`Compress_120*120`，用于设置Icon；以及`png`、`jpg`，将图片转换成对应的类型。需要什么样的Service都可以去Automator中添加，用起来还是很方便的。

##3 添加图片到工程中

需要注意的是，不能将这些图添加到`Images.xcassets`中，只能添加在工程的目录结构中，原因是添加到`Images.xcassets`中的图片，如果图片名中存在`@2x`、`@3x`，则这些图会被处理，图片名删掉`@2x`、`@3x`，并且把图片放在相应的位置上。

## 总结

总的来说，通过给UIImage添加扩展的方法来选择不同尺寸设备的图片还是挺方面的，虽然说有许多限制，但最终还是能实现该效果。

## 参考

[http://stackoverflow.com/questions/26703083/asset-catalog-images-of-type-retina-4-2x-not-presented-on-iphone-6](http://stackoverflow.com/questions/26703083/asset-catalog-images-of-type-retina-4-2x-not-presented-on-iphone-6)

[http://stackoverflow.com/questions/25892207/how-to-specify-size-for-iphone-6-customised-edge-to-edge-image](http://stackoverflow.com/questions/25892207/how-to-specify-size-for-iphone-6-customised-edge-to-edge-image)