---
layout:     post
title:      内存管理（strong、weak）
subtitle:   strong weak copy retain assign
date:       2018-04-11
author:     BlackBones
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - 内存管理
---
从做iOS开发开始，就对@property的可选参数不怎么了解，具体什么情况用什么，后来我就遵循:如果是对象，就是用strong，基本数据类型使用assign，字符串使用copy，delegate用weak，block使用copy，但其实也要根据具体情况来定的，不能一概而论。
今天好好梳理一下。

## 1、atomic 和 nonatomic

atomic是默认的属性，表示对对象的操作属于原子操作，主要是在多线程的环境下，提供多线程访问的安全。我们知道在多线程的下对对象的访问都需要先上锁访问后再解锁，保证不会同时有几个操作针对同一个对象。如果编程中不涉及到多线程，不建议使用，因为使用atomic比nonatomic更耗费系统资源。nonatomic 表示访问器的访问不是原子操作，不支持多线程访问安全，但是访问性能高。

## 2、readwrite 和readonly

readwrite 是默认的属性，表示可以对对象进行读和写，会生成对象相应的setter和getter方法。readonly 表示只允许读取对象的值，只会生成对象的getter方法。当给readonly类型赋值时，会直接报错。

## 3、retain和assign

假设有一个属性property，我们对其赋值：

```
self.property = newValue;
```
然后分别用retain和assign去修饰。
对于retain，相当于完成了以下几步：

```
if(property != newValue)
{
    [property release];
    property = [newValue retain];
}
```
对于assign，则是：

```
property = newValue;
```
retain会先释放原来的对象，然后给新对象引用计数加1；而assign不增加引用计数，assign的作用是简单赋值，不改变引用计数，对基础数据类型（NSInteger，CGFloat）和C数据类型（int, float, double）等适用。

## 4、strong和weak

当一个对象不再有strong类型引用指向它的时候，它就会被释放，即使该对象还有weak类型引用指向它，并且还指向该对象的weak引用会被置为nil，防止野指针的出现。

在网络上看到一个很形象的例子，我觉得很好，可以看看：

将一个对象类比为一只狗，释放对象类比为狗要跑掉。
strong类型就像是栓住狗的绳子，只要有绳子拴住狗，它就不会跑掉。如果有5条绳子栓一只狗，除非5条绳子都脱落，否则狗是不会跑掉的，就相当于只有每个strong指针都被置为nil时，对象才会被释放。weak类型就像是看着狗的小孩子，只要狗一直被拴着，那么小孩子就能看到狗，会一直指向它，只要狗的绳子脱落，那么狗就会跑掉，不管有多少的小孩在看着它。最后，狗跑掉以后，小孩也就和狗之间没什么联系了。只要最后一个strong型指针不再指向对象，那么对象就会被释放，同时所有的weak型指针都将会被清除。

之所以要引入weak类型，最主要的防止strong类型之间形成循环，使大家都不能释放造成内存泄露。最明显的例子就是delegate，以一个viewcontroller和tableview为例，如果viewcontroller中有strong类型指向tableview，而tableview的delegate指向viewcontroller，如果delegate是strong类型，那么要销毁delegate就要销毁viewcontroller，而要viewcontroller则要先销毁delegate，这样就形成循环了。所以要将delegate声明为weak类型的，这样才能在viewcontroller需要销毁的时候进行销毁。

## 5、retain和strong，assign和weak的关系

retain和strong是一致的（声明为强引用）；assign和weak是基本一致的（声明为弱引用），之所以说它俩是基本一致是因为它俩还是有所不同的，weak严格的说应当叫“归零弱引用”，即当对象被销毁后，会自动的把它的指针置为nil，这样可以防止野指针错误。而assign销毁对象后不会把该对象的指针置nil，对象已经被销毁，但指针还在痴痴的指向它，这就成了野指针，这是比较危险的。 retain和assign是ARC之前使用的，strong和weak这两个都是在ARC下才加入的，strong是默认的修饰符。

## 6、copy

