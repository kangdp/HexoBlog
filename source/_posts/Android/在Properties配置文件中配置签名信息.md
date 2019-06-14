---
title: 在Properties配置文件中配置签名信息
date: 2018-09-25
updated: 2018-09-25
tags:
categories: Android
---

# 新建`Properties`文件
在该文件中填写好签名的配置信息，等号左边为key，右边为value，在build需要根据key来获取value

    key_alias=name
    key_password=123456
    store_file=../mykeystore.jks
    store_password=123456

# 定义方法
在app的`build.gradle`文件中定义一个方法，该方法中创建一个`Properties`对象，使用它来加载刚刚创建的签名配置文件(`sign.properties`)，代码如下：

     读取签名配置并返回Properties
    def buildSign() {
         def Properties buildProperties = new Properties()
         buildProperties.load(new FileInputStream(file("../local.properties")))
         return buildProperties
     }

# 配置签名
在app的`build.gradle`文件中`android`标签下配置你项目的`release`签名，然后调用`buildSign()`方法来获取`Properties`对象，通过此对象来获取对应的签名信息的`value`值，如下所示：

     android {

        signingConfigs {
                release {
                      def Properties buildPro = buildSign()
                      keyAlias buildPro['key_alias']
                      keyPassword buildPro['key_password']
                      storeFile file(buildPro['store_file'])
                      storePassword buildPro['store_password']
                      }
              }
      ...
     }
