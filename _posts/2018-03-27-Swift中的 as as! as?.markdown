---
layout:     post
title:      Swift中的 as as! as?
subtitle:   as as! as?
date:       2018-03-27
author:     BlackBones
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - Swift
---

## as
使用场景:
1. 从派生类转换为基类，向上转型（upcasts）

```
// 定义人员基类
class Person {
    var name : String
    
    init(_ name: String){
        self.name = name
    }
}

// 定义学生类
class Student : Person {
}

// 定义教师类
class Teacher : Person {
}

// 处理人员对象的函数(或工厂模式处理操作等)
func showPersonName(_ people : Person){
    let name = people.name
    print("这个人的名字是: \(name)")
}

// 定义一个学生对象 tom
var tom = Student("Tom");

// 定义一个教师对象 kevin
var kevin = Student("Kevin Jakson");

// 先把学生对象向上转型为一般的人员对象
let person1 = tom as Person
let person2 = kevin as Person

// 再调用通用的处理人员对象的showPersonName函数
showPersonName(person1)
showPersonName(person2)
```
运行结果：
这个人的名字是: Tom
这个人的名字是: Kevin Jakson


2. 消除二义性，数值类型转换


```
let age = 28 as Int
let money = 20 as CGFloat
let cost = (50 / 2) as Double
```
3. switch 语句中进行模式匹配，通过switch语法检测对象的类型，根据对象类型进行处理。
4. 
```
switch person1 {
    case let person1 as Student:
        print("是Student类型，打印学生成绩单...")
    case let person1 as Teacher:
        print("是Teacher类型，打印老师工资单...")
    default: break
}
```
运行结果:
是Student类型，打印学生成绩单...

## as！
向下转型（Downcasting）时使用。由于是强制类型转换，如果转换失败会报 runtime 运行错误。

```
let person : Person = Teacher("Jimmy Lee")
let jimmy = person as! Teacher
```
## as？
as? 和 as! 操作符的转换规则完全一样。但 as? 如果转换不成功的时候便会返回一个 nil 对象。成功的话返回可选类型值。由于 as? 在转换失败的时候也不会出现错误，所以对于如果能确保100%会成功的转换则可使用 as!，否则使用 as?

```
let person : Person = Teacher("Jimmy Lee")

if let someone = person as? Teacher{
    print("这个人是教师, 名字是 \(someone.name)")
} else {
    print("这个人不是教师, 'tom'的值是 nil")
}
```
运行结果:
这个人是教师, 名字是 Jimmy Lee


