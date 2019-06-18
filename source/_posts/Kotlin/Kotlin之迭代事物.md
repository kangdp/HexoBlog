---
title: Kotlin之迭代事物
date: 2019-01-05
updated: 2019-01-05
tags:
- Kotlin
categories: Kotlin
---



# while循环

**Kotlin**有`while`循环和`do-while`循环，它们的语法和Java中的循环没什么区别:

    fun testWhile(){
    //先执行do一次，之后当while条件满足时继续执行do语句
    var flag : Boolean
    do {
        flag = true
        println("first run")
    }
    while (!flag)
    }
---
    
# 迭代数字:区间和数列

迭代左右都为闭区间

    fun testForLoop(){
        val oneToBai = 1..100 //一个起始值,一个结束值,使用 .. 运算符表示区间
        for (i in oneToBai){
            println("计数: $i")
        }
    }
迭代带步长的数列

    fun testForLoop2(){
        for (i in 100 downTo 1){ //数列递减,步长为1 (可使用 step     指定步长)
            println("递减: $i")
        }
    }

迭代不包含指定结束值的半闭合区间,使用`until` 函数可以创建这样的区间,但是仅支持递增

    fun testForLoop3(){
        val size = 10
        for (i in 0 until size){
            println("递增: $i")
        }
        //上面的for循环等同于
        for (i in 0 .. size-1){
            println("递增: $i")
        }
    }
---    
# 迭代Map

使用map[key]读取值,并使用 map[key] = value 设置它们

    fun testMap(){
        val binaryReps = TreeMap<Char,String>()
        val a:Char = 'a'
        for (c in 'A'..'B'){
            val binary = Integer.toBinaryString(c.toInt()) //转成二进制
            binaryReps[c] = binary
            println("key: = $c" + " " + "value: ${binaryReps[c]}")
        }
        println(a)
    }

使用 `in` 运算符来检查一个值是否在区间中,或者它的逆运算。`!in` 来检查这个值是否不在这个区间中

    fun isLetter(c:Char) = c in 'a'..'z' || c in 'A'..'Z'
    fun isNumber(c:Char) = c !in '0'..'9'

`in` 运算符也可用在when表达式中

    fun recognize(c:Char) =
        when(c){
            in 'a'..'z' -> "It is a letter!"
            in '0'..'9' -> "It is a digit"
            else -> "I don't know..."
    }

区间不仅限于字符，只要一个类实现了 java.lang.Comparable接口,就可以创建这种类型的对象的区间。如果是这样的对象，并不能列举出这个区间中的所有对象，但是仍然可以使用 `in` 运算符来检查一个其他的对象是否属于这个区间

    fun inObject(){
        println("Kotlin" in "Java".."Scale") //这里的字符串是按自然顺序比较的
    }

`in` 同样适用于集合

    fun inSet(){
        println("Kotlin" in setOf("Java","Scale")) //这个集合不包含字符串Kotlin
    }