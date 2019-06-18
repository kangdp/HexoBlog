---
title: Kotlin之类继承结构
date: 2019-01-10
updated: 2019-01-10
tags:
- Kotlin
categories: Kotlin
---

# 接口
使用`interface`关键字来声明一个接口
    
    interface Clickable{
        fun click()
    }
接下来实现这个接口

    class Button : Clickable{
        override fun click() = println("I was clicked")
    }
    
Kotlin中使用冒号代替了Java中的`extends`和`implements`关键字。和Java一样，一个类可以实现多个任意接口，但只能继承一个类

与Java中的`@Override`注解类似，`override`修饰符用来标注被重写的父类的接口的方法和属性。


接口可以有一个默认的实现，当然也可以在子类中重新定义`showOff`函数的实现

    interface Clickable{
        fun click()
        fun showOff() = println("I'm clickable!") //带默认实现的方法
    }

定义另一个实现了同样方法的接口

    interface Focusable{
        fun setFocus(b:Boolean) = println("I ${if (b) "got" else "lose"} focus")
        fun showOff() = println("I'm focusable!")
    }

然后在一个子类中同时实现这两个接口，它们每一个都包含了带默认实现的`showOff`方法，那么子类中你必须要实现这个方法，在Java中可以把基类的名字放在`super`关键字的前面；但是在Kotlin中需要把基类的名字放在尖括号中

    class Button2 : Clickable,Focusable{
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
    override fun click() {}
}

**注意**：在Java中实现Kotlin的接口，Java并不支持接口中的默认方法，子类必须实现方法体

# open、final和abstract修饰符:默认为final

声明一个带`open`的`open`类

    open class RichButton : Clickable{ //这个类是open的:其他类可以继承它
        override fun click() {}//这个函数重写了一个open函数并且它本身同样是open的
        fun disable(){} //这个函数是final的：不能在子类中重写它
        open fun animate(){}//这个函数是open的：可以在子类中重写它
    }
**注意**：如果你重写了一个基类或者接口的成员，重写了的成员同样默认是`open`的。如果你想阻止你类的子类重写你的实现，可以显示地将重写的成员标注为`final`

    open class RichButton: Clickable {
        final override fun click(){}
    }

# 抽象类
在Kotlin中，与Java一样，可以将一个类声明成`abstract`的，这种类不能被实例化。并且它内部的抽象成员始终是`open`的，因此不需要显示地使用`open`修饰符

    abstract class Animated{
        abstract fun animate()
        open fun stopAnimating(){}
        fun animateTwice(){}
    }
    

# 修饰符

- 访问修饰符
<br>普通类中成员默认为`final`;接口、抽象类中成员默认为`open`;若子类继承的父类方法为`open`，那么子类重写后的方法也为`open`

|修饰符|相关成员|评注
|-|-|-|
|final|不能被重写|类中成员默认使用
|open|可以被重写|需要明确的表明
|abstract|必须被重写|只能在抽象类中使用;抽象成员不能有实现
|override|重写父类或接口中的成员|如果没有使用final表明,重写的成员默认是开放的

- 可见性修饰符(默认为`public`)

|修饰符|类成员|顶层声明
|-|-|-
|public(默认)|所有地方可见|所有地方可见
|internal|模块中可见|模块中可见
|protected|子类中可见|------------
|private|类中可见|文件中可见

# 内部类和嵌套类
Kotlin没有显示的嵌套类与Java中的`static`嵌套类一样，要把它变成内部类来持有一个外部类的引用要用`inner`，下面是Java和Kotlin在这个行为上的不同：


|类A在另一个类B中声明|在Java中|在Kotlin中
|-|-|-
|嵌套类(不存储外部类的引用)|static class A| class A
|内部类(存储外部类的引用)|class A|inner class A


# 密封类:定义受限的类继承结构

在使用`when`表达式时，Kotlin编译期会强制检查默认选项。所以不得不添加一个默认的分支。Kotlin为这个问题提供了一个解决方案：`sealed`类。为父类添加一个`sealed`修饰符，对可能创建的子类做出严格的限制，所有的直接子类都必须嵌套在父类中

     sealed class Expr  //将基类标记成密封的，sealed修饰符隐含的是open的
    {
        class Num(val value:Int) : Expr()   //将所有可能的类作为嵌套类
        class Sum(val left:Expr,val right:Expr) : Expr()
    }

    fun simplifyEval2(e:Expr):Int =
        when(e){// “when”表达式涵盖了所有可能的情况，所以不再需要“else”分支
           is Expr.Num ->
                e.value
           is Expr.Sum ->
                simplifyEval2(e.left) + simplifyEval2(e.right)
        }




