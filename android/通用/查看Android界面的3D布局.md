#  查看Android界面的3D布局

>作者: 菜刀文

说到查看界面的布局, 第一反应是用 hierarchyviewer   

但是hierarchyviewer有些许的小问题:  
>1. 查看界面不够直观.   
2. 需要连开发工具   
3. 操作不够帅够炫 : )      
 
JackWharton大神的Scalpel给你另外一个选择    

## Scalpel简介

先来看看效果图   
 
![](http://upload-images.jianshu.io/upload_images/4048192-25f996136f902f85.gif?imageMogr2/auto-orient/strip)


很帅有木有,通过Scalpel
>1. 可以很清楚的看到每个view的层级.    
2. 支持拖动旋转查看  

在需要给别人演示自定义View,找布局层级的时候, 开这么个功能 逼格满满的.  


## 使用Scalpel
Scalpel使用很简单,几分钟搞定~     

### 1. 依赖scalpel的库      

```java     
compile 'com.jakewharton.scalpel:scalpel:1.1.2'       
``` 

### 2. Activity的布局中引入ScalpelFrameLayout   
需要修改Activity的根布局为ScalpelFrameLayout

```xml      
<!-- 使用scalpel需要修改Activity的根布局为ScalpelFrameLayout -->
<com.jakewharton.scalpel.ScalpelFrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scalpel"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
  <!-- 你的布局-->
</com.jakewharton.scalpel.ScalpelFrameLayout>

```    

如果你只想在debug模式中开启Scalpel.  可以简单的加个判断,debug的时候动态添加 ScalpelFrameLayout. 

```java   
@Override  
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    
if(BuildConfig.DEBUG){
        View activityView = getLayoutInflater().inflate(R.layout.sample_activity,null);    
        ScalpelFrameLayout scalpelView = new ScalpelFrameLayout(this);
        scalpelView.addView(activityView);
        setContentView(scalpelView);
    }else{
        setContentView(R.layout.sample_activity);
    }
}
```

### 3.配置3d显示效果   
- setLayerInteractionEnabled(boolean)  配置是否开启3d效果 

- setDrawViews(boolean) 配置是否绘制真实的View, 如果为false则仅绘制布局线框图

- setDrawIds(boolean) 配置是否在界面上显示控件的Id.

- setChromeColor，setChromeShadowColor 配置线框图颜色. 



## 代码链接   
[Scalpel源码下载](https://github.com/helen-x/scalpel)   

