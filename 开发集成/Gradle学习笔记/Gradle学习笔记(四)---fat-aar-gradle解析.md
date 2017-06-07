#fat-aar.gradle是什么？
在做android应用程序开发时，我们一般都会构建多个模块，来达到解耦的目的，但是有的需求是需要我们提供一个依赖库给外部使用，这时候就遇到一个问题：多个module确实达到了解耦的目的，同时也意味着对外提供依赖库时要提供多个aar，一个依赖module对应一个aar。**fat-aar 的功能简单来说就是让你能够合并和插入各种依赖到一个aar中**。
#实现原理
首先，我们来看打包的一个aar的结构：
```java
它的文件后缀名是.aar，它本身是一个zip文件，强制包含以下文件：
/AndroidManifest.xml
/classes.jar
/res/
/R.txt

另外，AAR文件可以包括以下可选条目中的一个或多个：
/assets/
/libs/name.jar
/jni/abi_name/name.so (where abi_name is one of the Android supported ABIs)
/proguard.txt
/lint.jar
```
这里假设有两个libs，分别为module1和module2。此时，为了合成一个aar，需要将module2中的.jar,res,assets中的文件资源插入到module1中，将AndroidManifest.xml和R.txt相互合并Merge。
**没错，核心思想就是将一个file下的文档拷贝到另一个下面，合并名称相同的两个文件，当然，这些都是在恰当的Task前或后插入的**

这里就引申出下面一个问题：
1. 如何确认需要合并的module？
2. 确定需要合并的module后，如何找到相应的文件路径？
3. 不同的文件在Gradle Tasks中的哪个位置插入？

