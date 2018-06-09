---
title: 进程间通信及Binder原理解析
date: 2018-06-09 16:47:19
tags:
- Android
- Process
- AIDL
- Binder
categories: Android
---

# 前言
>在网上有很多关于android跨进程通信的文章，但是大多数看了之后还是不知所云，一脸懵逼，最近抽时间整理了一下这方面的知识，为了能让大家从应用层上清楚地了解跨进程通信的实现原理，我会将我所讲的内容分为**多进程**、**Binder**和这两个部分，接下来我会按照我的思路给大家一一讲解。
# 多进程
在讲跨进程通信之前，我们必须先要了解Android中的多进程模式。通过在四大组件中指定`android:process`属性，便可以轻易的开启多进程模式，这样似乎看起来很简单，其实并不是这样，多进程远远没有我们想象的这么简单，有时候我们通过多进程得到的好处甚至都不足以弥补使用多进程带来的代码层面的负面影响。下面是一个例子，描述了如何在Android中创建多进程：

    <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
    </activity>

    <activity android:name=".FirstActivity"
              android:label="@string/app_name"
              android:process=":remote" />

    <activity android:name=".SecondActivity"
              android:label="@string/app_name"
              android:process="com.process.demo.remote"/>

上面的**FirstActivity**和**SecondActivity**都指定了`process`属性，但是属性值不同，相当于当前应用又新添加了两个进程，而**MainActivity**没有指定`process`属性，那么它是运行在默认的进程中，而系统默认的进程就是当前应用的包名；**FirstActivity**中的`process`属性值为`:remote`，这个`:`的含义是指在当前进程名前面附加当前的包名，也就是说**FirstActivity**的完整进程名为**com.process.demo.remote**；而对于**SecondActivity**中的`process`是一种完整命名方式，不会附加包名信息。另外，进程名以`:`开头的进程属于当前应用的私有进程，其它应用的组件不能和它跑在一个进程中，而进程名不以`:`开头的进程属于全局进程，其它应用通过`ShareUID`的方式可以和它跑在同一个进程中，Android系统会为每个应用分配一个唯一的**UID**，具有相同**UID**的应用才能共享数据，关于**ShareUID**的详细信息大家可以查一下资料，我就不多做解释了。

