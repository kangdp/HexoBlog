---
title: Kotlin之类和函数
date: 2018-06-13 11:36:44
tags:
- Kotlin
- fun
- Any
- open
categories: Kotlin
---

# 怎么定义一个类
如果你想定义一个类，你只需要使用`class`关键字，这一点和**java**一样。
   
       class Kotlin{
       
       }
它有一个默认的构造器，而且大部分你也只需要这个默认的构造器。你只需要在类名后面写上它的参数。如果这个类中没有任何内容，可以省略大括号。

       class Kotlin(a: String，b: String)

那么构造函数的函数体你可以写在`init`块中

       class Kotlin(a: String，b: String){

             init{
                ...
             }

       }

# 类继承
默认任何类都是继承`Any`(与java中的object类似)，但是我们可以继承其它类，所有的类默认都是不可继承的(final)，所以我们只能继承那些明确声明`open`或者`abstract`的类：

     open class Parent(a:String) //定义一个父类可以被子类继承

     class Child(a:Int,b:Int) : Parent(a.toString())  //继承
    {
            init {
               // ....
            }
    }
当我们只有单个构造器时，我们需要在从父类继承下来的构造器中指定需要的参数。这是用来替换Java中的super调用的。

# 函数
函数使用`fun`关键字来定义，例如定义一个`add`方法：

    fun add(a:Int,b:Int){
        println(a+b)
    }
关于返回值，默认返回`Unit`，相当于Java中的`void`，但是`Unit`是一个真正的对象。如果想要指定返回值，那么如下所示：

     fun add(a:Int,b:Int) : Int{
         return a+b
     }
>**提示:分号不是必须的**

然而如果返回的结果可以使用一个表达式计算出来，你可以不使用括号而是使用等号：
 
      fun add (a:Int,b:Int) : Int = a + b

#构造方法和函数参数
Kotlin中的参数与Java中的有些不同，Kotlin是先写参数的名字再写它的类型：

      fun add(a:Int,b:Int) : Int{
         return a+b
     }
我们可以给参数指定一个默认值，这里有一个例子，在Activity中创建了一个函数用来toast一段信息
     
      fun toast(message: String, length: Int = Toast.LENGTH_SHORT) {
       Toast.makeText(this, message, length).show()
      }
如你所见，我们给第二个参数(length)制定了默认值，当你在调用方法的时候第二个参数可以传或者不传，这样可以避免你需要的重载函数
  
     toast("Hello")
     toast("Hello", Toast.LENGTH_LONG)

这个与下面的Java代码是一样的：

     void toast(String message){
     }

     void toast(String message, int length){
     Toast.makeText(this, message, length).show();
     }

这跟你想象的一样复杂。再看看这个例子：

     fun niceToast(message: String,
                tag: String = javaClass<MainActivity>().getSimpleName(),
                length: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, "[$className] $message", length).show()
    }

我增加了第三个默认值是类名的tag参数。如果在Java中总数开销会以几何增长。现在可以通过以下方式调用：

    toast("Hello")
    toast("Hello", "MyTag")
    toast("Hello", "MyTag", Toast.LENGTH_SHORT)
而且甚至还有其它选择，因为你可以使用参数名字来调用，这表示你可以通过在值前写明参数名来传入你希望的参数：

      toast(message = "Hello", length = Toast.LENGTH_SHORT)

>**小提示：String模版
你可以在String中直接使用模版表达式。它可以帮助你很简单地在静态值和变量的基础上编写复杂的String。在上面的例子中，我使用了"[$className] $message"。
如你所见，任何时候你使用一个$符号就可以插入一个表达式。如果这个表达式有一点复杂，你就需要使用一对大括号括起来："Your name is ${user.name}"。**
