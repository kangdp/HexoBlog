---
title: Kotlin之可变参数、中缀调用和解构声明
date: 2019-01-08
updated: 2019-01-08
tags:
- Kotlin
categories: Kotlin
---

# 可变参数
当你调用一个函数创建列表时，可以传递任意个数的参数给它：
    
    val list = listOf("a","b","c","d")
点进`listOf`方法内部你会发现

    public fun <T> listOf(vararg elements: T): List<T> {...}
你可能对Java中的可变参数已经很熟悉了。Kotlin的可变参数与Java类似，但语法略有不同：
- Kotlin在该类型后不会再使用三个点，而是在参数上使用`vararg`修饰符
- 当需要传递的参数已经被包装到数组中。在Java中可以按照原样传递数组；而在Kotlin中要求你显示的解包数组，以便每个数组元素能够在函数中作为单独的参数来调用，不过需要再对应的参数前面加一个`*`

# 中缀调用、解构声明

先看一个例子，使用`mapOf`函数创建一个`map`

    val map = mapOf(1 to "a",2 to "b",3 to "c")

这段代码中的单词`to`不是内置结构，而是一种特殊的函数调用，被称为**中缀调用**。

以下这两种方式是等价的：

    "A".to(1) //一般to函数的调用
    "A" to 1 //使用中缀符号调用to函数
    
中缀函数可以与只有一个参数的函数一起使用，并且普通函数和扩展函数都可以使用。`to`函数会返回一个`Pair`类型的对象

    infix fun Any.to(other:Any) = Pair(this,other)

可以直接使用`Pair`的内容来初始化两个变量，这个功能被称为**解构声明**

    val (number,name) = 1 to "one"
![](https://upload-images.jianshu.io/upload_images/2349677-c151bc84a169ba7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
用**to**函数创建一个**pair**，然后用解构声明来展开


也适用于循环，例如使用`withIndex`函数的实现

    val list = listOf("A","B","C","D")
    for ((key,value) in list.withIndex())
        println("key = $key   value = $value")
