#  Android Studio插件大全 

>作者: Remind   
博客: https://ydmmocoo.github.io/        
 
现在Android的开发者基本上都使用Android Studio进行开发(如果你还在使用eclipse那也行，毕竟你乐意怎么样都行)。使用好Android Studio插件能大量的减少我们的工作量。
# 1.[GsonFormat](http://plugins.jetbrains.com/plugin/7654?pr=androidstudio)
快速将json字符串转换成一个Java Bean，免去我们根据json字符串手写对应Java Bean的过程。

![](http://upload-images.jianshu.io/upload_images/4048192-a4599aee295b99e4.png?imageMogr2/auto-orient/strip)

使用方法：快捷键Alt+S也可以使用Alt+Insert选择GsonFormat

# 2.[Android ButterKnife Zelezny](http://plugins.jetbrains.com/plugin/7369?pr=androidstudio)
配合ButterKnife实现注解，从此不用写findViewById，想着就爽啊。在Activity，Fragment，Adapter中选中布局xml的资源id自动生成butterknife注解。

![](http://upload-images.jianshu.io/upload_images/4048192-f398f9b74be3c434.png?imageMogr2/auto-orient/strip)

使用方法：Ctrl+Shift+B选择图上所示选项

# 3.[Android Code Generator](http://plugins.jetbrains.com/plugin/7595?pr=androidstudio)
根据布局文件快速生成对应的Activity，Fragment，Adapter，Menu。

![](http://upload-images.jianshu.io/upload_images/4048192-b3813154e9fb1392.png?imageMogr2/auto-orient/strip)
![](http://upload-images.jianshu.io/upload_images/4048192-a4942db594ae8757.png?imageMogr2/auto-orient/strip)

# 4.[Android Parcelable code generator](http://plugins.jetbrains.com/plugin/7332?pr=androidstudio)
JavaBean序列化，快速实现Parcelable接口。

![](https://segmentfault.com/image?src=http://img.blog.csdn.net/20160416104459926&objectId=1190000005092842&token=ab29ed79d41be9e42b3a3d2ed1ec3bef)

# 5.[Android Methods Count](http://plugins.jetbrains.com/plugin/8076?pr=androidstudio)
显示依赖库中得方法数

![](http://upload-images.jianshu.io/upload_images/4048192-6a1a78698b201ea2.png?imageMogr2/auto-orient/strip)

# 6.[Lifecycle Sorter](http://plugins.jetbrains.com/plugin/7742?pr=androidstudio)
可以根据Activity或者fragment的生命周期对其生命周期方法位置进行先后排序，快捷键Ctrl + alt + K

![](http://upload-images.jianshu.io/upload_images/4048192-2e1348678bf9c679.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-62d215df682239b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 7.[CodeGlance](http://plugins.jetbrains.com/plugin/7275?pr=androidstudio)
在右边可以预览代码，实现快速定位

![](http://upload-images.jianshu.io/upload_images/4048192-279a58fac4b41947.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-3d667c37487dfddf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 8.[findBugs-IDEA](http://plugins.jetbrains.com/plugin/3847?pr=androidstudio)
查找bug的插件，Android Studio也提供了代码审查的功能（Analyze-Inspect Code...）

![](http://upload-images.jianshu.io/upload_images/4048192-cffa8ac968862b53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 9.[ADB WIFI](http://plugins.jetbrains.com/plugin/7856?pr=androidstudio)
使用wifi无线调试你的app，无需root权限
也可参考以下文章：
[Android wifi无线调试App新玩法ADB WIFI](http://www.jianshu.com/p/21d1b65d92a4)

![](http://upload-images.jianshu.io/upload_images/4048192-b8321d99b7bfe2c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 10.[AndroidPixelDimenGenerator](https://github.com/succlz123/AndroidPixelDimenGenerator)
Android Studio自动生成dimen.xml文件插件

![](https://github.com/succlz123/AndroidPixelDimenGenerator/raw/master/snapshot/1.jpeg)

# 11.[JsonOnlineViewer](http://plugins.jetbrains.com/plugin/7838?pr=androidstudio)
在Android Studio中请求、调试接口

![](http://upload-images.jianshu.io/upload_images/4048192-3668768dfed4e575.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 12.[Android Styler](http://plugins.jetbrains.com/plugin/7972?pr=androidstudio)
根据xml自动生成style代码的插件

![](http://upload-images.jianshu.io/upload_images/4048192-6550fd44b4b07a53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-6d945ed374fb408b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-35d69f9bca582a00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Usage:
a. copy lines with future style from your layout.xml file 
b. paste it to styles.xml file with Ctrl+Shift+D (or context menu) 
c. enter name of new style in the modal window
d. your style is prepared! 

# 13.[Android Drawable Importer](http://plugins.jetbrains.com/plugin/7658?pr=androidstudio)
这是一个非常强大的图片导入插件。它导入Android图标与Material图标的Drawable ，批量导入Drawable ，多源导入Drawable（即导入某张图片各种dpi对应的图片）

![](http://upload-images.jianshu.io/upload_images/4048192-0667fb2c5a62d94d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-02bdb68119caf84a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-a91e01e75eec2589.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-224adcd75e063333.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-ef5a299189c40d67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-b0a9d54ef06b2471.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-a65ca1bf6b1b51ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-7afc9d04d1608a0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 14.[SelectorChapek for Android](http://plugins.jetbrains.com/plugin/7298?pr=androidstudio)
通过资源文件命名自动生成Selector文件。

![](http://upload-images.jianshu.io/upload_images/4048192-811052b6e16391bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-b357d8d5c56f2316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/4048192-a613c19dc8c69b34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 15.[GenerateSerialVersionUID](http://plugins.jetbrains.com/plugin/185?pr=androidstudio)
实现Serializable序列化bean

Adds a new action 'SerialVersionUID' in the generate menu (alt + ins). The action adds an serialVersionUID field in the current class or updates it if it already exists, and assigns it the same value the standard 'serialver' JDK tool would return. The action is only visible when IDEA is not rebuilding its indexes, the class is serializable and either no serialVersionUID field exists or its value is different from the one the 'serialver' tool would return.

# 16.[genymotion](http://plugins.jetbrains.com/plugin/7269?pr=androidstudio)
速度较快的android模拟器

![](http://upload-images.jianshu.io/upload_images/4048192-ff8fa114030d7005.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 17.[LeakCanary](https://github.com/square/leakcanary)
帮助你在开发阶段方便的检测出内存泄露的问题，使用起来更简单方便。
可以参考以下文章：
[LeakCanary 中文使用说明](http://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)

![](http://upload-images.jianshu.io/upload_images/4048192-efdb85b458f08833.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 18.[Android Postfix Completion](http://plugins.jetbrains.com/plugin/7775?pr=androidstudio)
可根据后缀快速完成代码，这个属于拓展吧，系统已经有这些功能，如sout、notnull等，这个插件在原有的基础上增添了一些新的功能，我更想做的是通过原作者的代码自己定制功能，那就更爽了

![](http://upload-images.jianshu.io/upload_images/4048192-70c17b2804047b51.png?imageMogr2/auto-orient/strip)

# 19.[Android Holo Colors Generator](https://plugins.jetbrains.com/plugin/7366?pr=)
通过自定义Holo主题颜色生成对应的Drawable和布局文件

![](https://plugins.jetbrains.com/files/7366/screenshot_14379.png)

# 20.[dagger-intellij-plugin](https://github.com/square/dagger-intellij-plugin)
dagger可视化辅助工具

![](https://github.com/square/dagger-intellij-plugin/raw/master/images/inject-to-provide.gif)

# 21.[GradleDependenciesHelperPlugin](https://github.com/ligi/GradleDependenciesHelperPlugin)
maven gradle 依赖支持自动补全

![](https://camo.githubusercontent.com/d9b1b39eda21e0e33b656e2821f01897d915f7c5/68747470733a2f2f6c68332e676f6f676c6575736572636f6e74656e742e636f6d2f2d51364e7970315864594c772f556a73325a5175666634492f414141414141414144624d2f624d704c516742664d6b632f773538372d683330392d6e6f2f696465615f677261646c655f706c7567696e2e706e67)

# 22.[RemoveButterKnife](https://github.com/u3shadow/RemoveButterKnife)
ButterKnife这个第三方库每次更新之后，绑定view的注解都会改变，从bind,到inject，再到bindview，搞得很多人都不敢升级，一旦升级，就会有巨量的代码需要手动修改，非常痛苦
当我们有一些非常棒的代码需要拿到其他项目使用，但是我们发现，那个项目对第三方库的使用是有限制的，我们不能使用butterknife，这时候，我们又得从注解改回findviewbyid
针对上面的两种情况，如果view比较少还好说，如果有几十个view，那么我们一个个的手动删除注解，写findviewbyid语句，简直是一场噩梦（别问我为什么知道这是噩梦）
所以，这种有规律又重复简单的工作为什么不能用一个插件来实现呢？于是RemoveButterKnife的想法就出现了。

[具体介绍](http://www.u3coding.com/2016/06/24/androidstudio-plugin-removebutterknife-di/)

![](https://camo.githubusercontent.com/0327cda5b531ab6f2b803abe295c42225668d28d/687474703a2f2f7777772e7533636f64696e672e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031362f30362f312e676966)

# 23.[AndroidProguardPlugin](https://github.com/zhonghanwen/AndroidProguardPlugin)
一键生成项目混淆代码插件，值得你安装~(不过目前可能有些第三方项目的混淆还未添加完全)

![](http://upload-images.jianshu.io/upload_images/4048192-8d94865113f43c64.gif?imageMogr2/auto-orient/strip)

# 24.[otto-intellij-plugin](https://github.com/square/otto-intellij-plugin)
otto事件导航工具。

![](https://github.com/square/otto-intellij-plugin/raw/master/images/produce-to-subscribe.gif)
![](https://github.com/square/otto-intellij-plugin/raw/master/images/event-to-subscribe.gif)

# 25.[eventbus-intellij-plugin](https://github.com/kgmyshin/eventbus-intellij-plugin)
eventbus导航插件(对于最新版的 EventBus 3.0.0 好像无效,请替换为eventbus3-intellij-plugin此插件地址在本文第51个)

![](https://raw.githubusercontent.com/kgmyshin/eventbus-intellij-plugin/master/art/cap.gif)

# 26.[idea-markdown](https://github.com/nicoulaj/idea-markdown)
markdown插件

![](https://github.com/nicoulaj/idea-markdown/raw/assets/screenshots/preview.png)

# 27.[Sexy Editor](https://plugins.jetbrains.com/plugin/1833?pr=androidstudio)
设置AS代码编辑区的背景图

首先点击界面的设置按钮 进入设置界面，选中Plugins,右边选择 Browser ... ，输入Sexy ... 下面自动弹出候选插件，右边点击Install 安装
 ![](http://upload-images.jianshu.io/upload_images/4048192-34120323b5bd8337.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240))
安装成功 后需要重启AS

![](http://upload-images.jianshu.io/upload_images/4048192-3376b497c5d9c95b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
重启完成之后 进入设置界面 选择other Setting 下的Sexy Editor ， 右侧 insert 一张或多张图片即可，上面的其他设置可以设置方位 间隔时间 透明度等等，设置完成后，要关闭打开的文件，重新打开项目文件即可在代码编辑区显示插入的图片，作为代码编辑区的背景图。

![](http://upload-images.jianshu.io/upload_images/4048192-aa816e7625372cdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 28.[folding-plugin](https://github.com/dmytrodanylyk/folding-plugin)
布局文件分组的插件

![](https://github.com/dmytrodanylyk/folding-plugin/raw/master/screenshots/Preview.png)

# 29.[Android-DPI-Calculator](https://github.com/JerzyPuchalski/Android-DPI-Calculator)
DPI计算插件

![](https://camo.githubusercontent.com/ce3be2aaa3b1f70b90f5b825c529694509d70313/68747470733a2f2f7261772e6769746875622e636f6d2f4a65727a7950756368616c736b692f416e64726f69642d4450492d43616c63756c61746f722f6d61737465722f696d672f6469616c6f672e706e67)

使用：
![](https://camo.githubusercontent.com/598d3b5c9efc5f0b57b58c25a79a323d06307fad/68747470733a2f2f7261772e6769746875622e636f6d2f4a65727a7950756368616c736b692f416e64726f69642d4450492d43616c63756c61746f722f6d61737465722f696d672f616374696f6e2e706e67)
或者
![](https://camo.githubusercontent.com/7a8f977de7a1ba6cd23fb64cbd37566690c27cdc/68747470733a2f2f7261772e6769746875622e636f6d2f4a65727a7950756368616c736b692f416e64726f69642d4450492d43616c63756c61746f722f6d61737465722f696d672f6d656e752e706e67)

# 30.[gradle-retrolambda](https://github.com/evant/gradle-retrolambda)
在java 6 7中使用 lambda表达式插件

修改编译的jdk为java8:
![](http://upload-images.jianshu.io/upload_images/4048192-cbb7f0cee48b908c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 31.[Android Studio Prettify](https://plugins.jetbrains.com/plugin/7405)
可以将代码中的字符串写在string.xml文件中

选中字符串鼠标右键选择图中所示
![](https://plugins.jetbrains.com/files/7405/screenshot_14417.png)

这个插件还可以自动书写findViewById
![](https://plugins.jetbrains.com/files/7405/screenshot_14418.png)

![](https://plugins.jetbrains.com/files/7405/screenshot_14416.png)

![](https://plugins.jetbrains.com/files/7405/screenshot_14501.png)

![](https://plugins.jetbrains.com/files/7405/screenshot_14419.png)

![](https://plugins.jetbrains.com/files/7405/screenshot_14415.png)

# 32.[Material Theme UI](https://plugins.jetbrains.com/plugin/8006?pr=)
添加Material主题到你的AS

![](https://plugins.jetbrains.com/files/8006/screenshot_15722.png)

![](https://plugins.jetbrains.com/files/8006/screenshot_15723.png)

![](https://plugins.jetbrains.com/files/8006/screenshot_15721.png)

# 33.[.ignore](https://plugins.jetbrains.com/plugin/7495?pr=)
我们都知道在Git 中想要过滤掉一些不想提交的文件，可以把相应的文件添加到.gitignore 中，而.gitignore 这个Android Studio 插件根据不同的语言来选择模板，就不用自己在费事添加一些文件了，而且还有自动补全功能，过滤文件再也不要复制文件名了。我们做项目的时候，并不是所有文件都是要提交的，比如构建的build 文件夹，本地配置文件，每个Module 生成的iml 文件，但是我们每次add，commit 都会不小心把它们添加上去，而gitignore 就是解决这种痛点的，如果你不想提交的文件，就可以在创建项目的时候将这个文件中添加即可，将一些通用的东西屏蔽掉。

![](https://plugins.jetbrains.com/files/7495/screenshot_14960.png)

![](https://plugins.jetbrains.com/files/7495/screenshot_14958.png)

![](https://plugins.jetbrains.com/files/7495/screenshot_14959.png)

# 34.[CheckStyle-IDEA](https://plugins.jetbrains.com/plugin/1065?pr=)
CheckStyle-IDEA 是一个检查代码风格的插件，比如像命名约定，Javadoc，类设计等方面进行代码规范和风格的检查，你们可以遵从像Google Oracle 的Java 代码指南 ，当然也可以按照自己的规则来设置配置文件，从而有效约束你自己更好地遵循代码编写规范。

# 35.[Markdown Navigator](https://plugins.jetbrains.com/plugin/7896?pr=)
github:[Markdown Navigator](https://github.com/vsch/idea-multimarkdown/wiki)
Markdown插件

![](https://plugins.jetbrains.com/files/7896/screenshot_15818.png)

# 36.[ECTranslation](https://github.com/Skykai521/ECTranslation)
Android Studio Plugin,Translate English to Chinese. Android Studio 翻译插件,可以将英文翻译为中文。

![](https://github.com/Skykai521/ECTranslation/raw/master/img/translation_img.png)

# 37.[PermissionsDispatcher plugin](https://plugins.jetbrains.com/plugin/8349)
github:[PermissionsDispatcher plugin](https://github.com/shiraji/permissions-dispatcher-plugin)
自动生成6.0权限的代码

![](https://github.com/shiraji/permissions-dispatcher-plugin/raw/master/website/images/pd.gif)

# 38.[WakaTime](https://plugins.jetbrains.com/plugin/7425?pr=)
github:[WakaTime](https://github.com/wakatime/jetbrains-wakatime)
记录你在IDE上的工作时间

![](https://plugins.jetbrains.com/files/7425/screenshot_14794.png)

# 39.[AndroidWiFiADB](https://github.com/pedrovgs/AndroidWiFiADB)
无线调试应用

![](https://github.com/pedrovgs/AndroidWiFiADB/raw/master/art/screenshot1.gif)

![](https://github.com/pedrovgs/AndroidWiFiADB/raw/master/art/android_devices_window.png)

# 40.[AndroidLocalizationer](https://github.com/westlinkin/AndroidLocalizationer)
可用于将项目中的 string 资源自动翻译为其他语言的 Android Studio/IntelliJ IDEA 插件

![](https://raw.githubusercontent.com/westlinkin/AndroidLocalizationer/master/screen_shot_3.png)

![](https://raw.githubusercontent.com/westlinkin/AndroidLocalizationer/master/screen_shot_2.png)

# 41.[TranslationPlugin](https://github.com/YiiGuxing/TranslationPlugin)
又一翻译插件,可中英互译。

![](https://raw.githubusercontent.com/YiiGuxing/TranslationPlugin/master/images/1.png)

![](https://raw.githubusercontent.com/YiiGuxing/TranslationPlugin/master/images/3.png)

# 42.[SingletonTest](https://github.com/luhaoaimama1/SingletonTest)
快速生成单例模式的预设

![](https://github.com/luhaoaimama1/SingletonTest/raw/master/demo/tip1.png)

![](https://github.com/luhaoaimama1/SingletonTest/raw/master/demo/tip2.png)

![](https://github.com/luhaoaimama1/SingletonTest/raw/master/demo/tip3.png)

# 43.[BorePlugin](https://github.com/boredream/BorePlugin)
Android Studio 自动生成布局代码插件

![](https://github.com/boredream/BorePlugin/raw/master/screenshot/LayoutCreator.gif)

## 代码生成规则

a.自动遍历目标布局中所有带id的文件, 无id的不会识别处理
b.控件生成的变量名默认为id名称, 可以在弹出确认框右侧的名称输入栏中自行修改
c.所有的Button或者带clickable=true的控件, 都会自动在代码中生成setOnClickListener相关代码
d.所有EditText控件, 都会在代码中生成非空判断代码, 如果为空会提示EditText的hint内容, 如果hint为空则提示xxx字符串不能为空字样, 最后会把所有输入框的验证合并到一个submit方法中
e.会自动识别布局中的include标签, 并读取对应布局中的控件

# 44.[jimu Mirror](http://www.jimumirror.com/mirror-downloads/)
能够实时预览Android布局，它会监听布局文件的改动，如果有代码变化，就会立即刷新UI。

# 45.[jRebel For Android](http://zeroturnaround.com/software/jrebel-for-android/)
不仅能够做到UI布局的实时预览，它甚至做到了让你更改java代码后就能实时替换apk中的类文件，达到应用实时刷新，官网的介绍是：Skip build, install and run，因此它可以节约我们很多很多的时间，它的效果也十分不错。

![](http://upload-images.jianshu.io/upload_images/4048192-7ee698871065246b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/4048192-8c50d19d8de4221c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/4048192-1883d7c6f7043055.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 46.[sdk-manager-plugin](https://github.com/JakeWharton/sdk-manager-plugin)
SDK管理插件，自动检测更新并下载。(图片与插件无关哈)

![](https://camo.githubusercontent.com/95469d65798f62a50a9fcabe21e2cc303a1b859c/687474703a2f2f692e696d6775722e636f6d2f384a734a587a6e2e6a7067)

# 47.[Codota](http://www.codota.com/)
搜索最好的Android代码。(Studio里面直接可以搜到此插件)

# 48.[LayoutFormatter](https://github.com/drakeet/LayoutFormatter)
drakeet 开发一个一键格式化你的 XML 文件的 Android Studio 插件，至于为什么不用 Android Studio 自带的格式化功能而用这个插件，可以看下作者的一篇 Blog -> [当我们谈 XML 布局文件代码的优雅性](https://drakeet.me/layoutformatter)

![](http://upload-images.jianshu.io/upload_images/4048192-b6bd4964ebb85ff3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 49.[android-strings-search-plugin](https://github.com/konifar/android-strings-search-plugin)
一个可以通过输入文字找到strings.xml资源的插件

![](https://github.com/konifar/android-strings-search-plugin/raw/master/art/demo.gif)

# 50.[ideaVim](http://plugins.jetbrains.com/plugin/164?pr=androidstudio)
vim 本身就是一款很优秀的文本编辑器，而Android Studio 更是一款编写APP应用的神器。如果两个款优秀的软件结合在一起感觉会怎样呢？
详细请看文章:[Android Studio ＋Vim](http://www.jianshu.com/p/43862126b88f)

![](http://upload-images.jianshu.io/upload_images/1825722-8b55d9654777599e.gif?imageMogr2/auto-orient/strip)

# 51.[eventbus3-intellij-plugin](https://github.com/likfe/eventbus3-intellij-plugin/blob/master/README-zh.md)
引导 EventBus 的 post 和 event(对于最新版的 EventBus 3.0.0 有效)
主要Bug修复工作：
修改包名和方法名以适应 EventBus 3.X
替换一个在新版的 intellij plugin SDK 已经不存在的类
增加若干 try-catch ，防止插件崩溃

![](https://raw.githubusercontent.com/likfe/eventbus3-intellij-plugin/master/art/cap.gif)

# 52.[Exynap](http://exynap.com/)
Exynap 一个帮助开发者自动生成样板代码的 AndroidStudio 插件

![](http://upload-images.jianshu.io/upload_images/4048192-b3555a820720dd4c.gif?imageMogr2/auto-orient/strip)

# 53.[gradle-cleaner-intellij-plugin](https://github.com/Softwee/gradle-cleaner-intellij-plugin)
Force clear delaying & no longer needed Gradle tasks.

![](https://camo.githubusercontent.com/cb48bca7f8bd0b513f350f7320c74054d1b9fbce/687474703a2f2f6936352e74696e797069632e636f6d2f726a687863382e706e67)

# 54.[MVPHelper](http://androidwing.net/index.php/27)
一款Intellj IDEA 和Android Studio的插件，可以为MVP生成接口以及实现类，解放双手。
具体请查看[Android Studio插件之MVPHelper，一键生成MVP代码](http://androidwing.net/index.php/27)一文

![](https://github.com/githubwing/MVPHelper/raw/master/img/mvp_presenter.gif) 

#55.[Matchmaker](https://github.com/lypeer/Matchmaker)
这是一款专为微信小程序开发的插件，目前可在 IntelliJ IDEA 中使用。它可以帮你完成重复机械无趣麻烦的绑定方法的过程，自动的将需要新建的方法注入到 js 文件中去。
![](https://raw.githubusercontent.com/lypeer/Matchmaker/master/gif/plugin.gif)  

# 56.[Emoji Support Plugin](https://plugins.jetbrains.com/plugin/9174)
让 Intellij 支持 Emoji 输入提醒   

![](https://github.com/shiraji/emoji/raw/master/website/images/commit.gif)

# 57.[Open-Uploader](https://github.com/fingerart/Open-Uploader)  
上传apk文件到指定的地址，提供自定义参数  

![](https://camo.githubusercontent.com/aa9d4468da25629928fc71256834a81fdaf2bebc/687474703a2f2f66696e6765726172742e71696e6975646e2e636f6d2f323031362d31302d31322d6f70656e5f75706c6f616465725f707265766965772e676966)
