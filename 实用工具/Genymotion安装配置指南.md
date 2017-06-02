#  Genymotion安装配置指南  

>作者: snowdream  
公众号: sn0wdr1am  
简书:http://www.jianshu.com/u/748f0f7e6432

##简介
Genymotion是一款基于x86架构的Android模拟器，由于系统启动速度，应用运行速度远远快于Android SDK自带模拟器而受到广泛应用。

## 优缺点
### 优点
>1. 系统启动速度快
2. 应用运行速度快
3. 跨平台   
4. IDE支持

## 缺点
>1. 与真机相比，无法支持一些硬件相关的传感器特性等  
2. 由于市场上大部分应用都是基于ARM架构来编译的，因此，与默认模拟器，真机相比，对于包含仅支持ARM架构的so的应用，默认不支持。   

注：基于x86架构的模拟器/真机，兼容ARM指令有两个解决方案：
>1. 对于x86真机，x86处理器已经能够基本兼容ARM指令了。参考《涨姿势！x86处理器兼容ARM架构App的秘密》
2. 对于Genymotion模拟器，则通过安装ARM_Translation_Android来进行兼容。   


## 安装Genymotion
### 安装步骤
1. 安装虚拟机VirtualBox      https://www.virtualbox.org/wiki/Downloads
2. 注册Genymotion帐号      https://www.genymotion.com/account/create/
3. 登录，下载并安装Genymotion   https://www.genymotion.com/download/

### 安装指南
详细安装步骤，请参考以下文章：
>1.  Installation https://docs.genymotion.com/Content/01_Get_Started/Installation.htm
2. Genymotion安装方法    http://www.jianshu.com/p/cb0e43e0996e

![[Genymotion效果图]](https://mmbiz.qlogo.cn/mmbiz_png/JPT2ofNCn4HxjTTHIJlw2ic4mDX4pfYkkGpELvxr7p1tFBPo15De2cicxxic5HicaEpUcDnoRXxJ4L0ED3aX30Fa0g/640?wx_fmt=png)

## 安装ARM_Translation_Android系列包
由于genymotion是基于x86的，而大部分应用都是基于ARM的，因此，我们需要安装一个ARM_Translation_Android系列包来增强兼容性。
### 安装步骤
1. 点击下载ARM_Translation_Android系列包
>1) Android 4.4及以下：  ARM Translation Installer v1.1     http://www.mirrorcreator.com/files/0ZIO8PME/Genymotion-ARM-Translation_v1.1.zip_links
2) Android 5.x： ARM_Translation_Lollipop    https://mega.nz/#!Mt8kyBxa!iVJYC7eI7ruLVoaarWIa85QOm_VlH53G0knVGpoSlAE
3) Android 6.x： ARM_Translation_Marshmallow      https://mega.nz/#!p4lFlCZR!TFsb8dMqNdAJjKoCDPDDvNtcQdEB0-KkVlTgQVwG20s

2. 将下载的zip包，拖进Genymotion模拟器窗口，按照提示安装
3. 安装成功后，重启Genymotion模拟器即可。

### 安装指南
1. Genymotion with Google Play Services    https://gist.github.com/wbroek/9321145
2. Use ARM Translation on 5.x image    http://23pin.logdown.com/posts/294446-genymotion-use-arm-translation-on-5x-image
3. Use ARM Translation on 6.x image    http://23pin.logdown.com/posts/691046-genymotion-use-arm-translation-on-6x-image

注：以上步骤，便可满足大部分的开发测试需求。以下的步骤，都是可选步骤。
下面是安装微信的效果
![[wechat]](https://mmbiz.qlogo.cn/mmbiz_png/JPT2ofNCn4HxjTTHIJlw2ic4mDX4pfYkkHZ1yvCCsUyetpNVb0p04tYf1QeYMNYqNnYJs7ovDeJqribp6TJWntZg/640?wx_fmt=png)

## 安装Google Apps
1. 根据平台，android版本等选择不同的安装包，下载。  
http://opengapps.org/   
https://github.com/opengapps/opengapps
2. 将下载的zip包，拖进Genymotion模拟器窗口，按照提示安装
3. 安装成功后，重启Genymotion模拟器即可。 
