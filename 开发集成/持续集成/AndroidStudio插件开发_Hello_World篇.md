#  AndroidStudio插件开发（Hello World篇）   


>作者:华超  
>博客:http://blog.csdn.net/huachao1001  


工欲善其事必先利其器，自打从Eclipse转战AndroidStudio以来，还没彻底摆脱Eclipse。打算从开发AndroidStudio插件开始，彻底摆脱Eclipse。   
AndroidStudio基于IntelliJ平台，因此，开发AndroidStudio插件其本质只是开发IntelliJ平台的插件。通常我们开发的IntelliJ平台插件主要分为如下几类：   

>1. 自定义编程语言的支持（Custom language support）：包括语法高亮、文件类型识别、代码格式化、代码查看和自动补全等等
2. 框架集成（Framework integration）：其实就是类似基于IntelliJ开发一个IDE出来，比如AndroidStudio 将Android SDK集成进IntelliJ。其他的插件如Java EE中的Spring、Struts等framework集成到IntelliJ。使用户在IntelliJ上面使用特定的框架更方便。
3. 工具集成（Tool integration）：对IntelliJ定制一些个性化或者是实用的工具。
4. 附加UI（User interface add-ons）：对标准的UI界面进行修改，如在编辑框里加一个背景图片等  

 
## 1. 下载IntelliJ IDEA

在开发IntelliJ插件时，我们使用的是IntelliJ IDEA自身来开发。为什么不用AndroidStudio来开发呢？主要是AndroidStudio是面向Android开发的，在针对IntelliJ插件的各种环境都没有，当然，你也可以自己下载插件开发环境然后在AndroidStudio上去配置。但是个人觉得过于麻烦，IntelliJ IDEA集成了插件开发环境，下载后可以直接拿来开发插件。IntelliJ IDEA下载地址如下：

> https://www.jetbrains.com/idea/  
 
# 2. 创建项目

选择“File>New>Project…”,将Project选择为IntelliJ Platform Plugin，然后再点击Next。如下图所示：

![](http://upload-images.jianshu.io/upload_images/4048192-7fb703e7358c032a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择IntelliJ Platform Plugin

填写Project名称及项目保存路径，其中Project Name可以认为是插件名称。点击Finish，如下图所示。

![](http://upload-images.jianshu.io/upload_images/4048192-30adcef18997b12d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

完成后，创建的项目结构如下所示：  
![](http://upload-images.jianshu.io/upload_images/4048192-f38e44b5a98faeae?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

我们比较关心的主要是src目录和resources/META-INF/plugin.xml文件。  
- src目录存放的是插件对应的Java源码， 
- resources/META-INF/plugin.xml是配置Action的文件.   
关于Action后面讲，现在暂时可以将resources/META-INF/plugin.xml看成是插件的配置文件。

## 3. 创建Action

我们在IntelliJ自定义的插件可以添加到菜单项目（如右键菜单中）或者是放在工具栏中。当用户点击时触发一个动作事件，IntelliJ则会回调AnAction类的actionPerformed函数。因此我们只需重写actionPerformed函数即可。 
在src目录中创建包名：com.huachao.plugin,然后，在com.huachao.plugin中创建java类，类为FirstPlugin.java

![](http://upload-images.jianshu.io/upload_images/4048192-9b196b30a40ebce8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

将FirstPlugin继承AnAction类，并重写actionPerformed函数。 

```java  
package com.huachao.plugin;

import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;
import com.intellij.openapi.actionSystem.PlatformDataKeys;
import com.intellij.openapi.project.Project;
import com.intellij.openapi.ui.Messages;

/**
 * Created by HuaChao on 2016/12/24.
 */
public class FirstPlugin extends AnAction {
    @Override
    public void actionPerformed(AnActionEvent event) {
        Project project = event.getData(PlatformDataKeys.PROJECT);
        Messages.showMessageDialog(project, "Hello World!", "Information", Messages.getInformationIcon());
    }
}
 ```
 
 
## 4. 修改plugin.xml

上一节我通过继承AnAction类来定义Action，现在我们需要将我们自定义的插件放入到工具类或者是菜单子项中，这就是通过plugin.xml指定。打开resources/META-INF/plugin.xml文件，IntelliJ帮我们自动生成内容如下：

```java   

<idea-plugin version="2">
    <id>com.your.company.unique.plugin.id</id>
    <name>Plugin display name here</name>
    <version>1.0</version>
    <vendor email="support@yourcompany.com" url="http://www.yourcompany.com">YourCompany</vendor>

    <description><![CDATA[
        Enter short description for your plugin here.<br>
        <em>most HTML tags may be used</em>
      ]]></description>

    <change-notes><![CDATA[
        Add change notes here.<br>
        <em>most HTML tags may be used</em>
      ]]>
    </change-notes>

    <!-- please see http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/build_number_ranges.html for description -->
    <idea-version since-build="145.0"/>

    <!-- please see http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/plugin_compatibility.html
         on how to target different products -->
    <!-- uncomment to enable plugin in all products
    <depends>com.intellij.modules.lang</depends>
    -->

    <extensions defaultExtensionNs="com.intellij">
      <!-- Add your extensions here -->
    </extensions>

    <actions>
      <!-- Add your actions here -->
    </actions>

</idea-plugin>
```    


每个标签的具体作用在注释中解释的很详细，我们现在只关心<actions>标签，暂时先忽略其他标签。在<actions>标签中添加<action>子标签，如下所示：

```xml
<actions>

    <action id="HelloWorld.FirstPlugin" class="com.huachao.plugin.FirstPlugin" text="Hello World" description="A test menu ">
        <add-to-group group-id="HelpMenu" anchor="first"/>
    </action>

</actions>
```  

<action>标签属性的简单说明：

>id：作为<action>标签的唯一标识。一般以<项目名>.<类名>方式。   
class：即我们自定义的AnAction类   
text：显示的文字，如我们自定义的插件放在菜单列表中，这个文字就是对应的菜单项    
description：对这个AnAction的描述    


另外还有<add-to-group>标签，这个标签指定我们自定义的插件应该放入到哪个菜单下面。在IntelliJ IDEA菜单栏中有很多菜单如File、Edit、View、Navigate、Code、……、Help等。他们的ID一般是菜单名+Menu的方式。比如，我们想将我们自定义的插件放到Help菜单中，作为Help菜单的子选项。那么在<add-to-group>标签中指定group-id="HelpMenu"。<add-to-group>标签的anchor属性用于描述位置，主要有四个选项：first、last、before、after。他们的含义如下：

>first：放在最前面   
last：放在最后   
before：放在relative-to-action属性指定的ID的前面   
after：放在relative-to-action属性指定的ID的后面  
relative-to-action也是<add-to-group>的属性。  

## 5. 运行

完成plugin.xml的修改后，点击运行。会发现，运行时是自动再启动新的IntelliJ IDEA。而新启动的IntelliJ IDEA由于没有可打开的项目会停留在如下界面：

![](http://upload-images.jianshu.io/upload_images/4048192-d0de263d500ff5a9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了能查看到我们的插件，可以点击Create New Project或者是导入项目，总之，让它正确进入到开发界面就好。

接下来，点击help菜单，会看到如下：  
![](http://upload-images.jianshu.io/upload_images/4048192-911b9d99e3858d81?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 

点击·”Hello World”项，运行如下：

![](http://upload-images.jianshu.io/upload_images/4048192-3f4c5316fd12bf23?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 6. 其他

## 6.1 自动配置plugin.xml

前面我们通过收到创建Java类，然后基础AnAction类的方式创建Action。并且最后还需要收到配置plugin.xml。其实可以无需手动编写，直接通过New>Action的方式创建，如下图所示。

![](http://upload-images.jianshu.io/upload_images/4048192-1ca1898826941e96?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开的图形化创建界面如下：

![](http://upload-images.jianshu.io/upload_images/4048192-c4ba83923e04a38c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，需要填写的部分跟我们plugin.xml中的一一对应。

## 6.2 卸载插件

当我们按照上面的方面再创建一个插件时，发现上一次的插件还会出现。而且我们创建的新的插件不会出现，这是什么原因呢？这主要是，我们没有修改插件名称，并且没有修改版本。这样的话自然就没有覆盖原先的插件了。因此我们有2种方法，第一种就是将原来的插件卸载，第二种就是将新建的插件名称与原先的区分开来（在plugin.xml中 <name>标签中指定）。第二种方法比较简单，我们看看第一种方法。  

首先，点击“File>Settings>Plugins”，如下图。

![](http://upload-images.jianshu.io/upload_images/4048192-07d491b46ce4bb7e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  


找到插件名称，这里也就是“Plugin display name here”,因为一开始我们没有修改插件名称，这个名词是自动生成的。然后点击Uninstall，最后重启或者是关闭IntelliJ IDEA完成卸载。

## 6.3 打包插件并在AndroidStudio中安装

回到主题，我们开发的插件是希望运行在AndroidStudio中。因此少不了在IntelliJ中打包和在AndroidStudio中安装的过程。点击“Build>Prepare All Plugin Modules For Deployment”，如下图：  

![](http://upload-images.jianshu.io/upload_images/4048192-6c269993f5ede23e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时在HelloWorld项目中多了一个HelloWorld.jar文件，如下图：

![](http://upload-images.jianshu.io/upload_images/4048192-2c7b640a9fa6f1cd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个文件即为我们导出的插件。接下来打开AndroidStudio，点击“File>Settings>Plugins”  

![](http://upload-images.jianshu.io/upload_images/4048192-4f714f41621c5f67?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击“install plugin from disk…”,将HelloWorld.jar包加入即可完成安装。  

我在安装过程中出现如下错误（Plugin display name here为插件名称）：  

![](http://upload-images.jianshu.io/upload_images/4048192-2b5b7e957b9279f4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从错误提示上看，是不兼容错误。回到plugin.xml，从中找到一行：

```xml
 <idea-version since-build="145.0"/>
```  

这是指定IntelliJ IDEA为2016年1月发布的版本（点击这里查看对应的版本）,显然，我的AndroidStudio还没使用那么新的IntelliJ IDEA，因此把145修改小一点就好，比如我修改为105.0，重新打包再安装。运行如下：

![](http://upload-images.jianshu.io/upload_images/4048192-fd57cfbfc494b58b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  


点击后：  

![](http://upload-images.jianshu.io/upload_images/4048192-1f708cc125bf39f8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行结果
参考资料：http://www.jetbrains.org/intellij/sdk/docs/index.html 
