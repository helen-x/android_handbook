# Android进程管理三部曲[1]-进程的创建

## 基础   
>作者:  强波  (阿里云OS平台部-Cloud Engine)  
>博客: http://qiangbo.space/



对于操作系统来说，进程管理是其最重要的职责之一。

考虑到这部分的内容较多，因此会拆分成几篇文章来讲解。

本文是进程管理系统文章的第一篇，会讲解Android系统中的进程创建。

本文适合Android平台的应用程序开发者，也适合对于Android系统内部实现感兴趣的读者。
 

# 概述
Android系统以Linux内核为基础，所以对于进程的管理自然离不开Linux本身提供的机制。例如：

* 通过fork来创建进行
* 通过信号量来管理进程
* 通过proc文件系统来查询和调整进程状态
等

对于Android来说，进程管理的主要内容包括以下几个部分内容：

* 进程的创建
* 进程的优先级管理
* 进程的内存管理
* 进程的回收和死亡处理

本文会专门讲解进程的创建，其余部分将在后面的文章中讲解。

# 主要模块
为了便于下文的讲解，这里先介绍一下Android系统中牵涉到进程创建的几个主要模块。

同时为了便于读者更详细的了解这些模块，这里也同时提供了这些模块的代码路径。

这里提到的代码路径是指AOSP的源码数中的路径。

