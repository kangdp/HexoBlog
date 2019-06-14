---
title: Gradle入门之环境配置、编写第一个脚本
date: 2018-05-29
updated: 2018-05-29
tags:
- Gradle
categories: Gradle
---

# 准备
在进行**Gradle**安装之前要确保已经安装配置好**Java**环境，要求**JDK6**以上，并且在环境变量里配置了**JAVA_HOME**，查看Java版本可以在终端输入如下命令：

       java -version
# Windows下搭建Gradle构建环境
以Windows为例，先到**Gradle**官网**https://gradle.org/**下载**Gralde SDK**，直接下载地址为**https://downloads.gradle.org/distributions/gradle-3.3-all.zip**，当然如果有**Android Studio**的朋友，由于Gradle默认已经下载，就不用重新下载了，之后解压出来得到如下目录清单：
- **docs**
API、DSL、指南等文档
- **getting-started.html**
入门链接
- **init.d**
gradle的初始化脚本目录
- **lib**
相关库
- **LICENSE**
- **media**
一些icon资源
- **NOTICE**
- **samples
事例
- **src**
源文件

# 配置Gradle环境变量

打开我的电脑>属性面板>高级系统设置>环境变量，新建一个**GRADLE_HOME**变量，变量值为Gradle的**bin**目录，例如我直接使用的是Android Studio中Gradle的目录
![](https://upload-images.jianshu.io/upload_images/2349677-be42ca4714a33a7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后将**GRADLE_HOME/bin**添加到你的环境变量**PATH**的路径里才可以。

好了，现在我们已经配置完了，要验证我们的配置是否正确，是否可以运行**Gradle**，我们只需要打开终端，输入`gradle -v`命令查看即可。如果能正确显示**Gradle版本号、Groovy版本号、JVM等相关信息**，那么说明你已经配置成功了。这里以验证我的配置为例：


![](https://upload-images.jianshu.io/upload_images/2349677-7567f7b4c9ca66a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

环境搭建好了，那么我们就开始写一个**Hello World**版的**Gradle脚本**。
# 编写第一个脚本
新建好一个目录比如**Android-Gradle**,然后在该目录下创建一个名为**build.gralde**的文件，打开编辑该文件，输入以下内容：

         task hello{
             doLast{
                 println 'Hello World'
              }     
          }

打开终端，然后进入**Android-Gradle**目录下，使用`gradle -q hello`命令来执行构建脚本：


      F:\android\Android-Gradle>gradle -q hello
      Hello World

接下来解释一下上面所产生的步骤和原因：

**build.gradle**是**Gradle**默认的构建脚本文件，执行**Gradle**命令的时候，会默认加载当前目录下的**build.gradle**文件。这个构建脚本定义一个任务(**Task**)，任务名字叫**hello**，并且给任务**hello**添加了一个动作，官方名字是**Action**，我把它看作是业务代码逻辑或者回调实现更加贴切一些，因为**doLast**就意味着在**Task**执行完毕之后要回调**doLast**的这部分闭包的代码实现。

再看**gradle -q hello**这段运行命令，意思是要执行**build.gradle**脚本中定义的名为**hello**的**Task**，**-q**参数用于控制**gradle**输出的日志级别，以及哪些日志可以输出被看到。

上面中的**println 'Hello World'**会输出 **Hello World**，它其实就是**System.out.println("Hello World")；**的简写方式。**Gradle**可以识别它，是因为**Groovy**已经把**println**这个方法添加到了**java.lang.Object**中了，而在**Groovy**中，方法的调用可以省略签名中的括号，以一个空格分开即可，所以就有了上面的写法。还有一点儿要说明的是，在**Groovy**中，单引号和双引号所包含的内容都是字符串；不像**Java**中，单引号是字符，双引号才是字符串。

