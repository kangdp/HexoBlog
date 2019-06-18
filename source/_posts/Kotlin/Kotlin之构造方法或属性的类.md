---
title: Kotlin之构造方法或属性的类
date: 2019-01-14
updated: 2019-01-14
tags:
- Kotlin
categories: Kotlin
---

# 主构造方法和初始化语句块

声明一个主构造方法

    class User constructor(_nickname: String){<!-- 带一个参数的主构造方法-->
        val nickname: String
        init { <!--初始化语句块-->
            nickname = _nickname
        }
    }
在上面的例子中，由于主构造方法中有限制，不能包含初始化代码，因此需要使用初始化语句块。

在这个例子中，也可以去掉`constructor`关键字

    class User2 constructor(_nickname: String){
        val nickname = _nickname
    }
    
如果属性用相应的构造方法参数来初始化，代码可以通过把`val`关键字加在参数前的方式来进行简化，这样就可以替换类中的属性定义

    class User(val nickname:String)
    
可以向函数参数一样为构造方法参数声明默认值

    class User(val nickname: String,val isSubscribed: Boolean = true)
    
如果一个类中具有父类，主构造方法中同样需要初始化父类

    open class User(val nickname: String){...}
    
    class TwitterUser(name:String) : User(name){...}
    
如果父类没有提供任何的构造方法，必须显示地调用父类的构造方法，即使它没有任何的参数

    open class Button
    class RadioButton : Button()
    
不要让类外部的代码实例化它，可以将构造方法标记为`private`

    class Secretive private constructor(){...}
    
# 用不同的方式初始化父类

当需要重写父类多个构造方法时，需要声明多个从构造方法
    
    class MyButton : View{
    
        constructor(ctx:Context):this(ctx, null)<!--调用自己的构造方法-->
        constructor(ctx: Context,attr: AttributeSet?):super(ctx,attr) <!--调用父类的构造方法-->
    }    
    
# 实现在接口中声明的属性

    interface User {
        val nickname: String
    }
    
- 在子类的主构造方法的参数前面加上`override`关键字，表明这个属性实现了来自于父类`User`的抽象属性
    
        class PrivateUser(override val nickname: String) : User 
    

- `nickname`属性通过自定义`getter`的方式来实现

        class SubscribingUser(val email: String) : User {
            override val nickname: String
                get() = email.substringBefore("@")
        }
- 在初始化时将`nickname`属性与值关联的方式来实现

    
    class FaceBookUser(val accountId：Int) : User {
        override val nickname = getFacebookName(accountId)
    }
    
除了抽象属性声明外，接口还可以包含具有`getter`和`setter`的属性
    
    interface User {
        val email: String
        val nickname: String
            get() = email.substringBefore("@")
    }
这个接口包含抽象属性`email`，同时`nickname`属性有一个自定义的`getter`。第一个属性必须在子类中重写，第二个属性是可以被继承的。

# 通过`getter`或`setter`访问支持字段

声明一个可变属性，并且在每次`setter`访问时执行额外的代码

    class User(val name: String){
        var address: String = "unspecified"
            set(value) {
                println("Address was changed for $name: $address -> $value") <!--读取支持字段的值-->
                field = value <!--更新支持字段的值-->
            }
    }
    
    val user = User("kangdongpu");
    user.address = "HeBei,SJZ"
    <!--输出-->
    Address was changed for kangdongpu: unspecified -> HeBei,SJZ
    
在`setter`的函数体中，使用了特殊的标识符`field`来访问支持字段的值。在`getter`中，只能读取值；而在`setter`中，即能读取也能修改它

# 修改访问器的可见性

声明一个具有`private setter`的属性，让其不能在类外部被修改

    class LengthCounter {
        var counter:Int = 0
        private set <!--不能在类外部修改此属性值，将setter的可见性改为private-->

        fun addWord(word:String){<!--可在类内部通过方法修改-->
            counter+=word.length
        }
    }
