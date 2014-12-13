---
layout: post
title: "切换移动网络开关的扩展"
date: 2014-12-13 15:12:31 +0800
comments: true
categories: 技术
---


当初看到[6david9](http://weibo.com/6david9?from=myfollow_group)的一篇博客[使用 CORETELEPHONY 中的私有接口设置移动网络开关](http://blog.cocoabit.com/blog/2014/09/29/shi-yongcoretelephony-zhong-de-si-you-jie-kou-she-zhi-yi-dong-wang-luo-kai-guan/)，然后想到给它加一个Today类型的Extension，直接下拉就可以快速设置，配上系统自带的上拉设置WiFi，堪称完美。

<!--more-->

项目地址：https://github.com/victorjiang/MobileDataSwitch

设置网络开关的私有接口：

```
extern BOOL CTCellularDataPlanGetIsEnabled();   // 查询
extern void CTCellularDataPlanSetIsEnabled(BOOL enabled);   // 设置
```

需要添加 CoreTelephony.framework 框架

第一次使用Widget，TodayViewController的视图是居中显示的

<!-- 图1 -->
->![cellular_extension_1](http://victorjiang.github.io/images/2014/cellular_extension_1.jpg)<-

之后下拉的时候，Widget的内容会向上偏移

<!-- 图2 -->
->![cellular_extension_2](http://victorjiang.github.io/images/2014/cellular_extension_2.jpg)<-

原因是TodayViewController的`- widgetMarginInsetsForProposedMarginInsets:defaultMarginInsets`方法

```
- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets
{
    //defaultMarginInsets = (top = 0, left = 47, bottom = 39, right = 0)
}
```

默认下边距有39个单位，解决办法为修改下边距为0，这样视图就居中了。

```
- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets
{
    defaultMarginInsets.bottom = 0;
    return defaultMarginInsets;
}
```

<!-- 图3 -->
->![cellular_extension_3](http://victorjiang.github.io/images/2014/cellular_extension_3.jpg)<-



