# 几条经验美化你的GitHub开源项目  


> 作者: hotBitmapGG  
> 简书:http://www.jianshu.com/u/566d6cec0ebc    

很多同学都会在Github上有自己的开源项目，这里我把自己知道一些小经验分享一下。  

## 1. [shields.io-生成标签](http://shields.io/)  

![](http://upload-images.jianshu.io/upload_images/4048192-906d02790b651ae2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

这个网站可以找到各种README的标签，标签的主要功能是展示项目的各种信息，当然主要用途还是持续性地跟踪当前库的各种状态并展示出来，而且它是自动的。比如某一个提交导致项目无法通过编译，编译标签会给出编译失败的状态，告诉大家这个库现在编译不了。再比如当前库中的某个测试用例测试无法通过时，测试标签会显示无法通过测试的状态，告诉大家，这个库现在还些问题等等。 

当然还可以自定义标签，这里简单介绍下怎么自定义标签，把网站拉到最下边，可以看到自定义的地方。 

![](http://upload-images.jianshu.io/upload_images/4048192-35b60f0e7e531198.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

![](http://upload-images.jianshu.io/upload_images/4048192-1812c472bd449ff5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

比如这里定义一个Gradle的版本标签。
![](http://upload-images.jianshu.io/upload_images/4048192-3d06e2d16b99342e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

点击Make Badge会生成一个SVG格式的图片，把url复制下在返回，然后拉到网站上边，找一个现有的标签，点击会出现一个弹窗。
![](http://upload-images.jianshu.io/upload_images/4048192-86c16fb188b9e241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)   

把image位置的链接改为我们自己刚生成自定义的标签的image地址，黏贴上去即可。
![](http://upload-images.jianshu.io/upload_images/4048192-7afd49e762210ccb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

这里还可以选择需要的样式，3种样式选择,然后在Link位置填上点击跳转的地址，这里按自己的需求写上即可，最后复制markdown的链接到你项目的README中使用就可以啦。
![](http://upload-images.jianshu.io/upload_images/4048192-36d76dbfa380eda4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

ok,到这里Shield.io就介绍完了。  

## 2. [progressed.io-生成进度](https://github.com/fehmicansaglam/progressed.io)  
![](http://upload-images.jianshu.io/upload_images/4048192-28c67b58c403bbc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

这个是GitHub上一个开源项目，有兴趣的同学可以关注下。


## 3. [ezgif-生成GIF](http://ezgif.com/)
这里介绍一个录制Gif超级棒的网站，录制出来的质量很清晰，下边介绍下怎么使用。

先用AndroidStudio或者其他方式录制视频文件. 
 
1. 这里你已经录制好了需要的视频，然后打开[ezgif.com](http://ezgif.com/),找到Video to Gif

![](http://upload-images.jianshu.io/upload_images/4048192-9add67e843c755a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.选择需要转换的视频文件，点击upload按钮上传

![](http://upload-images.jianshu.io/upload_images/4048192-ec239645d0962446.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.上传完成会出现这个界面，Start time (seconds)，End time (seconds)这两个地方设置gif的开始和结束时间，其他的设置默认即可，然后点击Convert to Gif按钮，开始转换gif文件

![](http://upload-images.jianshu.io/upload_images/4048192-26e4801ac8f287b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.稍等片刻转换成功，下图是转换后的gif信息，点击保存图片就ok了

![](http://upload-images.jianshu.io/upload_images/4048192-3c286282f7801ef7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4. [带壳截图](http://sspai.com/27937)
来自少数派的带壳截图App，让你的开源项目截图马上有逼格起来。
![](http://upload-images.jianshu.io/upload_images/4048192-bcd1f86a1eaef275.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/4048192-210e2544556a0a70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/4048192-4e717e05e7f199d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/4048192-be3746ca04f9b8a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

请在各大渠道市场下载该App使用即可。

## 5. [Material design icon](https://android-material-icon-generator.bitdroid.de/)
最后介绍一个MD的icon制作网站。
![](http://upload-images.jianshu.io/upload_images/4048192-2e09675bbf9ad7d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个使用就很简单了，左边是可以自定义拖一个svg的图标进来，来生成一个你需要的icon，右边的是选择Icons are copied from the Google Material icons collection.Google的图标合集，来生成icon，下边简单演示下如何生成一个Material design icon。

![](http://upload-images.jianshu.io/upload_images/4048192-2151700b54fcaacf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

操作很简单，选择一个已有的图标或者自己拖动一个svg格式的图片进来，就会进入到这个页面，按照左边的配置栏进行个性化的配置即可。 

- Background color 设置背景颜色
- Background shape 设置形状，圆形或者方形
- Center icon 图片是否居中
- Shadow length 阴影长度
- Shadow intensity 阴影的深度
- Shadow fading 阴影衰落
- Icon size 图标大小 