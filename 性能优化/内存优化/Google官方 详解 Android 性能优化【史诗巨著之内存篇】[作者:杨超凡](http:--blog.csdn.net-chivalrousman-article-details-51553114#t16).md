# Google官方 详解 Android 性能优化【史诗巨著之内存篇】[作者:杨超凡](http://blog.csdn.net/chivalrousman/article/details/51553114#t16)
## 为什么关注性能
对于一款APP，用户首先关注的是 app的性能，而不是APP本身的属性功能，用户不关心你是否是搞社交，是否搞电商，是否是一款强大的美图滤镜app，用户首先关注的是 性能—-性能不好，用户会直接卸载，在应用市场给一个恶狠狠得差评，小则影响产品口碑，大则影响公司的品牌和声誉，作为程序员，app的性能更应该作为我们关注的一个功能，而不是出了问题 才去门头苦恼加班加点的负担。

**老实说，提高app性能的确非常难，处理这些问题 你必须知道：**

1.应用速度慢的原因
2.还必须正确使用分析工具来分析数据

**为此诞生了本文：**

1.了解这些工具并用它们找到造成性能不好的原因
2.从理论角度了解它们

## 如何优化app的性能？
**性能优化听上去是一项非常艰巨的功能，但仔细思考，其实非常简单：**
1.获取信息:
有人说你应用慢，应用闪退（崩掉，crash掉）的时候你需要找到原因。通过运行 分析和反馈工具软件来收集应用相关的信息，我们需要明确哪些可以测量，哪些可以优化也就是说任何应用开始优化时，整个过程取决于问题的可测性以及性能优化的可评价性。 开发中经常遇到的坎，问题不可复现，以及对于某一个细节是否需要优化 拿不定主意，这个需要我们自己身处其境 考虑分析各方面因素 得出结论，而不是纯粹得靠感觉。
2.分析数据:
很多时候，我们并不能直接理解问题的原因，比如内存溢出（OOM）的error，判断内存溢出需要计算很多个变量得内存大小，我们并不能直观通过眼球看出来一个app 运行过程那些变量的内存，这里我们就需要运行分析工具来帮助我们，将其转化为可视化的图表，在这里，我们可能还是看不懂那些图表，横线竖线，具体是个什么玩意，没有关系，去弄懂它们！就可以成为性能大师了.现在你看那些内存中的二进制转换成图表的过程，就类似于古代的算命大师，步骤1和步骤2 会不断的循环，搜集数据，分析数据···有时候我们不只使用一种搜集工具和分析工具，这就需要自己针对性能得种类来深入研究了.
3.Tack action!
发现了问题，找到了问题所在以及发生的原因，我们必须要恰当的去解决它，根据项目进度，该性能的优化成本，性能优先级，考虑项目中使用的<font color=#FF0000>Java</font>库或者<font color=#FF0000>Android</font>开源框架，其中的一些严格限制，在你的方案提出之前，这些因素都是我们需要考虑的，因为提出的优化方案，不一定会被公司高层接受（除非你就是高层）。

```工具不是规则，理解事物的规则和流程更重要```

## 为什么关注内存
```内存大小属于手机性能之一```

举个简单的例子，内存就像你的卧室一样，当你在老家住着动辄几百平的村庄，舒服惯了，突然变卖家产一门心思想创业来到北京，家里的老本只够你住几平米的卫生间的时候，你就会注意到内存【房间】大小的重要性了。
首先我们要知道内存是如何影响系统运行通常我们认为代码执行速度等同于物理硬件的执行速度，我们的代码指令都是通过使用内存来完成的。通过为实例对象，常量，变量分配内存，来完成操作，但是如何释放这些内存，通常我们并不清楚。

![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/2222.png)

一旦分配出去的内存没有及时回收，会引造成系统卡顿，执行操作缓慢现象，这种现象称之为内存泄漏，Memory leak

![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/12a89c01adce51b97a72f7d025e9a47372b58006/333.png)

### java垃圾回收机制官方详解
```java中的JVM就是一个抽象的计算机，和实际的计算机一样，它具有指令集并使用不同的存储区域，JVM负责执行代码，管理数据，管理内存和寄存器。```

垃圾回收机制只做两件基本的事情：

发现无用的对象
回收被无用对象占用的内存空间，使得该空间可以被程序再次利用
通常，垃圾回收具有如下特点：
1.垃圾回收机制的工作目标是回收无用对象的内存空间，这些内存空间都是JVM堆内存的内存资源，对于其他物理资源，比如<font color=#FF0000>数据库</font>连接，磁盘I/O 等资源无能为力
2.为了让垃圾回收机制尽快回收那些对象，可以将该对象引用变量置为null
3.<font color=#4169E1>垃圾回收发生的不可预知性</font>，不同JVM 采用不同的<font color=#FF0000>算法</font>和机制，有可能定时回收，有可能cpu空闲回收，也有可能内存消耗极限发生，即使通过Runtime对象的gc（），System.gc（）来建议系统进行回收，但这之属于建议，不能精确控制垃圾回收机制的执行,
```意思就是说，垃圾回收机制什么时候开始执行，并不是我们程序员能控制的，我们只能给予建议。```
**那么问题来了，如何精确的进行垃圾回收呢？**

回答很明确，确保每一个对象都进行了有效的释放。对于不再需要的对象，不要引用他们，一旦在别的地方保持对这个对象的引用，垃圾回收机制 暂时不会回收该对象，则会导致严重得问题—-<font color=#DA70D6>系统可用内存越来越少，垃圾回收执行的频率越来越高，cpu全都被垃圾回收的操作占有了，系统性能自然而然就下降了！</font>

java8 已经删除了永生代内存，即一些常驻内存，不会回收的数据，而是改为使用本地内存来存储类的元数据，称之为元空间（Metaspace），不过貌似和Android开发没关系(-__-)。

回顾完java垃圾回收，下面介绍
### Android 自己的回收机制-Runtime 回收机制
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/androidruntimeheap.png)