#脚本解构
带着上面的问题，我们来看下包含两个libs的这个例子：
![module1 和 module2](http://upload-images.jianshu.io/upload_images/22193-2cb3d093eeb5e1ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
来看下module1下的build.gradle,引入了fat-aar.gradle:
```groovy
apply plugin: 'com.android.library'
apply from: 'fat-aar.gradle'
apply from: 'debug.gradle'
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"

    defaultConfig {
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
    embedded project(':module2') //注意这里的embedded 
}
```
在第一次使用fat-aar时，对这个**embedded **特别的不解，为什么不是**compile**或者**provided** ？然后，我看了下fat-aar.gradle的代码，发现了这个：
```groovy
configurations { 
    embedded  //这里是定义了一个configuration，而这个configurations 映射的则是一个 ConfigurationContainer对象
}

dependencies {
    compile configurations.embedded
}

```

![configurations](http://upload-images.jianshu.io/upload_images/22193-8d1ad7f5e6bbb71d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
embedded  实际上是一个 configuration,上面的` embedded project(':module2')` 则是将这个project作为dependency add 进了这个configuration，同时返回这个Dependency对象。


![调用embedded project(':module2')](http://upload-images.jianshu.io/upload_images/22193-310eae756637c3e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
而这个Configuration对象包含哪些属性呢？当然，肯定是有dependencies，另外一个就是artifacts。

这里回到第一个问题， 如何确认需要合并的module？
**通过自定义configuration embedded将这个module2关联起来。**

而第二个问题，确定需要合并的module后，如何找到相应的文件路径？
**我们知道在build一个aar的时候，都会在各个module下生成一个build文件夹，看下图：**

![fat-aar中不同文件的路径定义](http://upload-images.jianshu.io/upload_images/22193-bcc1887cb5002ac2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**我们生成aar相关联的文件夹是这个exploded-aar,当我看了下这个文件夹的结果时，发现了一个有趣的事情：除了support和test,多出了一个MyApplication/module2。哈哈，这个其实在`complie project(':module2')`时也是有的。辣么，我们便可以通过这个路径获取下面编译阶段生成的不同类型文件合并就可以了**

![module1下的exploded-aar](http://upload-images.jianshu.io/upload_images/22193-fafc848e43a02d36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里有个大前提，获取路径，这里就要先获取路径名称：
```groovy
//定义一个包含Dependencies的list
 def dependencies = new ArrayList(configurations.embedded.resolvedConfiguration.firstLevelModuleDependencies)
    //遍历依赖将确定的路径放到另一个list embeddedAarDirs 中。
    dependencies.reverseEach { 
        def aarPath = "${exploded_aar_dir}/${it.moduleGroup}/${it.moduleName}/${it.moduleVersion}"
        println("shang--->" + aarPath)
        println("shang--->" + configurations.embedded.getName())
        it.moduleArtifacts.each {
            artifact ->
                if (artifact.type == 'aar') {
                    if (!embeddedAarDirs.contains(aarPath)) {
                        embeddedAarDirs.add(aarPath)
                    }

                } else if (artifact.type == 'jar') {
                    def artifactPath = artifact.file
                    if (!embeddedJars.contains(artifactPath))
                        embeddedJars.add(artifactPath)
                } else {
                    throw new Exception("Unhandled Artifact of type ${artifact.type}")
                }
                println("shang--->artifact:" + artifact)
        }
    }
```
我就好奇的打印了aar和artifact:
```java
shang--->D:/work/MyApplication/module1/build/intermediates/exploded-aar/MyApplication/module2/unspecified
shang--->embedded
shang--->artifact:[ResolvedArtifact dependency:org.gradle.api.internal.artifacts.ivyservice.dynamicversions.DefaultResolvedModuleVersion@b5eb5da name:module2 classifier:null extension:aar type:aar]
```
可以看到路径的问题也得到了解决。

继续下一个问题，不同的文件在Gradle Tasks中的哪个位置插入？
fat-aar是这样做的，通过不同的task之间的dependsOn ，mustRunAfter 这种方式来插入自定义的task，如下：
 ```groovy
 // Merge Assets
generateReleaseAssets.dependsOn embedAssets
embedAssets.dependsOn prepareReleaseDependencies

// Embed Resources by overwriting the inputResourceSets
packageReleaseResources.dependsOn embedLibraryResources
embedLibraryResources.dependsOn prepareReleaseDependencies

// Embed JNI Libraries
bundleRelease.dependsOn embedJniLibs
embedJniLibs.dependsOn transformNative_libsWithSyncJniLibsForRelease

// Merge Embedded Manifests
bundleRelease.dependsOn embedManifests
embedManifests.dependsOn processReleaseManifest

// Merge proguard files
embedLibraryResources.dependsOn embedProguard
embedProguard.dependsOn prepareReleaseDependencies

// Generate R.java files
compileReleaseJavaWithJavac.dependsOn generateRJava
generateRJava.dependsOn processReleaseResources

// Bundle the java classes
bundleRelease.dependsOn embedJavaJars
embedJavaJars.dependsOn compileReleaseJavaWithJavac

// If proguard is enabled, run the tasks that bundleRelease should depend on before proguard
if (tasks.findByPath('proguardRelease') != null) {
    proguardRelease.dependsOn embedJavaJars
} else if (tasks.findByPath('transformClassesAndResourcesWithProguardForRelease') != null) {
    transformClassesAndResourcesWithProguardForRelease.dependsOn embedJavaJars
}
```
看到这里，有些地方还是不明白，那就是android app 编译过程中task都有哪些？预定义的task都是什么？每个task的功能有是什么？
这里在build.gradle里调用一下脚本便可以打印该模块下所有的task：
```groovy
afterEvaluate {
    tasks.each {
        task ->
            task << {
                //checkNewFiles()
            }
            println task
    }
}
```
在console下键入：
```xml
gradlew clean assembleDebug > log.txt

```

再这里截取module1里执行的task，以及自定义task执行的位置，顺序执行，关注有颜色的行，我们便大致知道了执行的顺序，包括预定义的task执行：
```xml
:module1:prepareComAndroidSupportAnimatedVectorDrawable2511Library
:module1:prepareComAndroidSupportAppcompatV72511Library
:module1:prepareComAndroidSupportSupportCompat2511Library
:module1:prepareComAndroidSupportSupportCoreUi2511Library
:module1:prepareComAndroidSupportSupportCoreUtils2511Library
:module1:prepareComAndroidSupportSupportFragment2511Library
:module1:prepareComAndroidSupportSupportMediaCompat2511Library
:module1:prepareComAndroidSupportSupportV42511Library
:module1:prepareComAndroidSupportSupportVectorDrawable2511Library
:module1:prepareMyApplicationModule2UnspecifiedLibrary
:module1:prepareReleaseDependencies
:module1:compileReleaseAidl
:module1:compileReleaseNdk UP-TO-DATE
:module1:compileLint
:module1:copyReleaseLint UP-TO-DATE
:module1:compileReleaseRenderscript
:module1:generateReleaseResValues
:module1:generateReleaseResources
:module1:mergeReleaseResources
:module1:processReleaseManifest
:module1:processReleaseResources
:module1:generateRJava

<Running FAT-AAR Task :generateRJava>

:module1:generateReleaseBuildConfig
:module1:generateReleaseSources
:module1:incrementalReleaseJavaCompilationSafeguard
:module1:compileReleaseJavaWithJavac
:module1:compileReleaseJavaWithJavac - is not incremental (e.g. outputs have changed, no previous execution, etc.).
:module1:collectRClass
:module1:embedRClass
:module1:embedJavaJars

<Running FAT-AAR Task :embedJavaJars>

:module1:mergeReleaseShaders
:module1:compileReleaseShaders
:module1:embedAssets

<Running FAT-AAR Task :embedAssets>

:module1:generateReleaseAssets
:module1:mergeReleaseJniLibFolders
:module1:transformNative_libsWithMergeJniLibsForRelease
:module1:transformNative_libsWithSyncJniLibsForRelease
:module1:embedJniLibs

<Running FAT-AAR Task :embedJniLibs>

======= Copying JNI from D:/work/MyApplication/module1/build/intermediates/exploded-aar/MyApplication/module2/unspecified/jni
:module1:embedManifests

<Running FAT-AAR Task :embedManifests>

========== INFO : Loading library manifest D:\work\MyApplication\module1\build\intermediates\exploded-aar\MyApplication\module2\unspecified\AndroidManifest.xml
========== INFO : Merging main manifest D:\work\MyApplication\module1\build\intermediates\bundles\release\AndroidManifest.orig.xml

========== INFO : Merging library manifest D:\work\MyApplication\module1\build\intermediates\exploded-aar\MyApplication\module2\unspecified\AndroidManifest.xml
========== INFO : Merging manifest with lower AndroidManifest.xml:2:1-17:12
========== INFO : Merging uses-sdk with lower AndroidManifest.xml:7:5-9:41
========== INFO : Merging application with lower AndroidManifest.xml:11:5-15:19
========== INFO : Merging result:SUCCESS
========== INFO : Merged manifest saved to D:\work\MyApplication\module1\build\intermediates\bundles\release\AndroidManifest.xml
========== INFO : Merged aapt safe manifest saved to D:\work\MyApplication\module1\build\intermediates\manifests\aapt\release\AndroidManifest.xml
:module1:extractReleaseAnnotations
:module1:mergeReleaseAssets
:module1:mergeReleaseProguardFiles UP-TO-DATE
:module1:packageReleaseRenderscript UP-TO-DATE
:module1:embedProguard

<Running FAT-AAR Task :embedProguard>

:module1:embedLibraryResources

<Running FAT-AAR Task :embedLibraryResources>

:module1:packageReleaseResources
:module1:processReleaseJavaRes UP-TO-DATE
:module1:transformResourcesWithMergeJavaResForRelease
:module1:transformClassesAndResourcesWithSyncLibJarsForRelease
:module1:bundleRelease
```
而各个task的功能大家可以关注下文末链接android-fat-aar，在这里就不过多分析，基本是文件的操作。
#总结反思
在阅读fat-aar脚本的时候，开始有些地方似懂非懂，就接着往下看，结果看到最后再回头看前面就更不懂了，在此反思，其实不是“悟性”不够的问题，而是解决问题的方式的问题，对于这类“脚本”,首先看下文档，理清大致的结构，在分析的时候，要步步为营，不懂的地方要查看文档，合理的猜测验证；再找一些实际的例子一一分析，最后自己尝试写一些自定义的东西，这也是写这一系列文字的原因。
#参考
* [android-fat-aar](https://github.com/adwiv/android-fat-aar)
* [Configuration 属性概览](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.Configuration.html#org.gradle.api.artifacts.Configuration)
