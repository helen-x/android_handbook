# Android渠道打包技术小结  

>作者:贼寇  
>简书:http://www.jianshu.com/u/b47c3eefb32f

## 导读
本文对比了渠道4种渠道打包方式:

| |是否需要重新编译|是否需要压缩/解压缩|是否需要重新签名|是否需要重新Zipalign|平均耗时|
|:----:|:----:|:----:|:----:|:----:|:----:|
|利用Gradle Product Favor打包|✔|✔|✔|✔|超过2分钟|
|替换Assets资源打包|❌|✔|✔|✔|约10秒|
|META-INF加入空文件|❌|✔|❌|不确定|约0.05秒|
|利用Zip的Comment打包|❌|❌|❌|❌|少于0.01秒| 


与iOS的单一渠道（AppStore）不同，Android平台在国内的渠道多入牛毛。以我们的App为例，就有27个普通渠道（应用宝，百度，360这种）和更多的推广专用渠道。我们打包技术也经过了若干次的改进。     

## 1.利用Gradle Product Favor打包
Product Favor是Gradle的自带的功能，配置很容易：  

```java
android {
    productFlavors {
        base {
            manifestPlaceholders = [ CHANNEL:”0"]
        }
        yingyongbao {
            manifestPlaceholders = [ CHANNEL:"1" ]
        }
        baidu {
            manifestPlaceholders = [ CHANNEL:"2"]
        }
    }
}
``` 

AndroidManifest.xml   

```xml  
<!-- 自用渠道号设置 -->
<meta-data
     android:name="CHANNEL"
     android:value="${CHANNEL}”/>
```

原理很简单，gradle编译的时候，会根据这个配置，把manifest里对应的metadata占位符替换成指定的值。然后Android这边在运行期再去取出来就是： 

```java   
public static String getChannel(Context context) {
    String channel = "";
    PackageManager pm = sContext.getPackageManager();
    try {
        ApplicationInfo ai = pm.getApplicationInfo(
            context.getPackageName(),
            PackageManager.GET_META_DATA);

        String value = ai.metaData.getString("CHANNEL");
        if (value != null) {
            channel = value;
        }
    } catch (Exception e) {
        // 忽略找不到包信息的异常
    }
    return channel;
}  
```

这个办法，缺点很明显，每打一个渠道包都会完整得执行一遍apk的编译打包流程，非常慢。近30个包要打一个多小时…优点就是不依赖其他工具，gradle自己就能搞定。  


## 2.替换Assets资源打包
assets用于存放一些资源。不同与res，assets里的资源编译时原样保留，不需要生成什么resouce id之类的东西。因此，我们可以通过替换assets里的文件打出不同的渠道包，而不用每次都重新编译。   

我们知道apk本质上就是个zip文件，那么我们就可以通过解压缩->替换文件->压缩的办法来搞定：   

这里给出一份Python3的实现  

```python  
# 解压缩
src_file_path = '原始apk文件路径'
extract_dir = '解压的目标目录路径'
os.makedirs(extract_dir, exist_ok=True)

os.system(UNZIP_PATH + ' -o -d %s %s' % (extract_dir, src_file_path))

# 删除签名信息
shutil.rmtree(os.path.join(extract_dir, 'META-INF'))

# 写入渠道文件assets/channel.conf
channel_file_path = os.path.join(extract_dir, 'assets', 'channel.conf')
with open(channel_file_path, mode='w') as f:
    f.write(channel)  # 写入渠道号写进去

os.chdir(extract_dir)

output_file_name = '输出文件名称'
output_file_path = '输出文件路径'
output_file_path_tmp = os.path.join(output_dir, output_file_name + '_tmp.apk')

# 压缩
os.system(ZIP_PATH + ' -r %s *' % output_file_path)
os.rename(output_file_path, output_file_path_tmp)

# 重新签名
# jarsigner -sigalg MD5withRSA -digestalg SHA1 -keystore your_keystore_path
# -storepass your_storepass -signedjar your_signed_apk, your_unsigned_apk, your_alias
signer_params = ' -verbose -sigalg MD5withRSA -digestalg SHA1' + \
                ' -keystore %s -storepass %s %s %s -sigFile CERT' % \
                (
                    sign, # 签名文件路径
                    store_pass, # 存储密码
                    output_file_path_tmp,
                    alias # 别名
                )

os.system(JAR_SIGNER_PATH + signer_params)

# Zip对齐
os.system(ZIP_ALIGN_PATH + ' -v 4 %s %s' % (output_file_path_tmp, output_file_path))
os.remove(output_file_path_tmp)
```