经由为数据分配内存的类型，以及系统如何有效的利用gc回收内存，并为新的对象分配内存。

所有要申请的内存都被划分到内存空间中，根据这些特点，哪些数据分配到哪些内存中，取决于Android的版本,

最为重要的一点，Android系统为每个运行中的app分配了预设的内存通常为16m-32m之间，当分配的内存越多，系统内存不足时，系统就可能会执行内存清理，注意，是可能会执行，是否执行垃圾清理是由系统自己判断的

进行垃圾回收，以确保有足够的内存分配给其他的应用操作，不同的Android版本，会有不同的gc操作，例如在davailk中，gc代表终止程序操作

![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/%E7%A9%BA%E9%97%B4%E9%A2%84%E8%AE%BE%E5%A4%A7%E5%B0%8F.png)
**代码是如何影响程序执行的？**
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc1.png)
上图是正常的界面刷新流程，
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc2.png)
上图，gc占据了一大块的时间，对于我们人类来说很短，但是对于系统来说很长了。
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc3.png)
综合三张图分析：代码质量很差，使得系统为我们的app分配了过多内存，而且没有及时回收，系统需要更多的时间去执行gc回收，那么系统就没时间去保持界面的活跃，所以就造成了卡顿的现象。
```在一个循环体中，重复得创建对象，就会造成内存污染，马上就会有很多gc 启动，由于这一额外的内存压力，内存泄漏仍然会产生，当可用内存降低到一定总量时，会强制系统gc执行，那循环体中的那部分操作会显示出卡顿的情况，甚至有可能在内存极限的时候，我们开发的应用会闪退。```
**所以，唯一的解决办法是：减少代码申请的内存量，不使用的对象及时回收。**
#### 使用内存分析工具Memory Monitor
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc4.png)
整个层叠图，代表还有多少内存可用 
深蓝色区域：代表正在使用的内存大小 
浅灰色区域：代表空闲未分配内存
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc5.png)
这个红色箭头所指的坡度表示急需大量的内存，内存分配也急剧的增加。 
上图是一个内存管理良好的例子

下图我们看一个内存糟糕的例子
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc6.png)
**分析:**
<font color=#FF0000>这里有一部分代码占用了大量的内存，然后又一下子释放了内存，生成不断重复又窄又长的曲线，这就是程序在花大量的时间在进行垃圾清理，运行垃圾清理的时间越多，其他操作可用的时间就越少，比如跟网络交互数据，页面刷新，打电话，听歌等等，这样就造成了卡顿.</font>
#### 使用Montior过程中遇到的 No Debugable Application的问题
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/montior.png)
```solution：Tools-Android-Android adb interact 最初并不会见效，重启app即可.```
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/montior2.png)
### 内存泄漏
在这里我才开始引入内存泄漏的原因【虽然文章前面已经提到，但是在这里才着重拿出来作为一节】

网络上和一些书籍对内存泄漏解释是 应该回收的对象没有回收，有点不全面，我认为深一点来说，内存泄漏是针对系统而言的，内存泄漏指的是不能被使用的内存，但是垃圾回收器无法识别出来，对其进行回收，这些对象一直存在于堆中，并且持续占据着内存空间，无法被删除，随着不断泄漏，系统可用的内存就越来越小，意味着系统又需要花更多的时间 去进行内存清理操作，进行垃圾回收操作的次数越来越多.
```简单的内存泄漏：对没有使用的对象 循环引用 复杂一点的：在listview还没有绘制完成时就添加到activity```
**Heap Viewer**
```Heap viewer使用步骤，我录制了gift图，详情请看：```
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/heapviewer.gif)
```了解Heap Viewer: Heap Viewer可以有效的分析程序在堆中分配的数据类型及数量和 大小小```
![](http://img.blog.csdn.net/20160601160253124)
```这里表示 byte数组和boolean数组的数量为177，占用了1.423M的内存```
### 内存泄漏的情况
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc7.png)

