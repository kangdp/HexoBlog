---
title: Kotlin之数据类
date: 2018-06-13 11:36:56
tags:
- Kotlin
- 数据类
- componentx
- data
categories: Kotlin
---

>数据类是一种非常强大的类，它可以让你避免创建Java中的用于保存状态但又操作非常简单的POJO的模版代码。它们通常只提供了用于访问它们属性的简单的getter和setter。定义一个新的数据类非常简单：

       data class Person(val name:String,val age:Int,val sex:String)

# 额外的函数
通过数据类，我们可以方便地得到很多有趣的函数，一部分是来自属性，我们之前已经讲过（从编写getter和setter函数）

- equals(): 它可以比较两个对象的属性来确保他们是相同的。
- hashCode(): 我们可以得到一个hash值，也是从属性中计算出来的。
- copy(): 你可以拷贝一个对象，可以根据你的需要去修改里面的属性。我们会在稍后的例子中看到。
- 一系列可以映射对象到变量中的函数。我也很快就会讲到这个。

# 复制一个数据类
如果使用不可修改的对象，假如现在需要修改这个对象的状态，必须创建一个或多个属性被修改的实例。这个任务是非常重复且不简洁的。
举个例子，如果我们需要修改Person中的age（年龄），我们可以这么做：

        val person1 = Person("张三",20,"男")
        val person2 = person1.copy(age = 25)

如上，我们拷贝了第一个Person对象然后只修改了age的属性而没有修改这个对象的其它状态。

# 映射对象到变量中
映射对象的每一个属性到一个变量中，这个过程就是所谓的多声明，这就是为什么会有`componentx`函数被自动创建，使用上面的`Person`类举个例子:
 
       val person1 = Person("张三",20,"男")
       val(name,age,sex) = person1

上面这个多声明会被编译成下面的代码:

       val name= f1.component1()
       val age= f1.component2()
       val sex= f1.component3()

这个特性背后的逻辑是非常强大的，它可以在很多情况下帮助我们简化代码。举个例子，Map类含有一些扩展函数的实现，允许它在迭代时使用key和value：

        for ((key,value) in map){
            Log.d("map","key:$key,vales:$value")
        }
