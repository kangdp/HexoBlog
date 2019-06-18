---
title: Kotlin之枚举和when
date: 2019-01-04
updated: 2019-01-04
tags:
- Kotlin
categories: Kotlin
---


# 声明枚举类

## 声明简单的枚举

    
    enum class Color{
        RED,ORANGE,YELLOW,GREEN
    }

## 声明带属性的枚举，在每个常量创建的时候指定属性值

    enum class Color2(val r:Int,val g:Int,val b:Int){
        RED(255,0,0),
        GREEN(0,255,0),
        BLUE(0,0,255),
        ORANGE(255,255,0),
        YELLOW(0,255,255),
        WHITE(0,0,0);//这里必须要有分号,Kotlin中唯一必须使用分号的地方,如果在类中定义任何方法,则要使用分号把枚举常量列表和方法分开

        fun rbg() = (r*256+g) * 256 + b
    }
---

# 使用"when"处理枚举类

**when**类似于Java中的`switch`关键字，不过它比`switch`功能更强大。它是一个有返回值的表达式，因此可以直接写一个直接返回when表达式的表达式函数体，


    fun getColor(color : Color2) = when(color) {
        Color2.RED -> "red"
        Color2.GREEN -> "green"
        Color2.BLUE -> "blue"
        Color2.ORANGE -> "orange"
        Color2.YELLOW -> "yellow"
        Color2.WHITE -> "white"
     }

在一个`when`分支上合并多个选项


    fun getColor2(color:Color2) = when (color){
        Color2.RED,Color2.GREEN -> "red or green"
        Color2.BLUE,Color2.YELLOW -> "blue or yellow"
        Color2.ORANGE -> "orange"
        Color2.WHITE -> "white"
    }

`when`表达式的实参可以是任何对象,它被检查是否与分支条件相等

     when (setOf(c1,c2)) {//c1和c2分别是 RED、GREEN,反过来也行,它们的混合结果还是 ORANGE
        // 列举能够混合的颜色
        setOf(Color2.RED, Color2.GREEN) -> Color2.ORANGE
        setOf(Color2.GREEN, Color2.BLUE) -> Color2.YELLOW
        //如果没有任何其它分支，匹配这里就会执行
        else -> Color2.WHITE
    }
    
使用不带参数的`when`，上面代码的效率可能有点低,每次调用fix这个函数时都会去创建一些`Set`实例，为了达到更好性能，可以换成以下这种方式,来避免创建垃圾对象

    fun mixOptimized(c1:Color2,c2:Color2) =
        when{
            c1 == Color2.RED && c2 == Color2.GREEN || c1 == Color2.GREEN && c2 == Color2.RED -> Color2.ORANGE
            c1 == Color2.GREEN && c2 == Color2.BLUE || c1 == Color2.BLUE && c2 == Color2.GREEN -> Color2.YELLOW
            else -> Color2.WHITE
        }
---
        
# 智能转换:合并类型检查和转换

假如要对`(1+2)+4`这样的算术表达式求值,如下：

    interface Expr  //定义一个接口
    class Num(val value:Int) : Expr   //使用冒号来标记Num类实现了Expr这个接口
    class Sum(val left:Expr,val right:Expr) : Expr //Sum运算的实参可以是任何Expr:Num或者另外一个Sum

计算表达式的值

    fun sum() = eval(Sum(Sum(Num(1),Num(2)),Num(4)))
    
    fun eval(e:Expr):Int{
    if (e is Num){//使用 is 来判断一个变量是否是某种类型,类似于java中的instanceOf,如果检查过一个变量是某种类型,后面就不需要再转换它
        val n = e as Num //使用as关键字来表示到特定类型的显示转换,在这里转换成Num其实是多余的
        return n.value
    }

    if (e is Sum){
        return eval(e.left) + eval(e.right)  //变量 e 被智能的转换了类型
    }
    throw IllegalArgumentException("Unknown expression")
    }
    
简化上面的的`eval`函数，使用表达式函数体

    fun simplifyEval(e:Expr):Int =
        if (e is Num) e.value //如果if分支下是一个代码块,则代码块中的最后一个表达式会被作为结果返回
        else if (e is Sum) eval(e.left) + eval(e.right)
        else throw IllegalArgumentException("Unknown expression")
        
重构:用`when`代替`if`分支

    fun simplifyEval2(e:Expr):Int =
        when(e){
           is Num ->
                e.value
           is Sum ->
                simplifyEval2(e.left) + simplifyEval2(e.right)
            else ->
                throw IllegalArgumentException("Unknown expression")
        }

代码块作为`if`和`when`的分支

    fun evalWithLogging(e:Expr):Int =
    when(e){
        is Num -> {
            println("num: ${e.value}")
            e.value //代码块中最后的表达式会作为结果被返回
        }
        is Sum -> {
            val left = evalWithLogging(e.left)
            val right = evalWithLogging(e.right)
            println("sum: $left+$right")
            left+right
        }
        else ->
            throw IllegalArgumentException("Unknown expression")
    }