绿色箭头标出来的那部分 代码就是有问题的部分，原因在于，可以内存几乎为0，所有的内存已经被程序占用，首先记住，我们的代码有问题，造成了内存泄漏，并且，垃圾回收机制无法回收那部分内存空间

下图为30s之后的内存回收情况
![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc8.png)

启动第二次gc，此时Android调整并提高应用的内存上限，这样做的同时，如果漏洞没有修复，表明内存泄漏仍然存在，那么还会有第三次，第四次同样的gc操作，直至系统无法调整提高给应用更高的内存上限，造成内存溢出，甚至可能死机.
### Trace Viewer分配追踪器
![](http://img.blog.csdn.net/20160601164024041)
```Trace Viewer可以精确追踪到代码的位置，限于篇幅请按照上图点击 那几个按钮 自行摸索考功它的功能能```

## 高效加载图片图片
**为什么只关注图片加载，而不去处理其他数据来解决内存不足的问题？**

1.Android 加载图片会创建Bitmap,drawable实例，占用内存空间，如果不进行高效处理，程序会很快达到 Android系统分配给APP的内存上限，直至挂掉

2.图片资源相比文本资源，在内存中会占据更大的内存，从字节数就可以看出来

3.在我们的应用中正确恰当高效的加载 图片资源 是一件非常棘手的事情

1.Android 系统会分配给单个APP至少 16M左右的内存Android Compatibility Definition Document (CDD)中，根据不同手机的尺寸和屏幕像素来要求应用最小内存，我们开发 的应用需要优化内存至最低内存限制，然而请记住，许多手机对内存有着更高的要求。
2.图片消耗大量的内存，尤其是高像素的图片，比如入门级单反相机拍摄出来的一张图片，都有可能超出APP的最低内存限制
3.app 中一些常见的UI 比如 ListView, GridView and ViewPager，都需要立刻加载大量的图片，注意是立刻，这对内存管理提出了很高的要求。
**所以我们需要高效得加载图片。**
**高效的加载大图：**
一张图片的像素，尺寸，分别率，由可能超过Android的UI组件本身大小，UI组件的大小是由手机设备屏幕决定的，这些超出的部分，会消耗更多的内存。

应该让图片去匹配我们的手机设备，所以，我们需要对图片进行处理： 
1. 不能超过每个应用程序的内存限制 
2. 用最小的内存加载图片
**只需三步**
#### 1. 读取内存中Bitmap的尺寸和类型BitmapFactory类提供了很多解析Bitmap的方法（decodeByteArray(), decodeFile(), decodeResource(), etc.），每一种解析方法都有一个额外的参数BitmapFactory.Options，设置inJustDecodeBounds 属性为true可以禁止应用分配内存，此时bitmap返回为null，但是我们可以通过BitmapFactory.Options对象来获取很多有用的参数此时 你可以通过BitmapFactory.Options来读取图片的尺寸和类型
```BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
int imageHeight = options.outHeight;
int imageWidth = options.outWidth;
String imageType = options.outMimeType;
```

```总结：为了避免java.lang.OutOfMemory ，在加载图片之前需要检查原图的大小是否超出最低内存限制。```

#### 2.加载一张小图，使得系统分配较少的内存给它。
```现在我们已经获取到了图片的尺寸，加载一张图片之前，我们需要考虑：```

* 计算整张图片需要多大的内存
* 我们希望给它多大的内存
* 加载图片的组件比如Imageview的尺寸是多大
* 当前手机设备的屏幕尺寸和分辨率

例如一张1080* 720的图片要展示在一个128*72的Imageview上

实际 项目中，比如一张2048x1536的图片，我们通过设置inSampleSize为4，来创建实际大小为512x384的bitmap，这样需要的内存为0.75MB而不是之前的12MB（色彩模式都是ARGB_8888的情况下），

Google官方提供了两个方法来供我们使用，可以封装到自己的工具类中：

```public static int calculateInSampleSize( BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // 获得内存中图片的宽高
final int height = options.outHeight;
final int width = options.outWidth;
int inSampleSize = 1;
 if (height > reqHeight || width > reqWidth) {

final int halfHeight = height / 2;
final int halfWidth = width / 2;

 // 计算出一个数值，必须符合为2的幂（1，2，4，8，tec），赋值给inSampleSize
        // 图片宽高应大于期望的宽高的时候，才进行计算
        while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }
    return inSampleSize;
}
```

```
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // 第一次解析 inJustDecodeBounds=true 只是用来获取bitmap在内存中的尺寸和类型，系统并不会为其分配内存，
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);

    // 计算出一个数值
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // 根据inSampleSize 数值来解析bitmap
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}
```
`研究一个新的函数，我们先关注函数的输入和输出提高阅读能力`
#### 3.接着为我们的UI组件ImageView设置一张缩略图咯：**
```
mImageView.setImageBitmap(decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
```
**为了完全理解这一部分，初学者请自行查阅1.BitmapFactory.decode2.BitmapFactory.Options**

### 使AsyncTask加载Bitmap
当图片资源来自网络或者硬盘的时候，最好不要直接在主线程中加载它，例如IO资源或者数据库资源都会占用CPU，CPU 要做的事情过多，Android手机会造成卡顿得现象，

好在Google 提供了解决办法–AsyncTask异步加载工具

```
class BitmapWorkerTask extends 
AsyncTask<Integer, Void, Bitmap>{
    
    private final 
    WeakReference<ImageView> 
    imageViewReference;
    private int data = 0;
    public BitmapWorkerTask(ImageView imageView) {
        // 用弱引用确保能被垃圾回收机制回收
        imageViewReference = new WeakReference<ImageView>(imageView);
    }
    //在后台解析bitmap
    @Override
    protected Bitmap doInBackground(Integer... params) {
        data = params[0];
        return decodeSampledBitmapFromResource(getResources(), data, 100, 100));
    }
    // 一旦完成，imageView将会加载bitmap
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            if (imageView != null) {
    imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```
**接着我们在主线程中执行它即可**    
```public void loadBitmap(int resId, ImageView imageView) {
    BitmapWorkerTask task = new BitmapWorkerTask(imageView);
    task.execute(resId);
}
```
### 使用Lrucache缓存图片
#### 1. 为什么要缓存图片？
对于如何高效的加载一张图片 ，我们似乎已经得心应手了，这里要泼一盆凉水给大家，因为我们的应用不仅只是加载一张图片这么简单，比如ListView, GridView or ViewPager，RecyclerView，需要立刻加载出大量的bitmap，滑动的过程不断加载bitmap，还要求不卡顿，内存够用，这似乎又是一件棘手的事情。

Google 又提供了一种解决思路：对于ListView，RecyclerView，有可见的item和不可见的item，回收不可见的item 内存，分配给可见的，这样内存得到了重复利用，避免重复创建对象，不断申请并分配新的内存空间，触发最低内存限制的危险。

所以我们要管理 这些 已经创建好的内存。
#### 2. 使用内存缓存
1.为什么优先使用内存缓存？

答：相比硬盘缓存的读取速度，读取内存中的数据更快

2.Google官方有什么建议？

答：Google推荐Lrucache类，底层使用用强引用封装的LinkedHashMap，来存储最近使用的对象，它自动会回收最近使用的对象当中，使用的最少的那一个对象的内存，这一点毋庸置疑值得推荐！
**一种过时的做法，是用虚引用或者弱引用来标记bitmap，这种方法在Android 3.0以后已经不提倡了，因为类似JDK1.8那样，bitmap的内存是放在本地内存中的，它的回收是不确定的，有可能导致APP挂掉，切勿使用。**
1.Lrucache这么棒，我们该如何用？

Lrucache就可以当作一种存储数据的结构，类似list，set，可以存储对象，获取对象，对应的有add（） 和get（）方法，它与数组一样，初始化的时候需要指定一个初始的大小。

那么Lrucache实例 初始的大小该如何确定？ 
在计算大小之前，我们需要明确几件事情：

* 我们的Activity和Application还有多少可用内存？
* 第一次加载的时候，需要为多少图片分配内存？可见的item数量，决定了图片的数量，图片的数量*每张图片的内存大小就是初始化需要分配的内存。
* 用户当前手机的屏幕尺寸和屏幕密度（为什么要这俩参数？大屏幕手机 ，初始化的时候会加载更多的item，需要的内存更大）
* 这张图片我们要怎样配置？图片的尺寸如何设置，颜色模式如何配置？根据不同的需求，比如是做用户头像，还是信息展示？都有各自的应用场景的要求。我们需要分类判断.
* cache大小是由内存大小决定的，而不是 它存储数据的个数决定

```这让我想起一个在项目开发中常见的bug，use a bitmap which has bean recycler```

#### 3. Lrucache使用实例
```private LruCache<String, Bitmap> mMemoryCache;
@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // 当Lrucache使用的内存大小超过虚拟机最大的可用内存时候，Android会抛出OutOfMemory exception
    
    final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);

    // 使用虚拟机可用1/8
        final int cacheSize = maxMemory / 8;

    //  在Lrucache构造函数中初始化它的大小
    // int in its constructor
    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {

     //cache大小是由内存大小决定的，而不是 它存储数据的个数决定
         return bitmap.getByteCount() / 1024;
        }
   public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
        }
   }
 public Bitmap getBitmapFromMemCache(String key) {
    return mMemoryCache.get(key);
}
```  


**loadBitmap的过程很简洁：如果内存缓存中有这张bitmap，则直接刷新imageview，如果bitmap为空，则启动后台线程去加载bitmap，接着刷新imageview**

```public void loadBitmap(int resId, ImageView imageView) {
    final String imageKey = String.valueOf(resId);

    final Bitmap bitmap = getBitmapFromMemCache(imageKey);
    if (bitmap != null) {
        mImageView.setImageBitmap(bitmap);
    } else {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }
}  
```
**异步线程 BitmapWorkerTask**

```class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final Bitmap bitmap = decodeSampledBitmapFromResource(
                getResources(), params[0], 100, 100));
        addBitmapToMemoryCache(String.valueOf(params[0]), bitmap);
        return bitmap;
    }
    ...
}
```
#### 4. 磁盘缓存
与内存缓存相结合的还有磁盘缓存，虽然磁盘读取速度较慢，但是持久存储的，不像内存缓存那样，在内存极限情况下仍然会被清理，比如后台正在执行数据加载，突然打进来一个电话，内存不足系统可能会进行垃圾回收。缓存就没有了

Google官方提供直接DiskLruCache 类

为什么要使用DiskLruCache 很明了：就是解决当内存缓存不可用的情形

**内当需要频繁访问缓存的图片资源时，比如APP的画廊功能，可以考虑使用ContentProvider解决更为妥当。**

```private DiskLruCache mDiskLruCache;
private final Object mDiskCacheLock = new Object();
private boolean mDiskCacheStarting = true;
private static final int DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB
private static final String DISK_CACHE_SUBDIR = "thumbnails";

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Initialize memory cache
    ...
    // Initialize disk cache on background thread
    File cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR);
    new InitDiskCacheTask().execute(cacheDir);
    ...
}

class InitDiskCacheTask extends AsyncTask<File, Void, Void> {
    @Override
    protected Void doInBackground(File... params) {
        synchronized (mDiskCacheLock) {
            File cacheDir = params[0];
            mDiskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE);
            mDiskCacheStarting = false; // Finished initialization
            mDiskCacheLock.notifyAll(); // Wake any waiting threads
        }
        return null;
    }
}

class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final String imageKey = String.valueOf(params[0]);

        // Check disk cache in background thread
        Bitmap bitmap = getBitmapFromDiskCache(imageKey);

        if (bitmap == null) { // Not found in disk cache
            // Process as normal
            final Bitmap bitmap = decodeSampledBitmapFromResource(
                    getResources(), params[0], 100, 100));
        }

        // Add final bitmap to caches
        addBitmapToCache(imageKey, bitmap);

        return bitmap;
    }
    ...
}

public void addBitmapToCache(String key, Bitmap bitmap) {
    // Add to memory cache as before
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }

    // Also add to disk cache
    synchronized (mDiskCacheLock) {
        if (mDiskLruCache != null && mDiskLruCache.get(key) == null) {
            mDiskLruCache.put(key, bitmap);
        }
    }
}

public Bitmap getBitmapFromDiskCache(String key) {
    synchronized (mDiskCacheLock) {
        // Wait while disk cache is started from background thread
        while (mDiskCacheStarting) {
            try {
                mDiskCacheLock.wait();
            } catch (InterruptedException e) {}
        }
        if (mDiskLruCache != null) {
            return mDiskLruCache.get(key);
        }
    }
    return null;
}

// Creates a unique subdirectory of the designated app cache directory. Tries to use external
// but if not mounted, falls back on internal storage.
public static File getDiskCacheDir(Context context, String uniqueName) {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir
    // otherwise use internal cache dir
    final String cachePath =
            Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                    !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                            context.getCacheDir().getPath();

    return new File(cachePath + File.separator + uniqueName);
}
```
#### 5.处理运行时变更的缓存 
当屏幕旋转，或者其他原因导致Activity restart，这个时候难道又让我们重新创建大量的图像资源？ 
回答是否定的，Google提供了一种解决方案：

通过在Activity中使用fragment，构造Fragment时，通过设置 setRetainInstance(true))来设置缓存， 
话不多说，直接上代码：

