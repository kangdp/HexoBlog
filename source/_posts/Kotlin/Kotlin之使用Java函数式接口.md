---
title: Kotlin之使用Java函数式接口
date: 2019-01-21
updated: 2019-01-21
tags:
- Kotlin
categories: Kotlin
---

# 把lambda当做参数传递给Java方法

可以把`lambda`传给任何期望函数式接口的方法

    <!--Java-->
    void postponeCompution(int delay,Runnable computation);

在Kotlin中，可以调用它并把一个`lambda`作为实参传给它。编译器会自动把它转换成一个`Runnable`的实例：

    postponeCompution(1000){println(42)}
    
通过显示地创建一个实现了`Runnable`的匿名对象也能达到同样的效果

    postponeCompution(1000,object:Runnable{
            override fun run(){
                println(42)
            }
    })
但这里有一点不一样，当你显示地声明对象时，每次调用都会创建一个新的实例。而使用`lambda`，如果`lambda`没有访问任何来自定义它的函数的变量，相应的匿名类实例可以在多次调用之间重用

     postponeCompution(1000){println(42) <!--整个程序只会创建一个Runnable的实例-->

因此，完全等价的实现应该是下面这段代码中的显示`object`声明，它把`Runnable`实例存储在一个变量中，并且每次调用的时候都是用这个变量：
    
    val runnable = Runnable{ println(42) } <!--编译成全局变量：程序中仅此一个实例-->
    fun handleCompution(){
        postponeCompution(1000,runnable) <!--每次postponeCompution调用时用的是一个对象-->
    }
    
如果`lambda`从包围它的作用域中捕捉了变量，每次调用就不再可能重用同一个实例了。这种情况下，每次调用时编译器都会创建一个新对象

    fun handleCompution(id:String){ <!--lambda捕捉id这个变量-->
        postponeCompution(1000){println(id)} <!--每次handleCompution调用时都会创建一个Runnable的新实例-->
    }

# SAM构造方法：显示地把lambda转换为函数式接口

`SAM`构造方法是编译器生成的函数，让你执行从`lambda`到函数式接口实例的显示转换。如果一个方法返回的是一个函数是接口的实例，不能直接返回一个`lambda`，要用`SAM`构造方法把它包装起来

    fun createAllDoneRunnable() : Runnable{
        return Runnable{ println("All done!") }
    }

`SAM`构造方法只接收一个参数：一个被用作函数式接口单抽象方法体的`lambda`，并返回实现这个接口的类的一个实例

    val listener = OnClickListener { view ->
        
        val text = when(view.id){
            R.id.button1 -> "First button"
            R.id.button2 -> "Second button"
            else -> "Unknow Button"
        }
        
        toast(text)
    }
    
    button1.setOnClickListener(listener)
    button2.setOnClickListener(listener)
