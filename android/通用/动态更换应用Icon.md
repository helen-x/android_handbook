# 动态更换应用Icon  

> 作者: 徐宜生 《Android群英传》《Android群英传:神兵利器》作者     
>博客: http://blog.csdn.net/eclipsexys

 
产品：我们可以动态更换App在Launcher里面的Icon吗  
开发：不可以  
产品：我们可以动态更换App在Launcher里面的Icon吗  
开发：不可以  
产品：我们可以动态更换App在Launcher里面的Icon吗  
开发：不可以  
产品：我们可以动态更换App在Launcher里面的Icon吗  
开发：让我想想……  

## 原理1——activity-alias

在AndroidMainifest中，有两个属性：

```java  
// 决定应用程序最先启动的Activity
android.intent.action.MAIN 
// 决定应用程序是否显示在程序列表里
android.intent.category.LAUNCHER  
``` 

另外，还有一个activity-alias属性，这个属性可以用于创建多个不同的入口，相信做过系统Setting和Launcher开发的开发者在系统的源码中应该见过很多。

## 原理2——PM.setComponentEnabledSetting

PackageManager是一个大统领类，可以管理所有的系统组件，当然，如果Root了，你还可以管理其它App的所有组件，一些系统优化工具就是通过这个方式来禁用一些后台Service的。

使用方式异常简单：

```java  
private void enableComponent(ComponentName componentName) {
    mPm.setComponentEnabledSetting(componentName,
            PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
            PackageManager.DONT_KILL_APP);
}

private void disableComponent(ComponentName componentName) {
    mPm.setComponentEnabledSetting(componentName,
            PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
            PackageManager.DONT_KILL_APP);
}
```  


根据  

```java  
PackageManager.COMPONENT_ENABLED_STATE_ENABLED    
PackageManager.COMPONENT_ENABLED_STATE_DISABLED   
```  
这两个标志量和对应的ComponentName，就可以控制一个组件的是否启用。

## 动态换Icon

有了上面的两个原理，来实现动态更换Icon就只剩下思路问题了。

首先，我们创建一个Activity，作为默认的入口并带着默认的图片，再创建一个双11的activity-alias，指向默认的Activity并带有双11的图片，再创建一个双12的activity-alias，指向默认的Activity并带有双12的图片……等等等。

```java  
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>

<activity-alias
    android:name=".Test11"
    android:enabled="false"
    android:icon="@drawable/s11"
    android:label="双11"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity-alias>

<activity-alias
    android:name=".Test12"
    android:enabled="false"
    android:icon="@drawable/s12"
    android:label="双12"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity-alias>

``` 

等等，这样有个问题，那就是这样会在Launcher上显示3个入口，所以，默认我们会把这些activity-alias先禁用，等到要用的时候再启用，养兵千日，用兵一时。  


```java
public class MainActivity extends AppCompatActivity {

    private ComponentName mDefault;
    private ComponentName mDouble11;
    private ComponentName mDouble12;
    private PackageManager mPm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDefault = getComponentName();
        mDouble11 = new ComponentName(
                getBaseContext(),
                "com.xys.changeicon.Test11");
        mDouble12 = new ComponentName(
                getBaseContext(),
                "com.xys.changeicon.Test12");
        mPm = getApplicationContext().getPackageManager();
    }

    public void changeIcon11(View view) {
        disableComponent(mDefault);
        disableComponent(mDouble12);
        enableComponent(mDouble11);
    }

    public void changeIcon12(View view) {
        disableComponent(mDefault);
        disableComponent(mDouble11);
        enableComponent(mDouble12);
    }

    private void enableComponent(ComponentName componentName) {
        mPm.setComponentEnabledSetting(componentName,
                PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
                PackageManager.DONT_KILL_APP);
    }

    private void disableComponent(ComponentName componentName) {
        mPm.setComponentEnabledSetting(componentName,
                PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
                PackageManager.DONT_KILL_APP);
    }
}
```  

OK了，禁用默认的Activity后，启用双11的activity-alias，结果不变还是指向了默认的Activity，但图标已经发生了改变。

根据ROM的不同，在禁用了组件之后，会等一会，Launcher会自动刷新图标。

效果参考下图。  
![](https://dn-mhke0kuv.qbox.me/741b9286e80ea75ba1a4.gif)  