关于如何获取AOSP源码请参见这里：[Downloading the Source](https://source.android.com/source/downloading.html)。

**本文以Android N版本的代码为示例，所用到的[Source Code Tags](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds)是：android-7.0.0_r1。**

**相关模块**：

* **app_process**

代码路径：`frameworks/base/cmds/app_process`

说明：app_process是一个可执行程序，该程序的主要作用是启动`zygote`和`system_server`进程。

* **Zygote**

代码路径：`frameworks/base/core/java/com/android/internal/os/ZygoteInit.java`

说明：`zygote`进程是所有应用进程的父进程，这是系统中一个非常重要的进程，下文我们会详细讲解。

* **ActivityManager**

代码路径：`frameworks/base/services/core/java/com/android/server/am/`

说明：am是ActivityManager的缩写。

这个目录下的代码负责了Android全部四大组件（Activity，Service，ContentProvider，BroadcastReceiver）的管理，并且还掌控了所有应用程序进程的创建和进程的优先级管理。

因此，这个部分的内容将是本系列文章讲解的重点。

# 进程与线程
Android官方开发网站的这篇文章：[Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html) 非常好的介绍了Android系统中进程相关的一些基本概念和重要知识。

**在阅读下文之前，请务必将这篇文章浏览一遍。**

# 关于进程

在Android系统中，进程可以大致分为**系统进程**和**应用进程**两大类。

**系统进程**是系统内置的（例如：`init`，`zygote`，`system_server`进程），属于操作系统必不可少的一部分。系统进程的作用在于：

* 管理硬件设备
* 提供访问设备的基本能力
* 管理应用进程

**应用进程**是指应用程序运行的进程。这些应用程序可能是系统出厂自带的（例如Launcher，电话，短信等应用），也可能是用户自己安装的（例如：微信，支付宝等）。

系统进程的数量通常是固定的（出厂或者系统升级之后就确定了），并且系统进程通常是一直存活，常驻内存的。系统进程的异常退出将可能导致设备无法正常使用。

而应用程序和应用进程在每个人使用的设备上通常是各不一样的。如何管理好这些不确定的应用进程，就是操作系统本身要仔细考虑的内容。也是衡量一个操作系统好坏的标准之一。

在本文中，我们会介绍**`init`**，**`zygote`**和**`system_server`**三个系统进程。

除此之外，**本系列文章将会把主要精力集中在讲解Android系统如何管理应用进程上**。

# init进程
init进程是一切的开始，在Android系统中，所有进程的进程号都是不确定的，唯独init进程的进程号一定是1。

因为这个进程一定是系统起来的第一个进程。

并且，init进程掌控了整个系统的启动逻辑。

我们知道，Android可能运行在各种不同的平台，不同的设备上。因此，启动的逻辑是不尽相同的。
为了适应各种平台和设备的需求，init进程的初始化工作通过`init.rc`配置文件来管理。

你可以在AOSP源码的`system/core/rootdir/`路径找到这些配置文件。

配置文件的主入口文件是`init.rc`，这个文件会通过`import`引入其他几个文件。

在本文中，我们统称这些文件为`init.rc`。

init.rc通过[Android Init Language](https://android.googlesource.com/platform/system/core/+/master/init/readme.txt)来进行配置。
建议读者大致阅读一下其 [语法说明](https://android.googlesource.com/platform/system/core/+/master/init/readme.txt) 。

init.rc中配置了系统启动的时候该做哪些事情，以及启动哪些系统进程。

这其中有两个特别重要的进程就是：**zygote**和**system_server**进程。

* **zygote**的中文意思是“受精卵“。这是一个很有寓意的名称：所有的应用进程都是由`zygote` fork出来的子进程，因此zygote进程是所有应用进程的父进程。
* **system_server** 这个进程正如其名称一样，这是一个系统服务器。Framework层的几乎所有服务都位于这个进程中。这其中就包括管理四大组件的`ActivityManagerService`。

# Zygote进程

init.rc文件会根据平台不一样，选择下面几个文件中的一个来启动zygote进程：

* init.zygote32.rc
* init.zygote32_64.rc
* init.zygote64.rc
* init.zygote64_32.rc

这几个文件的内容是大致一致的，仅仅是为了不同平台服务的。这里我们以init.zygote32.rc的文件为例，来看看其中的内容：

```
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
class main
socket zygote stream 660 root system
onrestart write /sys/android_power/request_state wake
onrestart write /sys/power/state on
onrestart restart audioserver
onrestart restart cameraserver
onrestart restart media
onrestart restart netd
writepid /dev/cpuset/foreground/tasks /dev/stune/foreground/tasks
```

在这段配置文件中（如果你不明白这段配置的含义，请阅读一下文档：[Android Init Language](https://android.googlesource.com/platform/system/core/+/master/init/readme.txt)），启动了一个名称叫做`zygote`的服务进程。这个进程是通过`/system/bin/app_process` 这个可执行程序创建的。

并且在启动这个可执行程序的时候，传递了`-Xzygote /system/bin --zygote --start-system-server
class main` 这些参数。

要知道这里到底做了什么，我们需要看一下app_process的源码。

app_process的源码在这个路径：`frameworks/base/cmds/app_process/app_main.cpp`。

这个文件的main函数的有如下代码：

```java
int main(int argc, char* const argv[])
{
...
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            zygote = true;
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;
        ...
    }
    ...
   if (!className.isEmpty()) {
        ...
    } else {
       ...
    
       if (startSystemServer) {
           args.add(String8("start-system-server"));
       }
    }
...
    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
        return 10;
    }
}
```

这里会判断，

* 如果执行这个命令时带了`--zygote`参数，就会通过`runtime.start`启动`com.android.internal.os.ZygoteInit`。
* 如果参数中带有`--start-system-server`参数，就会将`start-system-server`添加到args中。

这段代码是C++实现的。在执行这段代码的时候还没有任何Java的环境。而`runtime.start`就是启动Java虚拟机，并在虚拟机中启动指定的类。于是接下来的逻辑就在ZygoteInit.java中了。

这个文件的`main`函数主要代码如下：：

```java
public static void main(String argv[]) {
   ...

   try {
       ...

       boolean startSystemServer = false;
       String socketName = "zygote";
       String abiList = null;
       for (int i = 1; i < argv.length; i++) {
           if ("start-system-server".equals(argv[i])) {
               startSystemServer = true;
           } else if (argv[i].startsWith(ABI_LIST_ARG)) {
               ...
           }
       }
       ...
       registerZygoteSocket(socketName);
       ...
       preload();
       ...
       Zygote.nativeUnmountStorageOnInit();

       ZygoteHooks.stopZygoteNoThreadCreation();

       if (startSystemServer) {
           startSystemServer(abiList, socketName);
       }

       Log.i(TAG, "Accepting command socket connections");
       runSelectLoop(abiList);

       closeServerSocket();
   } catch (MethodAndArgsCaller caller) {
       caller.run();
   } catch (RuntimeException ex) {
       Log.e(TAG, "Zygote died with exception", ex);
       closeServerSocket();
       throw ex;
   }
}
```

在这段代码中，我们主要关注如下几行：

1. 通过 `registerZygoteSocket(socketName);` 注册**Zygote Socket**
2. 通过 `preload();` 预先加载所有应用都需要的公共资源
3. 通过 `startSystemServer(abiList, socketName);` 启动`system_server`
4. 通过 `runSelectLoop(abiList);` 在Looper上等待连接

这里需要说明的是：zygote进程启动之后，会启动一个socket套接字，并通过Looper一直在这个套接字上等待连接。

**所有应用进程都是通过发送数据到这个套接字上，然后由zygote进程创建的。**

这里还有一点说明的是：
在Zygote进程中，会通过`preload`函数加载需要应用程序都需要的公共资源。

预先加载这些公共资源有如下两个好处：

* **加快应用的启动速度** 因为这些资源已经在zygote进程启动的时候加载好了
* **通过共享的方式节省内存** 这是Linux本身提供的机制：父进程已经加载的内容可以在子进程中进行共享，而不用多份数据拷贝（除非子进程对这些数据进行了修改。）

preload的资源主要是Framework相关的一些基础类和Resource资源，而这些资源正是所有应用都需要的：

开发者通过Android SDK开发应用所调用的API实现都在Framework中。

```java
static void preload() {
   Log.d(TAG, "begin preload");
   Trace.traceBegin(Trace.TRACE_TAG_DALVIK, "BeginIcuCachePinning");
   beginIcuCachePinning();
   Trace.traceEnd(Trace.TRACE_TAG_DALVIK);
   Trace.traceBegin(Trace.TRACE_TAG_DALVIK, "PreloadClasses");
   preloadClasses();
   Trace.traceEnd(Trace.TRACE_TAG_DALVIK);
   Trace.traceBegin(Trace.TRACE_TAG_DALVIK, "PreloadResources");
   preloadResources();
   Trace.traceEnd(Trace.TRACE_TAG_DALVIK);
   Trace.traceBegin(Trace.TRACE_TAG_DALVIK, "PreloadOpenGL");
   preloadOpenGL();
   Trace.traceEnd(Trace.TRACE_TAG_DALVIK);
   preloadSharedLibraries();
   preloadTextResources();

   WebViewFactory.prepareWebViewInZygote();
   endIcuCachePinning();
   warmUpJcaProviders();
   Log.d(TAG, "end preload");
}
```

# system_server进程

上文已经提到，zygote进程起来之后会根据需要启动`system_server`进程。

`system_server`进程中包含了大量的系统服务。例如：

* 负责网络管理的NetworkManagementService
* 负责窗口管理的WindowManagerService
* 负责震动管理的VibratorService
* 负责输入管理的InputManagerService

等等。关于system_server，我们今后会其他的文章中专门讲解，这里不做过多说明。

在本文中，我们只关注`system_server`中的`ActivityManagerService`这个系统服务。

# ActivityManagerService
上文中提到：zygote进程在启动之后会启动一个socket，然后一直在这个socket等待连接。

而会连接它的就是`ActivityManagerService`。

因为`ActivityManagerService`掌控了所有应用进程的创建。

**所有应用程序的进程都是由`ActivityManagerService`通过socket发送请求给`Zygote`进程，然后由zygote fork创建的。**

`ActivityManagerService`通过`Process.start`方法来请求`zygote`创建进程：

```java
public static final ProcessStartResult start(final String processClass,
                             final String niceName,
                             int uid, int gid, int[] gids,
                             int debugFlags, int mountExternal,
                             int targetSdkVersion,
                             String seInfo,
                             String abi,
                             String instructionSet,
                             String appDataDir,
                             String[] zygoteArgs) {
   try {
       return startViaZygote(processClass, niceName, uid, gid, gids,
               debugFlags, mountExternal, targetSdkVersion, seInfo,
               abi, instructionSet, appDataDir, zygoteArgs);
   } catch (ZygoteStartFailedEx ex) {
       Log.e(LOG_TAG,
               "Starting VM process through Zygote failed");
       throw new RuntimeException(
               "Starting VM process through Zygote failed", ex);
   }
}
```
这个函数会将启动进程所需要的参数组装好，并通过socket发送给zygote进程。然后zygote进程根据发送过来的参数将进程fork出来。

在`ActivityManagerService`中，调用**Process.start**的地方是下面这个方法：

```java
private final void startProcessLocked(ProcessRecord app, String hostingType,
       String hostingNameStr, String abiOverride, String entryPoint, String[] entryPointArgs) {
       
...
  Process.ProcessStartResult startResult = Process.start(entryPoint,
          app.processName, uid, uid, gids, debugFlags, mountExternal,
          app.info.targetSdkVersion, app.info.seinfo, requiredAbi, instructionSet,
          app.info.dataDir, entryPointArgs);
...
}
```

下文中我们会看到，所有四大组件进程的创建，都是调用这里的`startProcessLocked`这个方法而创建的。

**对于每一个应用进程，在`ActivityManagerService`中，都有一个`ProcessRecord`与之对应。这个对象记录了应用进程的所有详细状态。**

PS：对于`ProcessRecord`的内部结构，在下一篇文章中，我们会讲解。

为了查找方便，对于每个`ProcessRecord`会存在下面两个集合中。

* **按名称和uid组织的集合**：

```java
/**
* All of the applications we currently have running organized by name.
* The keys are strings of the application package name (as
* returned by the package manager), and the keys are ApplicationRecord
* objects.
*/
final ProcessMap<ProcessRecord> mProcessNames = new ProcessMap<ProcessRecord>();
```

* **按pid组织的集合：**

```java
/**
* All of the processes we currently have running organized by pid.
* The keys are the pid running the application.
*
* <p>NOTE: This object is protected by its own lock, NOT the global
* activity manager lock!
*/
final SparseArray<ProcessRecord> mPidsSelfLocked = new SparseArray<ProcessRecord>();
```

下面这幅图小节了上文的这些内容：

![](http://upload-images.jianshu.io/upload_images/4048192-522905df79b4c116.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 关于应用组件
[Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html) 提到：

“**当某个应用组件启动且该应用没有运行其他任何组件时，Android 系统会使用单个执行线程为应用启动新的 Linux 进程。**”

因此，四大组件中的任何一个先起来都会导致应用进程的创建。下文我们就详细看一下，它们启动时，各自是如何导致应用进程的创建的。

PS：四大组件的管理本身又是一个比较大的话题，限于篇幅关系，这里不会非常深入的讲解，这里主要是讲解四大组件与进程创建的关系。

在应用程序中，开发者通过：

* `startActivity(Intent intent)` 来启动Activity
* `startService(Intent service)` 来启动Service
* `sendBroadcast(Intent intent)` 来发送广播
* `ContentResolver` 中的接口来使用ContentProvider

这其中，`startActivity`，`startService`和`sendBroadcast`还有一些重载方法。

其实这里提到的所有这些方法，最终都是通过Binder调用到ActivityManagerService中，由其进行处理的。

这里特别说明一下：应用进程和`ActivityManagerService`所在进程（即`system_server`进程）是相互独立的，两个进程之间的方法通常是不能直接互相调用的。

而Android系统中，专门提供了Binder框架来提供进程间通讯和方法调用的能力。

调用关系如下图所示：

![](http://upload-images.jianshu.io/upload_images/4048192-862293248ed47fc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Activity与进程创建

在ActivityManagerService中，对每一个运行中的Activity都有一个`ActivityRecord`对象与之对应，这个对象记录Activity的详细状态。

ActivityManagerService中的`startActivity`方法接受`Context.startActivity`的请求，该方法代码如下：

```java
@Override
public final int startActivity(IApplicationThread caller, String callingPackage,
       Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
       int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
   return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
           resultWho, requestCode, startFlags, profilerInfo, bOptions,
           UserHandle.getCallingUserId());
}
```

Activity的启动是一个非常复杂的过程。这里我们简单介绍一下背景知识：

* ActivityManagerService中通过Stack和Task来管理Activity
* 每一个Activity都属于一个Task，一个Task可能包含多个Activity。一个Stack包含多个Task
* ActivityStackSupervisor类负责管理所有的Stack
* Activity的启动过程会牵涉到：
    * Intent的解析
    * Stack，Task的查询或创建
    * Activity进程的创建
    * Activity窗口的创建
    * Activity的生命周期调度

Activity的管理结构如下图所示：

![](http://upload-images.jianshu.io/upload_images/4048192-f840fc6f0ed70682.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在Activity启动的最后，会将前一个Activity pause，将新启动的Activity resume以便被用户看到。

在这个时候，如果发现新启动的Activity进程还没有启动，则会通过`startSpecificActivityLocked`将其启动。整个调用流程如下：

* `ActivityManagerService.activityPaused` =>
* `ActivityStack.activityPausedLocked` =>
* `ActivityStack.completePauseLocked` =>
* `ActivityStackSupervisor.ensureActivitiesVisibleLocked` =>
* `ActivityStack.makeVisibleAndRestartIfNeeded` =>
* `ActivityStackSupervisor.startSpecificActivityLocked` =>
* `ActivityManagerService.startProcessLocked`
    
`ActivityStackSupervisor.startSpecificActivityLocked` 关键代码如下：

```java
void startSpecificActivityLocked(ActivityRecord r,
       boolean andResume, boolean checkConfig) {
   // Is this activity's application already running?
   ProcessRecord app = mService.getProcessRecordLocked(r.processName,
           r.info.applicationInfo.uid, true);

   r.task.stack.setLaunchTime(r);

   if (app != null && app.thread != null) {
       ...
   }

   mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
           "activity", r.intent.getComponent(), false, false, true);
}
```
这里的`ProcessRecord app` 描述了Activity所在进程。

# Service与进程创建

Service的启动相对于Activity来说要简单一些。

在ActivityManagerService中，对每一个运行中的Service都有一个`ServiceRecord`对象与之对应，这个对象记录Service的详细状态。

ActivityManagerService中的`startService`方法处理`Context.startService`API的请求，相关代码：

```java
@Override
public ComponentName startService(IApplicationThread caller, Intent service,
       String resolvedType, String callingPackage, int userId)
       throws TransactionTooLargeException {
   ...
   synchronized(this) {
       final int callingPid = Binder.getCallingPid();
       final int callingUid = Binder.getCallingUid();
       final long origId = Binder.clearCallingIdentity();
       ComponentName res = mServices.startServiceLocked(caller, service,
               resolvedType, callingPid, callingUid, callingPackage, userId);
       Binder.restoreCallingIdentity(origId);
       return res;
   }
}
```

这段代码中的`mServices`对象是`ActiveServices`类型的，这个类专门负责管理活动的Service。

启动Service的调用流程如下：

* `ActivityManagerService.startService` =>
* `ActiveServices.startServiceLocked` =>
* `ActiveServices.startServiceInnerLocked` =>
* `ActiveServices.bringUpServiceLocked` =>
* `ActivityManagerService.startProcessLocked`

`ActiveServices.bringUpServiceLocked`会判断如果Service所在进程还没有启动，

则通过`ActivityManagerService.startProcessLocked`将其启动。相关代码如下：

```java
// Not running -- get it started, and enqueue this service record
// to be executed when the app comes up.
if (app == null && !permissionsReviewRequired) {
  if ((app=mAm.startProcessLocked(procName, r.appInfo, true, intentFlags,
          "service", r.name, false, isolated, false)) == null) {
      String msg = "Unable to launch app "
              + r.appInfo.packageName + "/"
              + r.appInfo.uid + " for service "
              + r.intent.getIntent() + ": process is bad";
      Slog.w(TAG, msg);
      bringDownServiceLocked(r);
      return msg;
  }
  if (isolated) {
      r.isolatedProc = app;
  }
}
```
这里的`mAm` 就是ActivityManagerService。

# Provider与进程创建

在ActivityManagerService中，对每一个运行中的ContentProvider都有一个`ContentProviderRecord`对象与之对应，这个对象记录ContentProvider的详细状态。

开发者通过ContentResolver中的`insert`, `delete`, `update`, `query`这些API来使用ContentProvider。在ContentResolver的实现中，无论使用这里的哪个接口，ContentResolver都会先通过`acquireProvider` 这个方法来获取到一个类型为`IContentProvider`的远程接口。这个远程接口对接了ContentProvider的实现提供方。

同一个ContentProvider可能同时被多个模块使用，而调用ContentResolver接口的进程只是ContentProvider的一个客户端而已，真正的ContentProvider提供方是运行自身的进程中的，两个进程的通讯需要通过Binder的远程接口形式来调用。如下图所示：

![](http://upload-images.jianshu.io/upload_images/4048192-bca53bc54c2b547e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`ContentResolver.acquireProvider` 最终会调用到`ActivityManagerService.getContentProvider`中，该方法代码如下：

```java
@Override
public final ContentProviderHolder getContentProvider(
       IApplicationThread caller, String name, int userId, boolean stable) {
   enforceNotIsolatedCaller("getContentProvider");
   if (caller == null) {
       String msg = "null IApplicationThread when getting content provider "
               + name;
       Slog.w(TAG, msg);
       throw new SecurityException(msg);
   }
   // The incoming user check is now handled in checkContentProviderPermissionLocked() to deal
   // with cross-user grant.
   return getContentProviderImpl(caller, name, null, stable, userId);
}
```

而在`getContentProviderImpl`这个方法中，会判断对应的ContentProvider进程有没有启动，

如果没有，则通过`startProcessLocked`方法将其启动。



# Receiver与进程创建

开发者通过`Context.sendBroadcast`接口来发送广播。`ActivityManagerService.broadcastIntent` 方法了对应广播发送的处理。

广播是一种一对多的消息形式，广播接受者的数量是不确定的。因此发送广播本身可能是一个很耗时的过程（因为要逐个通知）。

在ActivityManagerService内部，是通过队列的形式来管理广播的：

* `BroadcastQueue` 描述了一个广播队列
* `BroadcastRecord` 描述了一个广播事件

在`ActivityManagerService`中，如果收到了一个发送广播的请求，会先创建一个`BroadcastRecord`接着将其放入`BroadcastQueue`中。

然后通知队列自己去处理这个广播。然后`ActivityManagerService`自己就可以继续处理其他请求了。

广播队列本身是在另外一个线程处理广播的发送的，这样保证的`ActivityManagerService`主线程的负载不会太重。

在`BroadcastQueue.processNextBroadcast(boolean fromMsg)` 方法中真正实现了通知广播事件到接受者的逻辑。在这个方法，如果发现接受者（即BrodcastReceiver）还没有启动，便会通过`ActivityManagerService.startProcessLocked` 方法将其启动。相关如下所示：

```java
final void processNextBroadcast(boolean fromMsg) {
    ...
       // Hard case: need to instantiate the receiver, possibly
       // starting its application process to host it.

       ResolveInfo info =
           (ResolveInfo)nextReceiver;
       ComponentName component = new ComponentName(
               info.activityInfo.applicationInfo.packageName,
               info.activityInfo.name);
    ...
       // Not running -- get it started, to be executed when the app comes up.
       if (DEBUG_BROADCAST)  Slog.v(TAG_BROADCAST,
               "Need to start app ["
               + mQueueName + "] " + targetProcess + " for broadcast " + r);
       if ((r.curApp=mService.startProcessLocked(targetProcess,
               info.activityInfo.applicationInfo, true,
               r.intent.getFlags() | Intent.FLAG_FROM_BACKGROUND,
               "broadcast", r.curComponent,
               (r.intent.getFlags()&Intent.FLAG_RECEIVER_BOOT_UPGRADE) != 0, false, false))
                       == null) {
           // Ah, this recipient is unavailable.  Finish it if necessary,
           // and mark the broadcast record as ready for the next.
           Slog.w(TAG, "Unable to launch app "
                   + info.activityInfo.applicationInfo.packageName + "/"
                   + info.activityInfo.applicationInfo.uid + " for broadcast "
                   + r.intent + ": process is bad");
           logBroadcastReceiverDiscardLocked(r);
           finishReceiverLocked(r, r.resultCode, r.resultData,
                   r.resultExtras, r.resultAbort, false);
           scheduleBroadcastsLocked();
           r.state = BroadcastRecord.IDLE;
           return;
       }

       mPendingBroadcast = r;
       mPendingBroadcastRecvIndex = recIdx;
   }
}
```

至此，四大组件的启动就已经分析完了。

# 结束语

进程管理本身是一个非常大的话题，本文讲解了Android系统中进程创建的相关内容。

进程启动之后该如何管理就是下一篇文章要讲解的内容了。

敬请期待。

# 参考资料与推荐读物

[UNIX环境高级编程(第3版)](https://www.amazon.cn/UNIX环境高级编程-史蒂文斯/dp/B00KMR129E/)

[深入理解LINUX内核(第3版)](https://www.amazon.cn/深入理解LINUX内核-博韦/dp/B0011F5RYM/)

[Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html)

[Android Booting](http://elinux.org/Android_Booting)  