在这里，几个PATH分别表示zip、unzip、jarsigner和zipalign这几个可执行文件的路径。   

签名是apk的一个重要机制，它给apk里的每一个文件（META-INF目录下的除外）计算一个hash值，记录在META-INF下的若干文件里。Zip对齐能够优化运行时Android读取资源的效率，这一步虽然不是必须的，但还是推荐做一下。   

采用这个方法，我们不需要再编译Java代码，速度有极大地提升。大约每10秒就能打一个包。    

同时给出读取渠道号的实现代码：   

```java     
public static String getChannel(Context context) {
    String channel = "";
    InputStream is = null;
    try {
        is = context.getAssets().open("channel.conf");
        byte[] buffer = new byte[100];
        int l = is.read(buffer);

        channel = new String(buffer, 0, l);
    } catch (IOException e) {
        // 如果读不到，那么取缺省值
    } finally {
        if (is != null) {
            try {
                is.close();
            } catch (Exception ignored) {
            }
        }
    }
    return channel;
}   
```  

顺便说一下，还可以用aapt这个工具来替代zip&unzip来实现文件替换：      

```java     
# 替换assets/channel.conf
os.chdir(base_dir)
os.system(AAPT_PATH + ' remove %s assets/channel.conf' % output_file_path_tmp)
os.system(AAPT_PATH + ' add %s assets/channel.conf' % output_file_path_tmp)
```

