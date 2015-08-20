---
layout: post
title: "Android Develop Tips(一)"
date: 2014-12-11 00:07:34 +0800
comments: true
categories: Android
---

## Android常用项目结构
在Android的开发过程中一直在不断的调整自己对于项目结构的架构方式，经过了多次的调整之后，现在基本每个项目都会以如下方式来架构：

* ***api***
放置一些API口类
* ***app***
放置一些与整个东西pp相关的类，如继承自Application的当前应用的Appliation类、继承自系统Resources写的当前应用的Resurces类、应用的版本管理VersionManager类等
* ***bean***
主要用来放置java bean的类
* ***io***
放置一些io操作的类，如网络请求、文件读写、数据库操作等等
* ***provider***
放置provider相关的类
* ***receiver***  
放置整应用里边需要的reiver类
* ***service***
放置整个应用里边的sevice类
* ***ui***
  * ***activity***
  整个应用的actvity类
  * ***adapter***
  整个应用的adapter类
  * ***controller***
  整个用中用到的ew层进行操纵的roller类
  * ***fragment***
  整个应中的fragemnt类
  * ***util***
  对应用中ui进行操作的一些helper类
  * ***view***
  整个应中可以分离出来的view
  * ***widget***
  整个应用的自定义控件
* ***util***
各种各样的工具类

我现在基本上每个项目都是这种架构方式了，另外，我习惯将自己平时的积累的一些工具类之类的类整理到一个Common的library module中，自己的每个项目都会把这个common module倒入进来，这样子方便了很多。

<!--more-->


## Gradle
Gradle只是提供了一个构建项目的框架,然后通过各种各样的Plugin来构建我们自己的项目。Gradle本身的领域对象主要有Project和Task，Project为task的执行提供了上下文。Gradle默认会将当前目录下的build.gradle作为项目的构建文件。接下来以一个实际项目为例类详细介绍gradle构建Android项目。

```java
apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion '21.1.1'

    defaultConfig {
        applicationId "us.drilight.android"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }

    signingConfigs {
        YourApp {
            storeFile file("Your keystore path")
            storePassword "Your keystore password"
            keyAlias "Your keyAlias"
            keyPassword "Your keyPassword"
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.YourApp
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }

    productFlavors {
        baidu_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu_market"]
        }
        meizu_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "meizu_market"]
        }
        mumayi_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "mumayi_market"]
        }
        xiaomi_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi_market"]
        }
        wandoujia_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia_market"]
        }
        play_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "play_market"]
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile project(':common')
    compile 'io.realm:realm-android:0.71.0'
    compile 'com.google.code.gson:gson:2.3'
    compile 'com.mcxiaoke.volley:library:1.0.6'
}
```

`apply plugin: 'com.android.application'` 显然这是一个构建Android项目Gradle Plugin，另外，如果希望当前module是以一个library project被编译的话应该使用 `apply plugin: 'com.android.library'` 这个plugin。


```java
android{
}

这一块是plugin中的android project构建。

```

```java
compileSdkVersion 21
buildToolsVersion '21.1.1'

这一块是用来指定CompileSdkVersion和BuildToolsVersion的。
```

```java
defaultConfig {
        applicationId "us.drilight.android"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }

这一块是用来对project进行一个初步的配置，包括applicationId,minSdkVersion,versionCode,versionName。
这个地方有个需要注意的就是在gradle构建的project中versionCode和versionName以这个地方的为准，
AndroidMainfest.xml中关于这一块的配置不再有效。
```

```java
lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }

这一块主要是对编译过程的配置，如果这个地方不做处理的话，在编译的时候就连warning之类的都会导致编译通不过，所以配
置这一块基本是必须的。
```

```java
signingConfigs {
        YourApp {
            storeFile file("Your keystore path")
            storePassword "Your keystore password"
            keyAlias "Your keyAlias"
            keyPassword "Your keyPassword"
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.YourApp
        }
    }

这一块是对签名的配置
```

```java
sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }

这一块主要是对sourceSet的配置这个配置可以将libs下的.so库编译进当前project。sourceSets这一块Android
Studio会默认配置，当然也可以自己写比如：
(
java.srcDirs = ['src']
resources.srcDirs = ['src']
idl.srcDirs = ['src']
renderscript.srcDirs = ['src']
res.srcDirs = ['res']
assets.srcDirs = ['assets'])
这个视个人项目文件结构来具体配置。
```

```java
productFlavors {
        baidu_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu_market"]
        }
        meizu_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "meizu_market"]
        }
        mumayi_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "mumayi_market"]
        }
        xiaomi_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi_market"]
        }
        wandoujia_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia_market"]
        }
        play_market {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "play_market"]
        }
    }

这一块是对多渠道包的配置，具体每一个渠道包的包名等等之类的信息都可以在这个地方配置，示例中是每一个
productFlavor中加入了友盟的多渠道信息统计。
```

```java
 dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile project(':common')
    compile 'io.realm:realm-android:0.71.0'
    compile 'com.google.code.gson:gson:2.3'
    compile 'com.mcxiaoke.volley:library:1.0.6'
}

这一块是对第三方依赖的配置。compile fileTree(dir: 'libs', include: ['*.jar']) 这个是将当前module项目
目录下的所有jar包以lib的形式编译到当前project中来。 compile project(':common') 这个的话是将第三方的
library project编译进当前project，这个地方要注意的是得先在settings.build中声明这个library,类似这样的
include ':app', ':Common' 。 compile 'com.google.code.gson:gson:2.3' 这个是通过从远程maven仓库的
形式来编译第三方library到当前project，我个人现在目前更倾向于这种形式导入第三方library。

```
