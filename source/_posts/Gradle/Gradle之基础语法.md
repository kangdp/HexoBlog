---
title: Gradle之基础语法
date: 2018-06-03
updated: 2018-06-03
tags:
- Grovvy
categories: Gradle
---

# 字符串
在**Grovvy**中，单引号和双引号都可以定义一个字符串常量，不同的是单引号标记的是纯粹的字符串常量，而不是对字符串里的表达式做运算，但是双引号可以。

    task printStringVar << {
        def name = "张三"
        println '单引号的变量计算:${name}'
        println "双引号的变量计算:${name}"
    }
输出结果如下：

    单引号的变量计算:${name}
    双引号的变量计算:张三

字符串连接运算规则：一个美元符号紧跟着一对花括号,花括号里放表达式，只有一个变量的时候可以省略花括号，如 $name。

# 集合
>常见的集合：List、Set、Map、Queue。

- **List**
在 Java 中定义一个List,需要New一个实现了List接口的类，在Grovvy中则非常简单，例如

      task printList << {
        def numList = [1,2,3,4,5,6];
        println numList.getClass().name
      }
  通过输出发现，numList是一个ArrayList类型。

  如果访问集合中的元素？**Grovvy**提供了非常简便的方法：

      task printList << {
        def numList = [1,2,3,4,5,6];
        println numList.getClass().name
        
        println numList[1] //访问第二个元素
        println numList[-1] //访问最后一个元素
        println numList[-2] //访问倒数第二个元素
        println numList[1..3] //访问第二个到第四个元素
      }
  输出结果：
       
      2
      6
      5
      [2，3，4]

  **Grovvy**还为**List**提供了非常方便的迭代操作，这就是**each**方法。该方法接收一个闭包作为参数，可以访问**List**里的每个元素：

      task printList << {
        def numList = [1,2,3,4,5,6];

        numList.each {
           println it
        }
      }
  输出结果：
     
      1
      2
      3
      4
      5
      6

  **it**变量为正在迭代的元素。
- **Map**
定义一个**Map**，访问元素，采用**map[key]**或者**map.key**的方式都可以，例如：

      task printlnMap << {
         def map1 = ['width':1024,'height':768]
         println map1['width']
         println map1.height
       }
  输出结果：
 
      1024
      768

  对于**Map**的迭代，当然少不了**each**方法，只不过迭代的元素是一个**Map.Entry**的实例：

      task printlnMap << {
         def map1 = ['width':1024,'height':768]
         map1.each {
           println "Key:${it.key},Value:${it.value}"
         }
       }
  对于集合，**Grovvy**还提供了诸如**collect、find、findAll**等便捷的方法，可以自己查阅文档看一下。

# 方法
- **括号是可以不写的**
定义一个方法：

      def method1 (int a,int b) {
       println a+b
      }

    关于方法的调用，在**Grovvy**中就要灵活得多，可以省略**()**，如下：

      task invokeMethod << {
         method1 (1,2)
         method1  1,2
        }
  上面的两种调用方式的结果是一样的，但第二种更简洁，**Gradle**中的方法调用都是这样的写法。

- **return是可以不写的**

    在**Grovvy**中，我们定义有返回值的方法时，return语句不是必需的。当没有return的时候，Grovvy会把方法执行过程中的最后一句代码作为其返回值：

      def method1 (int a,int b) {
         if(a > b) {
              a
         }else{
              b
         }
      }
      task invokeMethod << {

         def add1 = method1 1,2
         def add2 = method1 5,3

         println "add1:${add1},add2:${add2}"
      }

  输出结果：

      add1:2，add2:5

- **代码块可以作为参数传递**
呆板的写法   
    
        task printList2 << {
        def list = ['a','b','c','d','e']
        list.each({println it})
      }
  格式化一下，变成
      
      task printList2 << {
        def list = ['a','b','c','d','e']
        list.each({
          println it
        })
      }
    Grovvy规定，如果方法的最后一个参数是闭包，可以放到方法外面     

      task printList2 << {
        def list = ['a','b','c','d','e']
        list.each(){
          println it
        }
      }
    省略方法后，就是      

      task printList2 << {
        def list = ['a','b','c','d','e']
        list.each{
          println it
        }
      }

