# Android Studio使用自定义framework.jar

>作者:为何是Hex的昵称  
>简书:http://www.jianshu.com/u/0c6ff6654a6c

开发中,有时需要用到非公开的API，在以前，一般是通过反射去调用隐藏的API，但是这样就会存在性能隐患。这里介绍如何将 framework.jar 导入到 Android Studio 中，以去掉反射   
 
## 1. 准备 framewrok.jar

因为我是做系统应用开发，经常需要编译整个系统源码，所以 framework.jar 可以直接得到。路径：  
`out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes.jar`  

改名得到 framework.jar

## 2. 把 framework.jar 放到项目中

把 framework.jar 放到 app 模块的 libs 目录下

## 3. 添加 app 模块对 framework.jar 的依赖

依次打开 File –> Project Structure –> Modules 中找到 app ，在右边选择 Dependencies 选项卡，点击左下角的 + 按钮，选择 File dependency ，在弹出的 Select Path 窗口中选择 libs 中的 framework.jar

## 4. 修改 Scope 为 Provided

在新增的 Dependencies 记录的右边，将 Compile 修改为 Provided ，点击 OK 保存修改，Provided 的作用是只参与编译，但不打包到apk中

## 5. 修改项目根目录的 build.gradle 文件

在项目根 build.gradle 中加入以下内容

```java  
allprojects {
    repositories {
        jcenter()
    }
    gradle.projectsEvaluated {
        tasks.withType(JavaCompile) {
            options.compilerArgs << '-Xbootclasspath/p:app/libs/framework.jar'
        }
    }
} 
```  

## 6. 编写代码

按照以上5步修改完成后，就可以写代码了，需要注意的是，隐藏的API依然关联不到，显示红色的，但是可以顺利编译通过。

## 最后

解释一下代码的作用 allprojects 是要作用到所有的子模块上，tasks.withType(JavaCompile) 是在 javac 的 task 中加入一个参数，就是在 Xbootclasspath 增加自己的 jar 包