```private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    RetainFragment retainFragment =
            RetainFragment.findOrCreateRetainFragment(getFragmentManager());
    mMemoryCache = retainFragment.mRetainedCache;
    if (mMemoryCache == null) {
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            ... // Initialize cache here as usual
        }
        retainFragment.mRetainedCache = mMemoryCache;
    }
    ...
}

class RetainFragment extends Fragment {
    private static final String TAG = "RetainFragment";
    public LruCache<String, Bitmap> mRetainedCache;

    public RetainFragment() {}

    public static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {
        RetainFragment fragment = (RetainFragment) fm.findFragmentByTag(TAG);
        if (fragment == null) {
            fragment = new RetainFragment();
            fm.beginTransaction().add(fragment, TAG).commit();
        }
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }
}
```

#### 补充-UI组件并发性问题
**这一部分涉及到UI细节，例如ListView，Viewpager的优化**

ViewPager是一个非常棒的组件，通过使用ViewPager和PagerAdapter可以实现类似新闻标题的滑动条以及常见的画廊功能。

谈到优化，Google 建议使用 PagerAdapter的子类FragmentStatePagerAdapter ，它可以在后台自动销毁和保存ViewPager中使用的Fragment的数据，通过使用它可以节省内存开销。
`Note：当然，如果我们可以使用PagerAdapter来完成少许图片资源的展示，毕竟我们的优化是为了节省内存开销。`
下面实例展示了带有ImageView的ViewPager，在主activity持有ViewPager和它的适配器。