# JavaBean

  在**Java**中为了访问和修改**JavaBean**的属性，我们不得不重复生 成**getter/setter**方法，并且使用它们，太繁琐，这在**Grovvy**中得到了很大的改善：

    task helloJavaBean << {
       Person p = new Person();
       println "名字是:${p.name},年龄:${p.age}"
       p.name = '张三'
       p.age = 25
       println "名字是:${p.name},年龄:${p.age}" 
     }

     class Person {
       private String name
       private int age
     }

在**Grovvy**中，并不是一定要定义成员变量才能作为类的属性访问，我们直接用**getter/settter**方法，也一样可以当作属性访问：

    task helloJavaBean << {
       Person p = new Person();
       println "名字是:${p.name},年龄:${p.age}"
       p.name = '张三'
       println "名字是:${p.name},年龄:${p.age}" 
     }
     class Person {
         private String name
          public int getAge(){
                12
          }
     }

上面的例子并没有定义一个**age**的成员变量，但是一样可以通过**p.age**获取到该值，这是因为定义了**getAge()**方法。那么这时候我们不能修改**age**的值，因为我们没有为其定义**setter**方法。

# 闭包
- **向闭包传递参数**
当闭包有一个参数时，默认就是**it**；当有多个参数时，**it**就不能表示了，我们需要把参数一一列出：

      task helloClosure << {
         //使用我们自定义的闭包
           eachMap {key,value ->
           println "$key,$value"
          }
      }

  自定义一个闭包方法

      def eachMap(closure) {
       def map = ["name":"张三","age":25]
        map.each {
        closure(it.key,it.value)
        }
      }

  我们为闭包传递了两个参数，一个**key**，另一个**value**，这时我们就不能使用**it**了，必须要显式声明出来，如例子中的**key、value ->**用于把闭包的参数和主体区分开来。

- **闭包委托**
**Grovvy**闭包的强大之处在于它支持闭包方法的委托。**Grovvy**的闭包有**thisObject、owner、delegate**三个属性，当你再闭包内调用方法时，由它们来确定使用哪个对象来处理。默认情况下**delegate**和**owner**是相等的，但是**delegate**是可以被修改的，这个功能是非常强大的，**Gradle**中的闭包的很多功能都是通过修改**delegate**实现的：

         task helloDelegate << {

            new Delegate().test {
            println "thisObject:${thisObject.getClass()}"
            println "owner:${owner.getClass()}"
            println "delegate:${delegate.getClass()}"
            method1()
            it.method1()
            }
          }

         def method1(){
          println "Context this:${this.getClass()} in root"
          println "method1 in root"
         }
          class Delegate {

            def method1(){
              println "Delegate this:${this.getClass()} in Delegate"
              println "method1 in Delegate"
           }
            def test(Closure<Delegate> closure) {
                closure(this)
            }
         }
  输出结果：

      thisObject:class build_e27c427w88bo0afju9niqltzf
      owner:class build _e27c427w88bo0afju9niqltzf$_run_closure2
      delegate:class build _e27c427w88bo0afju9niqltzf$_run_closure2
      Context this:class build_e27c427w88bo0afju9niqltzf in root
      method1 in root
      Delegate this:class Delegate in Delegate
      method1 in Delegate

  通过上面的例子发现，**thisObject**的优先级最高，默认情况下，优先使用**thisObject**来处理闭包中调用的方法，如果有则执行。从输出中我们也可以看到，这个**thisObject**其实就是这个构建脚本的上下文，它和脚本中的**this**对象是相等的。
从例子中也证明了**delegate**和**owner**是相等的，它们两个的优先级是：**owner**要比**delegate**高。所以闭包内方法的处理顺序是：**thisObject>owner>delegate**。

  在**DSL**中，比如**Gradle**，我们一般会指定**delegate**为当前的**it**，这样我们在闭包内就可以对改**it**进行配置，或者调用其方法：

      task closureStudent << {
          student {
            name = '张三'
            age = 23
            dumpStudent()
           }
      }

      class Student {
        String name
        int age
        def dumpStudent() {
           println "name is ${name},age is ${age}"
        }
       }

       def student(Closure<Student> closure) {
          Student p = new Student();
          closure.delegate = p
          closure.setResolveStrategy(Closure.DELEGATE_FIRST);
          closure(p)
       }
    在例子中我们设置了委托对象为当前创建的**Person**实例，并且设置了委托模式优先，所以，我们在使用**person**方法创建一个**Person** 的实例时，可以在闭包里直接对该**Person**实例配置。























