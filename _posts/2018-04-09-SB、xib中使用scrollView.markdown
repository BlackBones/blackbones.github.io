---
layout:     post
title:      SB、xib中使用scrollView
subtitle:   StoryBoard Xib scrollView
date:       2018-04-09
author:     BlackBones
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - OC storyboard xib
---

在使用AutoLayout给UIScrollView进行布局的时候,总会出现点这样那样莫名其妙的问题.挣扎许久最后都以放弃storyboard改为代码实现而告终.今天好好研究了哈,遂拿出来说说.

先从最基础的开始,我们试着在storyboard上添加一个UIScrollView,并且在内部添加一个和它一样大的UIImageView.

首先,拖一个UIScrollView到storyboard,设置约束如下:

顶部距离控制器view距离为50
左侧和右侧距离控制器view距离分别为0![]()
高度固定250(多吉利的数字)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1fq6euje51ij307s066gll.jpg)
scrollView的约束

然后拖一个UIImageView到scrollView中,我们想让它在{0,0}处并和scrollView的尺寸一样,于是设置上下左右距离父控件都为0:

imageView相对scrollView的约束
好像没什么问题,可是Xcode告诉我们这样安排约束是错误的.
UIScrollView的子控件添加约束与普通view不同,仅仅这4个约束不足以满足它的需求.

那么,怎样才是正确的做法呢?

首先:

#### scrollView自身的约束(scrollView的位置和尺寸)可以像正常的UIView一样参照其父控件添加.

正如上面我们第一步所做的,在给scrollView添加子控件之前,那四个约束决定了scrollView的大小和位置,这步是没有问题的.

#### 问题的关键在于如何给scrollView内部的子控件添加约束.

scrollView内部子控件约束的添加需要遵循两个原则:

1、scrollView内部子控件的尺寸不能以scrollView的尺寸为参照
2、scrollView内部的子控件的约束必须完整
首先,子控件的尺寸不能以scrollView的尺寸为参照,那么我们有两种选择:

提供一个具体值的约束(比如200)
子控件的尺寸可以参照scrollView以外其它的控件的尺寸(如控制器的view的尺寸)
其次,约束"完整"的意思是说:子控件在水平及竖直方向上的约束要把scrollView"撑满".

也就是说,在水平方向上,我们需要设置:

子控件左侧与父控件的距离
子控件自身的宽度
子控件右侧距父控件的距离.
竖直方向上也一样,要设置:

子控件顶部距父控件的距离
子控件的高度
子控件底部距父控件的距离.
如图:
![](https://ws3.sinaimg.cn/large/006tKfTcgy1fq6f6vj56sj30ak07q3ye.jpg
)
scrollView子视图的约束条件图1

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fq6f74qa9fj30ar081aa0.jpg
)
scrollView子视图的约束条件图2
两张图片中,所有红色线条的长度都要确定(黄线表示对齐),才能保证AutoLayout不会报错.

为什么scrollView如此不同?
#### 因为scrollView需要根据添加在其内部的子控件的宽高及与四周的距离计算出它的contentSize.

举个栗子:
一个添加在scrollView内部的imageView的宽高为{80, 50}, imageView距离上左下右的距离分别为:100, 200, 300, 400,那么不需要用代码赋值contentSize,我们就可以打印出scrollView的contentSize为{680, 450}.
如图:
![](https://ws4.sinaimg.cn/large/006tKfTcgy1fq6f7uszd1j30bu07ojre.jpg
)
IB添加约束的原理

如果理解起来还是有困难,我们可以把scrollView的contentSize的范围想象成一块UIView(上图中的蓝色区域),暂且叫它container(实际是没有这个东西的).当我们在storyboard或xib中设置子控件与scrollView之间约束时,实际上设置的是子控件与container之间的约束.

也就是说,**子控件的约束决定了container的尺寸(contentSize).**
这就说明了为什么我们要在水平和竖直方向用约束"撑满".如果不撑满,container不知道它自己应该多大.

也正是因为container的尺寸由子控件的约束决定,所以子控件的尺寸不能再反过来参照container的尺寸.不然你等于我,我等于你,那到底是多少呢?如果你是Xcode你也会抓狂.(再次强调那个container不是真的,只是为了方便理解)

理论部分解释完毕,回到一开始的案例上.

imageView有了上下左右四个约束,还缺少宽高,我们再添加个固定宽高的约束(可以大一点,太小了看不到滚动效果),问题就可以解决了.

强调一下, 用代码给scrollView添加约束(包括使用Masonry的情况)是一样的.