## 3.美团给出的一种方案
刚才上文提到META-INF目录对签名机制是豁免的，往这里面放东西就可以免去重签名这一步，[美团技术团队](http://tech.meituan.com/mt-apk-packaging.html)就是这么做的。   

```java
import zipfile
zipped = zipfile.ZipFile(your_apk, 'a', zipfile.ZIP_DEFLATED)
empty_channel_file = "META-INF/mtchannel_{channel}".format(channel=your_channel)
zipped.write(your_empty_file, empty_channel_file)
```   

给META-INFO目录加入一个名为“mtchannel_渠道号”的空文件，在Java这边查找到这个文件，取得文件名即可：   

```java     
public static String getChannel(Context context) {
    ApplicationInfo appinfo = context.getApplicationInfo();
    String sourceDir = appinfo.sourceDir;
    String ret = "";
    ZipFile zipfile = null;
    try {
        zipfile = new ZipFile(sourceDir);
        Enumeration<?> entries = zipfile.entries();
        while (entries.hasMoreElements()) {
            ZipEntry entry = ((ZipEntry) entries.nextElement());
            String entryName = entry.getName();
            if (entryName.startsWith("mtchannel")) {
                ret = entryName;
                break;
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (zipfile != null) {
            try {
                zipfile.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    String[] split = ret.split("_");
    if (split != null && split.length >= 2) {
        return ret.substring(split[0].length() + 1);

    } else {
        return "";
    }
}
```

这个方法省去了重签名这一步，速度提升也很大。他们的描述是“900多个渠道不到一分钟就能打完”，也就是不到0.06s一个包。  

## 4.利用Zip文件comment的终极方案
另外给出了一个终极方案：我们知道Zip文件末尾有一块区域，可以用来存放文件的comment。改动这个区域，丝毫不会影响Zip文件的内容。  

打包的代码很简单：    

```java  
shutil.copyfile(src_file_path, output_file_path)

with zipfile.ZipFile(output_file_path, mode='a') as zipFile:
zipFile.comment = bytes(channel, encoding=‘utf8')

```
 

这个方法比前一个方法的区别在于，它不会修改Apk的内容，也就不必重新打包，速度又有提升！    
按文档中的说法，这个方法1s内可以打300多个包，也就是说单个包的时间小于10毫秒！  


读取的代码稍微复杂一些。    
Java 7的ZipFile类，有getComment方法，可以轻易地读取comment值。然而这个方法只在Android 4.4以及更高版本才可用，我们就需要多花点时间把这段逻辑移植过来。所幸这里的逻辑不复杂，我们查看源码，可以看到主要逻辑都在ZipFile的一个私有方法readCentralDir里，一小部分读取二进制数据的逻辑在libcore.io.HeapBufferIterator，全部搬过来，整理一下就搞定了：


```java    
public static String getChannel(Context context) {
   String packagePath = context.getPackageCodePath();

   RandomAccessFile raf = null;
   String channel = "";
   try {
      raf = new RandomAccessFile(packagePath, "r");
      channel = readChannel(raf);
   } catch (IOException e) {
      // ignore
   } finally {
      if (raf != null) {
         try {
            raf.close();
         } catch (IOException e) {
            // ignore
         }
      }
   }

   return channel;
}

private static final long LOCSIG = 0x4034b50;
private static final long ENDSIG = 0x6054b50;
private static final int ENDHDR = 22;

private static short peekShort(byte[] src, int offset) {
   return (short) ((src[offset + 1] << 8) | (src[offset] & 0xff));
}

private static String readChannel(RandomAccessFile raf) throws IOException {
   // Scan back, looking for the End Of Central Directory field. If the zip file doesn't
   // have an overall comment (unrelated to any per-entry comments), we'll hit the EOCD
   // on the first try.
   // No need to synchronize raf here -- we only do this when we first open the zip file.
   long scanOffset = raf.length() - ENDHDR;
   if (scanOffset < 0) {
      throw new ZipException("File too short to be a zip file: " + raf.length());
   }

   raf.seek(0);
   final int headerMagic = Integer.reverseBytes(raf.readInt());
   if (headerMagic == ENDSIG) {
      throw new ZipException("Empty zip archive not supported");
   }
   if (headerMagic != LOCSIG) {
      throw new ZipException("Not a zip archive");
   }

   long stopOffset = scanOffset - 65536;
   if (stopOffset < 0) {
      stopOffset = 0;
   }

   while (true) {
      raf.seek(scanOffset);
      if (Integer.reverseBytes(raf.readInt()) == ENDSIG) {
         break;
      }

      scanOffset--;
      if (scanOffset < stopOffset) {
         throw new ZipException("End Of Central Directory signature not found");
      }
   }

   // Read the End Of Central Directory. ENDHDR includes the signature bytes,
   // which we've already read.
   byte[] eocd = new byte[ENDHDR - 4];
   raf.readFully(eocd);

   // Pull out the information we need.
   int position = 0;
   int diskNumber = peekShort(eocd, position) & 0xffff;
   position += 2;
   int diskWithCentralDir = peekShort(eocd, position) & 0xffff;
   position += 2;
   int numEntries = peekShort(eocd, position) & 0xffff;
   position += 2;
   int totalNumEntries = peekShort(eocd, position) & 0xffff;
   position += 2;
   position += 4; // Ignore centralDirSize.
   // long centralDirOffset = ((long) peekInt(eocd, position)) & 0xffffffffL;
   position += 4;
   int commentLength = peekShort(eocd, position) & 0xffff;
   position += 2;

   if (numEntries != totalNumEntries || diskNumber != 0 || diskWithCentralDir != 0) {
      throw new ZipException("Spanned archives not supported");
   }

   String comment = "";
   if (commentLength > 0) {
      byte[] commentBytes = new byte[commentLength];
      raf.readFully(commentBytes);
      comment = new String(commentBytes, 0, commentBytes.length, Charset.forName("UTF-8"));
   }
   return comment;
}
```

>需要注意的是，Android 7.0加入了APK Signature Scheme v2技术。在Android Plugin for Gradle 2.2，这一技术是缺省启用的，这会导致第三、第四两种方法打出的包在Android 7.0下面校验失败。解决方法有二，一是把Gradle版本改低，二是在signingConfigs/release下面加上配置v2SigningEnabled false。详细说明见[谷歌的文档](https://developer.android.com/about/versions/nougat/android-7.0.html#apk_signature_v2)

## 总结
用表格说话

| |是否需要重新编译|是否需要压缩/解压缩|是否需要重新签名|是否需要重新Zipalign|平均耗时|
|:----:|:----:|:----:|:----:|:----:|:----:|
|利用Gradle Product Favor打包|✔|✔|✔|✔|超过2分钟|
|替换Assets资源打包|❌|✔|✔|✔|约10秒|
|META-INF加入空文件|❌|✔|❌|不确定|约0.05秒|
|利用Zip的Comment打包|❌|❌|❌|❌|少于0.01秒| 