然后继续，将程序运行起来后，通过adb命令(adb shell ps 包名)查看当前应用下的进程列表，如图所示：
![](https://upload-images.jianshu.io/upload_images/2349677-ecd432cacfc69e48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`PID`为当前进程的id，可以发现这三个id都不一样，也就是说成功的开启了三个进程。而在前面我们说到使用多进程会产生一些负面影响，接下来验证一下这个问题。
首先我们新建一个**Constant**类，并声明一个`int`类型的静态常量

    public class Constant {
         public static int id = 1;
    }
接着在**MainActivity**的'onCreate()'方法中修改这个`id`值，给它改为`2`，最后在**MainActivity**和**FirstActivity**中分别去打印这个`id`值，打印出来的日志如下：
![](https://upload-images.jianshu.io/upload_images/2349677-760ec1949c3caf23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


看了图中的日志，发现结果貌似跟我们想象中的不一样啊，**FirstActivity**打印出来的值咋还是`1`呢，我们的确在**MainActivity**给`id`赋值了，然后在**MainActivity**又启动了**FirstActivity**。到这里，想必大家已经明白了这就是多进程所带来的问题。所以说，多进程并非我们想象中的那么简单。

出现以上结果原因就在于**MainActivity**和**FirstActivity**是两个不同的进程，而Android会为每个应用分配一个独立的虚拟机，或者说为每个进程分配一个独立的虚拟机，而每个独立的虚拟机都有它自己独享的内存，不同的虚拟机在内存分配上有不同的地址空间，因此不同进程是无法共享内存中的数据的，对于上面例子而言，其实`com.process.demo`和`com.process.demo.remote`这两个进程中都有一个**Constant**类，而且这两个类是互不干扰的，所以说在**FirstActivity**中的`id`值并没有发生改变。

最后我们总结一下，使用多进程造成的如下几方面的问题：

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharedPreferences的可靠性下降
- Application会多次创建

第一个问题已经分析；第二个问题其实和第一个本质上是一样的，想想就知道了，进程都不一样了，锁的对象都不是同一个对象了，还怎么去保证线程同步；第三个问题是因为SharedPreferences不支持两个进程去同时执行写的操作，否则会导致一定几率的数据丢失，这是因为SharedPreferences底层是通过读/写**XML**文件来实现的，并发写显然是可能出问题的，甚至并发读/写都有可能出问题；第四个问题也是显而易见的，当一个组件跑在一个新的进程中的时候，由于系统要在创建新的进程同时分配独立的虚拟机，所以这个过程其实就是启动了一个应用的过程，因此会重新创建新的**Application**，不信的话，大家可以去测试一下。

# Binder
上面我们分析了多进程所带来的问题，那么接下来就是讲解如何解决它，其实系统提供了很多跨进程通信的方法，虽然不能共享内存但是通过跨进程通信我们还是可以实现数据交互。其实Android的这四大组件中都可以实现跨进程通信，而AIDL也是处理进程间通信的一种方式，AIDL跨进程的核心实现其实就是Binder，接下来我们就详细分析一下Binder的实现原理。

先写一个简单的例子：
比如我有一个需求，服务端提供给客户端添加书籍和获取所有书籍列表的方法。

1、在包名目录下创建一个bean目录，在bean目录下创建一个Book实体类
![](https://upload-images.jianshu.io/upload_images/2349677-917ffe837b219152.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后让Book实现Parcleable接口，代码如下：

    public class Book implements Parcelable{
    public int bookId;
    public String bookName;

    protected Book(Parcel in) {
        bookId = in.readInt();
        bookName = in.readString();
    }

    public Book(int bookId, String bookName) {
        this.bookId = bookId;
        this.bookName = bookName;
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(bookId);
        dest.writeString(bookName);
    }
}



2、 创建Book.aidl、IBookManager.aidl文件。Book.aidl是Book类在AIDL中的声明，IBookManager.aidl是我们定义的一个接口，里面有两个方法：`getBookList`和`addBook`,其中`getBookList`用来获取服务端书籍列表，`addBook`用来往服务端添加书籍，这里需要注意的是 Book类和Book.aidl所在的包名要一致，不然AS会编译失败，如图所示：
![](https://upload-images.jianshu.io/upload_images/2349677-ed72123717c18d0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还需要在IBookManager.aidl中使用**import**导入Book类

    package com.kdp.aidl;
    import com.kdp.aidl.bean.Book;

    interface IBookManager {

    void addBook(in Book book);

    List<Book> getBookList();
    
    }

Book.aidl代码如下：

       package com.kdp.aidl.bean;


       parcelable  Book;


最后编译一下，AS会自动帮你生成一个IBookManager.java的一个类，该类所在目录如下：
![](https://upload-images.jianshu.io/upload_images/2349677-f0cc3b5f0b287a7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而这个类就是Binder的核心实现，现在我们需要对这个类进行分析，先看一下整体代码：

       package com.kdp.aidl;
       public interface IBookManager extends android.os.IInterface
       {
           /** Local-side IPC implementation stub class. */
           public static abstract class Stub extends android.os.Binder implements  com.kdp.aidl.IBookManager
       {
        private static final java.lang.String DESCRIPTOR ="com.kdp.aidl.IBookManager";
     /** Construct the stub at attach it to the interface. */
         public Stub()
       {
         this.attachInterface(this, DESCRIPTOR);
       }
    /**
    * Cast an IBinder object into an com.kdp.aidl.IBookManager interface,
    * generating a proxy if needed.
    */
     public static com.kdp.aidl.IBookManager asInterface(android.os.IBinder obj)
     {
            if ((obj==null)) {
              return null;
            }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
      if (((iin!=null)&&(iin instanceof com.kdp.aidl.IBookManager))) {
          return ((com.kdp.aidl.IBookManager)iin);
      }
        return new com.kdp.aidl.IBookManager.Stub.Proxy(obj);
      }
       @Override public android.os.IBinder asBinder()
      {
      return this;
      }
       @Override public boolean onTransact(int code, android.os.Parcel data, 
      android.os.Parcel reply, int flags) throws android.os.RemoteException
     {
      switch (code)
     {
      case INTERFACE_TRANSACTION:
     {
      reply.writeString(DESCRIPTOR);
      return true;
      }@Override public boolean onTransact(int code, android.os.Parcel data, 
      android.os.Parcel reply, int flags) throws android.os.RemoteException
     {
      switch (code)
     {
      case INTERFACE_TRANSACTION:
     {
      reply.writeString(DESCRIPTOR);
      return true;
      }
      case TRANSACTION_addBook:
      {
      data.enforceInterface(DESCRIPTOR);
      com.kdp.aidl.bean.Book _arg0;
      if ((0!=data.readInt())) {
       _arg0 = com.kdp.aidl.bean.Book.CREATOR.createFromParcel(data);
       }
       else {
      _arg0 = null;
       }
      this.addBook(_arg0);
      reply.writeNoException();
      return true;
      }
      case TRANSACTION_getBookList:
      {
         data.enforceInterface(DESCRIPTOR);
         java.util.List<com.kdp.aidl.bean.Book> _result = this.getBookList();
         reply.writeNoException();
         reply.writeTypedList(_result);
         return true;
      }
     }
        return super.onTransact(code, data, reply, flags);
    }

       private static class Proxy implements com.kdp.aidl.IBookManager
       {
         private android.os.IBinder mRemote;
         Proxy(android.os.IBinder remote)
         {
            mRemote = remote;
         }
          @Override public android.os.IBinder asBinder()
        {
          return mRemote;
        }
          public java.lang.String getInterfaceDescriptor()
       {
          return DESCRIPTOR;
       }
        @Override public void addBook(com.kdp.aidl.bean.Book book) throws 
           android.os.RemoteException
      {
           android.os.Parcel _data = android.os.Parcel.obtain();
           android.os.Parcel _reply = android.os.Parcel.obtain();
        try {
               _data.writeInterfaceToken(DESCRIPTOR);
               if ((book!=null)) {
                 _data.writeInt(1);
                   book.writeToParcel(_data, 0);
                }
               else {
                 _data.writeInt(0);
                }
               mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
               _reply.readException();
              }
             finally {
               _reply.recycle();
              _data.recycle();
             }
           }
          @Override public java.util.List<com.kdp.aidl.bean.Book> getBookList() throws android.os.RemoteException
           {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<com.kdp.aidl.bean.Book> _result;
                try {
                  _data.writeInterfaceToken(DESCRIPTOR);
                  mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                  _reply.readException();
                  _result = _reply.createTypedArrayList(com.kdp.aidl.bean.Book.CREATOR);
                    }
            finally {
                      _reply.recycle();
                     _data.recycle();
                    }
                 return _result;
             }
      }
         static final int TRANSACTION_addBook = 
         (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_getBookList = 
        (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }
        public void addBook(com.kdp.aidl.bean.Book book) throws android.os.RemoteException;
        public java.util.List<com.kdp.aidl.bean.Book> getBookList() throws android.os.RemoteException;
}

粗略的看了一下，发现这里面有很多内部类，我们先看最外层的这个IBookManager，它是一个接口，此接口定义了两个方法，分别是`addBook`，`getBookList`，另外还有两个常量id，
 
      static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
      static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);

这两个常量id是服务端用来区分这两个方法的，当客户端发起一个请求时，服务端会根据这个id来判断客户端请求的是哪个方法；此外IBookManager继承了 `android.os.IInterface`接口，该接口中只定义了一个方法，如图：
![](https://upload-images.jianshu.io/upload_images/2349677-e9f6f0b24af7366c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个方法表示返回当前**Binder**对象。
接下来我们继续往下看，在**IBookManager**内部有一个**Stub**类，而且是抽象的，这个**Stub**你有没有感觉很熟悉，其实它就是我们在Service中需要实例化的**Binder**对象，我们一般会在Service中这样做
![](https://upload-images.jianshu.io/upload_images/2349677-4189f00c7efb2199.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而这个**Stub**继承了`android.os.Binder`并且实现了IBookManager接口类，我们看**Stub**的构造方法，里面有一个`attachInterface`方法
   
      public Stub()
      {
      this.attachInterface(this, DESCRIPTOR);
      }
第一个参数不用多说，第二个参数`DESCRIPTOR`是**Binder**的唯一标识，一般用当前**Binder**的类名表示，比如本例中的`com.kdp.aidl.IBookManager"`。点开进入此方法内部，发现这个方法在**Binder**类中做了如下赋值:
![](https://upload-images.jianshu.io/upload_images/2349677-e8c5cc4211bb64a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将**Binder**类的唯一标识赋值给了`mDescriptor`变量，现在大家肯定有疑问，这个`mDescriptor`变量到底有什么用?当然有用了，接下来我们会用到它。

下面接着看IBookManager.java这个类，在Stub构造方法下面有一个`asInterface`的静态方法，
![](https://upload-images.jianshu.io/upload_images/2349677-33ad6a83a8dcdb36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个方法想必大家都不会陌生，这个就是我们在客户端需要调用的方法，我们一般在客户端这样写：

        IBookManager binder = IBookManager.Stub.asInterface(binder);
此方法传入的是服务端的**Binder**对象，而这个方法就是用来将服务端的**Binder**对象转换成客户端所需的**AIDL**接口类型的对象，这种转换过程是区分进程的，如果客户端和服务端位于同一进程，那么此方法返回的就是服务端的**Stub**对象本身，否则返回的是系统封装后的**Stub.proxy**对象。
在这个方法中我们发现有一个`queryLocalInterface`方法，此方法就是用来判断客户端和服务端是否位于同一进程，该方法传入当前**Binder**类的唯一标识，进入此方法内部来看看里面做了什么？

![](https://upload-images.jianshu.io/upload_images/2349677-f8fd68b1e79a1105.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们发现传入的`DESCRIPTOR`要跟`mDescroptor`变量做对比，等等，这个`mDescroptor`不就是我们在`attachInterface`方法中赋值的那个变量吗？没错，也就是说，当客户端和服务端位于同一个进程时，`mDescroptor`变量和我们传入的`DESCRIPTOR`是相同的，所以直接就会返回当前的**Binder**对象本身；如果客户端和服务端不在同一个进程，那么客户端所在进程中的`mDescriptor`变量就会为空，由于我们只在服务端通过new的方式来实例化了**Stub**对象并给`mDescriptor`赋值，也就是说服务端所在的进程中`mDescriptor`变量是有值的，但是在客户端只是通过`asInterface`方法来获取**Stub**对象，而并没有给`mDescriptor`变量赋值，所以客户端所在进程中的`mDescriptor`变量是空的，回想上面我们讲多进程数据共享的时候举过的一个例子，内存中的数据只有在同一进程才会被共享。

当客户端和服务端不在同一个进程时，`asInterface`方法则会返回系统封装后的**Stub.Proxy**对象，将服务端的**Binder**对象通过构造方法传了进去，并且这个**Proxy**是**Stub**中的一个内部类，该类也实现了**IBookManager**接口，并实现了`addBook`和`getBookList`这两个方法，而这两个方法是我们在客户端需要调用的，比如当客户端调用`getBookList`方法时，如下图所示：
![](https://upload-images.jianshu.io/upload_images/2349677-833f2f549afa0bbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此方法内部首先会创建该方法所需要的输入型**Parcel**对象**_data**、输出型**Parcel**对象**_reply**和返回值对象**List**；
然后把该方法的参数信息写入**_data**中(如果有参数的话)，接着会调用**transact**方法来发起**RPC**(远程过程调用)请求，同时当前线程挂起；然后服务端的**onTarnsact**方法会被调用，直到**RPC**过程返回后，当前线程继续执行，并从**_reply**中取出**RPC**过程的返回结果，最后返回**_reply**中的数据。

在调用**transact**方法发起RPC请求时，服务端的`onTarnsact`方法会被调用，我们进入`transact`方法的内部会发现
![](https://upload-images.jianshu.io/upload_images/2349677-1c9f19f806c77e25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此方法中调用了`onTransact`方法，这个方法也是定义在系统的**Binder**类中，我们之前讲了**Stub**类继承自**Binder**，那么在我们的**Stub**类中重写了**Binder**类中的`onTransact`方法，而此方法是运行在服务端的**Binder**线程池中，当客户端发起请求时，远程请求会通过系统底层封装后交由此方法来处理。该方法原型为`public Boolean onTransact(int code，android.os.Parcel data，android.os.Parcel.reply，int flags)。
看一下该方法内部做了什么
   
        @Override public boolean onTransact(int code, android.os.Parcel data, 
      android.os.Parcel reply, int flags) throws android.os.RemoteException
     {
      switch (code)
     {
      case INTERFACE_TRANSACTION:
     {
      reply.writeString(DESCRIPTOR);
      return true;
      }@Override public boolean onTransact(int code, android.os.Parcel data, 
      android.os.Parcel reply, int flags) throws android.os.RemoteException
     {
      switch (code)
     {
      case INTERFACE_TRANSACTION:
     {
      reply.writeString(DESCRIPTOR);
      return true;
      }
      case TRANSACTION_addBook:
      {
      data.enforceInterface(DESCRIPTOR);
      com.kdp.aidl.bean.Book _arg0;
      if ((0!=data.readInt())) {
       _arg0 = com.kdp.aidl.bean.Book.CREATOR.createFromParcel(data);
       }
       else {
      _arg0 = null;
       }
      this.addBook(_arg0);
      reply.writeNoException();
      return true;
      }
      case TRANSACTION_getBookList:
      {
         data.enforceInterface(DESCRIPTOR);
         java.util.List<com.kdp.aidl.bean.Book> _result = this.getBookList();
         reply.writeNoException();
         reply.writeTypedList(_result);
         return true;
      }
     }
        return super.onTransact(code, data, reply, flags);
    }

在该方法中，服务端会根据传入的**code**来判断当前客户端请求的是哪个方法，这个**code**就是这两个方法的id，然后服务端就会调用相应的目标方法，比如当客户端调用`getBookList`方法时，最终实现此方法是在Sercvice中(因为我们是在Service中**new Stub**然后实现**IBookManager**接口中的这两个方法的)，而调用是在服务端的`onTransact`方法中，服务端会先判断**code**，然后再调用`getBooList`方法拿到数据，之后便将`List<Book>`序列化后写入到**reply**这个**Parcel**对象中，接着我们再回到客户端的`getBookList`方法

           @Override public java.util.List<com.kdp.aidl.bean.Book> getBookList() 
      throws android.os.RemoteException
        {
           android.os.Parcel _data = android.os.Parcel.obtain();
           android.os.Parcel _reply = android.os.Parcel.obtain();
           java.util.List<com.kdp.aidl.bean.Book> _result;
        try {
          _data.writeInterfaceToken(DESCRIPTOR);
          mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
         _reply.readException();
        _result = _reply.createTypedArrayList(com.kdp.aidl.bean.Book.CREATOR);
          }
          finally {
        _reply.recycle();
        _data.recycle();
        }
         return _result;
        }
在调用**Binder**的`transact`方法之后，此时的`_reply`中已经有了客户端所需要的数据，然后将`_reply`反序列化取出`List<!-- <Book> -->`，返回给客户端，这样一个完整的远程请求就结束了

最后给大家看一下**Binder**的工作机制图

![Binder的工作机制图.png](https://upload-images.jianshu.io/upload_images/2349677-93100ed114e985d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

