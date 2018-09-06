---
layout:     post
title:      Instruments的学习
subtitle:   Instruments 
date:       2017-06-05
author:     BlackBones
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - Instruments  Time Profiler  Zombies Allocations Leaks
---

# Instruments的学习
Instruments是用来动态跟踪和分析MacOS X和iOS代码的实用工具。
有网友说：**Instruments的价值在于，它使我们深刻理解我们代码的内部运作**。
我觉得说得很好。👏👏鼓掌
Instruments里面工具很多，我也不知道，说几个我觉得还比较有用的
* Time Profiler:分析代码的执行时间，找出导致程序变慢的原因。
* Zombies:检查是否访问了僵尸对象,但是这个工具只能从上往下检查,不智能（我反正没用过）
* Allocations:用来检查内存分配,写算法的那批人也用这个来检查
* Leaks:检查否有内存泄露，对付复杂的循环引用没用

至于Instruments如何打开，这里就不说了。
## 1、Time Profiler  时间分析器 
用来检测app中每个方法所用的时间，并且可以排序，并查找出哪些函数占用了大量时间。页面如下：
![](https://upload-images.jianshu.io/upload_images/1440398-419a4530df61be5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/462)
使用Time Profile前有两点需要注意的地方：
1. 一定要使用真机调试
在开始进行应用程序性能分析的时候，一定要使用真机。因为模拟器运行在Mac上，然而Mac上的CPU往往比iOS设备要快。相反，Mac上的GPU和iOS设备的完全不一样，模拟器不得已要在软件层面（CPU）模拟设备的GPU，这意味着GPU相关的操作在模拟器上运行的更慢，尤其是使用CAEAGLLayer来写一些OpenGL的代码时候，这就导致模拟器性能数据和用户真机使用性能数据相去甚远

2. 应用程序一定要使用发布配置
在发布环境打包的时候，编译器会引入一系列提高性能的优化，例如去掉调试符号或者移除并重新组织代码。另iOS引入一种"Watch Dog"[看门狗]机制，不同的场景下，“看门狗”会监测应用的性能，如果超出了该场景所规定的运行时间，“看门狗”就会强制终结这个应用的进程。开发者可以crashlog看到对应的日志，但Xcode在调试配置下会禁用"Watch Dog"
![](https://upload-images.jianshu.io/upload_images/1440398-51fd6c89d63d6e6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/605)

下面解释了每一个选项对左侧列表中数据的显示起了什么作用:

**Separate by Thread**：每个线程被单独考虑。这能让你知道哪一个线程占用CPU最多。

**Invert Call Tree**：选中该选项后，调用栈会自上至下显示。这通常是你需要的，因为你想知道CPU花费时间的那个最深的方法。

**Hide System Libraries**：选中该选项后，只有你自己app中出现的符号会被显示出来。通常选中该选项是有用的，因为你只关心CPU在你自己的代码中的哪一部分花费时间，你没法对系统库使用CPU做多少改变。

**Flatten Recursion**：该选项将每一个调用栈中的递归函数（调用它们自身的函数）视作单一入口，而不是多入口。

**Top Functions**：选上这一选项让Instruments将花费在一个函数中的总时间视作在该函数中直接花费的时间加上调用的其他函数花费的时间。所以如果函数A调用了函数B，那么函数A花费的总时间被记为A花费的时间加上B花费的时间。这一选项非常有用，因为它能让你在每次进入调用栈时找到花费最长的时间，瞄准你最耗时的方法。

看到网上有人说主线程耗时过多进行优化的，有的是网络请求耗时过多进行优化的，有的是UIImage耗时过多进行优化的，总之，可以看到哪个函数耗时多，进而优化，在这里不由得想起了本文开头提到的一位网友说的：**Instruments的价值在于，它使我们深刻理解我们代码的内部运作**。

**总结**：性能优化是在所有更能实现完成时要做的事，使用Time Profile工具分析app每个流程的执行情况，发现耗时的地方，合理优化，提升用户体验，切记，优化后要做一遍详细的测试，别修了东墙坏了西墙。

## 2、Zombies  僵尸

翻译英文：专注于检测过度释放的“僵尸”对象。还提供了数据对象分配的类以及所有活动分配内存地址的历史。

这里我们可以看到一个词语叫“over-release”,过度释放。我们在项目中见到最多的就是“EXC_BAD_ACCESS”或者是这样的：Thread 1: Program received signal:"EXC_BAD_ACCESS"，这就是访问了被释放的内存地址造成的。过度释放，是对同一个对象释放了过多的次数，其实当引用计数降到0时，对象占用的内存已经被释放掉，此时指向原对象的指针就成了“悬垂指针”，如若再对其进行任何方法的调用，（原则上）都会直接crash（然而由于某些特殊的情况，不会马上crash）。过度释放简单的说就是对release的对象再release，就是过度释放。
重新回顾几个概念：
1. 内存泄漏：对象使用完没有释放，导致内存浪费。
2. 僵尸对象：已经被销毁的对象(不能再使用的对象)
3. 野指针：指向僵尸对象(不可用内存)的指针。给野指针发消息会报**EXC_BAD_ACCECC**错误。
4. 空指针：没有指向储存空间的指针(里面存的是nil,也就是0)。在oc中使用空指针调中方法不会报错。
**温馨提示**:通常为了避免野指针的常见方法是在对象被销毁之后,将指向对象的指针变为空指针。

Zombies暂时不说吧，我也不太明白。


## 3、Allocations 分配
循环引用导致的内存泄漏，Leaks工具查不出来的。
可以利用Allocations进行简单的方法检测内存泄漏。
接下来我们根据内存泄漏的情况对内存分配进行分析，内存泄漏分两种：

1. 为对象A申请了内存空间，之后再也没用到A，也没有释放A导致内存泄漏。这种是Leaked Memory内存泄漏。
2. 类似于递归，不断的申请内存导致的内存泄漏。根据下边的链接文章我们知道这种情况就是Abandoned Memory被遗弃的内存。
第二种情况根据以下图的操作可以清晰的找到对应的问题代码，当然不一定是我们自己代码的原因，也有可能是系统框架的问题。
![](https://upload-images.jianshu.io/upload_images/1440398-d7b18f84b7056f06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
![](https://upload-images.jianshu.io/upload_images/1440398-5e47bfea247d855a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/988)
下边是关于寻找这个Abandoned Memory被遗弃的内存的方法：
![](https://upload-images.jianshu.io/upload_images/1440398-520b2f9411c5d60f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
![](https://upload-images.jianshu.io/upload_images/1440398-654d8bb7d708a585.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
我没有亲测，不过看着挺可靠的，具体的步骤告诉了，按照步骤一步一步的走下来应该就能找到Abandoned Memory（被遗弃的内存）。

## 4、Leaks 泄漏
首先还是来说哈内存溢出和内存泄漏：
**内存溢出 out of memory**：是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory；比如申请了一个integer,但给它存了long才能存下的数，那就是内存溢出。
**内存泄露 memory leak**：是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光。
**memory leak会最终会导致out of memory！**

在前面的ALLcations里面我们提到过内存泄漏就是应该释放而没有释放的内存。而内存泄漏分为两种：Leaked Memory 和Abandoned Memory。前面我们讲到了如何找到Abandoned Memory被遗忘的内存，现在我们研究的就是Leaked Memory。
![](https://upload-images.jianshu.io/upload_images/1440398-0bd1c8d1995ffa51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/965)

下边我们介绍Instruments里面的Leaked的用法，首先打开Leaked，跑起工程来，点击要测试的页面，如果有内存泄漏，会出现下图中的红色的❌。然后按照后边的步骤进行修复即可。
![](https://upload-images.jianshu.io/upload_images/1440398-482e80e6ff4390db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
![](https://upload-images.jianshu.io/upload_images/1440398-2a353cef7eb3812d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
下图是对Leaked页面进一步的理解：
![](https://upload-images.jianshu.io/upload_images/1440398-cfe69945d6a4e955.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
当然了，关于内存泄漏我们还可以用 command ＋shift ＋B 的方式进行检测，这个快捷键调起的是内存管理器**Analyze**。
关于内存管理器Analyze解决问题可以参考
[APP Analyze（静态分析）](https://www.jianshu.com/p/9e2789e23f67)
[Analyze问题解决](https://www.jianshu.com/p/264e570cff0a)

使用上述两种方法（1）**Instruments－Leaked **（2）内存管理器**Analyze ** 来检查内存泄漏，是我们最常用的两种方法。

站在巨人的肩膀上 [Instruments性能检测](https://www.jianshu.com/p/9e94e42cfb01)
