---
title: Gradle之Wrapper详解
date: 2018-06-01
updated: 2018-06-01
tags:
- Gradle
- Wrapper
categories: Gradle
---

# 介绍
**Wrapper**，看到这个单词大家应该都不陌生，它就是位于**Android**项目根目录下的**gradle**文件夹中的**gradle-wrapper.properties**脚本文件。其实它就是对**Gradle**的一层包装，我们都知道一个**Android**的项目需要使用**Gradle**来构建，但是不同版本的项目需要不同版本的**Gradle**，而**Wrapper**简化了**Gradle**的部署，手动去部署的话就比较麻烦了，这里说得项目的版本指的是开发此项目所使用的**Android Studio Gradle**插件的版本，也就是我们通常所说的编译器的版本，而**Android Studio Gradle**插件和**Gradle**的版本对应关系如下：

![](https://upload-images.jianshu.io/upload_images/2349677-4a232efe9ab386c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以，我们在导入别人的项目之前，要先根据自己的编译器版本去修改**Gradle**的版本，如果版本不对应会出现以下错误：

![](https://upload-images.jianshu.io/upload_images/2349677-60cdad7be2b27726.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/550)

我的编译器版本为**2.3.3**，它所对应的**Gradle**版本最低为**3.3**，而当前我的项目的**Gradle**版本为**2.14.1**，那么显而易见项目就会构建失败，解决它很简单，只需要修改**gradle-wrapper.properties**文件中的**distributionUrl**字段即可，将**2.14.1**改为**3.3**，如图：

![](https://upload-images.jianshu.io/upload_images/2349677-914550994f1edbc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/550)

最后再重新**build**，这样**Wrapper**就会检查**Gradle**有没有被下载关联，如果没有就会从配置的地址(Gradle的官方地址)进行下载并运行构建。

# 生成Wrapper
我们创建一个新的项目，**Gradle**会自动为我们生成**Wrapper**所需的目录文件，其实在**Gradle**中内置了一个**Wrapper task**，如果我们要手动去生成**Wrapper**，只需要在项目的根目录下输入**gradle wrapper**即可，

![](https://upload-images.jianshu.io/upload_images/2349677-aeeb16786014ef8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

生成的文件如下：
![](https://upload-images.jianshu.io/upload_images/2349677-32c6055ee5b5037e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Wrapper 配置
我们在终端执行**gradle wrapper**生成相关文件时，可以为其指定一些参数，来控制**Wrapper**的生成，比如依赖的版本等，如下：

|  参数名 | 说明 |  
|------------|-------|
| --gradle-version | 用于指定使用的**Gradle**版本 |
| --gradle-distribution-url | 用于指定下载**Gradle**发行版的**url**地址 |

使用方法为**gradle wrapper --gradle-version 3.3**，这样我们所配置的**Wrapper**就会使用**3.3**版本的**Gradle**，它会影响**gradle-wrapper.properties**中的**distributionUrl**的值，该值的规则是`http\://services.gradle.org/distributions/gradle-${gradleVersion}-bin.zip`
如果我们在执行**gradle wrapper**的时候不添加任何参数，那么就会使用你当前**Gradle**的版本作为生成的**Wrapper**的**gradle version**。例如，你当前安装的**Gradle**是**3.3**版本的，那么生成的**Wrapper**也是**3.3**版本的。

# gradle-wrapper.properties
该文件是**wrapper**的配置文件，我们上面执行任务的配置都会被写进此文件中。该文件的配置字段如下：

| 字段名 | 说明 |
| --- | --- | 
| distributionBase | 下载的Gradle压缩包解压后存储的主目录 |
| distributionPath | 相对于distributionBase的解压后的Gradle压缩包的路径 |
| zipStoreBase | 同distributionBase，只不过是存放zip压缩包的 |
| zipStorePath | 同distributionPath，只不过是存放zip压缩包的 |
| distributionUrl | Gradle发行版压缩包的下载地址 |

而**distributionUrl**就是我们**gradle wrapper**所依赖的**Gradle**版本。一般生成的都是这样的**https\://services.gradle.org/distributions/gradle-3.3-bin.zip，**通常都会把**bin**改成**all**，这样在开发过程中，就可以看到**Gradle**的源代码了。

# 自定义Wrapper Task

**gradle-wrapper.properties**是由**Wrapper Task**生成的，那么我们当然也可以自己来配置该**Wrapper Task**来达到我们配置**gradle-wrapper.properties**的目的，在**build.gradle**构建文件中录入如下脚本：

	//构建wrapper
    task wrapper(type: Wrapper){
       gradleVersion = '3.3'
    }

然后我们执行**gradle wrapper**的时候，就会默认生成**3.3**版本的**wrapper**了，而不用使用`--gradle-version 3.3`进行了指定了。同样的你也可以配置其它参数，例如：

	//构建wrapper
    task wrapper(type: Wrapper){
       gradleVersion = '3.3'
       archiveBase = 'GRADLE_USER_HOME'
       archivePath = 'wrapper/dists'
       distributionBase = 'GRADLE_USER_HOME'
       distributionPath = 'wrapper/dists'
       distributionUrl = 'http://services.gradle.org/distributions/gradle-3.3-all.zip'
    }


















