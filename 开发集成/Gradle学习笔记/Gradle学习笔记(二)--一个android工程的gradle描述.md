#前言
看了大多数写Gradle的文章，一开始就先讲解各种概念，最后被绕的云里雾里，心里默默感慨道：讲的真tm的好，但是我还是不懂。
所以，我这里想先从全局上给大家来一个通俗的解释，尽量简洁明了，而后再各个深入进去。
#阐明主题
**Gradle作为一个构建工具，主要功能是打包用**。没错，对于android工程来讲，就是最后产出一个apk或者aar，中间你可以插入你自定义的一些操作，比如改字节码，改资源名称之类的。
所以，大致的过程需要说一下：
```
1. 首先，读取工程根目录下settings.gradle文件，创建Settings 对象，获取 include 的 Projects
2. 根据获取的projects ,创建Project对象实例层次,单module就是单实例，多module就是多实例。
3. 然后会通过读取每个module下的build.gradle文件来评估这个Project，以及Project之间的先后顺序。
4. 最后会顺序执行这些project的build.gradle 的task，生成目标文件apk或aar。
```
**注意，每个build.gradle都对应一个Project对象实例**
#一个例子
我们新建一个空白工程，只有一个主工程module，多module类似。
![一个空白工程](http://upload-images.jianshu.io/upload_images/22193-c95e379a656bcfa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到有三个.gradle文件。settings.gradle,两个build.gradle，不过一个是application的，一个是module的。
****
*settings.gradle*
``` gradle
include ':app'

```
很简单，就**执行了**一个include方法，当然，多module的话，只需在后面加 `，'：module1'`类似即可，当你加入一个新的module以后，一个[ProjectDescriptor
](https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/ProjectDescriptor.html)便会被创建出来，然后你就可以用这个对象去改某个module的特定属性了，这里大部分人应该没怎么注意，看不懂也没关系，用到了就会懂了。
 注意，我这里说**执行了**，为什么这么说呢？看这里: include确实是 [Settings](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html)的一个方法，要多看api文档，很多问题都会迎刃而解。 
****
*build.gradle(Application)*
```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.
看上面就应该清楚和module的区别了吧
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter() //jcenter仓库
    }
}
这里是继承了Delete基类的 一个task，作用就是删除 rootProject.buildDir下的文件。
辣么问题来了，rootProject是什么鬼？为什么会有这个对象？
task clean(type: Delete) {
    delete rootProject.buildDir
}
```
这里要介绍[Build script structure](https://docs.gradle.org/current/dsl/#N10060)，**BS是一个以闭包为参数的方法调用，对就是这样**。在x{y{}}中，这里的x{}和x{}里的y{}都是BS。
从下图可以看出，每一个BS都一一映射到一个类。BS中属性就对应于这些类的成员变量。我们可以参照官方文档来了解我们不知道的一些细节，也能为我们以后自己写插件做铺垫。

![ 官方文档](http://upload-images.jianshu.io/upload_images/22193-fcc68ba5777b7e8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
****
*build.gradle(module)*
相信大家对这个就比较熟悉了，我们就先提出几个问题吧：apply 是什么鬼？为什么这样就把android主module用到的BS全部都包含进去了？android{}这个东西是怎么来的？
```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.1"
    defaultConfig {
        applicationId "com.shang.myapplication"
        minSdkVersion 10
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.1.1'
    testCompile 'junit:junit:4.12'
}

```
Ctrl+鼠标放到apply可以看到下图:

![PluginAware](http://upload-images.jianshu.io/upload_images/22193-8fde9c5d334c04e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
前面我们讲个每个build.gradle文件都对应一个Project对象，辣么很显然这个apply就应该是这个类的方法，我就毫不犹豫的打开了[Project类
](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)介绍，Project 实现了PluginAware接口，全局搜了下apply，豁然明了:apply to ,apply plugin ,apply from。**apply plugin 对应的是插件的id，而这个插件里则定义了很多个属性，闭包，来提供给我们做个性化定制使用。**

![apply](http://upload-images.jianshu.io/upload_images/22193-1ac834f1a051da71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

****
写到这里，对一个简单工程的gradle分析大致结束了，其实很多东西，网上的人云亦云，照抄照搬，看来看去，还不如直接对照官方文档来得方便，希望能够多多阅读api文档，与大家共勉。
#参考
* [Gradle官方指导文档](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)
