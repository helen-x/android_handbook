#前言
在学习Gradle时，首先要对Groovy这个脚本语言的常用的东西有个概念，以后也好阅读别人的脚本或者自己写插件。
#Groovy中的基本数据类型
数据类型可以在[Groovy Api 在线文档](http://www.groovy-lang.org/api.html)的lang包下看到，大致有以下三类：
```gradle
1. java中的基本类型
2. 容器类型（List ，Map，Range）
3. 闭包
```
* List
类似ArrayList ,   def alist = ['a',1,true];   alist[1] == 1; alist.size == 3
* Map
类似LinkedHashMap，def amap = ['key1':value1,'key2':value2]; 
* Range
对List的扩展
**[IntRange](http://docs.groovy-lang.org/latest/html/gapi/groovy/lang/IntRange.html)** ：Represents a list of Integer objects from a specified int up (or down) to and including a given to
**[ObjectRange](http://docs.groovy-lang.org/latest/html/gapi/groovy/lang/ObjectRange.html)**：Represents an inclusive list of objects from a value to a value using comparators.
def arange = 1..5 //表示1，2，3，4，5
def arange = 1..<5//表示1，2，3，4
* [Closure](http://docs.groovy-lang.org/latest/html/gapi/groovy/lang/Closure.html) 
闭包的定义:
```java
 def a = 1
 def c = { a }
 assert c() == 1
//或者
 def a = 1
 def c = {a}
 assert c.call() == 1
```
在闭包的学习中，有以下需要注意的地方：
 * it
 如果闭包没定义参数的话，则隐含有一个参数，这个参数名字叫 it ，和 this  的作用类似。it  代表闭包的参数。
 *  省略圆括号
闭包在 Groovy 中大量使用，比如很多类都定义了一些函数，这些函数最后一个参数都
是一个闭包。比如：
```java
public static <T> List<T> each(List<T> self, Closure closure)
```
上面这个函数表示针对List的每一个元素都会调用closure做一些处理。这里的closure，
就有点回调函数的感觉。但是，在使用这个 each 函数的时候，我们传递一个怎样的 Closure
进去呢？比如：
```gradle
def iamList = [1,2,3,4,5] //定义一个 List
iamList.each{ //调用它的 each，这段代码的格式看不懂了吧？each 是个函数，圆括号去哪了？
          println it
}
```
上面代码有两个知识点：
  each  函数调用的圆括号不见了！原来，Groovy 中，当函数的最后一个参数是闭包的话，可以省略圆括号。比如
```gardle
def testClosure(int a1,String b1, Closure closure){
       //do something
        closure() //调用闭包
}
那么调用的时候，就可以免括号！
testClosure   4, "test", {
         println "i am in closure"
} 
```
省略圆括号虽然使得代码简洁，看起来更像脚本语言，但是它这经常会让我 confuse（不知道其他人是否有同感），以 doLast 为例，完整的代码应该按下面这种写法：
```java
doLast({
      println 'Hello world!'
})
```
有了圆括号，你会知道 doLast 只是把一个 Closure 对象传了进去。很明显，它不代表这段脚本解析到 doLast 的时候就会调用 println 'Hello world!' 。但是把圆括号去掉后，就感觉好像 println 'Hello world!'立即就会被调用一样！
 * 如何确定 Closure 的参数
另外一个比较让人头疼的地方是，Closure 的参数该怎么搞？还是刚才的 each 函数：
```java
public static <T> List<T> each(List<T> self, Closure closure)
```
each 函数说明中，将给指定的 closure 传递 Set 中的每一个 item。所以，closure的参数只有一个。
 findAll 中呢，就不知道该怎么办了。一个是没说明往 Closure 里传什么。另外没说明 Closure的返回值是什么.....。
Closure 虽然很方便，但是它一定会和使用它的上下文有极强的关联。要不，作为类似回调这样的东西，我如何知道调用者传递什么参数给 Closure 呢？此问题如何破解？只能通过查询 API 文档才能了解上下文语义。所以最初阶段，SDK 查询是必不可少的。

#文件IO和XML操作
文件读写：[File](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/File.html)
```java
def targetFile = new File(文件名) <==File 对象还是要创建的。

看看 Groovy 定义的 API：

1 读该文件中的每一行：eachLine 的唯一参数是一个 Closure。Closure 的参数是文件每一行的内容
其内部实现肯定是 Groovy 打开这个文件，然后读取文件的一行，然后调用 Closure...

    targetFile.eachLine{
        String oneLine ->
        println oneLine
    } <==是不是令人发指？？！

2 直接得到文件内容
    targetFile.getBytes() <==文件内容一次性读出，返回类型为 byte[]
   //注意前面提到的 getter 和 setter 函数，这里可以直接使用 targetFile.bytes....

3 使用 InputStream.InputStream 的 SDK 在http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html
    def ism = targetFile.newInputStream()
    //操作 ism，最后记得关掉
    ism.close

4 使用闭包操作 inputStream，以后在 Gradle 里会常看到这种搞法
    targetFile.withInputStream{ ism ->
       操作 ism. 不用 close。Groovy 会自动替你 close
    }

5 写文件
    def srcFile = new File(源文件名)
    def targetFile = new File(目标文件名)
    targetFile.withOutputStream{ os->
                srcFile.withInputStream{ ins->
              os << ins //利用 OutputStream 的<<操作符重载，完成从 inputstream 到 OutputStream的输出
    }
}
```
Xml解析：  [GPathResult](http://docs.groovy-lang.org/latest/html/gapi/groovy/util/slurpersupport/GPathResult.html),[XmlSlurper](http://docs.groovy-lang.org/latest/html/gapi/groovy/util/XmlSlurper.html)
```java
  def androidManifest = new XmlSlurper().parse("AndroidManifest.xml")
  println androidManifest['@android:versionName']
  或者
  println androidManifest.@'android:versionName'
```

#Gradle模型常用数据类型
* [Project
](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)
映射到build.gradle文件，从Project你可以获取Gradle的所有features，其中主要的Method如下：

![Project Method 1](http://upload-images.jianshu.io/upload_images/22193-a1c9251312178038.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Project Method 2](http://upload-images.jianshu.io/upload_images/22193-adb9cd15f9755613.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Project Method 3](http://upload-images.jianshu.io/upload_images/22193-e07cfbf0443889b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Project Method 4](http://upload-images.jianshu.io/upload_images/22193-ddd7e45d99f79d1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* [Task
](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html)
一个工程是很多个Tasks的集合，每个Task都会有一个基本的独特的功能，或者单元测试，或者单纯的打一个jar包。
在script中可以这样定义Task:
  ```gradle
task myTask
task myTask { configure closure }
task myTask(type: SomeType)
task myTask(type: SomeType) { configure closure }
  ```
TaskContainer包含了所有的task任务，我们可以通过在Project中的getTasks().findByPath('packageReleaseJniLibs')来获取特定名字的task，然后可以通过dependsOn，mustRunAfter 等来插入自己定义的Task。
![Task Property](http://upload-images.jianshu.io/upload_images/22193-bdbbf35b7272fc29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* [Gradle
](https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html)
Gradle对象实例，通过[Project.getGradle()](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:gradle)可以 获取。我们需要重点关注下其中的method方法，对于我们定制化有很大的意义。

![Gradle的methods 1](http://upload-images.jianshu.io/upload_images/22193-4e667cb4f5a80037.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Gradle的methods 2](http://upload-images.jianshu.io/upload_images/22193-4b9a88adda38ed61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* [Settings
](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html)
映射到settings.gradle文档
* [Script
](https://docs.gradle.org/current/dsl/org.gradle.api.Script.html)
简单来说，Script 是一个接口，Gradle中的脚本会implement这个接口并实现自己的功能，并且每个Script都会有个Groovy对象相应，例如一个 build   script 对应一个project,一个init script 对应一个Gradle 对象。在此Script对象上找不到的任何属性引用或方法调用都将转发到委托对象。
#参考
* [Groovy Api 在线文档](http://www.groovy-lang.org/api.html)
* [Gradle Api 在线文档](https://docs.gradle.org/current/javadoc/)
