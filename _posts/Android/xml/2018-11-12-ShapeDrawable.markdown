---
layout: post
title:  "ShapeDrawable"
date:   2018-11-12 23:44:00 +0800
categories: Android
tags: xml
author: pepe
description: 『 ShapeDrawable 』
---

### **shape**

先来看语法：

```
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    
    <!-- 圆角 -->
    <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
        
    <!-- 渐变 -->
    <gradient
        android:angle="integer"
        android:centerX="integer"
        android:centerY="integer"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
        
    <!-- 间隔 -->
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
        
    <!-- 大小 -->
    <size
        android:width="integer"
        android:height="integer" />
        
    <!-- 填充 -->
    <solid
        android:color="color" />
        
    <!-- 描边 -->
    <stroke
        android:width="integer"
        android:color="color"
        android:dashWidth="integer"
        android:dashGap="integer" />
</shape>
```
### **android:shape**

    `android:shape的几个值`：rectagle矩形（默认），oval椭圆，line水平直线，ring环形。

其中**`android:shape="ring"`会单独多出几个属性**：

 - android:innerRadius：内环的半径
 - android:innerRadiusRatio：
 - android:thickness：
 - android:thicknessRatio：
 - android:useLevel：只有当我们的shape使用在LevelListDrawable中的时候，这个值为true，否则为false。
 
### **corners**

圆角半径， **仅在`shape="rectangle"(默认)`时使用**，radius值越大角越圆，radius默认为0。

 - android:radius：同时设置四个角的半径，会被下面每个特定的圆角属性覆盖。可以是0dp，表示没有圆角。

 - android:topLeftRadius：设置左上角的半径

 - android:topRightRadius：设置右上角的半径 

 - android:bottomLeftRadius：设置左下角的半径

 - android:bottomRightRadius：设置右下角的半径
 
### **gradient**

 - android:angle：**angle表示渐变的起始位置**，这个值必须为45的倍数，包括0，且默认为0，0表示从左往右渐变，逆时针旋转，依次是45，90，135.....，90表示从下往上渐变（亲自测试过），270表示从上往下渐变，剩下的大家依次去推理。

 - android:centerX：X轴上对渐变的中心的相对位置（0 - 1.0）

 - android:centerY：Y轴上对渐变的中心的相对位置（0 - 1.0）

 - android:startColor：表示渐变的起始颜色

 - android:centerColor：表示渐变的过渡颜色

 - android:endColor：表示渐变的结束颜

 - android:gradientRadius：渐变半径，当`android:type="radial"`时，需要设置

 - android:type：表示渐变的类型，有三种，分别是：

  - "linear"	线性变化（默认）
  - "radial"	辐射渐变，startColor即辐射中心的颜色，当type为radial时，我们要设置`android:gradientRadius=""`，这个表示渐变的半径（线性渐变和扫描渐变不需要设置）
  - "sweep"	扫描渐变
 - android:useLevel：只有当我们的shape使用在LevelListDrawable中的时候，这个值为true，否则为false。
 
### **padding**

内容与视图边界的距离

 - android:left：左边填充距离

 - android:top：顶部填充距离

 - android:right：右边填充距离

 - android:bottom：底部填充距离
 
### **size**
这个shape的大小。

 - android:height：这个shape的高度

 - android:width：这个shape的宽度

注意：默认情况下，这个shape会缩放到与他所在容器大小成正比。当你在一个ImageView中使用这个shape，你可以使用 `android:scaleType="center"`来限制这种缩放。

### **solid**

填充这个shape的颜色

 - android:color：颜色值，十六进制数，或者一个Color资源

### **stroke**

这个shape使用的笔画，**当android:shape="line"的时候，必须设置改元素。**

 -  android:width：笔画的粗细。

 -  android:color：笔画的颜色

 -  android:dashGap：每画一条线就间隔多少

 -  android:dashWidth：每画一条线的长度

dashWidth和dashGap两个属性，如果不设置则为实线，并且只要其中一个设置为0dp，则边框仍然为实线边框

### **动态设置**
```
GradientDrawable drawable =(GradientDrawable)view.getBackground();
//GradientDrawable drawable = new GradientDrawable();
drawable.setCornerRadius(5);
drawable.setStroke(1, Color.parseColor("#cccccc"));
drawable.setColor(Color.parseColor("#eeeeee"));
view.setBackgroundDrawable(drawable);
```

参考：

[Android样式的开发:shape篇](http://keeganlee.me/post/android/20150830)

[Android XML shape 标签使用详解（apk瘦身，减少内存好帮手） - popfisher - 博客园](https://www.cnblogs.com/popfisher/p/6238119.html)



