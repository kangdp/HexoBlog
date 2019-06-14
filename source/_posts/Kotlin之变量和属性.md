---
title: Kotlin之变量和属性
date: 2018-06-13 11:27:32
tags:
- Kotlin
categories: Kotlin
---

>在**Kotlin**中，**一切都是对象**。没有像Java那样的原始基本类型。这个是非常有帮助的，因为我们可以使用一致的方式来处理所有的可用的类型。

# 基本类型
- **数字类型中不会转型**
举个例子，你不能给`Double`变量分配一个`Int`。必须要做一个明确的类型转换，可以使用众多的函数之一：

       val i : Int = 7
       val d: Double = i.toDouble()

- **字符(Char)不能直接作为一个数字来处理**
在需要时我们需要把他们转换为一个数字：
 
       val c : Char = 'c'
       val i : Int = c.toInt()

- **位运算也有一点不同。**
在Android中，我们经常`flags`中使用“或”，所以我使用“and”和“or”来举例：
 
        //Java
        int bitwiseOr = FLAG1 | FLAG2;
        int bitwiseAnd = FLAG1 & FLAG2;

        //Kotlin
        val bitwiseOr = FLAG1 or FLAG2
        val bitwiseAnd = FLAG1 and FLAG2
>还有很多其他的位操作符，比如**sh1**，**shs**，**ushr**，**xor**，或**ivn**。当我们需要的时候，可以去Kotlin官网查看。

- **字面上可以写明具体的类型。这个不是必须的，但是一个通用的Kotlin实践时省略变量的类型(我们马上就能看到)，所以我们可以让编译器自己去推断出具体的类型。

        val i = 12 //An Int
        val iHex = 0x0f   //一个十六进制的Int类型
        val l = 3L    //A Long
        val d = 3.5  //A Double
        val f = 3.5F //A Float

- **一个String可以像数组那样访问，并且被迭代：
       
         val s = "Example"
         val c = s[2]  //这是一个字符'a'
          //迭代String
          for(c in s){
           println(c)
          }

# 变量
变量可以简单地定义成可变(**var**)和不可变(**val**)的变量。这个与Java中使用的`final`很相似。但是**不可变**在Kotlin中是一个很重要的概念。
>在Kotlin中，**应该尽可能的使用val**

# 属性
属性与Java中的字段是相同的，但是更加强大。属性做的事情是字段加上getter()加上setter。我们通过一个例子来比较他们的不同之处。这是Java中字段安全访问和修改所需要的代码：

       public class Person{
          private String name;
        
          public String getName(){
              return name;
           }

          public void setName(String name){
                this.name = name;
           }
        }

        ...
        Person person = new Person();
        person.setName("name");
        String name = person.getName();
在Kotlin中，只需要一个属性就可以了：

        public class Person {
          var name: String = ""
        }
        ...
       
         val person = Person();
         person.name = "name"
         val name = person.name

如果没有任何指定，属性会默认使用getter和setter。当然它也可以修改为你自定义的代码，并且不修改存在的代码：

          public class Person{
            var name: String = ""
            get() = field.toUpperVase()
            set(value){
               field = "Name: &value"
             }

           }

如果需要在getter和setter中访问这个属性自身的值，它需要创建一个backing field。可以使用field这个预留字段来访问，它会被编译器找到正在使用的并自动创建。需要注意的是，如果我们直接调用了属性，那我们会使用setter和getter而不是直接访问这个属性。backing field只能在属性访问器内访问。

就如在前面章节提到的，当操作Java代码的时候，Kotlin将允许使用属性的语法去访问在Java文件中定义的getter/setter方法。编译器会直接链接到它原始的getter/setter方法。所以当我们直接访问属性的时候不会有性能开销
