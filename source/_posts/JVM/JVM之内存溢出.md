---
title: JVM之内存溢出
date: 2019-03-18
updated: 2019-03-18
tags:
- JVM
- 内存溢出
categories: JVM
---


 # 内存溢出和内存泄漏的区别
- **内存溢出**
在Java虚拟机向系统申请内存时，由于虚拟机内部的各存储区域存储空间都有限制(可以通过指定虚拟机的某些参数来优化调整内存大小)，例如当堆内存被占满后，虚拟机再向系统申请内存是申请不到的，此时就会发生内存溢出

- **内存泄漏**
内存泄漏是针对**GC**(垃圾回收器)来说的，**GC**在进行对象回收时，一些无用对象仍然持续占有内存，无法得到及时释放，最后造成内存空间的浪费


# 关于内存溢出有以下几种分类
- **Java堆溢出**
- **虚拟机栈和本地方法栈溢出**
- **方法区和运行时常量池溢出**



## Java堆溢出
Java堆用于存储对象实例，只要不断的创建对象，并且保证**GC Roots**到对象之间有可达路径来避免垃圾回收机制清除这些对象，当对象数量达到堆的最大容量后就会产生内存溢出异常。

下面举个例子来验证一下堆内存溢出，为了方便实验，先把堆大小限制为10M，不可扩展(将堆的最小值`-Xms`参数与最大值`-Xmx`参数设置为一样即可避免堆自动扩展)，然后通过指定虚拟机的参数`-XX:+HeapDumpOnOutOfMemoryError`可以让虚拟机在出现内存溢出时`Dump`出当前的内存堆转储快照以便事后进行分析

    
    public class HeapTest {
    public static void main(String[] args){
        List<Object> list = new ArrayList<>();
        while (true) {
            list.add(new Object());
        }
        }
    }

结果如下：

    java.lang.OutOfMemoryError: Java heap space
    Dumping heap to java_pid13288.hprof ...
    Heap dump file created [13500082 bytes in 0.076 secs]
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at com.company.HeapTest.main(HeapTest.java:10)
	
当然Java堆溢出不仅仅是以上情况，还有可能是内存泄漏导致的，比如内存泄漏地比较多了，浪费了大量的空间内存，这样就会导致堆内存会快速的被占满，然后造成内存溢出，到底是内存溢出还是内存泄漏还需要通过工具去查证，这里就不详细说了，网上有很多教程。
	
## 虚拟机栈和本地方法栈溢出

虚拟机栈也就是我们平常所说的栈，它在Java虚拟机规范中有两种异常:

 - 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出**StackOverflowError**异常
 - 如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出**OverOfMemoryError**异常 

上面所说的栈深度其实就是栈中栈帧的数量，操作系统分给JVM的内存是有限的，而JVM分给虚拟机栈的内存也是有限的，如果方法调用过多，创建的栈帧的数量也就越多，那么最终就会导致虚拟机栈溢出；当然我们可以通过`-Xss`参数来指定虚拟机栈的最大深度。另外，如果将虚拟机栈设置为可动态扩展，那么同样的当栈深度不够时，JVM会自动申请扩展，如果此时申请不到足够的内存空间就会抛出**OverOfMemoryError**异常


## 方法区和运行时常量池溢出

在JDK1.6以前，很多人愿意把方法区称为“永久代”，本质上两者并不等价，其实方法区只是Java虚拟机规范的一种定义，或者说用永久代来实现方法区而已，也就是说永久代仅仅是HotSpot的概念，对于其它虚拟机来说是不存在永久代的概念的。而使用永久代来实现方法区，更容易遇到内存溢出问题，所以在JDK1.7的HotSpot中，原本放在永久代中的字符串常量池被移到了堆中。

接下来我们使用`String.intern()`方法来验证JDK1.6版本中方法区或者说是常量池溢出的情况，不过在此之前先来了解一下此方法在JDK1.6和JDK1.7中的作用：


**jdk1.6中**

- 如果字符串常量池中已经包含了一个等于(equals)此String对象的字符串，则返回常量池中这个字符串的String对象
- 否则，将此String对象包含的字符串添加到常量池中，并返回此String对象的引用。

**jdk1.7中**
- 如果字符串常量池中已经包含了一个等于(equals)此String对象的字符串，则返回常量池中这个字符串的String对象
- 否则，将复制一份此String对象的引用到字符串常量池中，并返回此引用

他们的区别其实就是：
如果字符串常量池中不包含一个等于(equals)此String对象的字符串，那么
- 在jdk1.6中，就会将堆中的字符串对象复制一份到字符串常量池中，然后返回该字符串对象
- 而在jdk1.7中，会将堆中的字符串对象的引用复制一份到字符串常量池中，然后返回该引用

为了方便测试jdk1.6中方法区的溢出，我们先通过`-XX:PermSize`和 `-XX:MaxPermSize`参数来限制方法区的大小，然后下面是测试代码

    
    public static void main(String[] args){
        List<String> list = new ArrayList<String>();
        int i = 0;
        while (true) {
            list.add(String.valueOf(i++).intern());
        }
        }

运行结果如下：



        java.lang.OutOfMemoryError: PermGen space
        Dumping heap to java_pid7588.hprof ...
        Heap dump file created [109527439 bytes in 0.777 secs]
        Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
	    at java.lang.String.intern(Native Method)
	    at com.company.HeapTest.main(HeapTest.jav   a:11)
        Process finished with exit code 1
    
我们应该很清楚的发现第一行报错信息：
`java.lang.OutOfMemoryError: PermGen space`
而`PermGen space`就是永久代，显而易见方法区溢出了，也说明了在jdk1.6中字符串常量池位于方法区内，而使用JDK1.7运行这段程序就不会得到相同的结果。

最后我们再来验证一下JDK1.7中字符串常量池是否被移到了堆中，首先设置一下VM参数` -Xms10m -Xmx10m`，将堆内存限制到10M，然后再运行上面的代码，会出现如下结果

    
    java.lang.OutOfMemoryError: Java heap space
    Dumping heap to java_pid10284.hprof ...
    Heap dump file created [21832557 bytes in 0.279 secs]
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.lang.Integer.toString(Integer.java:403)
	at java.lang.String.valueOf(String.java:3099)
	at com.company.HeapTest.main(HeapTest.java:14)
	
结果如出一辙，抛出了堆内存溢出的异常信息，说明JDK1.7中确实是把字符串常量池放到了堆中
    