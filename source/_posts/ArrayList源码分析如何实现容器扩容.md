---
title: ArrayList源码分析如何实现容器扩容
date: 2018-06-15 15:42:59
tags:
- ArrayList
categories: Java
---

# 前言
对于**ArrayList**，相信每个人都非常熟悉，在平常开发中，经常会用到它来存储数据，而它底层使用的是**数组**结构，对于数组来说，在创建的时候容量的大小就已经确定了，因此数组只支持**查询**和**修改**。但是，**ArrayList**在实例化的时候是不需要指定大小的，它在内部已经实现了自动扩容的功能，这样开发者就可以对它进行**增**、**删**、**改**、**查**。下面就来分析它内部的实现原理。


# 构造方法
**ArrayList**有三个构造方法：

- ArrayList()
- ArrayList(int initialCapacity)
- ArrayList(Collection<? extends E> c)

第一个是无参的构造方法；第二个是传一个**int**类型的参数，该参数指定了集合的初始化容量大小；而第三个接收了一个**Collection**类型的对象，其实就是为当前集合进行数据的初始化。
而我们平常用到最多就是第一个，那么来看看它内部做了什么：

      public ArrayList() {
       super();
        //默认初始化一个空数组对象，数组长度为0
       this.elementData = EMPTY_ELEMENTDATA;
    }
我们发现，在它内部仅仅是给**elementData**这个对象进行了赋值，而这个**elementData**是ArrayList内部声明的一个对象，**EMPTY_ELEMENTDATA**是一个空数组实例。

          /**
     * 用于空实例的共享空数组实例
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     *
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == EMPTY_ELEMENTDATA will be expanded to
     * DEFAULT_CAPACITY when the first element is added.
     *
     * 数组缓冲区，用来存储ArrayList的元素，ArrayList的容量就是缓冲区的大小
     * Package private to allow access from java.util.Collections.
     */
    transient Object[] elementData;

这样显然易见，该构造方法内部初始化了一个空数组的对象**elementData**，容量大小为0。

平常我们用到最多的就是直接使用**add()**方法给集合添加元素，而容器的扩容也是在该方法内部实现的，接下来看看它的源码：


# add(E e)

    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
看第二行代码我们发现，此方法将接收到的对象加入到ArrayList内部的**elementData**数组中，而在给数组赋值之前，先是调用了**ensureCapacityInternal(size + 1)**，而这个方法就是为了给数组进行扩容，然后再给数组中的指定位置进行赋值，这个逻辑应该没错的，先看一下该方法内部是如何给数组进行扩容的：

        private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

        private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
     /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
主要看**grow()**这个方法，其中**newCapacity**变量就是新的数组的容量大小，而它的大小最小不能小于**minCapacity**这个值，但是**newCapacity**也不是无限大的，从下面的代码中可以看出：
 
        private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

        if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);

        private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

如果**newCapacity**大于**MAX_ARRAY_SIZE**(ArrayList内部定义的一个常量)，那么调用**hugeCapacity()**方法重新计算新的数组的容量大小，大小不能超过**Integer.MAX_VALUE**。
计算出最终新数组的容量大小后，接下来就是创建新的数组，并把原数组中的元素复制到新数组中：

           elementData = Arrays.copyOf(elementData, newCapacity);
这个方法就是将原数组**elementData**，扩充后的新的数组大小为**newCapacity**，之后返回新数组对象，而**Arrays.copyOf()**方法内部其实就是重新**new**了一个数组，并调用**System.arraycopy()**将旧数组中的元素复制到新数组中：
          
       public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
这样在得到扩容后的新数组对象后，就可以将元素添加到数组中了

        public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }



