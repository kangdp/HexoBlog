---
title: Kotlin之函数和变量
date: 2019-01-03
updated: 2019-01-03
tags:
- Kotlin
categories: Kotlin
---

# 引言
> 自从Kotlin出来后，还没有好好地去学习它，而它带来的优势也是我所向往的。如今已经有很多大公司已经开始使用它进行项目开发了，作为一名Android开发工程师，接下来肯定是要学习一番，毕竟多掌握一门语言对本身而言也是一种优势。那么接下来，进入我的Kotlin学习之旅。

从一个经典的例子开始,打印一个`Hello World`,代码如下:

    fun main(args:Array<String>){
        println("Hello World")  
    }
从上面的例子可以明显的看出**Kotlin**和**Java**在声明函数上的不同:
- kotlin使用关键词**fun**来声明一个函数
- 可以省略每行代码后的分号
- 参数的类型写在名称的后面
- 使用`println(...)`代替了Java中的`System.out.println(...)`
---

# 函数

声明一个具有返回值的函数，求最大值

    fun max(a:Int,b:Int):Int {
        return if(a > b) a else b  //函数体
    }
    
当方法内部为表达式时，可用表达式作为完整的函数体，并去掉花括号和`return`语句
    
    fun max(a:Int,b:Int):Int = if(a > b) a else b //表达式函数体

还可以进一步优化`max`函数,省掉返回类型(**只有表达式函数体的返回值类型才可以省略**)

    fun max(a:Int,b:Int) = if(a > b) a else b
---
# 变量

在**Java**中声明变量的时候以类型开始，而**Kotlin**刚好相反，以关键字开始，然后是变量名称，最后加上类型(不加也可以)

    val question = "The Ultimate Question"
    val a = 1 //省略了类型
    val b : Int = 2 //不省略类型
    //如果使用浮点类型,则变量默认是 Double 类型
    val number = 7.5e6 // >> 7.5 * 10^6
---
# 字符串模板

使用`$`符号加变量名

    fun testStrTemplate(){
        val a = 10
        val  b = 15
        val  name = if (a > b) a else b
        println("Hello,$name")
    }

更复杂的表达式，需要用花括号括起来

    fun testStrTemplate2(){
        val array = arrayOf("a","b","c","d","e")
        println("测试: ${array[0]}")
    }
    
在双引号中嵌套双引号

    fun testStrTemplate3(){
        val array = arrayOf(1,2)
        println("测试: ${if (array[0] > 10) array[0] else "someone"}!")
    }
    




    


