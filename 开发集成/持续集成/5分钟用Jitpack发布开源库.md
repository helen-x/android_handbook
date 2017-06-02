# 5分钟用Jitpack发布开源库  

>作者: 菜刀文   
>Demo:https://github.com/helen-x/JitPackReleaseDemo
 
[![](https://jitpack.io/v/helen-x/JitpackReleaseDemo.svg)](https://jitpack.io/#helen-x/JitpackReleaseDemo)
 
项目开发中会用到很多开源库,  
他们一般通过Maven/Gradle依赖进来的.   



演而优则唱,开发越来越溜以后, 你是否也蠢蠢欲动,想发布自己的库呢.   

下面介绍怎么通过Jitpack进行发布Github代码,  
真的非常非常简单,几分钟搞定~  


## 为什么用Jitpack

现在Maven的两个主要仓库是:  
>1)Maven center   
2)jcenter       

他们使用面很广, 家大业大,所以带来的相应的问题:  
>1)发布过程比较麻烦,需要验证和审核  
2)发布的时候需要Group唯一,这个group得是一个域名.而现在很多开发者没有自己的域名.   


用Jitpack就没有这些烦恼了, 利用Github地址做自己域名, 发布配置也非常简单,不需要验证.  

话不多说,来看看怎么搞.   


## 步骤1: 新建Lib工程     

在AndroidStudio中新建Android Library工程,结构如下   
![](http://upload-images.jianshu.io/upload_images/4048192-755470ecd25ab04a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 解释:  
1.在项目的build.gradle的buildscript添加jitpack编译插件    

```java   
 buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.3'
        //添加jitpack依赖
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
    }
}
```  

2.在library的build.gradle中添加jitpack配置信息  

```java   
//启用Jitpack 插件
apply plugin: 'com.github.dcendents.android-maven'

//设置Jitpack发布的Group
//我的github账号是helen-x, 对应我的group就是com.github.helen-x
group='com.github.helen-x'

```   


## 步骤2: Github上发布代码      
### 1.上面代码发布到Github   
### 2.发布代码(Release/TAG)    
 找到对应项目,进入release页面 
 

![](http://upload-images.jianshu.io/upload_images/4048192-97d738c667e41eaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 进入release以后,进行代码发布.  
发布的时候可以用Releases也可以用Tags.   


![](http://upload-images.jianshu.io/upload_images/4048192-1896d6c6c531bfed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

填写发布信息后,就可以发布了  


![](http://upload-images.jianshu.io/upload_images/4048192-38f812569de24fd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 步骤3: Jitpack发布   

进入[Jitpack link](https://jitpack.io/).   
>1.填写仓库名称   
>2.搜索  
>3.使用"Get", 发布就成功啦~~


![](http://upload-images.jianshu.io/upload_images/4048192-4e9734e3520ba9f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


发布成功后,会列出仓库的地址信息, 别人利用这个坐标就可以用我们的开源库啦.  
比如,我的demo发布后的地址是: `com.github.helen-x:JitpackReleaseDemo:0.1`

## 步骤4: 使用我们的开源库
 1.在build.gradle中加入Jitpack仓库

```java   
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```  

  2.使用我们开源库 
```java    
	dependencies {
	        compile 'com.github.helen-x:JitpackReleaseDemo:0.1'
	}  
```  


## 拓展      
可以在仓库的readme.md中加入  
`[![](https://jitpack.io/v/helen-x/JitpackReleaseDemo.svg)](https://jitpack.io/#helen-x/JitpackReleaseDemo)`

就会自动会有一个Jitpack的bar,效果如下,瞬间显得很高端有木有~

[![](https://jitpack.io/v/helen-x/JitpackReleaseDemo.svg)](https://jitpack.io/#helen-x/JitpackReleaseDemo)

