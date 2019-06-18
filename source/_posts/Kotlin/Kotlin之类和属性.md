---
title: Kotlin之类和属性
date: 2019-01-03
updated: 2019-01-03
tags:
- Kotlin
categories: Kotlin
---




# 属性
在类中，声明属性和变量是一样的，使用`val`和`var`关键字，声明成`val`的属性是只读的，而`var`属性是可变的



    class Person{
        val name:String = "满天星"//只读属性，生成一个字段和一个简单的getter
        var age:Int = 20//可写属性，一个字段、一个getter和一个setter
    }
**另外:**
- 使用val声明属性，kotlin会把属性的getter方法暴露给java；
- 使用var声明属性,kotlin会把属性的getter和setter方法都暴露给java
- Kotlin也可以使用java的Person类，并且如果Java类定义了两个名称为 getName和setName的方法，就把它们当做名称为name的属性访问


调用构造方法不需要关键字`new`，可以直接访问属性,但其实调用的是`getter`方法


    fun testPerson() {
        val person = Person("康栋普", 25)
        person.age = 26
        println("name: ${person.name}" + "\n" + "age: ${person.age}")
        }
---

# 自定义访问器

假如声明一个矩形，它能判断自己是否为正方形，不需要单独的字段来存储这个值，而是随时可以通过检查矩形的长度是否相等来判断，
属性`isSquare`不需要字段来保存它的值,它只有一个自定义实现的`getter`

    class Rectangle(val width: Int, val height: Int) {
        val isSquare:Boolean
        get() {
            return width == height
        }
    }

更简洁的写法可以不使用带花括号的完整语法


    class Rectangle(val width: Int, val height: Int) {
        val isSquare: Boolean get() = width == height
    }
