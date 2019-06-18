---
title: Kotlin之创建集合
date: 2019-01-06
updated: 2019-01-06
tags:
- Kotlin
categories: Kotlin
---

# List

创建不可变的List集合,list引用不能通过 list[0] = "4" 这样的方式修改集合中的内容，尽管`listOf`返回的是一个`ArrayList`集合，但它不能改变对象内部的值

        var list = listOf("1","2","3")

创建一个可变的arrayList集合,可修改集合内部的值，如下修改集合中第0个元素的内容，然后分别打印出修改前后的集合

    val arrayList = arrayListOf("1","2","3")
        println("修改前 ${arrayList}")
        arrayList[0] = "0"
        println("修改后 ${arrayList}")

    打印
        修改前 [1, 2, 3]
        修改后 [0, 2, 3]

# Set和Map
        
`Set`、`Map`和`List`一样，它们的`setOf`和`mapOf`方法分别返回的是不可变的集合对象，除此之外的方式创建的集合对象都是可变的

使用类似的方法创建`Set`和`Map`集合，当然还有很多不同的集合，它们的创建方式都类似，这里就不一一列举了

    val set = hashSetOf(1,7,53)
    val map = hashMapOf(1 to "one",7 to "seven",53 to "fifty-three")
    //注意,to并不是一个特殊的结构，而是一个普通的函数，后面会探讨它
打印出对象的类型
    
    println(set.javaClass)
    class java.util.HashSet
    println(map.javaClass)
    class java.util.HashMap
    
从输出的内容可以看出，Kotlin并没有采用自己的集合类，而是使用标准的Java集合类，Kotlin可以更容易的和Java进行交互。但Kotlin不止如此,例如通过以下方式可以获取列表中最后一个元素或者可以获取到数字列表中最大值

     val strings = listOf("first","second","fourteenth")
     println("最后一个元素：${strings.last()}")
     val numbers = setOf(1,14,2)
     println("最大值：${numbers.max()}")
打印
    
    最后一个元素：fourteenth
    最大值：14

当然Kotlin还有更多其它的操作，在后续的章节中会详细讲解。

    

    

