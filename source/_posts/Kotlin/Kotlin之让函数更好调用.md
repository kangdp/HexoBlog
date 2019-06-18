---
title: Kotlin之让函数更好调用
date: 2019-01-07
updated: 2019-01-07
tags:
- Kotlin
categories: Kotlin
---

声明一个函数:使用分隔符分割每一个元素

        
    fun <T> joinToString(collection: Collection<T>,separator:String,prefix:String,postfix:String) : String{
        val result = StringBuilder(prefix)
        for ((index,element) in collection.withIndex()){

        if (index > 0) result.append(separator)
         result.append(element)
        }
            result.append(postfix)
        return result.toString()
    }
    
为了提高函数调用的可读性，当调用一个Kotlin定义的函数时，可以显示的标注一些参数的声明。如果在调用函数时，指明了一个参数的名称，那么它之后的所有参数都需要标明名称，就像下面这样(注意:当调用Java的函数时，不能采用命名参数)

    fun testJoinToString(){
        val resutl = joinToString(collection = arrayListOf("10","20","50","80"),separator = ",",prefix = "(",postfix = ")")
        println(resutl)
    }
    
# 默认参数值

在声明函数的时候，指定参数的默认值，来避免创建重载的函数，修改`javaToString`函数的参数，如下

    fun <T> joinToString(collection: Collection<T>,separator:String = "",prefix:String = "",postfix:String = ""):String{
        val result = StringBuilder(prefix)
        for ((index,element) in collection.withIndex()){

            if (index > 0) result.append(separator)
            result.append(element)
        }
            result.append(postfix)
        return result.toString()
    }
    
当使用常规的调用语句时，必须按照函数声明定义的参数顺序来给定参数，可以省略的只有排在末尾的参数

    fun testJoinToString(){
        val list = listOf("1","2","3")
        joinToString(list,"","","")
        joinToString(list,",","(")
        joinToString(list)
    }
如果使用命名参数，则可以省略中间的一些参数,也可以以你想要的任意顺序只给定你需要的参数

    fun testJoinToString(){
        val list = listOf("1","2","3")
        joinToString(list,prefix = "(",postfix = ")")
    }
    
考虑到Java中没有参数默认值的概念，当从Java中调用Kotlin函数的时候，必须显示地指定所有参数值，
如果需要从Java代码中频繁的调用，而且希望它能对Java的调用更简洁，可以使用`@JvmOverloads`注解它。
这个注解表示编译期生成Java重载函数，从最后一个开始省略每个参数，就像这样

    @JvmOverloads
    fun <T> joinToString(...):String{...}
    
# 消除静态工具类：顶层函数和属性

在Java中，如果有很多公共的函数，我们一般需要创建一个公共类，如下

    package com.kdp.java;
    public class JoinKt {
        public static String joinToString(...){
        ...
        }
    }
但是在Kotlin中，根本不需要创建这些无意义的类。你可以将这些函数直接放到代码文件的顶层，不用从属任何类。
这些放到文件顶层的函数依然是包内的成员,此时还需要改变包含Kotlin顶层函数的生成的类的名称
需要为这个文件添加`@JvmName`的注解，将其放到这个文件的开头，位于包名的前面

    @file:JvmName("AppUtils")   //注解指定类名
    package com.kdp.kotlin //包的声明跟在文件注解之后
    fun joinToString(...):String{...}
然后在Java中这样调用

    /*java*/
    import com.kdp.kotlin.AppUtils;
    public class Test {
        public static void main(String[] args){
            AppUtils.joinToString(...);
        }
    }
    
顶层属性，和函数一样，属性也可以放到文件的顶层，如下用var属性来计算一些函数被执行的次数

    var opCount = 0; //声明一个顶层属性

    fun performOperation(){
        opCount++      //改变该属性的值
    }
    fun reportOperationCount(){
        println(opCount) //读取该属性值
    }
像上面的`opCount`值会被存储在i个静态的字段中，也可以在代码中用顶层属性来定义常量

    val UNIX_LINE_SEPARTOR = "\n"
    
顶层属性可以通过访问器暴露给Java使用(如果是val就只有一个getter,如果是val就对应一对getter和setter)

    /*java*/
    AppUtils.getUNIX_LINE_SEPARTOR()

把一个常量以`public static final `的属性暴露给Java，可以使用`const`来修饰它
    
    /*kotlin*/
    const val UNIX_LINE_SEPARTOR = "\n"
    
等同于下面的Java代码

    /*java*/
    public static final String UNIX_LINE_SEPARTOR = "\n";
