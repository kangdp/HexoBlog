---
title: Kotlin之惰性集合操作：序列
date: 2019-01-19
updated: 2019-01-19
tags:
- Kotlin
categories: Kotlin
---

在使用集合函数比如`map`和`filter`。这些函数会及早的创建中间集合，也就是说每一步的中间结果都被存储在一个临时列表。而序列给了你另一种选择，可以让你避免创建这些临时的中间对象。如下一个例子：

    val peoples = listOf(Person("Alice",20),Person("Bob",31))
    peoples.map(Person::name).filter { it.startsWith("A") }

上面的例子中`map`和`filter`都会返回一个列表，也就是说链式的调用会创建两个列表，如果源列表中只有两个元素，这不是什么问题，但如果有一百万个元素，(链式)调用就会变得十分低效。
为提高效率，可以把操作变成使用序列，而不是直接使用集合

     peoples.asReversed() <!--把初始集合转换成序列-->
        .map ( People::name )  <!--序列支持和集合一样的API-->
        .filter {it.startsWith("A") }
        .toList() <!--把结果序列转换回列表-->

Kotlin惰性集合操作的入口就是`Sequence`接口，它的强大之处就在于其操作的实现方式，序列中元素的求值是惰性的。因此可以使用序列高效地对集合元素执行链式操作，而不需要创建额外的集合来保存过程中产生的中间结果。
可以调用扩展函数`asSequence`把任意集合转换成序列，调用`toList`来做反向的转换

# 中间和末端操作

序列操作分为两类：中间和末端。一次中间操作返回的是另一个序列，这个新序列知道如何变换原始序列中的元素。而一次末端操作返回的是一个结果 

![](https://upload-images.jianshu.io/upload_images/2349677-7495f4baeb2e96ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

中间操作始终是惰性的，看看下面这个缺少了末端操作的例子

    listOf(1,2,3,4).asSequence()
        .map{print("map ($it)");it * it}
        .filter { print("filter($it)");it % 2 == 0 }
执行以上这段代码不会在控制台输出任何内容。这意味着`map`和`filter`变换被延期了，它们只有在获取结果的时候caihui被应用(即末端操作被调用的时候)

在使用`map`和`filter`进行变换时，先应用`filter`有助于减少变换的次数。如果`map`在前，每个元素都被变换。而如果`filter`在前，不合适的元素会被尽早地过滤掉且不会发生变换


# 创建序列

使用`generateSequence`函数创建序列，给定序列中的前一个元素，这个函数会计算下一个元素

        val naturalNumbers = generateSequence(0){it + 1}
    val numberTo100 = naturalNumbers.takeWhile { it <= 100 }
    println(numberTo100.sum()) <!--在获取结果`sum`时，所有被推迟的操作都被执行-->


创建使用父目录的序列，如果元素的父元素和它的类型相同，你可能会对它的祖先组成的序列的物质感兴趣。下面的例子可以查询文件是否放在隐藏目录中，通过创建一个其父目录的序列并检查每个目录的属性来实现

    fun File.isInsideHiddenDirectory() =
        generateSequence(this){ it.parentFile}.any{ it.isHidden }
        
    val file = File("/Users/svtk/.HiddenDir/a.txt")
    println(file.isInsideHiddenDirectory())

