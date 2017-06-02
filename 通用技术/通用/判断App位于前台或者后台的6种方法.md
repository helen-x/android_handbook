# 判断App位于前台或者后台的6种方法  

>作者: wenmingvs    
> Github: https://github.com/wenmingvs/AndroidProcess 



## 用法

传入Context参数与想要判断是否位于前台的App的包名,会返回ture或者false表示App是否位于前台

``` java

//六种方法任选其一

//使用方法一
Boolean isForeground = BackgroundUtil.getRunningTask(context, packageName);
//使用方法二
Boolean isForeground = BackgroundUtil.getRunningAppProcesses(context, packageName);
//使用方法三
Boolean isForeground = BackgroundUtil.getApplicationValue(context);
//使用方法四
Boolean isForeground = BackgroundUtil.queryUsageStats(context, packageName);
//使用方法五
Boolean isForeground = BackgroundUtil.getFromAccessibilityService(context, packageName);
//使用方法六
Boolean isForeground = BackgroundUtil.getLinuxCoreInfo(context, packageName);
```

## 六种方法的区别

|方法|判断原理|需要权限|可以判断其他应用位于前台|特点
| ------ | ------ | ------ | ------ | ------ |
|方法一|RunningTask|否|Android4.0系列可以,5.0以上机器不行|5.0此方法被废弃
|方法二|RunningProcess|否|当App存在后台常驻的Service时失效|无
|方法三|ActivityLifecycleCallbacks|否|否|简单有效,代码最少
|方法四|UsageStatsManager|是|是|需要用户手动授权
|方法五|通过Android无障碍功能实现|否|是|需要用户手动授权
|方法六|读取/proc目录下的信息|否|是|当proc目录下文件夹过多时,过多的IO操作会引起耗时


## 方法一：通过RunningTask

 
**原理**  
>当一个App处于前台的时候，会处于RunningTask的这个栈的栈顶，所以我们可以取出RunningTask的栈顶的任务进程，看他与我们的想要判断的App的包名是否相同，来达到效果

**缺点**    
>getRunningTask方法在Android5.0以上已经被废弃，只会返回自己和系统的一些不敏感的task，不再返回其他应用的task，用此方法来判断自身App是否处于后台，仍然是有效的，但是无法判断其他应用是否位于前台，因为不再能获取信息

 
## 方法二：通过RunningProcess

 
**原理**  
>通过runningProcess获取到一个当前正在运行的进程的List，我们遍历这个List中的每一个进程，判断这个进程的一个importance 属性是否是前台进程，并且包名是否与我们判断的APP的包名一样，如果这两个条件都符合，那么这个App就处于前台

**缺点**：  
>在聊天类型的App中，常常需要常驻后台来不间断的获取服务器的消息，这就需要我们把Service设置成START_STICKY，kill 后会被重启（等待5秒左右）来保证Service常驻后台。如果Service设置了这个属性，这个App的进程就会被判断是前台，代码上的表现就是appProcess.importance的值永远是 ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND，这样就永远无法判断出到底哪个是前台了。

## 方法三：通过ActivityLifecycleCallbacks

 
**原理**  
AndroidSDK14在Application类里增加了ActivityLifecycleCallbacks，我们可以通过这个Callback拿到App所有Activity的生命周期回调。   
 
``` java
 public interface ActivityLifecycleCallbacks {
        void onActivityCreated(Activity activity, Bundle savedInstanceState);
        void onActivityStarted(Activity activity);
        void onActivityResumed(Activity activity);
        void onActivityPaused(Activity activity);
        void onActivityStopped(Activity activity);
        void onActivitySaveInstanceState(Activity activity, Bundle outState);
        void onActivityDestroyed(Activity activity);
    }
```
知道这些信息，我们就可以用更官方的办法来解决问题，当然还是利用方案二里的Activity生命周期的特性，我们只需要在Application的onCreate()里去注册上述接口，然后由Activity回调回来运行状态即可。

可能还有人在纠结，我用back键切到后台和用Home键切到后台，一样吗？以上方法适用吗？在Android应用开发中一般认为back键是可以捕获的，而Home键是不能捕获的（除非修改framework）,但是上述方法从Activity生命周期着手解决问题，虽然这两种方式的Activity生命周期并不相同，但是二者都会执行onStop（）；所以并不关心到底是触发了哪个键切入后台的。另外,Application是否被销毁,都不会影响判断的正确性

