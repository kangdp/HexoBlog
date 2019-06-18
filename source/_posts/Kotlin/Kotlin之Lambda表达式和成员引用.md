---
title: Kotlin之Lambda表达式和成员引用
date: 2019-01-18
updated: 2019-01-18
tags:
- Kotlin
categories: Kotlin
---


# Lambda介绍：作为函数参数的代码块
用匿名内部类实现监听器

    <!--Java-->
    button.setOnClickListener(new OnClickListener(){
           override
           public void onClick(View view){
              <!-- 点击后执行的动作-->
           }
        }
    );

现在用Kotlin的Lambda表达式来替换匿名内部类

    button.setOnClickListener{<!--点击后执行的动作-->}
    
# Lambda和集合
先看一个例子

    data class Person(val name:String,val age:Int)

然后创建一个Person集合，并找出集合中年龄最大的那个

    val list = listOf(Person("Alice",29),Person("Bob",31))
    println(list.maxBy{it.age})
    Person{name=Bob，age=31}
如上使用了Kotlin的库函数，`maxBy`函数可以在任何集合上调用，且只需要一个实参：一个函数，指定比较哪个值来找到最大值，而花括号中的代码`{it.age}`就是实现了这个逻辑的`lambda`

如果lambda刚好是属性的委托，可以用成员引用代替

    list.maxBy(Person::age)
    
# Lambda表达式语句

![](https://upload-images.jianshu.io/upload_images/2349677-829577c94c89a493.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)

还可以把`lambda`表达式存储在一个变量中

    val sum = { x:Int,y:Int -> x+y }
    println(sum(1,2))
    
还可以直接调用`lambda`表达式

    {println(42)}()
    
但是这样的语法毫无可读性，也没有什么意义，如果你确实需要把一小段代码封闭在一个代码块中，可以使用库函数`run`来执行传给它的`lambda`

    run { println(42) }
    
    
再回到上面的例子

    val list = listOf(Person("Alice",29),Person("Bob",31))
    println(list.maxBy{it.age})
    
如果不用简明的语法重写这个例子，你会得到下面的代码

    list.maxBy( { p:Person -> p.age} )
这个代码就一目了然了，花括号里面的代码片段是`lambda`表达式，把它作为实参传递给函数。这个`lambda`接受一个Person的参数并返回它的年龄

这个代码还可以简化，如果`lambda`表达式是函数调用的最后一个实参，它可以放到括号的外面

    list.maxBy(){ p:Person -> p.age }

当`lambda`是函数唯一的实参时，还可以去掉调用代码中的空括号

    list.maxBy { p:Person -> p.age }
    
    
省略`lambda`参数类型，和局部变量一样，如果`lambda`参数的类型可以被推导出来，你就不需要显示地指定它。这里以`maxBy`函数为例，其参数的类型始终和集合中元素的类型相同

    list.maxBy { p -> p.age }
    
使用默认参数类型，仅在实参名称没有显示地指定时这个默认的名称才会生成

    list.maxBy{ it.age }
**注意：** `it`约定能大大缩短你的代码，但你不应该滥用它。尤其在嵌套`lambda`的情况下，最后显示地声明每个`lambda`的参数。否则很难搞清楚`it`引用的到底是哪个值。

此外，`lambda`表达式还可以包含更多的语句

    val sum = { x:Int,y:Int -> 
        println("Computing the sum of $x and $y...")
        x+y
    }
    
    println(sum(1,2)

    Computing the sum of 1 and 2...
    3

# 在作用域中访问变量

当在函数中声明一个匿名内部类的时候，在匿名内部类中可以引用函数的参数和局部变量。如果函数内部使用`lambda`，也可以访问这个函数的参数

    fun printMessageWithPrefix(messages:Collec    tion<String>,prefix:String){
            messages.forEach {<!--接收lambda作为实参-->
            println("$prefix $it") <!--在lambda中访问prefix参数-->
        }
    }

这里Kotlin和Java的区别就是，在Kotlin中不会仅限于访问`final`变量，在`lambda`内部也可以修改这些变量

# 成员引用

如果你想要当做参数传递的代码已经被定义成了函数，那么你可以将这个函数转换成值，如下使用`::`运算符来转换

    val getAge = Person::age
这种表达式被称为成员引用，它提供简明语法，来创建一个调用单个方法或这访问单个属性的函数值。双冒号把类名称和你要引用的成员名称隔开

    
![](https://upload-images.jianshu.io/upload_images/2349677-d33319b76370bf2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如下这个`lambda`表达式

    val getAge = { person:Person -> person.age }
    
成员引用和调用函数的`lambda`具有一样的类型，所以可以相互转换

    list.maxBy(Person::age)
    
还可以引用顶层函数，这种情况省略了类名称，直接以`::`开头。成员引用`::salute`被当作实参传递给库函数`run`

    fun salute() = println("Salute!")
    run (::salute)
    Salute!
    
如果`lambda`要委托给一个接收多个参数的函数，提供成员引用代替它将会非常方便

    val action = {person:Person,message:String -> sendEmail(person,message)}
    
    val nextAction = ::sendEmail
    调用
    nextAction(...,...)
    

可以用构造方法引用存储或者延期执行创建类实例的动作，构造方法的引用方式是在双冒号后指定类名称

    val createPerson = ::Person <!--创建`Person`实例的动作被保存成了值-->
    val person = createPerson("kdp",25)
    println(person)

还可以使用同样的方式引用扩展函数

    fun Person.isAdult() = age >= 21
    val predicate = Person::isAdult
    
# 绑定引用

Kotlin1.1允许你使用成员引用语法捕捉特定实例对象上的方法引用

    val p = Person("Dmitry",34)
    val dmitrysAgeFunction = p::age
    println(dmitrysAgeFunction())

> 注意：dmitrysAgeFunction是一个零函数的参数，在Kotlin1.1之前，你需要显示地写出`lambda{p.age}`，而不是使用绑定成员引用`p::age`
