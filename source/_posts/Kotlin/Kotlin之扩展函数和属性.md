---
title: Kotlin之扩展函数和属性
date: 2019-01-07
updated: 2019-01-07
tags:
- Kotlin
categories: Kotlin
---

# 定义扩展函数

在系统的String类中添加一个扩展函数，例如获取字符串中最后一个字符(**注意:扩展函数时声明在类之外的**)

    //String为接收者类型          //this为接收者对象(this可以省略)
    fun String.lastChar() = this.get(this.length-1) //接收者类型是由扩展函数定义的，接收者对象是该类型的一个实例
    
在Kotlin中调用它

    fun testLastChar(){
        println("Kotlin".lastChar())
    }

实际上扩展函数是静态函数，它把调用对象作为了第一个参数。调用这个静态函数，然后把接收者对象作为第一个参数传进来即可，假如它声明在一个叫做`StringUtils.kt`文件中，然后在Java中调用

    /*java*/
    char c = StringUtils.lastChar("Kotlin");
    System.out.println("lastIndex = "+ c);
    

在扩展函数中，可以直接访问被扩展的类的其他方法和属性，就好像是在这个类自己的方法中访问它们一样(**注意：扩展函数不能访问私有的或者是受保护的成员**)

# 导入和扩展函数

如果要在不同包中使用定义好的扩展函数，需要进行导入。Kotlin允许用和导入单个的函数

    import com.kdp.kotlin.lastChar
    val char = "Kotlin".lastChar()
    
也可以使用`*`来导入

    import com.kdp.kotlin.*
    val char = "Kotlin".lastChar()
    
可以使用关键字`as`来修改导入的类或者函数名称

    import com.kdp.kotlin.lastChar as last
    val char = "Kotlin".last()
    
# 作为扩展函数的工具类

现在可以写一个`joinToString`函数的终极版了，它和你在Kotlin标准库中看到的一模一样

    fun <T> Collection<T>.joinToString(separator:String="",prefix:String="",postfix:String="") : String{
        val  result = StringBuilder(prefix)
        for ((index,element) in this.withIndex()){
            if (index > 0) result.append(separator)
                result.append(element)
        }
                result.append(postfix)
            return result.toString()
    }
测试

    fun test(){
        val list = listOf("A","B","C","D","E")
        println( list.joinToString(",","(",")"))
    }
    //输出:(A,B,C,D,E)
    
# 不可重写的扩展函
扩展函数并不是类的一部分，它是声明在类之外的，尽管可以给基类和子类都分别定义一个同名的扩展函数，当这个函数被调用时，它会用到哪一个函数呢?它由该变量的静态类型所决定的，而不是这个变量的运行时类型

# 扩展属性

声明一个扩展属性，和扩展函数一样，扩展属性也像接收者的一个普通的成员属性一样，这里必须定义`getter`函数,因为没有支持字段，因此没有默认`getter`的实现。同理，初始化也不可以，没有地方存储初始化值

    val String.lastChar : Char
        get() = get(this.length-1)
        
可以像访问使用成员属性一样访问它

    println("Kotlin".lastChar)
    
在Java中访问扩展属性需要显示的调用

    StringUtils.getLastChar("Java");