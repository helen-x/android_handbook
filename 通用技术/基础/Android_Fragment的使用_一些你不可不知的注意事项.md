# Android Fragment 的使用，一些你不可不知的注意事项  


>作者: 亦枫  
>博客: http://yifeng.studio/  

Fragment，俗称碎片，自 Android 3.0 开始被引进并大量使用。然而就是这样耳熟能详的一个东西，在开发中我们还是会遇见各种各样的问题，层出不穷。所以，是时候总结一波了。 



## Fragment 简介

作为 Activity 界面的一部分，Fragment 的存在必须依附于 Activity，并且与 Activity 一样，拥有自己的生命周期，同时处理用户的交互动作。   
同一个 Activity 可以有一个或多个 Fragment 作为界面内容，并且可以动态添加、删除 Fragment，灵活控制 UI 内容，也可以用来解决部分屏幕适配问题
另外，support v4 包中也提供了 Fragment，兼容 Android 3.0 之前的系统（当然，现在 3.0 之前的系统在市场上已经很少见了，可以不予考虑），使用兼容包需要注意两点：  

- Activity 必须继承自 FragmentActivity；
- 使用 getSupportFragmentManager() 方法获取 FragmentManager 对象； 



## 生命周期
作为宿主 Activity 的一部分，Activity 拥有的大部分生命周期函数在 Fragment 中同样存在，并与 Activity 保持同步。  同时，作为一个特殊情况的存在，Fragment 也有一些自己的生命周期函数，如 onAttach()、onCreateView() 等。    

至于 Activity 与 Fragment 之间生命周期函数的对应同步关系，来自 GitHub 的 xxv/android-lifecycle 项目用了一幅图完美地予以展示：  