```public class ImageDetailActivity extends FragmentActivity {
    public static final String EXTRA_IMAGE = "extra_image";

    private ImagePagerAdapter mAdapter;
    private ViewPager mPager;

    // A static dataset to back the ViewPager adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.image_detail_pager); // Contains just a ViewPager

        mAdapter = new ImagePagerAdapter(getSupportFragmentManager(), imageResIds.length);
        mPager = (ViewPager) findViewById(R.id.pager);
        mPager.setAdapter(mAdapter);
    }

    public static class ImagePagerAdapter extends FragmentStatePagerAdapter {
        private final int mSize;

        public ImagePagerAdapter(FragmentManager fm, int size) {
            super(fm);
            mSize = size;
        }

        @Override
        public int getCount() {
            return mSize;
        }

        @Override
        public Fragment getItem(int position) {
            return ImageDetailFragment.newInstance(position);
        }
    }
}
```

在适配器的getItem中使用了自定义的Fragment，它持有ImageView，并负责完成展示

**看上去已经大功告成 了，但事实真的如此吗？**

下面是ImageDetailFragment的细节，通过ImageDetailFragment，看上去已经大功告成 了，但事实真的如此吗？

```public class ImageDetailFragment extends Fragment {
    private static final String IMAGE_DATA_EXTRA = "resId";
    private int mImageNum;
    private ImageView mImageView;

    static ImageDetailFragment newInstance(int imageNum) {
        final ImageDetailFragment f = new ImageDetailFragment();
        final Bundle args = new Bundle();
        args.putInt(IMAGE_DATA_EXTRA, imageNum);
        f.setArguments(args);
        return f;
    }

    // Empty constructor, required as per Fragment docs
    public ImageDetailFragment() {}

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mImageNum = getArguments() != null ? getArguments().getInt(IMAGE_DATA_EXTRA) : -1;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        // image_detail_fragment.xml contains just an ImageView
        final View v = inflater.inflate(R.layout.image_detail_fragment, container, false);
        mImageView = (ImageView) v.findViewById(R.id.imageView);
        return v;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        final int resId = ImageDetailActivity.imageResIds[mImageNum];
        mImageView.setImageResource(resId); //为ImageView设置图片
    }
}
```

