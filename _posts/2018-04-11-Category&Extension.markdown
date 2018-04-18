---
layout:     post
title:      Category&Extension
subtitle:   category extension
date:       2018-04-12
author:     BlackBones
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - 基础知识 Category Extension
---
以前老是对分类（Category）和扩展（Extension）概念有点模糊，这次好好梳理一下。
## 1. Category简介
category是Objective-C 2.0之后添加的语言特性。category的主要作用是为已经存在的类添加方法。除此之外，apple还推荐了category的另外两个使用场景1

* 可以把类的实现分开在几个不同的文件里面。这样做有几个显而易见的好处，a)可以减少单个文件的体积 b)可以把不同的功能组织到不同的category里 c)可以由多个开发者共同完成一个类 d)可以按需加载想要的category 等等。
* 声明私有方法

不过除了apple推荐的使用场景，广大开发者脑洞大开，还衍生出了category的其他几个使用场景：

* 模拟多继承
* 把framework的私有方法公开
（坦白说，我好想没用过）

## 2. 与Extension对比

扩展（extension）看起来很像一个匿名的category，但是extension和有名字的category几乎完全是两个东西。 extension在编译期决定，它就是类的一部分，在编译期和头文件里的@interface以及实现文件里的@implement一起形成一个完整的类，它伴随类的产生而产生，亦随之一起消亡。extension一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加extension，所以你无法为系统的类比如NSString添加extension。

但是category则完全不一样，它是在运行期决议的。
就category和extension的区别来看，我们可以推导出一个明显的事实，extension可以添加实例变量，而category是无法添加实例变量的（因为在运行期，对象的内部结构已经确定，如果添加实例变量就会破坏类的内部结构，这对编译型语言来说是灾难性的）。

## 3. 详解

Category的在OC中的具体实现，这里就不做讨论了。（关键是我也不知道，留给以后吧）
从上面的对比中，我们可以发现Category里面是无法添加实例变量的，但是我们很多时候需要在category中添加和对象关联的值，这个时候可以使用runtime，利用关联对象来实现。
举个栗子：

```
#import "MyClass.h"

@interface MyClass (Category1)

@property(nonatomic,copy) NSString *name;

@end
```
此时这个属性并不存在getter和setter方法，还需要在.m文件里去实现，并关联。


```
#import "MyClass+Category1.h"
#import <objc/runtime.h>

@implementation MyClass (Category1)

+ (void)load
{
    NSLog(@"%@",@"load in Category1");
}

- (void)setName:(NSString *)name
{
    objc_setAssociatedObject(self,
                             "name",
                             name,
                             OBJC_ASSOCIATION_COPY);
}

- (NSString*)name
{
    NSString *nameObject = objc_getAssociatedObject(self, "name");
    return nameObject;
}

@end
```
这样，变量与对象就关联起来了。但是这个关联的变量与常规的不同，不能以下划线+属性名的方式访问。

**那么这些关联的变量合适被销毁呢？？？**

查了一下资料，这些关联的对象都是由单独的类管理起来的，对象在销毁时，会首先判断这个对象是否有关联的对象，如果有，就会调用管理类来清理。



