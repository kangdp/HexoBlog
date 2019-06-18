---
title: Kotlin之异常
date: 2019-01-06
updated: 2019-01-06
tags:
- Kotlin
categories: Kotlin
---



> 和java中的异常一样，唯一不同的是kotlin的 throw结构是一个表达式,能作为另一个表达式中的一部分使用

    val percentage = if(number in 0..100) number else throw IllegalArgumentException("A percentage must be between 0 and 100: $number")

---
#  "try" "catch" 和 "final"

Kotlin不区分受检异常和未受检异常，不需要指定函数抛出异常

    fun readNumber(reader:BufferedReader): Int? {
        try {
            val line = reader.readLine();
            return line.toInt()
        }catch (e:NumberFormatException){
            return null
        }finally {
            reader.close()
        }
    }
---
# "try"作为表达式,表达式中最后一句作为返回值

如果发生异常执行catch中的语句,那么catch中最后一句作为返回值，如果将return放到catch代码块中,如果发生异常，代码将不会继续往下执行，想继续往下执行，catch也需要一个值，例如将return改成 null，那么它就会是子居中最后一个表达式的值

    fun readNumber(reader: BufferedReader){
        val number = try {
                reader.readLine().toInt()
            }catch(e:NumberFormatException){
                return
            }finally {
                reader.close()
            }
        println(number)
    }