**阅读一段代码，除了学习外，还要思考它有什么缺点，如果是我，我该如何提高。**

希望你能注意到这些问题，上述代码是在UI线程中加载的，可能会导致我们开发的应用长时间无响应而被迫挂掉。所以，请使用AsyncTask开启子线程来加载图片：

`在阅读这段代码之前，请首先阅读上一篇博文 内存优化中的AsyncTsak部分`

```public class ImageDetailActivity extends FragmentActivity {
    ...

    public void loadBitmap(int resId, ImageView imageView) {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }

    ... // include BitmapWorkerTask class，
}

public class ImageDetailFragment extends Fragment {
    ...

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        if (ImageDetailActivity.class.isInstance(getActivity())) {
            final int resId = ImageDetailActivity.imageResIds[mImageNum];
            // 使用子线程来加载图片
            ((ImageDetailActivity) getActivity()).loadBitmap(resId, mImageView);
        }
    }
}
```

**请至少阅读完上一篇内存优化的LruCache部分才能保证你理解这里**

```public class ImageDetailActivity extends FragmentActivity {
    ...
    private LruCache<String, Bitmap> mMemoryCache;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        ...
        // initialize LruCache
    }

    public void loadBitmap(int resId, ImageView imageView) {
        final String imageKey = String.valueOf(resId);

        final Bitmap bitmap = mMemoryCache.get(imageKey);
        if (bitmap != null) {
            mImageView.setImageBitmap(bitmap);
        } else {
            mImageView.setImageResource(R.drawable.image_placeholder);
            BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
            task.execute(resId);
        }
    }

    ... // include updated BitmapWorkerTask 
}
```
`注意，只有阅读完上一篇内存优化中的Lrucache部分和AsyncTask部分才能保证你完全读懂上述代码`