![](http://upload-images.jianshu.io/upload_images/4048192-070370618e2e16a9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)    

关于 Fragment 各个生命周期函数的意义，这里就不一一叙述，可以参考官网介绍：[Fragment Lifecycle](https://developer.android.com/reference/android/app/Fragment.html)。  

## 创建实例   
像普通的类一样，Fragment 拥有自己的构造函数，于是我们可以像下面这样在 Activity 中创建 Fragment 实例：    

```java  
MainFragment mainFragment = new MainFragment(); 
```   

> 注意:不用使用带参构造函数的Framgment   
 
如果需要在创建 Fragment 实例时传递参数进行初始化的话，可以创建一个带参数的构造函数，并初始化 Fragment 成员变量等。这样做，看似没有问题，但在一些特殊状况下还是有问题的。    

我们知道，Activity 在一些特殊状况下会发生 destroy 并重新 create 的情形，比如屏幕旋转、内存吃紧时；对应的，依附于 Activity 存在的 Fragment 也会发生类似的状况。而一旦重新 create 时，Fragment 便会调用默认的无参构造函数，导致无法执行有参构造函数进行初始化工作。    

好在 Fragment 提供了相应的 API 帮助我们解决这个问题。利用 bundle 传递数据，参考代码如下：   


```java  
public static OneFragment newInstance(int args){
    OneFragment oneFragment = new OneFragment();

    Bundle bundle = new Bundle();
    bundle.putInt("someArgs", args);

    oneFragment.setArguments(bundle);
    return oneFragment;
}

@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    Bundle bundle = getArguments();
    int args = bundle.getInt("someArgs");
}
```




## 嵌入方式   
Activity 嵌入 Fragment 分为布局静态嵌入和代码动态嵌入两种。前者在 Activity 的 Layout 布局中使用 <fragment> 标签嵌入指定 Fragment，如：   

```java    
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <fragment
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        class="com.yifeng.samples.OneFragment"/>

</LinearLayout>
```
后者在 Activity 的 Java 代码中借助管理器类 FragmentManager 和 事务类 FragmentTransaction 提供的 replace() 方法替换 Activity 的 Layout 中的相应容器布局，如：   

```java   
FragmentManager fm = getFragmentManager();
FragmentTransaction ft = fm.beginTransaction();
ft.replace(R.id.fl_content, OneFragment.newInstance());
ft.commit();
```  

这两种嵌入方式对应的 Fragment 生命周期略有不同，从生命周期图中可以看出。相比布局静态嵌入方式，代码动态嵌入方式更为常用，毕竟后者能够实现灵活控制多个 Fragment，动态改变 Activity 中的内容。   


## getChildFragmentManager()  
像上面这样，在 Activity 嵌入 Fragment 时，需要使用 FragmentManager，通过 Activity 提供的 getFragmentManager() 方法即可获取，用于管理 Activity 里面嵌入的所有一级 Fragment。    
 
然而有时候，我们会在 Fragment 里面继续嵌套二级甚至三级 Fragment，即 Activity 嵌套多级 Fragment。此时在 Fragment 里管理子 Fragment 时，也需要使用到 FragmentManager。但是一定要使用 getChildFragmentManager() 方法获取 FragmentManager 对象！     

从官方文档注释上也可以看出这两个方法获取到的 FragmentManager 对象的区别：
Activity：getFragmentManager()    

>Return the FragmentManager for interacting with fragments associated with this activity.  

Fragment：getChildFragmentManager()   
>Return a private FragmentManager for placing and managing Fragments inside of this Fragment.  
 
 
## FragmentTransaction   
 
Fragment 的动态添加、删除等操作都需要借助于 FragmentTransaction 类来完成，比如上面提到的 replace() 操作。     

FragmentTransaction 提供有很多方法供开发人员操作 Activity 里面的 Fragment，具体可以参考官网介绍：FragmentTransaction Public methods，这里介绍几个常用的关键方法：  
 
- add() 系列：添加 Fragment 到 Activity 界面中；  
- remove()：移除 Activity 中的指定 Fragment；  
- replace() 系列：通过内部调用 remove() 和 add() 完成 Fragment 的修改；  
- hide() 和 show()：隐藏和显示 Activity 中的 Fragment；  
- addToBackStack()：添加当前事务到回退栈中，即当按下返回键时，界面回归到当前事物状态；  
- commit()：提交事务，所有通过上述方法对 Fragment 的改动都必须通过调用方法完成提交；    

>注意：动态切换显示 Activity 中的多个 Fragment 时，可以通过 replace() 实现，也可以 hide() 和 show() 方法实现。事实上，我们更倾向于使用后者，因为 replace() 方法不会保留 Fragment 的状态，也就是说诸如 EditText 内容输入等用户操作在 remove() 时会消失。当然，如果你不想保留用户操作的话，可以选择前者，视情况而定。    
 
 
## BackStack（回退栈）
通过 addToBackStack() 保存当前事务，当用户按下返回键时，如果回退栈中保存有之前的事务，便会执行事务回退，而不是 finish 掉当前 Activity。    

举个例子，比如 App 中有一个新用户注册功能，包括设置用户名、密码、手机号等等流程，设计师在 UI 设计上将每个流程单独设计成一个界面，引导用户一步步操作。作为开发人员，如果将每一个完善信息的流程单独设置成一个 Activity 的话操作起来就比较繁琐，并且也不易于应用里的逻辑处理，而如果使用 Fragment 并结合回退栈的话，就非常合适了。   

将每一个设置的流程写成一个 Fragment，通过状态控制显示不同的 Fragment，并利用回退栈实现返回上一步操作的功能。比如从 FirstStepFragment 进入 SecondStepFragment 时，比如可以在 LoginActivity.java 中这样操作：   

```java      
FragmentManager fm = getSupportFragmentManager();
FragmentTransaction ft = fm.beginTransaction();
ft.hide(firstStepFragment);
if (secondStepFragment==null){
    ft.add(R.id.fl_content, secondStepFragment);
}else {
    ft.show(secondStepFragment);
}
ft.addToBackStack(null);
ft.commit();
```   
>注意：这里使用了 hide() 方法，而不是 replace() 方法，因为我们当然希望用户返回上一步操作时，之前设置的内容不会消失。  


## 通信方式   
通常，Fragment 与 Activity 通信存在三种情形：Activity 操作内嵌的 Fragment，Fragment 操作宿主 Activity，Fragment 操作同属 Activity中的其他 Fragment。    

由于 Activity 持有所有内嵌的 Fragment 对象实例（创建实例时保存的 Fragment 对象，或者通过 FragmentManager 类提供的 findFragmentById() 和 findFragmentByTag() 方法也能获取到 Fragment 对象），所以可以直接操作 Fragment；Fragment 通过 getActivity() 方法可以获取到宿主 Activity 对象（强制转换类型即可），进而可以操作宿主 Activity；那么很自然的，获取到宿主 Activity 对象的 Fragment 便可以操作其他 Fragment 对象。    

虽然上述操作已经能够解决 Activity 与 Fragment 的通信问题，但会造成代码逻辑紊乱的结果，极度不符合这一编程思想：高内聚，低耦合。Fragment 做好自己的事情即可，所有涉及到 Fragment 之间的控制显示等操作，都应交由宿主 Activity 来统一管理。    

所以我们强烈推荐，使用对外开放接口的形式将 Fragment 的一些对外操作传递给宿主 Activity。具体实现方式如下：   

```java    
public class OneFragment extends Fragment implements View.OnClickListener{

    public interface IOneFragmentClickListener{
        void onOneFragmentClick();
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View contentView = inflater.inflate(R.layout.fragment_one, null);
        contentView.findViewById(R.id.edt_one).setOnClickListener(this);
        return contentView;
    }

    @Override
    public void onClick(View v) {
        if (getActivity() instanceof IOneFragmentClickListener){
             ((IOneFragmentClickListener) getActivity()).onOneFragmentClick();
        }
    }

}

```  

只要在宿主 Activity 实现 Fragment 定义的对外接口 IOneFragmentClickListener，便可以实现 Fragment 调用 Activity 的功能。  

当然，你可以这样做:  

```java     
public class OneFragment extends Fragment implements View.OnClickListener{
    
    private IOneFragmentClickListener clickListener;

    public interface IOneFragmentClickListener{
        void onOneFragmentClick();
    }

    public void setClickListener(IOneFragmentClickListener clickListener) {
        this.clickListener = clickListener;
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View contentView = inflater.inflate(R.layout.fragment_one, null);
        contentView.findViewById(R.id.edt_one).setOnClickListener(this);
        return contentView;
    }

    @Override
    public void onClick(View v) {
        clickListener.onOneFragmentClick();
    }

}
```   

原理是一样的，只是相比第一种方式，需要在宿主 Activity 中额外添加一步监听设置：   

```java   
oneFragment.setClickListener(this);
```    



## getActivity() 引用问题  
使用中，经常会在 Fragment 中通过 getActivity() 获取到宿主 Activity 对象，但稍有不慎便会引发下面这两个问题：   

第一个， Activity 的实例销毁问题。比如，Fragment 中存在类似网络请求之类的异步耗时任务，当该任务执行完毕回调 Fragment 的方法并用到宿主 Activity 对象时，很有可能宿主 Activity 对象已经销毁，从而引发 NullPointException 等异常，甚至造成程序崩溃。所以，异步回调时需要注意添加空值等判断（譬如:fragment.isAdd()，getActivity()!＝null 等），或者在 Fragment 创建实例时就通过 getActivity().getApplicationContext() 方法保存整个应用的上下文对象，再来使用；   

第二个，内存泄漏问题。如果 Fragment 持有宿主 Activity 的引用，会导致宿主 Activity 无法回收，造成内存泄漏。所以，如果可以的话，尽量不要在 Fragment 中持有宿主 Activity 的引用。  

为了解决 Context 上下文引用的问题，Fragment 提供了一个 onAttach(context) 方法，在此方法中我们可以获取到 Context 对象，如：  

```java    
@Override
public void onAttach(Context context) {
    super.onAttach(context);
    this.context = context;
}
```   


##  Fragment 重叠问题    

前面我们介绍 Fragment 初始化时提到 Activity 销毁重建的问题，试想一下，当 Activity 重新执行 onCreate() 方法时，是不是会再次执行 Fragment 的创建和显示等操作呢？而之前已经存在的 Fragment 实例也会销毁再次创建，这不就与 Activity 中 onCreate() 方法里面第二次创建的 Fragment 同时显示从而发生 UI 重叠的问题了吗？  

根据经验，通常我们会在 AndroidManifest 里将 Activity 设置为横屏模式，所以不会由于屏幕旋转导致这种问题的出现。一种比较多的出现方式是，应用长时间处于后台，但由于设备内存吃紧，导致 Activity 被销毁，而当用户再次打开应用时便会发生 Fragment 重叠的问题。但是这种问题在开发阶段由于应用的频繁使用导致我们很难遇见，但确确实实存在着。所以开发过程中，一定要注意这类问题。   

知道问题的根源所在之后，对应的解决方案也就有啦。就是在 Activity 中创建 Fragment 实例时，添加一个判断即可，处理方式有三种：  

第一种方式，在 Activity 提供的 onAttachFragment() 方法中处理：  

```java    
@Override
public void onAttachFragment(Fragment fragment) {
    super.onAttachFragment(fragment);
    if (fragment instanceof  OneFragment){
        oneFragment = (OneFragment) fragment;
    }
}
```   

第二种方式，在创建 Fragment 前添加判断，判断是否已经存在：   

```java     
Fragment tempFragment = getSupportFragmentManager().findFragmentByTag("OneFragment");
if (tempFragment==null) {
    oneFragment = OneFragment.newInstance();
    ft.add(R.id.fl_content, oneFragment, "OneFragment");
}else {
    oneFragment = (OneFragment) tempFragment;
}
```  


第三种方式，更为简单，直接利用 savedInstanceState 判断即可：   

```java   
if (savedInstanceState==null) {
    oneFragment = OneFragment.newInstance();
    ft.add(R.id.fl_content, oneFragment, "OneFragment");
}else {
    oneFragment = (OneFragment) getSupportFragmentManager().findFragmentByTag("OneFragment");
}
```    


## onActivityResult()    
Fragment 类提供有 startActivityForResult() 方法用于 Activity 间的页面跳转和数据回传，其实内部也是调用 Activity 的对应方法。但是在页面返回时需要注意 Fragment 没有提供 setResult() 方法，可以通过宿主 Activity 实现。  

举个例子，在 ActivityA 中的 FragmentA 里面调用 startActivityForResult() 跳转至 ActivityB 中，并在 ActivityB 中的 FragmentB 里面返回到 ActivityA，返回代码如下：  

```java  
Intent intent = new Intent();
// putExtra
getActivity().setResult(Activity.RESULT_OK, intent);
getActivity().finish();  

```   

在回调时，先会回调 ActivityA 中的 onActivityResult() 方法，然后再分发回调 FragmentA 中的 onActivityResult() 方法，从 FragmentActivity 类的源码中可以看出：   

```java     
/**
* Dispatch incoming result to the correct fragment.
*/
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    mFragments.noteStateNotSaved();
    int requestIndex = requestCode>>16;
    if (requestIndex != 0) {
        requestIndex--;

        String who = mPendingFragmentActivityResults.get(requestIndex);
        mPendingFragmentActivityResults.remove(requestIndex);
        if (who == null) {
            Log.w(TAG, "Activity result delivered for unknown Fragment.");
            return;
        }
        Fragment targetFragment = mFragments.findFragmentByWho(who);
        if (targetFragment == null) {
            Log.w(TAG, "Activity result no fragment exists for who: " + who);
        } else {
            targetFragment.onActivityResult(requestCode & 0xffff, resultCode, data);
        }
        return;
    }

    super.onActivityResult(requestCode, resultCode, data);
}
```  

再拓展一下，如果 FragmentA 中又嵌入一层 FragmentAA ，然后从 FragmentAA 中跳转至 ActivityB，那么在 FragmentAA 中的 onActivityResult() 方法中能收到回调吗？显然不能。从上述源码中可以看出 FragmentActivity 只进行到一级分发。所以，如果想实现多级分发，就得自己在各级 Fragment 中手动添加分发代码，至下一级 Fragment 中。  

## 状态变迁监听   

Fragment 的 hide 和 show 等状态变迁操作都会反应在相应的回调函数中，我们可以利用这些监听函数做一些界面刷新等功能。较为常见的一个监听函数就是 onHiddenChanged() 方法，这个方法的变化直接影响着 isHidden() 方法的返回值。   

除了 isHidden() 方法，还有一个 isVisible() 方法，也用于判断 Fragment 的状态，表明 Fragment 是否对用户可见，如果为 true，必须满足三点条件：1，Fragment 已经被 add 至 Activity 中；2，视图内容已经被关联到 window 上；3. 没有被隐藏，即 isHidden() 为 false。这三点，从 isVisible() 源码中可以看出：   


```java   
/**
* Return true if the fragment is currently visible to the user.  This means
* it: (1) has been added, (2) has its view attached to the window, and 
* (3) is not hidden.
*/
final public boolean isVisible() {
    return isAdded() && !isHidden() && mView != null
        && mView.getWindowToken() != null && mView.getVisibility() == View.VISIBLE;
}
```      

>注意：onHiddenChanged() 方法可以监听 hide() 和 show() 操作，与 setUserVisibleHint() 方法有所不同，后者常见的出现场景是在 ViewPager 和 Fragment 组合的 FragmentPagerAdapter 中使用。ViewPager 滑动时便是通过这个方法改变 Fragment 的状态，利用这个方法可以实现 Fragment 懒加载，后续文章中再详细描述实现方式。 