## 方法四:通过使用UsageStatsManager获取

 
**原理**  
>通过使用UsageStatsManager获取，此方法是Android5.0之后提供的新API，可以获取一个时间段内的应用统计信息，但是必须满足一下要求

**使用前提**  
1.此方法只在android5.0以上有效   
2.AndroidManifest中加入此权限      
   
```  java
<uses-permission xmlns:tools="http://schemas.android.com/tools"    
 android:name="android.permission.PACKAGE_USAGE_STATS"   
  tools:ignore="ProtectedPermissions" />
```
3.打开手机设置，点击安全-高级，在有权查看使用情况的应用中，为这个App打上勾  


![enter image description here](https://raw.githubusercontent.com/wenmingvs/AndroidProcess/master/sample/3.PNG)

## 方法五：通过Android自带的无障碍功能


非常感谢[@EffectiveMatrix](http://weibo.com/hatewx?refer_flag=1005050005_)大神带来的新的判断前后台的方法  

此方法属于他原创，具体的博文参照这里http://effmx.com/articles/tong-guo-android-fu-zhu-gong-neng-accessibility-service-jian-ce-ren-yi-qian-tai-jie-mian/  

此方法无法直观的通过下拉通知视图来进行前后台的观察，请到LogCat中进行观察即可，以下是LogCat中打印的信息

![enter image description here](http://upload-images.jianshu.io/upload_images/4048192-0f7c8c95e2884b4a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
 
**原理**  
>Android 辅助功能(AccessibilityService) 为我们提供了一系列的事件回调，帮助我们指示一些用户界面的状态变化。 我们可以派生辅助功能类，进而对不同的 AccessibilityEvent 进行处理。 同样的，这个服务就可以用来判断当前的前台应用

**优势**  
>1. AccessibilityService 有非常广泛的 ROM 覆盖，特别是非国产手机，从 Android API Level 8(Android 2.2) 到 Android Api Level 23(Android 6.0)  
2.  AccessibilityService 不再需要轮询的判断当前的应用是不是在前台，系统会在窗口状态发生变化的时候主动回调，耗时和资源消耗都极小  
3. 不需要权限请求  
4. 它是一个稳定的方法，与 “方法6”读取 /proc 目录不同，它并非利用 Android 一些设计上的漏洞，可以长期使用的可能很大  
5. 可以用来判断任意应用甚至 Activity, PopupWindow, Dialog 对象是否处于前台  

**劣势**  
>1. 需要要用户开启辅助功能  
2. 辅助功能会伴随应用被“强行停止”而剥夺  

## 方法六：读取Linux系统内核保存在/proc目录下的process进程信息

 
此方法并非我原创，原作者是国外的大神，[GitHub项目在这里](https://github.com/jaredrummler/AndroidProcesses)，也一并加入到工程中，供大家做全面的参考选择

**原理**  
>无意中看到乌云上有人提的一个漏洞，Linux系统内核会把process进程信息保存在/proc目录下，Shell命令去获取的他，再根据进程的属性判断是否为前台

**优点**  
>1. 不需要任何权限  
2. 可以判断任意一个应用是否在前台，而不局限在自身应用

**缺点**  
>1. 当/proc下文件夹过多时,此方法是耗时操作

**用法**  
获取一系列正在运行的App的进程  

``` java
List<AndroidAppProcess> processes = ProcessManager.getRunningAppProcesses();
```

获取任一正在运行的App进程的详细信息  

``` java
AndroidAppProcess process = processes.get(location);
String processName = process.name;

Stat stat = process.stat();
int pid = stat.getPid();
int parentProcessId = stat.ppid();
long startTime = stat.stime();
int policy = stat.policy();
char state = stat.state();

Statm statm = process.statm();
long totalSizeOfProcess = statm.getSize();
long residentSetSize = statm.getResidentSetSize();

PackageInfo packageInfo = process.getPackageInfo(context, 0);
String appName = packageInfo.applicationInfo.loadLabel(pm).toString();
```

判断是否在前台  

``` java
if (ProcessManager.isMyProcessInTheForeground()) {
  // do stuff
}
```

获取一系列正在运行的App进程的详细信息  

``` java
List<ActivityManager.RunningAppProcessInfo> processes = ProcessManager.getRunningAppProcessInfo(ctx);
``` 