#### RecyclerView GridView ListView
下面可能是大家经常的做法：

```
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    private ImageAdapter mAdapter;

    // A static dataset to back the GridView adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};

    // Empty constructor as per Fragment docs
    public ImageGridFragment() {}

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mAdapter = new ImageAdapter(getActivity());
    }

    @Override
    public View onCreateView(
            LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        final View v = inflater.inflate(R.layout.image_grid_fragment, container, false);
        final GridView mGridView = (GridView) v.findViewById(R.id.gridView);
        mGridView.setAdapter(mAdapter);
        mGridView.setOnItemClickListener(this);
        return v;
    }

    @Override
    public void onItemClick(AdapterView<?> parent, View v, int position, long id) {
        final Intent i = new Intent(getActivity(), ImageDetailActivity.class);
        i.putExtra(ImageDetailActivity.EXTRA_IMAGE, position);
        startActivity(i);
    }

    private class ImageAdapter extends BaseAdapter {
        private final Context mContext;

        public ImageAdapter(Context context) {
            super();
            mContext = context;
        }

        @Override
        public int getCount() {
            return imageResIds.length;
        }

        @Override
        public Object getItem(int position) {
            return imageResIds[position];
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ImageView imageView;
            if (convertView == null) { // if it's not recycled, initialize some attributes
                imageView = new ImageView(mContext);
                imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
                imageView.setLayoutParams(new GridView.LayoutParams(
                        LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
            } else {
                imageView = (ImageView) convertView;
            }
            imageView.setImageResource(imageResIds[position]); // Load image into ImageView
            return imageView;
        }
    }
}
```
看上去非常棒，对吗？但是我们如何才能做的更好？

如果你能发现主线程加载图片的问题，恭喜你，有一点点进步。但是重复提醒你使用AsyncTask，显然不是我写这一节的目的。

我们还需要警惕GridView 中的并发问题，因为iGridView 会回收子View的内存空间。
#### 并发问题
像ListView和GridView，与RecyclerView使用AsyncTask的时候，不能忽视一个问题：Android系统为了高效分配内存，这些组件都会在上下滑动的时候回收子view的内存。在滑动的时候，并不能保证AsyncTask可以完成当前任务。此外，也不能保证异步任务可以按照顺序完成。

Google在 Multithreading for Performance 提供了建议：在AsyncTask中，用弱引用来存储ImageView，通过使用弱引用来被检查ImageView是否加载完成

好吧，理论上是这样，我们开始行动：
#####1. 自定义Drawable去存储异步任务重的弱引用，只有这样，才能保证task执行完毕之后，ImageView才会显示图片

```
static class AsyncDrawable extends BitmapDrawable {
    private final WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;

    public AsyncDrawable(Resources res, Bitmap bitmap,
            BitmapWorkerTask bitmapWorkerTask) {
        super(res, bitmap);
        bitmapWorkerTaskReference =
            new WeakReference<BitmapWorkerTask>(bitmapWorkerTask);
    }

    public BitmapWorkerTask getBitmapWorkerTask() {
        return bitmapWorkerTaskReference.get();
    }
}
```
##### 2. 在执行BitmapWorkerTask之前，你需要创建 AsyncDrawable 并且绑定到ImageView上，通过如下代码：

```
public void loadBitmap(int resId, ImageView imageView) {
    if (cancelPotentialWork(resId, imageView)) {
        final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
        final AsyncDrawable asyncDrawable =
                new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
        imageView.setImageDrawable(asyncDrawable);
        task.execute(resId);
    }
}
```


##### 3. 仅仅这些代码还是不够的，Google 还引入 cancelPotentialWork（）来检查是否有其他的task任务在使用当前ImageView，如果有，就会取消当前任务，取消这个做法是不是很让人眼前一亮呢！

这是cancelPotentialWork（）的实现

```
public static boolean cancelPotentialWork(int data, ImageView imageView) {
    final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);

    if (bitmapWorkerTask != null) {
        final int bitmapData = bitmapWorkerTask.data;
        // If bitmapData is not yet set or it differs from the new data
        if (bitmapData == 0 || bitmapData != data) {
            // Cancel previous task
            bitmapWorkerTask.cancel(true);
        } else {
            // The same work is already in progress
            return false;
        }
    }
    // No task associated with the ImageView, or an existing task was cancelled
    return true;
}
```
##### 4. 我们还需要get方法获得相关的ImageView
```
private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
   if (imageView != null) {
       final Drawable drawable = imageView.getDrawable();
       if (drawable instanceof AsyncDrawable) {
           final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
           return asyncDrawable.getBitmapWorkerTask();
       }
    }
    return null;
}
```

