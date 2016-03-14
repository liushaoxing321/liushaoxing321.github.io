---
layout: post
title: Autolayout
date: 2016-02-16 15:32:24.000000000 +09:00
---

# Autolayout 基础

`Archives` `iOS`

* * *

_如果您觉得我的博客对您有帮助，请通过关注我的新浪微博 [MicroCai](http://weibo.com/1736763114/profile?rightmod=1&wvr=6&mod=personinfo) 支持我，谢谢！_

* * *

这两天自学的时候，复习了下 autolayout。本来想来写一篇文章记录下学习内容，搜了一下写的人真不少，也写得挺不错的。照理我就不用写了，但心里总有那么一点点遗憾，这么流行的东西，我博客里怎么能没有呢？既然如此，那就多写点基础内容。

警告：博主为博文贴了十几张图片，查克拉耗尽，生命垂危，关注 [MicroCai](http://weibo.com/1736763114/profile?rightmod=1&wvr=6&mod=personinfo) 或者送香吻一个就能唤醒博主，好人一生平安。

<div class="toc">

<div class="toc">

*   [Autolayout 基础](#autolayout-基础)
    *   [Interface Builder 介绍](#interface-builder-介绍)
        *   [Align（对齐）](#align对齐)
        *   [Pin：设置相对大小和位置](#pin设置相对大小和位置)
        *   [Resolve Auto Layout Issues：解决 autolayout 问题](#resolve-auto-layout-issues解决-autolayout-问题)
        *   [Resizing Behavior](#resizing-behavior)
        *   [SizeClass](#sizeclass)
            *   [重写布局](#重写布局)
            *   [使用 Xcode 6 预览](#使用-xcode-6-预览)
    *   [VFL（Visual Format Language）](#vflvisual-format-language)
    *   [Autolayout 常见问题](#autolayout-常见问题)
    *   [Masonry：替代 Autolayout](#masonry替代-autolayout)

</div>

</div>

## Interface Builder 介绍

在 storyboard 界面的右下角，有这么一排图标

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_1.png)

鼠标放上去停留一小段时间，就会告诉你它们的作用，从左至右依次是：

*   Align：用来设置对齐相关的约束；
*   Pin：设置相对大小和位置；
*   Resolve Auto Layout Issues：解决 autolayout 问题；
*   Resizing Behavior：设置重置大小会如何影响其他对象；

### Align（对齐）

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_2.png)

下面这些是两个视图层次中同一级的 View 的对齐。

    Leading Edges：头对齐
    Trailing Edges：尾对齐
    Top Edges：顶部对齐
    Bottom Edges：底部对齐

    Horizontal Centers：水平中心对齐
    Vertical Centers：垂直中心对齐
    BaseLines：基准线（默认 View 底部位置）水平对齐，用来对齐有文字的控件，如 UILabel、UIButton 等

下面这些是 SuperView 和 SubView 的对齐，SuperView 是 SubView 的 Container

    Horizontal Center in Container：View 的水平中心和容器的水平中心的相对距离
    Vertical Center in Container：View 的垂直中心和容器的垂直中心的相对距离

在对齐数值的白色输入框内，点击右侧下拉框可以选择“Use Current Canvas Value”，意思是使用当前 Xib/Storyboard 内的差值。

最后一个 Update Frames 表示如何更新 frame，有三个选项，默认为 None 不更新

    None：不更新 frame
    Items of New Constraints：更新新添加的 frame
    All Frames in Container：更新容器内所有的 frame

### Pin：设置相对大小和位置

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_3.png)

最上面有四个矩形框和四条虚线，原来用过 auto resizing 的童鞋应该会比较眼熟。矩形框里的数字表示当前的 View 到最近的 View （注意：不是 SuperView）边缘的距离。

在矩形框下面有一行灰色字的可选项“Constrain to margins”，意思是在设置上述约束是相对于 margins 设置的，而 margin 默认距离是 16。如和上边缘距离 306，加上 16，所以 View 的顶部和它上边最近的 View 的距离是 312。

其他的选项

    Width：设置宽度
    Height：设置高度

    Equal Widths：设置两个同级 View 的宽度关系
    Equal Heights：设置两个同级 View 的高度关系
    Aspect Ratio：设置 View 自身宽高比例

    Align：和前面所讲的 **Align** 一致。那为什么 **Align** 还会出现在这边呢？估计和 **Pin** 有关系，故而也放到这边。

### Resolve Auto Layout Issues：解决 autolayout 问题

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_4.png)

可以选择要处理的 Views：当前选中的 Views 或 Controller 内所有的 Views

    Update Frames：更新 frame 
    Update Constraints：更新约束
    Add Missing Constraints：添加遗漏的约束
    Reset to Suggested Constraints：重置约束
    Clear Constraints：清除约束

### Resizing Behavior

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_5.png)

设置重置大小会如何影响其他对象，有两个选项（默认已勾选）

    Sublings and Ancestors：影响同级兄弟 Views 和祖先 Views
    Descendants：影响后代 Views

_（这个地方我也没弄明白，我查了文档和一些博客，都只是做了简单文字说明，然后自己试了下勾选和未勾选的情况，还是找不到有什么区别，所以也没明白具体是如何影响。）_

### SizeClass

SizeClass 中文意思可以理解为尺寸等级，就是在 autolayout 的基础上，加上屏幕尺寸类型的定义。SizeClass 的宽高有三种类型：Compact（紧凑）、Any（任意）、Regular（普通）

_不同设备屏幕的宽高类型_  
![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_6.png)

当你具体选择尺寸时， IB 会显示出当前选择的屏幕尺寸的相关信息，如宽高类型，屏幕尺寸类型，这个尺寸适用于哪些设备等。

#### 重写布局

如果你设置的某种类型屏幕的约束布局，在其他类型屏幕下出现不符合意图的布局时，可以重写布局，即重新设置该屏幕类型下的布局。

如图的 UIView 设置了 wAny|hAny 上下左右四个约束

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_7.png)

点击这个 UIView，查看 Attribute Inspector 有个 install 选项被勾选上了。

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_8.png)

Installed/UnInstalled 表示的意思是当前布局是否被安装在什么类型的屏幕上。Installed/UnInstalled 前面如果没有东西，表示布局是安装在 wAny|hAny 类型的屏幕上。现在如果我们要单独设置某种类型屏幕的布局可以点击加号，选择屏幕类型

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_9.png)

将新的屏幕类型的勾选取消掉，则当前布局在该类型下不起作用，此时就可以切换类型，重新设置所有约束。

若只想修改一条约束，也可以点击 Size Inspector

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_10.png)

选择 Constraints —— All 显示所有约束，双击任意一条约束，会出现一个和之前类似的界面

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_11.png)

Installed 前面也有个加号，估计你也该猜到了，这里的修改和之前也是类似的。

如果 Xcode 的 Inspector 一些选项前面有加号，就表明它可以被重写，比如刚刚这幅图中，Installed  
选项上方的 Constant 前面也有加号，说明它也是可以被重写的。类似的还有 UILabel 的 font，**但是**：与重写布局不同，在不同的 size class 中改变文字的属性始终会影响基础布局中的文字。它不能像布局一样，在不同的size class中设置不同的属性值。我们通过下面的方法来解决这一问题。 [参考：Swift自适应布局（Adaptive Layout）教程（二）](http://www.devtalking.com/articles/adaptive-layout-2/)。

#### 使用 Xcode 6 预览

在屏幕类型多了这么多之后，做不同类型的屏幕适配也是需要花不少功夫。如果每次适配一种类型屏幕后，都要运行后才能查看效果，效率简直太低下了。Xcode 6 提供了 preview 预览功能，针对这个问题可以节省不少时间。

点击 Xcode Tool Bar 的 Assistant Editor 按钮，显示另一个窗口

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_12.png)

选择 Preview 展示预览界面

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_13.png)

如果想在预览中同时显示不同类型的屏幕，可以在预览界面的左下角点击加号，选择更多设备

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_14.png)

如果想要查看横屏/竖屏，点击设备下方的旋转按钮即可

![](http://blogofzuoyebuluo.qiniudn.com/image_note73867_15.png)

## VFL（Visual Format Language）

不得不说，VFL 的语法比手写 Constraints 的量少了很多，也有象形文字的感觉，但是和普通的 Swift 语法风格比起来实在不搭调。这个内容直接看 [官方文档](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html) 吧，都用图表示出来，说明的挺清楚的。

## Autolayout 常见问题

1.  _设置了 autolayout 之后，在代码中还能用 self.someView.frame 修改 frame 吗？_  
    不能

2.  _为什么有些控件只设置 x、y 值约束，也不会出错？_  
    UIKit 的一些控件如 UILabel、UIImageView 等有自适应特性，会根据内容自适应尺寸，所以不需要再约束其宽高。

3.  _同样的约束用在 UIScrollView 和 UIImageViwe 上，为什么会出现错误？_  
    参考 [Nonomori 的文章：ScrollView 与 Autolayout](http://nonomori.farbox.com/post/scrollview-yu-autolayout)

## Masonry：替代 Autolayout

Masonry 是目前为止公认的 autolayout 最好的替代方案，语法简洁、直观，不会出现各种意外之外的布局。根据网上了解到的一些情况，Masonry 在大型项目中的效果比 autolayout 好很多。