---
title: Kotlin之通用对象方法、数据类和委托类
date: 2019-01-16
updated: 2019-01-16
tags:
- Kotlin
categories: Kotlin
---

# 通用对象方法

声明一个类，用来存储客户名称和邮编

    class Client(val name:String,val postalCode:Int)

- **字符串表示 : toString()**

    默认的话，一个对象的字符串表示形如`Client@5e9f23b4`，这并不十分有用，要想改变它，需要重写`toString()`方法

    为`Client`实现`toString()`

        class Client(val name:String,val postalCode:Int){
            override fun toString(): String {
                return "Client(name = $name postalCode = $postalCode)"
            }
        }
    输出如下

        Client(name = 康栋普 postalCode = 123456)

- **对象相等性 ：equals()**

    假如想要将包含相等数据的对象视为相等
    
        <!---->
        val client1 = Client("康栋普",123456)
        val client2 = Client("康栋普",123456)
        println(client1 == client2) <!--在Kotlin中，==检查对象是否相等，而不是比较引用，这里会编译成调用`equals`-->
        false
        
        注意:
        - 在Java中，可以使用==运算符来比较基本数据类型和引用类型。在基本数据上，==比较的是值；然而在引用类型上，==比较的是引用(引用指向的地址)
        
        - 在Kotlin中，==运算符是比较两个对象的默认方式。本质上说它是通过equals方法来比较两个值的。所以如果你的equals在类中重写了，你能够很安全地使用==来比较实例。要想进行引用比较，可以使用===运算符，它与Java中的==比较对象引用是一样的
        
- **Hash容器 ：hashCode()**

    `hashCode`方法通常与`equals`一起被重写，它主要用来底层数据结果是`hash table`，例如: **HashSet、HashMap、HashTable**
    如果两个对象相等，它们的`hash`值也必须相等；一般在比较对象的时候会先去比较它们的`hash`值，如果不相等，那么对象就不相等；如果`hash`值相等，再使用`equals`去比较值。
    
    
# 数据类:自动实现通用方法

如果想要你的类是一个方便的数据容器,你需要重写这些方法: `toString、equals和hashCode`。而在Kotlin中你不需要再去生成这些方法了，给你的类添加`data`修饰符，必要的方法将会自动生成

     data class Client(val name:String,val postalCode:Int)
现在就得到了一个重写了所有标准Java方法的类

- equals用来比较实例
- hashCode用来作为例如HashMao这种基于哈希容器的键
- toString用来为类生成按声明顺序排列的所有字段的字符串表达式

数据类和不可变性：copy()方法

在数据类中，如果属性使用`val`，在对象被创建出来时，对象内部的值是无法改变的。为了让不可变数据的数据类可以变得更容易，Kotlin给它们提供了一个方法：一个允许`copy`类的实例的方法，并在`copy`的同时修改某些属性的值

下面为手动实现`copy方法

      class Client(val name:String,val postalCode:Int){
      fun copy(name:String = this.name,postalCode: Int = this.postalCode) = Client(name,postalCode)
    }
    
    val client = Client("kangdongpu",123456)
    println(client.copy("kdp123"))
输出
    
    Client(name=kdp123, postalCode=123456)
    
# 委托类：使用`by`关键字

例如要实现一个如`Collection`接口的装饰器，那么你需要实现它内部的所有方法，即使你不做任何行为的修改

    class DelegatingCollection<T> : Collection<T>{
        override val size: Int
            get() = this.size
        override fun contains(element: T): Boolean = this.contains(element)
        override fun containsAll(elements: Collection<T>): Boolean = this.containsAll(elements)
        override fun isEmpty(): Boolean = this.isEmpty()
        override fun iterator(): Iterator<T> = this.iterator()
    }
而现在Kotlin将委托作为一个语言级别的的功能做了头等支持。无论什么时候实现一个接口，你都可以使用`by`关键字将接口的实现委托到另一个对象

    class DelegatingCollection<T> (innerList: Collection<T> = ArrayList<T>()) : Collection<T> by innerList{
    }
类中所有的方法实现都消失了，编译期会自动实现它们，并且方法的实现和上面的`DelegatingCollection`的例子是相似的，而当你需要修改某些方法的行为时，再去重写这些方法
    
    
     


    
    
