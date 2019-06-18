---
title: Kotlin之Object关键字：将声明一个类与创建一个实例结合起来
date: 2019-01-17
updated: 2019-01-17
tags:
- Kotlin
categories: Kotlin
---

# 对象声明：创建单例易如反掌
Kotlin使用对象声明功能能将类声明与该类的单一实例声明结合到一起。需要注意的是对象声明可以包含属性、方法、初始化语句块等的声明，唯一不允许的就是构造方法。

    object Payroll{
        val allEmployee = arrayListOf<Person>()
        fun caculateSalary(){
            for (person in allEmployee){
            ...
            }
    }
    }
    
    Payroll.allEmployee.add(Person("kdp",20))
    Payroll.caculateSalary()
    
对象声明同样可以继承类和接口

    object CaseInsensitiveFileComparator : Comparator<File>{
        override fun compare(o1: File, o2: File): Int = o1.path.compareTo(o2.path,true)
    }
同样也可以在类中声明对象

# 伴生对象：工厂方法和静态成员

在类中定义的对象可以使用一个特殊的关键字来标记：`companion`。这样就可以直接通过容器类名称来访问这个对象的方法和属性。不需要显示的指明对象的名称。有点类似于Java中的静态方法的调用

    class A{
        companion object {
            fun bar(){
            println("This is a bar!")
            }
        }
    }
    
    A.bar()
    
伴生对象可以访问类中的所有private成员，包括private构造方法

    class User private constructor(val nickname:String){

        companion object{
            fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
            fun newFacebookUser(account:String) = User("facebook")
        }
    }
    
    
    val subscribing = User3.newSubscribingUser("Subscribing")
    val faceBook = User3.newFacebookUser("Facebook")
    println(subscribing.nickname)
    println(faceBook.nickname)
    
# 在伴生对象中实现接口

    interface JSONFactory<T>{
        fun fromJSON(jsonStr:String) : T
    }
    class Person(val name:String){
        companion object : JSONFactory<Person>{
            override fun fromJSON(jsonStr: String) : Person = ...
        }
    }
    
    fun <T>loadFromJSON (fact:JSONFactory<T>) : T = fact.fromJSON("kdp")
    
    loadFromJSON(Person) <!--将伴生对象实例传给它-->
    注意：Person类的名字被当做JSONFactory的实例

# 伴生对象扩展

伴生对象同样可以用到扩展函数上

    class Person(val firstName:String,val lastName:String){
        companion object{}
    }
    
    fun Person.Companion.fromJSON(json:String): Person{...} <!--声明一个扩展函数-->
    
    val p = Person.fromJSON(json)
    
# 对象表达式：改变写法的匿名内部类

使用匿名内部类实现事件监听

    window.addMouseListener{
        object : MouseAdapter(){
            override fun mouseClicked(e:MouseEvent){
                ...
            }
            
            override fun mouseEntered(e:MouseEvent){
                ...
            }
        }
    }
    
除了去掉了对象的名称外，语法和对象声明是相同的，对象表达式声明了一个类并创建了该类的一个实例。

可以给对象分配一个名字，将其存储在一个变量中

    val listener = object : MouseAdapter(){
            override fun mouseClicked(e:MouseEvent){
                ...
            }
            
            override fun mouseEntered(e:MouseEvent){
                ...
            }
        }
        
与Java匿名内部类不同的是：Kotlin的匿名对象可以实现多个接口或者不实现接口

**注意**: 与对象声明不同，匿名对象不是单例的。每次对象表达式被执行都被创建一个新的实例。
