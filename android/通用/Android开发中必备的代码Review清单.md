# Android开发中必备的代码Review清单  


>作者: InKenKa  
>简书: http://www.jianshu.com/u/97315b81287a

## 前言

本文收集了我自己工作以来提交代码前的所有检查点。事实证明，这样能有效提高自己的代码质量和功能的稳定性。所以推荐大家以后每次提交代码前，都可以看下这份Review清单哈。

此外，可能还有些检查点我并没有发现，欢迎大家踊跃在评论区补充哈～

## 清理操作

**1.页面退出时，是否完成必要的清理操作**  
1. 是否调用Handler的removeCallbacksAndMessages(null)来清空Handler里的消息  
2. 是否取消了还没完成的请求  
3. 在页面里注册的监听等，是否取消注册  

**2.数据库的游标是否已经关闭**  
这个点一般人都知道，出问题一般在于，没有考虑到多线程并发时的情况下，Cursor没有被释放。
所以数据库的操作需要加上同步代码块
详细可参考：http://www.2cto.com/kf/201408/329574.html

**3.打开过的文件流是否关闭**  

**4.使用完的Bitmap是否调用recycle()回收**  
否则会很耗内存，导致OOM或者卡顿

**5.WebView使用完是否调用了其destory()函数**  

## 是否能进一步优化自己的代码

**1.保存在内存中的图片，是否做过压缩处理再保存在内存里**    
否则可能由于图片质量太高，导致OOM

**2.Intent传递的数据太大，会导致页面跳转过慢。太大的数据可以通过持久化的形式传递，例如读写文件**  

**3.频繁地操作同一个文件或者执行同一个数据库操作，是否考虑把它用静态变量或者局部变量的形式缓存在内存里。用空间换时间**

**4.放在主页面的控件，是否可以考虑用ViewStub来优化启动速度**

## 要小心第三方包

**1.build.gradle远程依赖第三方包时，版本号建议写死，不要使用+号**  
避免由于新版本的第三方包引入了新的问题

**2.导入第三方工程时，记得把编码转换成自己工程当前是用的编码**  

**3.调用第三方的包或者JDK的方法时，要跳进他们的源码，看要不要加 try-catch**  
否则可能会导致自己应用的崩溃

**4.系统应用添加so时，是否在固件对应的Android.mk文件上加入新增的so，否则系统可能编译不过**

@lib/armeabi/libcommon.so     
@lib/armeabi/libabcdefg.so    

## 注意要成对出现的地方

**1.系统的、自己写的，注册和反注册的方法，是否成对出现**

**2.在生命周期的回调里，创建和销毁的代码是否对应起来**   
比如：onCreate()里面创建了Adapter，那么对应Adapter的退出处理操作(比如清空Image缓存)，一般就要写在onDestory()，而不能写在onDestoryView()。

类似的生命周期对应的代码有：  
onStart()、onStop();  
onCreate()、onDestory();  
onResume()、onPause();  
onCreateView()、onDestoryView()  

**3.若ListView的item复用了，对Item里View的操作是否成对出现**   
比如：

```java  

switch (type) {
    case ArticleListItem.TYPE_AD:
        ......
        mTitleView.setText(tencentAdBean.title);
        mGreenLabelView.setVisibility(VISIBLE);
        mRedLabelView.setText("");
        mRedLabelView.setVisibility(GONE);
        break;
    case ArticleListItem.TYPE_ARTICLE:
        ......
        mTitleView.setText(mzAdBean.adData.getTitle());
        mGreenLabelView.setVisibility(GONE);
        mRedLabelView.setText("ABC");
        mRedLabelView.setVisibility(VISIBLE);
        break;
}
```  

比如以上对mTitleView、mGreenLabelView和mRedLabelView的操作，都是成对出现。否则ListView可能会由于Item复用，导致Item显示错乱问题

## 防内存泄漏

**1.内部类，比如Handler、Listener、Callback是否是成static class**    
因为非静态内部类会持有外部类的引用。

**2.假如子线程持有了Activity，要用弱引用来持有**    
比如Request的Activity就应该用弱引用的形式，防止内存泄漏。

**3.要求传入Activity作为参数的函数，是否可以改用getApplicationContext()来作为参数**   

## Handler相关

**1.使用View.post()是否会有问题**   
因为在View处于detached状态期间，post()里面的Runnable是不会被执行的。只有在此View处于attached状态时才会被执行。

如果想改Runnable每次肯定会被执行，那么应该是用Handler.post来替代

**2.假如程序可能多次在同一个Handler里post同一个Runnable，每次post之前都应该先清空这个Handler中还没执行的该Runnable**
如：

```java  
if (mCloudRun != null) {
    mHandler.removeCallbacks(mCloudRun);
    mCloudRun = null;
}
mCloudRun = new Runnable() {
    @Override
    public void run() {
        CloudAccelerateSwitchRequest request = new CloudAccelerateSwitchRequest();
        request.setPriority(RequestTask.PRIORITY_LOW);
        RequestQueue.getInstance().addRequest(request);
    }
};
mHandler.post(mCloudRun);  

```   

## 其他

**1.多思考某些情况下，某变量是否会为空**  
而且在函数体内，处理参数前，必须加上判空语句

**2.回调函数是否处理好**   
回调函数很容易出问题。比如网络请求的回调，需要判断此时的Aciivity等是否还存在，再进行调用。因为异步操作回来，Activity可能就消失不存在了。
而且还要对一些可能被回收的变量进行判空。

**3.修改数据库后，是否把数据库的版本号+1**    

**4.启动第三方的Activity / Service时，是否加上try...catch**  
若Activity不在会爆出ActivityNotFoundException的异常

**5.除数是否做了非0判断**  

**6.不要在Activity的onCreate里调用PopupWindow的showAsLoaction方法，由于Activity还没被加载完，会报错**  

## 功能完成后，自测时的检查点

**1.某些情况下某个变量是否会造成空指针问题**

**2.自测功能，通用的几个检查点**   
 
1. 按下Home再返回是否正常；  
2. 熄灭屏幕再打开会怎样；  
3. 切换成其它应用再切换回来会怎样；  
4. onResume,onPause是否处理好;  

**3.从低版本升级上来，会不会有问题**

**4.打开 手机的开发者选项，看新功能是否会导致过度绘制、是否会掉帧**

**5.测试看是否影响启动速度**  
adb shell am start -W 包名/Activity

**6.对比看APK大小是否有增大**

**7.跑1小时Monkey测试其稳定性**   

**8.试试看看App是否能被反编译**   

**9.检查Log是否关闭**  

**10.看看是否走的Https,能否被抓包**