那么多修饰符中，我觉得copy算是比较麻烦的一个了，看别人的代码，大部分的时候NSString的属性都是copy类型，那copy与strong到底有什么区别呢?

首先我们看看将一个属性声明为copy类型时，做一个赋值操作：

self.property = newValue;

其实copy做了这么几件事：

```
if(property != newValue)
{
    [property release];//原本指向的对象技术减1
    property = [newValue copy];//指向新的对象，计数为1
}
```

我们再来看一个更加完整的例子，首先声明两个属性

```
@property (nonatomic, strong) NSString *strongStr;
@property (nonatomic, copy) NSString *cStr;
```

我们进行以下的操作

```
NSString *str1 = @"123";
NSMutableString *str2 = [NSMutableString stringWithString:@"abc"];
//case 1
self.strongStr = str1;
self.cStr = str1;
//case 2
self.strongStr = str2;
self.cStr = str2;
//case 3
[str2 appendString:@"asd"];
```

分别将三种情况下，各指针指向的值、指向的区域和本身存储的区域打印出来： 

![](https://ws1.sinaimg.cn/large/006tNc79gy1fq8ugpnxbzj309n04maa9.jpg)
 
对于case 1，str1是NSString，此时strongStr和cStr都指向str1指向的地方，这种情况下strong和copy类型并没有区别，两种都是是指针拷贝（浅拷贝），因为str1指向的区域内存是immutable的，所以不会产生什么问题。若str转而指向别的区域，对strongStr和cStr是没影响的。我的理解是对于一个immutable对象调用copy方法不会返回新的内存区域。

对于case 2，因为NSMutableString是NSString的子类，所以str2是可以赋值给strongStr和cStr的。strongStr和str2指向同一块区域，而cStr指向另一个内容相同的新区域。相当于strong类型是浅拷贝，copy是深拷贝。

对于case 3, 因为str2是mutable的，所以可以更改str2指向的区域内容，而strongStr与str2指向同一片区域，它的值也会随着一起更改，这在大多数情况下可能并不是我们想要的结果。而cStr因为是指向另一块新区域，所以并不会收到str2的影响，这可能才是我们想要的结果，所以大多数情况字符串都使用copy作为修饰符。

## 7、immutable和copy

我们来看一下这种情况：


```
@property (nonatomic, copy) NSMutableArray *array;

NSArray *array1 = [NSArray arrayWithObjects:@"234", @"323", nil];
NSMutableArray *array2 = [NSMutableArray arrayWithObjects:@"234", @"323", nil];
//case 1
self.array = array1;//将NSArray赋值给NSMutableArray，发出警告

//case 2
self.array = array2;
[self.array addObject:@"321"];//运行时报错
```
case 1好理解，对于case 2，前面说了，copy方法返回的是immutable对象，所以array其实是immutable的，尽管它是mutable类型，修改immutable对象时就会报错。所以对于mutable的属性，我们应该声明为strong类型。

8、关于copy和mutableCopy

再看一个例子

```
NSArray *originArray = [NSArray arrayWithObjects: @"234", @"323", nil];
NSMutableArray *originMutableArray = [NSMutableArray arrayWithArray:originArray];
NSArray *originArrayCopy = [originArray copy];
NSMutableArray *originArrayMutableCopy = [originArray mutableCopy];
NSMutableArray *originMutableArrayCopy = [originMutableArray copy];
NSMutableArray *originMutableArrayMutableCopy = [originMutableArray mutableCopy];

// case 1:运行时错误，immutable对象不可修改
[originMutableArrayCopy addObject:@"321"];
```

各个变量指向的内存区域： 

![](https://ws3.sinaimg.cn/large/006tNc79gy1fq8usfoev3j309x02bmx6.jpg)

这里再一次印证了前面的内容，对一个immutable对象调用copy方法并不会返回新的内存区域。同时copy返回的内存区域是不可改变的。

至此，大部分的修饰符，就理清楚了。
**注：**大部分内容摘抄于网络，再加上一些自己理解，主要是为了自己梳理知识用的，如冒犯立马撤下。


