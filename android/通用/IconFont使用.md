# 使用IconFont

>作者: 菜刀文  
>Demo地址: [IconFontDemo](https://github.com/helen-x/IconFontDemo)        
>阿里开发iconfont库:[http://www.iconfont.cn/plus](http://www.iconfont.cn/plus)  
 
## IconFont介绍 
__IconFont__实际是把图片做在字体文件(.ttf)中.   
ttf文件中每个图片对应的一个unicode码.  
TextView 设置文字的时候,使用对应的unicode码就能显示出图片.      

打开iconfont.ttf, 可以看到里面全是图片   


![Snip20161215_15.png](http://upload-images.jianshu.io/upload_images/4048192-212f17281f0835fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 

假如在iconfont.ttf里面.  unicode码"&\#xf1cb;"对应的是"visa"这个图片,   
那么__android:text="文字和图片&\#xf1cb;混排"__ 显示效果如下  
 
 
![Snip20161215_17.png](http://upload-images.jianshu.io/upload_images/4048192-4294ac1202cd280a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
</br>
是不是很帅~    
我们目前项目中都在使用IconFont.  他有很多优点:

## 优点 
1. apk文件变小了.   
    (以demo中的iconfont.ttf为例)   
    字体文件才139KB,里面有670+个字符.    
    要是放普通图片,那估计好几M,甚至十几M
2. IconFont可以随意设置大小,变大后不会发虚.
3. 可以随意设置颜色
4. 方便批量切换图标,只要更新iconfont.ttf文件就行了.
5. 很容易实现图文混排. 因为都是Icon也被看做为文字.  

  
## 缺点   
1. 图片只能显示纯色的    



## 使用方法   
下面以IconFontDemo为例,介绍一下iconfont的使用方法.     
demo效果如下    
 
![iconfont_2.png](http://upload-images.jianshu.io/upload_images/4048192-8e3e7fc4a01f21f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 
 
 
### 1.首先把字体文件拷贝到项目中   
 
![Snip20161215_13.png](http://upload-images.jianshu.io/upload_images/4048192-e6aaef3dd32b9d08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 2.自定义TextView,让他可以加载iconfont.ttf   

```java   
public class IconFontTextView extends TextView {

    ***
    public IconFontTextView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init(context);
    }

    private void init(Context context) {
        //加载字体文件
        Typeface typeface = IconFontTypeFace.getTypeface(context);
        this.setTypeface(typeface);
        //去掉padding,这样iconfont和普通字体容易对齐
        setIncludeFontPadding(false);
    }


public static class IconFontTypeFace {

        //用static,整个app共用整个typeface就够了
        private static Typeface ttfTypeface = null;

        public static synchronized Typeface getTypeface(Context context) {
            if (ttfTypeface == null) {
                try {
                    ttfTypeface = Typeface.createFromAsset(context.getAssets(), "fonts/iconfont.ttf");
                } catch (Exception e) {

                }
            }
            return ttfTypeface;
        }

        public static synchronized void clearTypeface() {
            ttfTypeface = null;
        }
    }
}


```     



### 3. 像正常使用TextView一样使用iconfont.   
要显示iconfont中图片, 需要把text设置为图片对应的unicode码.       
比如"&\#xf1f0;"对应的是一个图片,我们可以这样使用:    

```java 
 <com.helen.iconfont.widget.IconFontTextView
        android:id="@+id/itv_combine_txt_1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"    
        android:text="文字和图片混排"
        android:textColor="@color/csl_black_blue"
        android:textSize="25dp" />   
<com.helen.iconfont.widget.IconFontTextView
        android:id="@+id/itv_only_icon"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text=""
        android:textColor="@android:color/holo_green_dark"
        android:textSize="25dp" />
``` 
 

代码中也可以    


```java     
 //String.xml中设置
 <string name="iconfont_mastercard"></string>  
 
 //代码中使用
 tvTest.setText(R.string.iconfont_mastercard);
```      
  
### 链接   
IconFontDemo中字体[unicode对照表](http://fortawesome.github.io/Font-Awesome/cheatsheet/) 

</br>
## 拓展-阿里的iconfont库中使用  

__阿里iconfont库地址:__  [http://www.iconfont.cn/plus](http://www.iconfont.cn/plus)   


#### 1.选择图标文件,加入购物车   
先选中自己想要的图片, 挨个加入购物车   
 
![iconfont_download.png](http://upload-images.jianshu.io/upload_images/4048192-ae6d29e6c48c000f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.购物车中文件下载   
全部选好后,可以在购物车中下载文件.  


 
![Snip20161215_19.png](http://upload-images.jianshu.io/upload_images/4048192-b216dc6f921fea8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
</br>
#### 3.下载的文件中可以看到字体文件(iconfont.ttf) 和对应的编码表  
 
 
![Snip20161215_14.png](http://upload-images.jianshu.io/upload_images/4048192-3d1149c370ba82b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

</br>
#### 4. demo_unicode.html 中可以看到字符和unicode字符的对应表.   
这里 &\#xe641; 对应的就是X这个图片

![iconfont_unicode.png](http://upload-images.jianshu.io/upload_images/4048192-4099507cbda22465.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 
#### 5. 有了unicode字符表和iconfont.ttf文件,就可以妥妥的使用了 