##### 5. 最后就是更新task中的onPostExecute（）检查任务是否取消

```
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...

    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (isCancelled()) {
            bitmap = null;
        }

        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            final BitmapWorkerTask bitmapWorkerTask =
                    getBitmapWorkerTask(imageView);
            if (this == bitmapWorkerTask && imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```
至此，我们已经了解Google 对于并发加载的解决方案，只需要在getView（）实现它们就可以啦！

#### 在GridView解决并发

```
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    ...

    private class ImageAdapter extends BaseAdapter {
        ...

        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ...
            loadBitmap(imageResIds[position], imageView)
            return imageView;
        }
    }

    public void loadBitmap(int resId, ImageView imageView) {
        if (cancelPotentialWork(resId, imageView)) {
            final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
            final AsyncDrawable asyncDrawable =
                    new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
            imageView.setImageDrawable(asyncDrawable);
            task.execute(resId);
        }
    }

    static class AsyncDrawable extends BitmapDrawable {
        private final WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;

        public AsyncDrawable(Resources res, Bitmap bitmap,
                BitmapWorkerTask bitmapWorkerTask) {
            super(res, bitmap);
            bitmapWorkerTaskReference =
                new WeakReference<BitmapWorkerTask>(bitmapWorkerTask);
        }

        public BitmapWorkerTask getBitmapWorkerTask() {
            return bitmapWorkerTaskReference.get();
        }
    }

    public static boolean cancelPotentialWork(int data, ImageView imageView) {
        final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);

        if (bitmapWorkerTask != null) {
            final int bitmapData = bitmapWorkerTask.data;
            if (bitmapData != data) {
                // Cancel previous task
                bitmapWorkerTask.cancel(true);
            } else {
                // The same work is already in progress
                return false;
            }
        }
        // No task associated with the ImageView, or an existing task was cancelled
        return true;
    }

    private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
       if (imageView != null) {
           final Drawable drawable = imageView.getDrawable();
           if (drawable instanceof AsyncDrawable) {
               final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
               return asyncDrawable.getBitmapWorkerTask();
           }
        }
        return null;
    }

    ... // include updated BitmapWorkerTask class
```
`这样解决并发的办法，同理适用于ListView， Recycler 
这也是热门加载框架ImageLoader，Volley 的加载原理。原理还是Google提出的！`

这样就可以流畅的展示图片。


#### View Holder设计模式

我们的代码可能会调用findViewById（），尤其是当滑动ListView，RecyclerView，GridView的时候，会使得app性能变得糟糕。甚至Adapter会返回一个已经被Android系统回收的View，可是你仍然需要初始化加载这个view并刷新它。

Google提出了使用 “View Holder” 设计模式

##### 1. 首先创建 ViewHolder类，存储子View内部所有需要展示的布局

```
static class ViewHolder {
  TextView text;
  TextView timestamp;
  ImageView icon;
  ProgressBar progress;
  int position;
}
```

##### 2. 接着填充ViewHolder并且存储到布局中

```
ViewHolder holder = new ViewHolder();
holder.icon = (ImageView) convertView.findViewById(R.id.listitem_image);
holder.text = (TextView) convertView.findViewById(R.id.listitem_text);
holder.timestamp = (TextView) convertView.findViewById(R.id.listitem_timestamp);
holder.progress = (ProgressBar) convertView.findViewById(R.id.progress_spinner);
convertView.setTag(holder);
```

```
‘view holder’ 设计模式同样也适用于ListView，RecyclerView，GridView
```


**有趣的是**

当我通宵赶完这篇博客的时候，我的计算机也提醒我内存不足了 (●’◡’●)

![](https://raw.githubusercontent.com/ShaunSheep/BlogGifRes/master/gc9.png)

### 写本文所参考的：
[1.Markdown字体颜色写法](http://blog.csdn.net/testcs_dn/article/details/45719357)

[2.Google 官方 Android Performance 课程]()

[3.Google 官方 Android Diplaying Bitmaps Efficiently 课程](https://developer.android.com/training/building-graphics.html)

[4.Managing Your App’s Memory](https://developer.android.com/training/articles/memory.html)

[5.Google 官方 Processing Bitmaps Off the UI Thread课程](https://developer.android.com/training/displaying-bitmaps/process-bitmap.html#concurrency)

[6.Multithreading for Performance](https://developer.android.com/training/displaying-bitmaps/process-bitmap.html#concurrency)


