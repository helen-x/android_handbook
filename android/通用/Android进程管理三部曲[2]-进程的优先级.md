# Android进程管理三部曲[2]-进程的优先级

>作者:  强波  (阿里云OS平台部-Cloud Engine)  
>博客: http://qiangbo.space/ 



本文是Android进程管理系列文章的第二篇，会讲解进程管理中的优先级管理。

进程管理的第一篇文章：《[进程的创建](http://www.jianshu.com/p/96f43244f754)》 

本文适合Android平台的应用程序开发者，也适合对于Android系统内部实现感兴趣的读者。

 

# 前言

进程的优先级反应了系统对于进程重要性的判定。

在Android系统中，进程的优先级影响着以下三个因素：

* 当内存紧张时，系统对于进程的回收策略
* 系统对于进程的CPU调度策略
* 虚拟机对于进程的内存分配和垃圾回收策略

本文会主要讲解系统对于进程优先级的判断依据和计算方法。

在[Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html) （如果你还没有阅读，请**立即**阅读一下这篇文章）一文中，我们已经了解到，系统对于进程的优先级有如下五个分类：

1. 前台进程
2. 可见进程
3. 服务进程
4. 后台进程
5. 空进程

这只是一个粗略的划分。其实，在系统的内部实现中，优先级远不止这么五种。

# 优先级的依据
在[进程的创建](http://www.jianshu.com/p/96f43244f754)一文中我们提到：

* 每一个Android的应用进程中，都可能包含四大组件（`Activity`，`Service`，`ContentProvider`，`BroadcastReceiver`）中的一个/种或者多个/种。
* 对于每一个应用进程，在`ActivityManagerService`中都有一个`ProcessRecord`对象与之对应。

**而进程中四大组件的状态就是决定进程优先级的根本依据。**

对于运行中的Service和ContentProvider来说，可能有若干个客户端进程正在对其使用。

在`ProcessRecord`中，详细记录了上面提到的这些信息，相关代码如下：

```java
// all activities running in the process
final ArrayList<ActivityRecord> activities = new ArrayList<>();
// all ServiceRecord running in this process
final ArraySet<ServiceRecord> services = new ArraySet<>();
// services that are currently executing code (need to remain foreground).
final ArraySet<ServiceRecord> executingServices = new ArraySet<>();
// All ConnectionRecord this process holds
final ArraySet<ConnectionRecord> connections = new ArraySet<>();
// all IIntentReceivers that are registered from this process.
final ArraySet<ReceiverList> receivers = new ArraySet<>();
// class (String) -> ContentProviderRecord
final ArrayMap<String, ContentProviderRecord> pubProviders = new ArrayMap<>();
// All ContentProviderRecord process is using
final ArrayList<ContentProviderConnection> conProviders = new ArrayList<>();
```

这里的：

* `activities` 记录了进程中运行的Activity
* `services`，`executingServices` 记录了进程中运行的Service
* `receivers` 记录了进程中运行的BroadcastReceiver
* `pubProviders` 记录了进程中运行的ContentProvider

而：

* `connections` 记录了对于Service连接
* `conProviders` 记录了对于ContentProvider的连接

这里的**连接**是指一种使用关系，对于Service和ContentProvider是类似的。

它们都有可能同时被多个客户端使用，每个使用的客户端都需要记录一个连接，和如下图所示：
![](http://upload-images.jianshu.io/upload_images/4048192-d0b3659bc0ec501e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

连接的意义在于：**连接的客户端的优先级会影响被使用的Service和ContentProvider所在进程的优先级。**

例如：当一个后台的Service正在被一个前台的Activity使用，那么这个后台的Service就需要设置一个较高的优先级以便不会被回收。（否则后台Service进程一旦被回收，便会对前台的Activity造成影响。）

而**这些组件的状态就是进程优先级的决定性因素。** **组件的状态**是指：

* Activity是否在前台，用户是否可见
* Service正在被哪些客户端使用
* ContentProvider正在被哪些客户端使用
* BroadcastReceiver是否正在接受广播

# 优先级的基础

## oom_score_adj

对于每一个运行中的进程，Linux内核都通过[proc文件系统](http://man7.org/linux/man-pages/man5/proc.5.html)暴露这样一个文件来允许其他程序修改指定进程的优先级： 

**/proc/[pid]/oom_score_adj**。（修改这个文件的内容需要`root`权限）

这个文件允许的值的范围是：**-1000 ~ +1000之间。值越小，表示进程越重要**。

当内存非常紧张时，系统便会遍历所有进程，以确定那个进程需要被杀死以回收内存，此时便会读取`oom_score_adj` 这个文件的值。关于这个值的使用，在后面讲解进程回收的的时候，我们会详细讲解。

PS：在Linux 2.6.36之前的版本中，Linux 提供调整优先级的文件是`/proc/[pid]/oom_adj`。这个文件允许的值的范围是`-17 ~ +15`之间。数值越小表示进程越重要。
这个文件在新版的Linux中已经废弃。

但你仍然可以使用这个文件，当你修改这个文件的时候，内核会直接进行换算，将结果反映到`oom_score_adj`这个文件上。

Android早期版本的实现中也是依赖`oom_adj`这个文件。但是在新版本中，已经切换到使用`oom_score_adj`这个文件。

ProcessRecord中下面这些属性反应了`oom_score_adj`的值：

```java
int maxAdj;                 // Maximum OOM adjustment for this process
int curRawAdj;              // Current OOM unlimited adjustment for this process
int setRawAdj;              // Last set OOM unlimited adjustment for this process
int curAdj;                 // Current OOM adjustment for this process
int setAdj;                 // Last set OOM adjustment for this process
```
`maxAdj` 指定了该进程允许的`oom_score_adj`最大值。这个属性主要是给系统应用和常驻内存的进程使用，这些进程的优先级的计算方法与应用进程的计算方法不一样，通过设定`maxAdj`保证这些进程一直拥有较高的优先级（在后面”优先级的算法“中，我们会看到对于这个属性的使用）。

除此之外，还有四个属性。

这其中，`curXXX`这一组记录了这一次优先级计算的结果。在计算完成之后，会将`curXXX`复制给对应的`setXXX`这一组上进行备份。
（下文的其他属性也会看到curXXX和setXXX的形式，和这里的原理是一样的。）

另外，xxxRawAdj记录了没有经过限制的adj值，“没有经过限制”是指这其中的值可能是超过了`oom_score_adj`文件所允许的范围（-1000 ~ 1000）。

为了便于管理，ProcessList.java中预定义了`oom_score_adj`的可能取值。

其实这里的预定义值也是对应用进程的一种分类，它们是：

```java
static final int UNKNOWN_ADJ = 1001; // 未知进程
static final int PREVIOUS_APP_ADJ = 700; // 前一个应用
static final int HOME_APP_ADJ = 600; // 桌面进程
static final int SERVICE_ADJ = 500; // 包含了Service的进程
static final int HEAVY_WEIGHT_APP_ADJ = 400; // 重量级进程
static final int BACKUP_APP_ADJ = 300; // 备份应用进程
static final int PERCEPTIBLE_APP_ADJ = 200; // 可感知的进程
static final int VISIBLE_APP_ADJ = 100; // 可见进程
static final int VISIBLE_APP_LAYER_MAX = PERCEPTIBLE_APP_ADJ - VISIBLE_APP_ADJ - 1;
static final int FOREGROUND_APP_ADJ = 0; // 前台进程
static final int PERSISTENT_SERVICE_ADJ = -700; // 常驻服务进程
static final int PERSISTENT_PROC_ADJ = -800; // 常驻应用进程
static final int SYSTEM_ADJ = -900; // 系统进程
static final int NATIVE_ADJ = -1000; // native系统进程
```
这里我们看到，`FOREGROUND_APP_ADJ = 0`，这个是前台应用进程的优先级。这是用户正在交互的应用，它们是很重要的，系统不应当把它们回收了。

`FOREGROUND_APP_ADJ = 0`是普通应用程序能够获取到的最高优先级。

而`VISIBLE_APP_ADJ`，`PERCEPTIBLE_APP_ADJ`，`PREVIOUS_APP_ADJ`这几个级别的优先级就逐步降低了。

`VISIBLE_APP_ADJ`是具有可见Activity进程的优先级：同一时刻，不一定只有一个Activity是可见的，如果前台Activity设置了透明属性，那么背后的Activity也是可见的。

`PERCEPTIBLE_APP_ADJ`是指用户可感知的进程，可感知的进程包括：

* 进程中包含了处于pause状态或者正在pause的Activity
* 进程中包含了正在stop的Activity
* 进程中包含了前台的Service

另外，`PREVIOUS_APP_ADJ`描述的是前一个应用的优先级。所谓“前一个应用”是指：在启动新的Activity时，如果新启动的Activity是属于一个新的进程的，那么当前即将被stop的Activity所在的进程便会成为“前一个应用”进程。

而`HEAVY_WEIGHT_APP_ADJ` 描述的重量级进程是指那些通过Manifest指明不能保存状态的应用进程。

除此之外，Android系统中，有一些系统应用会常驻内存，这些应用通常是系统实现的一部分，如果它们不存在，系统将处于比较奇怪的状态，例如SystemUI（状态栏，Keyguard都处于这个应用中）。

所以它们的优先级比所有应用进程的优先级更高：`PERSISTENT_SERVICE_ADJ = -700`，`PERSISTENT_PROC_ADJ = -800`。

另外，还有一些系统服务的实现，如果这些系统服务不存在，系统将无法工作，所以这些应用的优先级最高，几乎是任何任何时候都需要存在的：`SYSTEM_ADJ = -900`，`NATIVE_ADJ = -1000`。

## Schedule Group

运行中的进程会能够获取的CPU时间片可能是不一样的，Linux本身提供了相关的API来调整，例如：[sched_setscheduler](http://man7.org/linux/man-pages/man2/sched_setscheduler.2.html)。

在ProcessRecord中，下面的属性记录了进程的Schedule Group：

```java
int curSchedGroup;          // Currently desired scheduling class
int setSchedGroup;          // Last set to background scheduling class
```

它们可能的取值定义在Process.java中：

```java
/**
* Default thread group -
* has meaning with setProcessGroup() only, cannot be used with setThreadGroup().
* When used with setProcessGroup(), the group of each thread in the process
* is conditionally changed based on that thread's current priority, as follows:
* threads with priority numerically less than THREAD_PRIORITY_BACKGROUND
* are moved to foreground thread group.  All other threads are left unchanged.
* @hide
*/
public static final int THREAD_GROUP_DEFAULT = -1;

/**
* Background thread group - All threads in
* this group are scheduled with a reduced share of the CPU.
* Value is same as constant SP_BACKGROUND of enum SchedPolicy.
* FIXME rename to THREAD_GROUP_BACKGROUND.
* @hide
*/
public static final int THREAD_GROUP_BG_NONINTERACTIVE = 0;

/**
* Foreground thread group - All threads in
* this group are scheduled with a normal share of the CPU.
* Value is same as constant SP_FOREGROUND of enum SchedPolicy.
* Not used at this level.
* @hide
**/
private static final int THREAD_GROUP_FOREGROUND = 1;
```

在Android中，`Process.setProcessGroup(int pid, int group)`用来设置进程的调度组。调度组会影响进程的CPU占用时间。

## Process State

ProcessRecord中的下面这几个属性记录了进程的状态：

```java
int curProcState; // Currently computed process state
int repProcState; // Last reported process state
int setProcState; // Last set process state in process tracker
int pssProcState; // Currently requesting pss for
```
进程的状态会影响虚拟机对于进程的内存分配和垃圾回收策略。

这些属性可能的取值定义在`ActivityManager`中，这些定义的注释很好的说明了这些值在什么时候会被用到：

```java
/** @hide Process does not exist. */
public static final int PROCESS_STATE_NONEXISTENT = -1;
/** @hide Process is a persistent system process. */
public static final int PROCESS_STATE_PERSISTENT = 0;
/** @hide Process is a persistent system process and is doing UI. */
public static final int PROCESS_STATE_PERSISTENT_UI = 1;
/** @hide Process is hosting the current top activities.  Note that this covers
* all activities that are visible to the user. */
public static final int PROCESS_STATE_TOP = 2;
/** @hide Process is hosting a foreground service due to a system binding. */
public static final int PROCESS_STATE_BOUND_FOREGROUND_SERVICE = 3;
/** @hide Process is hosting a foreground service. */
public static final int PROCESS_STATE_FOREGROUND_SERVICE = 4;
/** @hide Same as {@link #PROCESS_STATE_TOP} but while device is sleeping. */
public static final int PROCESS_STATE_TOP_SLEEPING = 5;
/** @hide Process is important to the user, and something they are aware of. */
public static final int PROCESS_STATE_IMPORTANT_FOREGROUND = 6;
/** @hide Process is important to the user, but not something they are aware of. */
public static final int PROCESS_STATE_IMPORTANT_BACKGROUND = 7;
/** @hide Process is in the background running a backup/restore operation. */
public static final int PROCESS_STATE_BACKUP = 8;
/** @hide Process is in the background, but it can't restore its state so we want
* to try to avoid killing it. */
public static final int PROCESS_STATE_HEAVY_WEIGHT = 9;
/** @hide Process is in the background running a service.  Unlike oom_adj, this level
* is used for both the normal running in background state and the executing
* operations state. */
public static final int PROCESS_STATE_SERVICE = 10;
/** @hide Process is in the background running a receiver.   Note that from the
* perspective of oom_adj receivers run at a higher foreground level, but for our
* prioritization here that is not necessary and putting them below services means
* many fewer changes in some process states as they receive broadcasts. */
public static final int PROCESS_STATE_RECEIVER = 11;
/** @hide Process is in the background but hosts the home activity. */
public static final int PROCESS_STATE_HOME = 12;
/** @hide Process is in the background but hosts the last shown activity. */
public static final int PROCESS_STATE_LAST_ACTIVITY = 13;
/** @hide Process is being cached for later use and contains activities. */
public static final int PROCESS_STATE_CACHED_ACTIVITY = 14;
/** @hide Process is being cached for later use and is a client of another cached
* process that contains activities. */
public static final int PROCESS_STATE_CACHED_ACTIVITY_CLIENT = 15;
/** @hide Process is being cached for later use and is empty. */
public static final int PROCESS_STATE_CACHED_EMPTY = 16;
```

# 优先级的更新

前文已经提到，系统会对处于不同状态的进程设置不同的优先级。但实际上，进程的状态是一直在变化中的。例如：用户可以随时会启动一个新的Activity，或者将一个前台的Activity切换到后台。在这个时候，发生状态变化的Activity的所在进程的优先级就需要进行更新。

![](http://upload-images.jianshu.io/upload_images/4048192-fe854b7d1f80db96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

并且，Activity可能会使用其他的Service或者Provider。当Activity的进程优先级发生变化的时候，其所使用的Service或者Provider的优先级也应当发生变化。

ActivityManagerService中有如下两个方法用来更新进程的优先级：

* `final boolean updateOomAdjLocked(ProcessRecord app)`
* `final void updateOomAdjLocked() `

其中，第一个方法是针对指定的一个进程更新优先级。另一个是对所有运行中的进程更新优先级。

在下面的这些情况下，需要对指定的应用进程更新优先级：

* 当有一个新的进程开始使用本进程中的ContentProvider
* 当本进程中的一个Service被其他进程bind或者unbind
* 当本进程中的Service的执行完成或者退出了
* 当本进程中一个BroadcastReceiver正在接受广播
* 当本进程中的BackUpAgent启动或者退出了

`final boolean updateOomAdjLocked(ProcessRecord app)` 被调用的关系如下图所示：

![](http://upload-images.jianshu.io/upload_images/4048192-bfc3c16c2e414282.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在如下一些情况下，系统会对所有应用进程的优先级进行更新：

* 当有一个新的进程启动时
* 当有一个进程退出时
* 当系统在清理后台进程时
* 当有一个进程被标记为前台进程时
* 当有一个进程进入或者退出cached状态时
* 当系统锁屏或者解锁时
* 当有一个Activity启动或者退出时
* 当系统正在处理一个广播事件时
* 当前台Activity发生改变时
* 当有一个Service启动时

`final void updateOomAdjLocked()` 被调用的关系图如下所示：
![](http://upload-images.jianshu.io/upload_images/4048192-64bc4df5d0c9af58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 优先级的算法

ActivityManagerService中的`computeOomAdjLocked`方法负责计算进程的优先级。

上文中已经提到，优先级计算的基础主要就是依赖以下信息:

* 一个进程中可能包含四个组件中的一个/种或多个多个/种
* 每个Service或者Provider的客户端连接

`computeOomAdjLocked`方法总计约700行，这个方法的执行流程主要包含如下10个步骤：

![](http://upload-images.jianshu.io/upload_images/4048192-491f58b875a23868.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面我们来详细看其中的每一个步骤：

* 1.确认该进程是否是空进程

   空进程中没有任何组件，因此主线程也为null（`ProcessRecord.thread`描述了应用进程的主线程）。
   
   如果是空进程，则不需要再做后面的计算了。直接设置为`ProcessList.CACHED_APP_MAX_ADJ`级别即可。

```java
if (app.thread == null) {
      app.adjSeq = mAdjSeq;
      app.curSchedGroup = ProcessList.SCHED_GROUP_BACKGROUND;
      app.curProcState = ActivityManager.PROCESS_STATE_CACHED_EMPTY;
      return (app.curAdj=app.curRawAdj=ProcessList.CACHED_APP_MAX_ADJ);
}
```

* 2.确认是否设置了maxAdj

   上文已经提到过，系统进程或者Persistent进程会通过设置maxAdj来保持其较高的优先级，对于这类进程不用按照普通进程的算法进行计算，直接按照`maxAdj`的值设置即可。

```java
if (app.maxAdj <= ProcessList.FOREGROUND_APP_ADJ) {
      app.adjType = "fixed";
      app.adjSeq = mAdjSeq;
      app.curRawAdj = app.maxAdj;
      app.foregroundActivities = false;
      app.curSchedGroup = ProcessList.SCHED_GROUP_DEFAULT;
      app.curProcState = ActivityManager.PROCESS_STATE_PERSISTENT;
      app.systemNoUi = true;
      if (app == TOP_APP) {
          app.systemNoUi = false;
          app.curSchedGroup = ProcessList.SCHED_GROUP_TOP_APP;
          app.adjType = "pers-top-activity";
      } else if (activitiesSize > 0) {
          for (int j = 0; j < activitiesSize; j++) {
              final ActivityRecord r = app.activities.get(j);
              if (r.visible) {
                  app.systemNoUi = false;
              }
          }
      }
      if (!app.systemNoUi) {
          app.curProcState = ActivityManager.PROCESS_STATE_PERSISTENT_UI;
      }
      return (app.curAdj=app.maxAdj);
  }
```
   
* 3.确认进程中是否有前台优先级的组件

    前台优先级的组件是指：
    
    1.前台的Activity; 2.正在接受广播的Receiver; 3.正在执行任务的Service;
    
    除此之外，还有Instrumentation被认为是具有较高优先级的。Instrumentation应用是辅助测试用的，正常运行的系统中不用考虑这种应用。
    
    假设进程中包含了以上提到的前台优先级的任何一个组件，则直接设置进程优先级为`FOREGROUND_APP_ADJ`即可。因为这已经是应用程序能够获取的最高优先级了。
    
```java
  int adj;
  int schedGroup;
  int procState;
  boolean foregroundActivities = false;
  BroadcastQueue queue;
  if (app == TOP_APP) {
      adj = ProcessList.FOREGROUND_APP_ADJ;
      schedGroup = ProcessList.SCHED_GROUP_TOP_APP;
      app.adjType = "top-activity";
      foregroundActivities = true;
      procState = PROCESS_STATE_CUR_TOP;
  } else if (app.instrumentationClass != null) {
      adj = ProcessList.FOREGROUND_APP_ADJ;
      schedGroup = ProcessList.SCHED_GROUP_DEFAULT;
      app.adjType = "instrumentation";
      procState = ActivityManager.PROCESS_STATE_FOREGROUND_SERVICE;
  } else if ((queue = isReceivingBroadcast(app)) != null) {
      adj = ProcessList.FOREGROUND_APP_ADJ;
      schedGroup = (queue == mFgBroadcastQueue)
              ? ProcessList.SCHED_GROUP_DEFAULT : ProcessList.SCHED_GROUP_BACKGROUND;
      app.adjType = "broadcast";
      procState = ActivityManager.PROCESS_STATE_RECEIVER;
  } else if (app.executingServices.size() > 0) {
      adj = ProcessList.FOREGROUND_APP_ADJ;
      schedGroup = app.execServicesFg ?
              ProcessList.SCHED_GROUP_DEFAULT : ProcessList.SCHED_GROUP_BACKGROUND;
      app.adjType = "exec-service";
      procState = ActivityManager.PROCESS_STATE_SERVICE;
  } else {
      schedGroup = ProcessList.SCHED_GROUP_BACKGROUND;
      adj = cachedAdj;
      procState = ActivityManager.PROCESS_STATE_CACHED_EMPTY;
      app.cached = true;
      app.empty = true;
      app.adjType = "cch-empty";
  }
```

* 4.确认进程中是否有较高优先级的Activity

   这里需要遍历进程中的所有Activity，找出其中优先级最高的设置为进程的优先级。
   
   即便Activity不是前台Activity，但是处于下面这些状态的Activity优先级也是被认为是较高优先级的：
   
   1. 该Activity处于可见状态
   2. 该Activity处于Pause正在Pause状态
   3. 该Activity正在stop

```java
if (!foregroundActivities && activitiesSize > 0) {
 int minLayer = ProcessList.VISIBLE_APP_LAYER_MAX;
 for (int j = 0; j < activitiesSize; j++) {
     final ActivityRecord r = app.activities.get(j);
     if (r.app != app) {
         Log.e(TAG, "Found activity " + r + " in proc activity list using " + r.app
                 + " instead of expected " + app);
         if (r.app == null || (r.app.uid == app.uid)) {
             // Only fix things up when they look sane
             r.app = app;
         } else {
             continue;
         }
     }
     if (r.visible) {
         // App has a visible activity; only upgrade adjustment.
         if (adj > ProcessList.VISIBLE_APP_ADJ) {
             adj = ProcessList.VISIBLE_APP_ADJ;
             app.adjType = "visible";
         }
         if (procState > PROCESS_STATE_CUR_TOP) {
             procState = PROCESS_STATE_CUR_TOP;
         }
         schedGroup = ProcessList.SCHED_GROUP_DEFAULT;
         app.cached = false;
         app.empty = false;
         foregroundActivities = true;
         if (r.task != null && minLayer > 0) {
             final int layer = r.task.mLayerRank;
             if (layer >= 0 && minLayer > layer) {
                 minLayer = layer;
             }
         }
         break;
     } else if (r.state == ActivityState.PAUSING || r.state == ActivityState.PAUSED) {
         if (adj > ProcessList.PERCEPTIBLE_APP_ADJ) {
             adj = ProcessList.PERCEPTIBLE_APP_ADJ;
             app.adjType = "pausing";
         }
         if (procState > PROCESS_STATE_CUR_TOP) {
             procState = PROCESS_STATE_CUR_TOP;
         }
         schedGroup = ProcessList.SCHED_GROUP_DEFAULT;
         app.cached = false;
         app.empty = false;
         foregroundActivities = true;
     } else if (r.state == ActivityState.STOPPING) {
         if (adj > ProcessList.PERCEPTIBLE_APP_ADJ) {
             adj = ProcessList.PERCEPTIBLE_APP_ADJ;
             app.adjType = "stopping";
         }
         if (!r.finishing) {
             if (procState > ActivityManager.PROCESS_STATE_LAST_ACTIVITY) {
                 procState = ActivityManager.PROCESS_STATE_LAST_ACTIVITY;
             }
         }
         app.cached = false;
         app.empty = false;
         foregroundActivities = true;
     } else {
         if (procState > ActivityManager.PROCESS_STATE_CACHED_ACTIVITY) {
             procState = ActivityManager.PROCESS_STATE_CACHED_ACTIVITY;
             app.adjType = "cch-act";
         }
     }
 }
 if (adj == ProcessList.VISIBLE_APP_ADJ) {
     adj += minLayer;
 }
}
```

* 5.确认进程中是否有前台Service

    通过`startForeground`启动的Service被认为是前台Service。给予这类进程`PERCEPTIBLE_APP_ADJ`级别的优先级。
  
```java
if (adj > ProcessList.PERCEPTIBLE_APP_ADJ
     || procState > ActivityManager.PROCESS_STATE_FOREGROUND_SERVICE) {
 if (app.foregroundServices) {
     // The user is aware of this app, so make it visible.
     adj = ProcessList.PERCEPTIBLE_APP_ADJ;
     procState = ActivityManager.PROCESS_STATE_FOREGROUND_SERVICE;
     app.cached = false;
     app.adjType = "fg-service";
     schedGroup = ProcessList.SCHED_GROUP_DEFAULT;
 } else if (app.forcingToForeground != null) {
     // The user is aware of this app, so make it visible.
     adj = ProcessList.PERCEPTIBLE_APP_ADJ;
     procState = ActivityManager.PROCESS_STATE_IMPORTANT_FOREGROUND;
     app.cached = false;
     app.adjType = "force-fg";
     app.adjSource = app.forcingToForeground;
     schedGroup = ProcessList.SCHED_GROUP_DEFAULT;
 }
}
```

* 6.确认是否是特殊类型进程

    特殊类型的进程包括：重量级进程，桌面进程，前一个应用进程，正在执行备份的进程。 
    重量级进程和前一个应用进程在上文中已经说过了。
    
```java
if (app == mHeavyWeightProcess) {
 if (adj > ProcessList.HEAVY_WEIGHT_APP_ADJ) {
     adj = ProcessList.HEAVY_WEIGHT_APP_ADJ;
     schedGroup = ProcessList.SCHED_GROUP_BACKGROUND;
     app.cached = false;
     app.adjType = "heavy";
 }
 if (procState > ActivityManager.PROCESS_STATE_HEAVY_WEIGHT) {
     procState = ActivityManager.PROCESS_STATE_HEAVY_WEIGHT;
 }
}
    
if (app == mHomeProcess) {
 if (adj > ProcessList.HOME_APP_ADJ) {
     adj = ProcessList.HOME_APP_ADJ;
     schedGroup = ProcessList.SCHED_GROUP_BACKGROUND;
     app.cached = false;
     app.adjType = "home";
 }
 if (procState > ActivityManager.PROCESS_STATE_HOME) {
     procState = ActivityManager.PROCESS_STATE_HOME;
 }
}
    
if (app == mPreviousProcess && app.activities.size() > 0) {
 if (adj > ProcessList.PREVIOUS_APP_ADJ) {
     adj = ProcessList.PREVIOUS_APP_ADJ;
     schedGroup = ProcessList.SCHED_GROUP_BACKGROUND;
     app.cached = false;
     app.adjType = "previous";
 }
 if (procState > ActivityManager.PROCESS_STATE_LAST_ACTIVITY) {
     procState = ActivityManager.PROCESS_STATE_LAST_ACTIVITY;
 }
}
    
if (false) Slog.i(TAG, "OOM " + app + ": initial adj=" + adj
     + " reason=" + app.adjType);
    
app.adjSeq = mAdjSeq;
app.curRawAdj = adj;
app.hasStartedServices = false;
    
if (mBackupTarget != null && app == mBackupTarget.app) {
 if (adj > ProcessList.BACKUP_APP_ADJ) {
     if (DEBUG_BACKUP) Slog.v(TAG_BACKUP, "oom BACKUP_APP_ADJ for " + app);
     adj = ProcessList.BACKUP_APP_ADJ;
     if (procState > ActivityManager.PROCESS_STATE_IMPORTANT_BACKGROUND) {
         procState = ActivityManager.PROCESS_STATE_IMPORTANT_BACKGROUND;
     }
     app.adjType = "backup";
     app.cached = false;
 }
 if (procState > ActivityManager.PROCESS_STATE_BACKUP) {
     procState = ActivityManager.PROCESS_STATE_BACKUP;
 }
}
```

* 7.根据所有Service的客户端计算优先级

    这里需要遍历所有的Service，并且还需要遍历每一个Service的所有连接。然后根据连接的关系确认客户端进程的优先级来确定当前进程的优先级。
    
    `ConnectionRecord.binding.client`即为客户端进程`ProcessRecord`，由此便可以知道客户端进程的优先级。
  
```java
for (int is = app.services.size()-1;
     is >= 0 && (adj > ProcessList.FOREGROUND_APP_ADJ
             || schedGroup == ProcessList.SCHED_GROUP_BACKGROUND
             || procState > ActivityManager.PROCESS_STATE_TOP);
     is--) {
 ServiceRecord s = app.services.valueAt(is);
 if (s.startRequested) {
     app.hasStartedServices = true;
     if (procState > ActivityManager.PROCESS_STATE_SERVICE) {
         procState = ActivityManager.PROCESS_STATE_SERVICE;
     }
     if (app.hasShownUi && app != mHomeProcess) {
         if (adj > ProcessList.SERVICE_ADJ) {
             app.adjType = "cch-started-ui-services";
         }
     } else {
         if (now < (s.lastActivity + ActiveServices.MAX_SERVICE_INACTIVITY)) {
             if (adj > ProcessList.SERVICE_ADJ) {
                 adj = ProcessList.SERVICE_ADJ;
                 app.adjType = "started-services";
                 app.cached = false;
             }
         }
         if (adj > ProcessList.SERVICE_ADJ) {
             app.adjType = "cch-started-services";
         }
     }
 }
    
 for (int conni = s.connections.size()-1;
         conni >= 0 && (adj > ProcessList.FOREGROUND_APP_ADJ
                 || schedGroup == ProcessList.SCHED_GROUP_BACKGROUND
                 || procState > ActivityManager.PROCESS_STATE_TOP);
         conni--) {
     ArrayList<ConnectionRecord> clist = s.connections.valueAt(conni);
     for (int i = 0;
             i < clist.size() && (adj > ProcessList.FOREGROUND_APP_ADJ
                     || schedGroup == ProcessList.SCHED_GROUP_BACKGROUND
                     || procState > ActivityManager.PROCESS_STATE_TOP);
```
      
* 8.根据所有Provider的客户端确认优先级

    这里与Service类似，需要遍历所有的Provider，以及每一个Provider的所有连接。然后根据连接的关系确认客户端进程的优先级来确定当前进程的优先级。
    
    类似的，`ContentProviderConnection.client`为客户端进程的`ProcessRecord`。

```java
for (int provi = app.pubProviders.size()-1;
     provi >= 0 && (adj > ProcessList.FOREGROUND_APP_ADJ
             || schedGroup == ProcessList.SCHED_GROUP_BACKGROUND
             || procState > ActivityManager.PROCESS_STATE_TOP);
     provi--) {
 ContentProviderRecord cpr = app.pubProviders.valueAt(provi);
 for (int i = cpr.connections.size()-1;
         i >= 0 && (adj > ProcessList.FOREGROUND_APP_ADJ
                 || schedGroup == ProcessList.SCHED_GROUP_BACKGROUND
                 || procState > ActivityManager.PROCESS_STATE_TOP);
         i--) {
     ContentProviderConnection conn = cpr.connections.get(i);
     ProcessRecord client = conn.client;
     if (client == app) {
         // Being our own client is not interesting.
         continue;
     }
     int clientAdj = computeOomAdjLocked(client, cachedAdj, TOP_APP, doingAll, now);
     ...
```
    
* 9.收尾工作
收尾工作主要是根据进程中的Service，Provider的一些特殊状态做一些处理，另外还有针对空进程以及设置了`maxAdj`的进程做一些处理，这里就不贴出代码了。

这里想专门说明一下的是，在这一步还会对Service进程做`ServiceB`的区分。相关代码见下文。

系统将Service进程分为ServiceA和ServiceB。ServiceA是相对来说较新的Service，而ServiceB相对来说是比较“老旧”的，对用户来说可能是不那么感兴趣的，因此ServiceB的优先级会相对低一些。

```java
static final int SERVICE_B_ADJ = 800;
static final int SERVICE_ADJ = 500;
```

而ServiceB的标准是：`app.serviceb = mNewNumAServiceProcs > (mNumServiceProcs/3);`
即：所有Service进程的前1/3为ServiceA，剩下为ServiceB。

```java
if (adj == ProcessList.SERVICE_ADJ) {
  if (doingAll) {
      app.serviceb = mNewNumAServiceProcs > (mNumServiceProcs/3);
      mNewNumServiceProcs++;
      if (!app.serviceb) {
          if (mLastMemoryLevel > ProcessStats.ADJ_MEM_FACTOR_NORMAL
                  && app.lastPss >= mProcessList.getCachedRestoreThresholdKb()) {
              app.serviceHighRam = true;
              app.serviceb = true;
          } else {
              mNewNumAServiceProcs++;
          }
      } else {
          app.serviceHighRam = false;
      }
  }
  if (app.serviceb) {
      adj = ProcessList.SERVICE_B_ADJ;
  }
}

app.curRawAdj = adj;
```
    
* 10.保存结果
    最终需要把本次的计算结果保存到ProcessRecord中：

```java
app.curAdj = app.modifyRawOomAdj(adj);
app.curSchedGroup = schedGroup;
app.curProcState = procState;
app.foregroundActivities = foregroundActivities;
```

# 优先级的生效

优先级的生效是指：将计算出来的优先级真正应用到系统中，`applyOomAdjLocked` 方法负责了此项工作。

前文中我们提到，优先级意味着三个方面，这里的生效就对应了三个方面：

1. `ProcessList.setOomAdj(app.pid, app.info.uid, app.curAdj);`
将计算出来的adj值写入到procfs中，即：`/proc/[pid]/oom_score_adj` 这个文件中。在进程回收的时候，这个值是被考虑的一个非常重要的因素，在下一篇文章中我们会详细讲解。

2. `Process.setProcessGroup(app.pid, processGroup);`
用来设置进程的调度组。

3. `app.thread.setProcessState(app.repProcState);`
这个方法会最终调用到

`VMRuntime.getRuntime().updateProcessState(dalvikProcessState);`将进程的状态设置到虚拟机中。

# 结束语
前言中我们提到，“优先级反应了系统对于进程重要性的判定。”

那么，系统如何评价进程的优先级，便是系统本身一个很重要的特性。了解系统的这一特性对于我们开发应用程序，以及对于应用程序运行的行为分析是很有意义的。

系统在判定优先级的时候，应当做到公平公正，并且不能让开发者有机可乘。

“公平公正”是指系统需要站在一个中间人的状态下，不偏倚任何一个应用，公正的将系统资源分配给真正需要的进程。并且在系统资源紧张的时候，回收不重要的进程。

通过上文的分析，我们看到，Android系统认为“重要”的进程主要有三类：

1. 系统进程
2. 前台与用户交互的进程
3. 前台进程所使用到的进程

不过对于这一点是有改进的空间的，例如，可以引入对于用户习惯的分析：如果是用户频繁使用的应用，可以给予这个应用更高的优先级来减少它们被回收的频度，以提升这些应用的效应速度。毕竟，冷启动和热启动，响应时间是差别很大的。

“不能让开发者有机可乘”是指：系统对于进程优先级的判定的因素应当是不能被开发者利用的。因为一旦开发者可以利用，每个开发者都肯定会将自己的设置为高优先级，来抢占更多的资源。

需要说明的是，Android在这个方面是存在缺陷的：在Android系统上，可以通过[startForeground](https://developer.android.com/reference/android/app/Service.html#startForeground(int,%20android.app.Notification))拿到前台的优先级的。后来Google也意识到这个问题，于是在API Level 18以上的版本上，调用`startForeground`这个API会在通知栏显示一条通知以告知用户。但是，**这个改进是有Bug的**：开发者可以同时通过`startForeground`启动两个Service，指定同样的通知id，然后退出其中一个，这样应用的不会在通知栏显示通知图标，并且拿到了前台的优先级。这个便是让开发者“有机可乘”了。

由于笔者认为这不是一个很好的行为，具体的做法不细说了，有兴趣自己去网上搜索。

本文，我们详细讲解的Android中进程优先级的计算方法，在下一篇文章中，我们会专门讲解与进程回收相关的内容，敬请期待。

# 参考资料与推荐读物

[Embedded Android: Porting, Extending, and Customizing](https://www.amazon.cn/Embedded-Android-Porting-Extending-and-Customizing-Yaghmour-Karim/dp/1449308295/)

[The proc filesystem](http://man7.org/linux/man-pages/man5/proc.5.html)

[sched_setscheduler](http://man7.org/linux/man-pages/man2/sched_setscheduler.2.html)  
