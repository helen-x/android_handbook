# Android进程管理三部曲[3]-内存的回收

>作者:  强波  (阿里云OS平台部-Cloud Engine)  
>博客: http://qiangbo.space/ 



本文是Android系统进程管理的第三篇文章。进程管理的前面两篇文章，请参见这里：

* [Android系统中的进程管理：进程的创建](http://www.jianshu.com/p/96f43244f754)
* [Android系统中的进程管理：进程的优先级](http://www.jianshu.com/p/0501bc2bbe7c)

本文适合Android平台的应用程序开发者，也适合对于Android系统内部实现感兴趣的读者。

 
# 前言

内存是系统中非常宝贵的资源，即便如今的移动设备上，内存已经达到4G甚至6G的级别，但对于内存的回收也依然重要，因为在Android系统上，同时运行的进程有可能会有几十甚至上百个之多。

如何将系统内存合理的分配给每个进程，以及如何进行内存回收，便是操作系统需要处理的问题之一。

本文会讲解Android系统中内存回收相关的知识。

对于内存回收，主要可以分为两个层次：

* **进程内的内存回收**：通过释放进程中的资源进行内存回收
* **进程级的内存回收**：通过杀死进程来进行内存回收

这其中，**进程内的内存回收**主要分为两个方面：

* 虚拟机自身的垃圾回收机制
* 在系统内存状态发生变化时，通知应用程序，让开发者进行内存回收

而**进程级的内存回收**主要是依靠系统中的两个模块，它们是：

* Linux OOM Killer
* LowMemoryKiller

在特定场景下，他们都会通过杀死进程来进行内存回收。

# Android系统的内存管理简介

在Android系统中，进程可以大致分为**系统进程**和**应用进程**两大类。

**系统进程**是系统内置的（例如：`init`，`zygote`，`system_server`进程），属于操作系统必不可少的一部分。系统进程的作用在于：

* 管理硬件设备
* 提供访问设备的基本能力
* 管理应用进程

**应用进程**是指应用程序运行的进程。这些应用程序可能是系统出厂自带的（例如Launcher，电话，短信等应用），也可能是用户自己安装的（例如：微信，支付宝等）。

Android中应用进程通常都运行在Java虚拟机中。在Android 5.0之前的版本，这个虚拟机是Dalvik，5.0及之后版本，Android引入了新的虚拟机，称作Android Runtime，简称“ART”。

关于ART和Dalvik可以参见这里：[ART and Dalvik](https://source.android.com/devices/tech/dalvik/index.html)。无论是Dalvik还是ART，本身都具有垃圾回收的能力，关于这一点，我们在后面专门讲解。

Android的应用程序都会依赖一些公共的资源，例如：Android SDK提供的类和接口，以及Framework公开的图片，字符串等。为了达到节省内存的目的，这些资源在内存中并不是每个应用进程单独一份拷贝。而是会在所有应用之间共享，因为所有应用进程都是作为Zygote进程fork出来的子进程。关于这部分内容，我们已经在[Android系统中的进程管理：进程的创建](http://www.jianshu.com/p/96f43244f754)一文中讲解过。

在Java语言中，通过`new`创建的对象都会在堆中分配内存。应用程序堆的大小是有限的。系统会根据设备的物理内存大小来确定每个应用程序所允许使用的内存大小，一旦应用程序使用的内存超过这个大小，便会发生[OutOfMemoryError](https://developer.android.com/reference/java/lang/OutOfMemoryError.html)。

因此开发者需要关心应用的内存使用状况。关于如何监测应用程序的内存使用，可以参见这里：[Investigating Your RAM Usage](https://developer.android.com/studio/profile/investigate-ram.html)。

# 开发者相关的API
下面是一些与内存相关的开发者API，它们是Android SDK的一部分。

## ComponentCallbacks2
Android系统会根据当前的系统内存状态和应用的自身状态对应用进行通知。这种通知的目的是希望应用能够感知到系统和自身的状态变化，以便开发者可以更准确的把握应用的运行。

例如：在系统内存充足时，为了提升响应性能，应用可以缓存更多的资源。但是当系统内存紧张时，开发者应当释放一定的资源来缓解内存紧张的状态。

`ComponentCallbacks2`接口中的`void onTrimMemory(int level)` 回调函数用来接收这个通知。关于这一点，在“开发者的内存回收”一节，我们会详细讲解。

## ActivityManager
ActivityManager，从名称中就可以看出，这个类是用来管理Activity的系统服务。但这个类中也包含了很多运行时状态查询的接口，这其中就包括与内存相关的几个：

- `int getMemoryClass ()` 获取当前设备上，单个应用的内存大小限制，单位是M。注意，这个函数的返回值只是一个大致的值。
- `void getMemoryInfo (ActivityManager.MemoryInfo outInfo)` 获取系统的内存信息，具体结构可以查看[ActivityManager.MemoryInfo](https://developer.android.com/reference/android/app/ActivityManager.MemoryInfo.html)，开发者最关心的可能就是`availMem`以及`totalMem`。
- `void getMyMemoryState (ActivityManager.RunningAppProcessInfo outState)` 获取调用进程的内存信息
- `MemoryInfo[] getProcessMemoryInfo (int[] pids)` 通过pid获取指定进程的内存信息
- `boolean isLowRamDevice()` 查询当前设备是否是低内存设备

## Runtime
Java应用程序都会有一个Runtime接口的实例，通过这个实例可以查询运行时的一些状态，与内存相关的接口有：

- `freeMemory()` 获取Java虚拟机的剩余内存
- `maxMemory()` 获取Java虚拟机所能使用的最大内存
- `totalMemory()` 获取Java虚拟机拥有的最大内存

# 虚拟机的垃圾回收
垃圾回收是指：虚拟机会监测应用程序的对象创建和使用，并在一些特定的时候销毁无用的对象以回收内存。

垃圾回收的基本想法是要找出虚拟机中哪些对象已经不会再被使用然后将其释放。其最常用的算法有下面两种：

## 引用计数算法

引用计数算法是为每个对象维护一个被引用的次数：对象刚创建时的初始引用计数为0，每次被一个对象引用时，引用计数加1，反之减1。当一个对象的引用计数重新回到0时便可以认为是不会被使用的，这些对象便可以被垃圾回收。

读者可能马上会想到，当有两个对象互相引用时，这时引用计数该如何计算。关于这部分内容，这里不再展开讲解。有兴趣的读者可以查询Google或者维基百科：[Garbage collection](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))

## 对象追踪算法
对象追踪算法是通过GC root类型的对象为起点，追踪所有被这些对象所引用的对象，并顺着这些被引用的对象继续往下追踪，在追踪的过程中，对所有被追踪到的对象打上标记。

而剩下的那些没有被打过标记的对象便可以认为是没有被使用的，因此这些对象可以将其释放。

这里提到的的GC root类型的对象有四类：

* 栈中的local变量，即方法中的局部变量
* 活动的线程（例如主线程或者开发者创建的线程）
* static变量
* JNI中的引用

下面这幅图描述了这种算法：
![](http://upload-images.jianshu.io/upload_images/4048192-e1c04336771e909a.GIF?imageMogr2/auto-orient/strip)


a）表示算法开始时，所有对象的标记为false，然后以GC root为起点开始追踪和打标记，b）中被追踪到的对象打上了标记。剩下的没有打上标记的对象便可以释放了。算法结束之后，c）中将所有对象的标记全部置为false。下一轮计算时，重新以GC root开始追踪。

Dalvik虚拟机主要用的就是垃圾回收算法，这里是其Source：[MarkSweep.cpp](https://android.googlesource.com/platform/dalvik.git/+/android-4.3_r2/vm/alloc/MarkSweep.cpp)

# 开发者的内存回收
内存回收并不是仅仅是系统的事情，作为开发者，也需要在合适的场合下进行内存释放。无节制的消耗内存将导致应用程序[OutOfMemoryError](https://developer.android.com/reference/java/lang/OutOfMemoryError.html)。

上文中提到，虚拟机的垃圾回收会回收那些不会再被使用到的对象。因此，开发者所需要做的就是：当确定某些对象不会再被使用时，要主动释放对其引用，这样虚拟机才能将其回收。对于不再被用到对象，仍然保持对其引用导致其无法释放，将导致内存泄漏的发生。

为了更好的进行内存回收，系统会一些场景下会通知应用，希望应用能够配合进行一些内存的释放。

`ComponentCallbacks2`接口中的 `void onTrimMemory(int level)`回调就是用来接收这个事件的。

`Activity`, `Service`, `ContentProvider`和`Application`都实现了这个接口，因此这些类的子类都可以接收这个事件。

`onTrimMemory`回调的参数是一个级别，系统会根据应用本身的状态以及系统的内存状态发送不同的级别，具体的包括：

- 应用处于Runnig状态可能收到的级别
    - `TRIM_MEMORY_RUNNING_MODERATE` 表示系统内存已经稍低
    - `TRIM_MEMORY_RUNNING_LOW` 表示系统内存已经相当低
    - `TRIM_MEMORY_RUNNING_CRITICAL` 表示系统内存已经非常低，你的应用程序应当考虑释放部分资源

- 应用的可见性发生变化时收到的级别
    - `TRIM_MEMORY_UI_HIDDEN` 表示应用已经处于不可见状态，可以考虑释放一些与显示相关的资源

- 应用处于后台时可能收到的级别
    - `TRIM_MEMORY_BACKGROUND` 表示系统内存稍低，你的应用被杀的可能性不大。但可以考虑适当释放资源
    - `TRIM_MEMORY_MODERATE` 表示系统内存已经较低，当内存持续减少，你的应用可能会被杀死
    - `TRIM_MEMORY_COMPLETE` 表示系统内存已经非常低，你的应用即将被杀死，请释放所有可能释放的资源

这里是这个方法实现的示例代码：[Release memory in response to events](https://developer.android.com/topic/performance/memory.html#release)

在前面的文章中我们提到过：`ActivityManagerService`负责管理所有的应用进程。

而这里的通知也是来自`ActivityManagerService`。在`updateOomAdjLocked`的时候，`ActivityManagerService`会根据系统内存以及应用的状态通过`app.thread.scheduleTrimMemory`发送通知给应用程序。

这里的`app`是`ProcessRecord`，即描述应用进程的对象，`thread`是应用的主线程。而`scheduleTrimMemory`是通过Binder IPC的方式将消息发送到应用进程上。这些内容在前面的文章中已经介绍过，如果觉得陌生，可以阅读一下前面两篇文章。

在`ActivityThread`中（这个是应用程序的主线程），接受到这个通知之后，便会遍历应用进程中所有能接受这个通知的组件，然后逐个回调通知。

相关代码如下：

```java
final void handleTrimMemory(int level) {
   if (DEBUG_MEMORY_TRIM) Slog.v(TAG, "Trimming memory to level: " + level);

   ArrayList<ComponentCallbacks2> callbacks = collectComponentCallbacks(true, null);

   final int N = callbacks.size();
   for (int i = 0; i < N; i++) {
       callbacks.get(i).onTrimMemory(level);
   }

   WindowManagerGlobal.getInstance().trimMemory(level);
}
```

# Linux OOM Killer
前面提到的机制都是在进程内部通过释放对象来进行内存回收。

而实际上，系统中运行的进程数量，以及每个进程所消耗的内存都是不确定的。

在极端的情况下，系统的内存可能处于非常严峻的状态，假设这个时候所有进程都不愿意释放内存，系统将会卡死。

为了使系统能够继续运转不至于卡死，系统会尝试杀死一些不重要的进程来进行内存回收，这其中涉及的模块主要是：Linux OOM Killer和LowMemoryKiller。

Linux OOM Killer是Linux内核的一部分，其源码可以在这里查看：[/mm/oom_kill.c](http://lxr.free-electrons.com/source/mm/oom_kill.c)。

Linux OOM Killer的基本想法是：

**当系统已经没法再分配内存的时候，内核会遍历所有的进程，对每个进程计算badness值，得分(badness)最高的进程将会被杀死**。

即：badness得分越低表示进程越重要，反之表示不重要。

Linux OOM Killer的执行流程如下：

`_alloc_pages -> out_of_memory() -> select_bad_process() -> oom_badness()`

这其中，`_alloc_pages` 是内核在分配内存时调用的函数。当内核发现无法再分配内存时，便会计算每个进程的badness值，然后选择最大的（系统认为最不重要的）将其杀死。

那么，内核是如何计算进程的badness值的呢？请看下面的代码：

```c++
unsigned long oom_badness(struct task_struct *p, struct mem_cgroup *memcg,
			  const nodemask_t *nodemask, unsigned long totalpages)
{
	long points;
	long adj;

	...

	points = get_mm_rss(p->mm) + p->mm->nr_ptes + get_mm_counter(p->mm, MM_SWAPENTS);
	task_unlock(p);

	if (has_capability_noaudit(p, CAP_SYS_ADMIN))
		points -= (points * 3) / 100;

	adj *= totalpages / 1000;
	points += adj;

	return points > 0 ? points : 1;
}
```

从这段代码中，我们可以看到，影响进程badness值的因素主要有三个：

* 进程的`oom_score_adj`值
* 进程的内存占用大小
* 进程是否是root用户的进程

即，`oom_score_adj`(关于`oom_score_adj`，在[Android系统中的进程管理：进程的优先级](http://qiangbo.space/2016-11-23/AndroidAnatomy_Process_OomAdj/)一文中我们专门讲解过。)值越小，进程占用的内存越小，并且如果是root用户的进程，系统就认为这个进程越重要。

反之则被认为越不重要，越容易被杀死。

# LowMemoryKiller

OOM Killer是在系统内存使用情况非常严峻的时候才会起作用。但直到这个时候才开始杀死进程来回收内存是有点晚的。因为在进程被杀死之前，其他进程都无法再申请内存了。

因此，Google在Android上新增了一个LowMemoryKiller模块。LowMemoryKiller通常会在Linux OOM Killer工作之前，就开始杀死进程。

LowMemoryKiller的做法是：

**提供6个可以设置的内存级别，当系统内存每低于一个级别时，将oom_score_adj大于某个指定值的进程全部杀死。**

这么说会有些抽象，但具体看一下LowMemoryKiller的配置文件我们就好理解了。

LowMemoryKiller在sysfs上暴露了两个文件来供系统调整参数，这两个文件的路径是：

* `/sys/module/lowmemorykiller/parameters/minfree`
* `/sys/module/lowmemorykiller/parameters/adj`

如果你手上有一个Android设备，你可以通过`adb shell`连上去之后，通过`cat`命令查看这两个文件的内容。

这两个文件是配对使用的，每个文件中都是由逗号分隔的6个整数值。

在某个设备上，这两个文件的值可能分别是下面这样：

* `18432,23040,27648,32256,55296,80640`
* `0,100,200,300,900,906`

这组配置的含义是；当系统内存低于80640k时，将oom_score_adj值大于906的进程全部杀死；当系统内存低于55296k时，将oom_score_adj值大于900的进程全部杀死，其他类推。

LowMemoryKiller杀死进程的时候会在内核留下日志，你可以通过`dmesg`
命令中看到。这个日志可能是这样的：

```
lowmemorykiller: Killing 'gnunet-service-' (service adj 0,
to free 327224kB on behalf of 'kswapd0' (21) because
cache 6064kB is below limit 6144kB for oom_score_adj 0
```

从这个日志中，我们可以看到被杀死进程的名称，进程pid和oom_score_adj值。另外还有系统在杀死这个进程之前系统内存还剩多少，以及杀死这个进程释放了多少。

LowMemoryKiller的源码也在内核中，路径是：`kernel/drivers/staging/android/lowmemorykiller.c`。

lowmemorykiller.c中定义了如下几个函数：

* `lowmem_shrink`
* `lowmem_init`
* `lowmem_exit`
* `lowmem_oom_adj_to_oom_score_adj`
* `lowmem_autodetect_oom_adj_values`
* `lowmem_adj_array_set`
* `lowmem_adj_array_get`
* `lowmem_adj_array_free`

LowMemoryKiller本身是一个内核驱动程序的形式存在，`lowmem_init`和`lowmem_exit`
 分别负责模块的初始化和退出清理工作。

在`lowmem_init`函数中，就是通过`register_shrinker`向内核中注册了`register_shrinker` 函数：

```c++
static int __init lowmem_init(void)
{
	register_shrinker(&lowmem_shrinker);
	return 0;
}
```

`register_shrinker`函数就是LowMemoryKiller的算法核心，这个函数的代码和说明如下：

```c++
static int lowmem_shrink(struct shrinker *s, struct shrink_control *sc)
{
	struct task_struct *tsk;
	struct task_struct *selected = NULL;
	int rem = 0;
	int tasksize;
	int i;
	short min_score_adj = OOM_SCORE_ADJ_MAX + 1;
	int minfree = 0;
	int selected_tasksize = 0;
	short selected_oom_score_adj;
	int array_size = ARRAY_SIZE(lowmem_adj);
	int other_free = global_page_state(NR_FREE_PAGES) - totalreserve_pages;
	int other_file = global_page_state(NR_FILE_PAGES) -
						global_page_state(NR_SHMEM) -
						total_swapcache_pages();
   
	if (lowmem_adj_size < array_size)
		array_size = lowmem_adj_size;
	if (lowmem_minfree_size < array_size)
		array_size = lowmem_minfree_size;
	// lowmem_minfree 和lowmem_adj 记录了两个配置文件中配置的数据
	for (i = 0; i < array_size; i++) {
		minfree = lowmem_minfree[i];
		// 确定当前系统处于低内存的第几档
		if (other_free < minfree && other_file < minfree) {
		   // 确定需要杀死的进程的oom_score_adj的上限
			min_score_adj = lowmem_adj[i];
			break;
		}
	}
	if (sc->nr_to_scan > 0)
		lowmem_print(3, "lowmem_shrink %lu, %x, ofree %d %d, ma %hd\n",
				sc->nr_to_scan, sc->gfp_mask, other_free,
				other_file, min_score_adj);
	rem = global_page_state(NR_ACTIVE_ANON) +
		global_page_state(NR_ACTIVE_FILE) +
		global_page_state(NR_INACTIVE_ANON) +
		global_page_state(NR_INACTIVE_FILE);
	if (sc->nr_to_scan <= 0 || min_score_adj == OOM_SCORE_ADJ_MAX + 1) {
		lowmem_print(5, "lowmem_shrink %lu, %x, return %d\n",
			     sc->nr_to_scan, sc->gfp_mask, rem);
		return rem;
	}
	selected_oom_score_adj = min_score_adj;

	rcu_read_lock();
	// 遍历所有进程
	for_each_process(tsk) {
		struct task_struct *p;
		short oom_score_adj;

		if (tsk->flags & PF_KTHREAD)
			continue;

		p = find_lock_task_mm(tsk);
		if (!p)
			continue;

		if (test_tsk_thread_flag(p, TIF_MEMDIE) &&
		    time_before_eq(jiffies, lowmem_deathpending_timeout)) {
			task_unlock(p);
			rcu_read_unlock();
			return 0;
		}
		oom_score_adj = p->signal->oom_score_adj;
		// 跳过那些oom_score_adj值比目标值小的
		if (oom_score_adj < min_score_adj) {
			task_unlock(p);
			continue;
		}
		tasksize = get_mm_rss(p->mm);
		task_unlock(p);
		if (tasksize <= 0)
			continue;
		// selected 是将要杀死的备选进程
		if (selected) {
		   // 跳过那些oom_score_adj比备选的小的
			if (oom_score_adj < selected_oom_score_adj)
				continue;
		   // 如果oom_score_adj一样，跳过那些内存消耗更小的
			if (oom_score_adj == selected_oom_score_adj &&
			    tasksize <= selected_tasksize)
				continue;
		}
		// 更换备选的目标，因为又发现了一个oom_score_adj更大，
		// 或者内存消耗更大的进程
		selected = p;
		selected_tasksize = tasksize;
		selected_oom_score_adj = oom_score_adj;
		lowmem_print(2, "select '%s' (%d), adj %hd, size %d, to kill\n",
			     p->comm, p->pid, oom_score_adj, tasksize);
	}
	
	// 已经选中目标，记录日志并杀死进程
	if (selected) {
		long cache_size = other_file * (long)(PAGE_SIZE / 1024);
		long cache_limit = minfree * (long)(PAGE_SIZE / 1024);
		long free = other_free * (long)(PAGE_SIZE / 1024);
		trace_lowmemory_kill(selected, cache_size, cache_limit, free);
		lowmem_print(1, "Killing '%s' (%d), adj %hd,\n" \
				"   to free %ldkB on behalf of '%s' (%d) because\n" \
				"   cache %ldkB is below limit %ldkB for oom_score_adj %hd\n" \
				"   Free memory is %ldkB above reserved\n",
			     selected->comm, selected->pid,
			     selected_oom_score_adj,
			     selected_tasksize * (long)(PAGE_SIZE / 1024),
			     current->comm, current->pid,
			     cache_size, cache_limit,
			     min_score_adj,
			     free);

		lowmem_deathpending_timeout = jiffies + HZ;
		set_tsk_thread_flag(selected, TIF_MEMDIE);
		send_sig(SIGKILL, selected, 0);
		rem -= selected_tasksize;
	}
	lowmem_print(4, "lowmem_shrink %lu, %x, return %d\n",
		     sc->nr_to_scan, sc->gfp_mask, rem);
	rcu_read_unlock();
	return rem;
}
```

# 进程的死亡处理

在任何时候，应用进程都可能死亡，例如被OOM Killer或者LowMemoryKiller杀死，自身crash死亡又或者被用户手动杀死。无论哪种情况，作为应用进程的管理者`ActivityManagerService`都需要知道。

在应用进程死亡之后，ActivityManagerService需要执行如下工作：

* **执行清理工作** ActivityManagerService内部的ProcessRecord以及可能存在的四大组件的相关结构需要全部清理干净
* **重新计算进程的优先级** 上文已经提到过，进程的优先级是有关联性的，有其中一个进程死亡了，可能会连到影响到其他进程的优先级需要调整。

ActivityManagerService是利用Binder提供的死亡通知机制来进行进程的死亡处理的。关于Binder请参阅其他资料，限于篇幅关系，这里不再展开讲解。

简单来说，死亡通知机制就提供了进程间的一种死亡监听的能力：当目标进程死亡的时候，监听回调会执行。

ActivityManagerService中的AppDeathRecipient监听了应用进程的死亡消息，该类代码如下：

```java
private final class AppDeathRecipient implements IBinder.DeathRecipient {
   final ProcessRecord mApp;
   final int mPid;
   final IApplicationThread mAppThread;

   AppDeathRecipient(ProcessRecord app, int pid,
           IApplicationThread thread) {
       mApp = app;
       mPid = pid;
       mAppThread = thread;
   }

   @Override
   public void binderDied() {
       synchronized(ActivityManagerService.this) {
           appDiedLocked(mApp, mPid, mAppThread, true);
       }
   }
}
```

每一个应用进程在启动之后，都会attach到ActivityManagerService上通知它自己的进程已经启动完成了。这时ActivityManagerService便会为其创建一个死亡通知的监听器。在这之后如果进程死亡了，ActivityManagerService便会收到通知。

```java
private final boolean attachApplicationLocked(IApplicationThread thread,
       int pid) {
    ...
        try {
            AppDeathRecipient adr = new AppDeathRecipient(
                    app, pid, thread);
            thread.asBinder().linkToDeath(adr, 0);
            app.deathRecipient = adr;
        } catch (RemoteException e) {
            app.resetPackageList(mProcessStats);
            startProcessLocked(app, "link fail", processName);
            return false;
        }
    ...
}
```

进程死亡之后的处理工作是`appDiedLocked`这个方法中处理的，这部分还是比较容易理解的，这里就不过多讲解了。

# 结束语

这三篇文章，我们详细讲解了Android系统中进程的创建，优先级的管理和内存回收。这些内容对于所有运行在Android系统中的应用进程都是适用的。

优秀的开发者应该充分了解这些内容，因为这是与应用的生命周期密切相关的。

由于篇幅所限，这其中有些知识我们没有详细展开讨论，但有些内容会在今后的文章中专门讲解。

由于笔者水平有限，文章中不免有所错漏，欢迎读者指出。

# 参考资料与推荐读物

[Overview of Android Memory Management](https://developer.android.com/topic/performance/memory-overview.html)

[Understanding Java Garbage Collection](http://www.cubrid.org/blog/dev-platform/understanding-java-garbage-collection/)

[Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html)

[Java Memory Management](https://www.dynatrace.com/resources/ebooks/javabook/how-garbage-collection-works/)

[Debugging ART Garbage Collection](https://source.android.com/devices/tech/dalvik/gc-debug.html)

[Android Runtime](https://www.google.com/events/io/io14videos/b750c8da-aebe-e311-b297-00155d5066d7)

[Taming the OOM killer](https://lwn.net/Articles/317814/)

[OOM Killer](https://linux-mm.org/OOM_Killer)

[Out Of Memory Management](https://www.kernel.org/doc/gorman/html/understand/understand016.html) 
