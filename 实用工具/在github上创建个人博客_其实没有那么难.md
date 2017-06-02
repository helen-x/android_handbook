# 在github上创建个人博客，其实没有那么难  


>作者:AriaLyy    
>简书: http://www.jianshu.com/p/c5dbd724f5ec    

## 写在前面
前段时间萌发自己搭建博客的念头，冲动之下买了个云服务器，奈何个人对html的东西实在不通，折腾了几天，blog依然丑的可以。后来无意间看见在github上可以搭建个人blog，就用谷歌折腾该如何在github上搭建blog，奈何网上很多教程都过于古老，或者很多细节都含糊不清，导致爬了好几天几天坑，才blog搭建了起来。

## 如何在github上搭建blog  

本文分为三个部分:  

- 本地环境搭建
- github 部署
- hexo 与github关联

需要的的原料 

- git
- github 账号
- node js
- hexo
- 个人域名(可选)

## 1. 本地环境搭建
- 安装git    

```java
https://www.git-scm.com/download/win 
```

选择你需要的版本，一路默认安装便可

- 安装node js     

32位下载地址
```java  
https://nodejs.org/dist/v4.2.3/node-v4.2.3-x86.msi  
```

64位下载地址  

```java  
https://nodejs.org/dist/v4.2.3/node-v4.2.3-x64.msi
```  

下载完成后双击安装，不做任何操作，一直默认下去便可。

- 安装完成后win+R调出CMD命令，输入node -v
，校验是否安装成功，如果出现以下信息，则表示安装成功
 
![](http://upload-images.jianshu.io/upload_images/4048192-91508d4bf47c37ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装完成node js后，在任意目录下打开cmd命令窗口，输入以下命令来安装hexo    

```java  
npm install -g hexo
```  

如果出现以下信息，则表示hexo安装成功


![](http://upload-images.jianshu.io/upload_images/4048192-40218cf0aa3b9577.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接着输入以下命令添加git依赖   
注意： 
1.  该命令必须在hexo g 之前完成
2. 部署这个命令一定要用git bash，否则会提示Permission denied   

```java  
 (publickey).npm install hexo-deployer-git --save  
```

如果出现以下信息，则表示hexo git依赖安装成功  

![](http://upload-images.jianshu.io/upload_images/4048192-8f3ae057a8923df5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 创建hexo工程     
在任意盘符中创建一个文件夹，把它命名为hexo，打开该文件夹，在该目录下打开cmd，输入以下命令    

```java  
hexo int  
```  

hexo 便自动创建了一个项目，目录结构如下:

![](http://upload-images.jianshu.io/upload_images/4048192-5078edecf08507f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在在cmd命令下输入以下命令    
```java  
hexo server -p 5001   
```  

如果出现以下信息，表示你已经生成了hexo工程。


![](http://upload-images.jianshu.io/upload_images/4048192-0bedf083cab0df48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在，是时候在浏览器中展示你的成果了，在浏览器地址栏中，输入以下地址，如果出现以下页面，该页面便是你以后的blog的预览效果。   
```java  
http://localhost:5001/  
```


![](http://upload-images.jianshu.io/upload_images/4048192-2c57da4bd9581227.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. Github部署
如果以上的网页能正常显示，那么恭喜你，本地环境搭建已经基本完成，现在是时候将hexo部署到github上了，将hexo部署你还需要完成几个步骤： 

- 创建一个github账号
- 创建一个新的软件仓库，名称格式必须为：github的用户名.github.io。如：我的github用户名为：AriaLyy，那么该软件仓库的名必须为:AriaLyy.github.io。最终软件仓库如下：   

![](http://upload-images.jianshu.io/upload_images/4048192-3bb60730e0890e59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在点击Settings，往下拉，如果你看见下图的信息 `Your site is published at http://AriaLyy.github.io/`


![](http://upload-images.jianshu.io/upload_images/4048192-6aa8c3a39a9a8804.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

只要出现可以点击的网络地址，那么表示github已经成功帮你自动生成一个静态网站了。

创建github ssk公钥在任意目录打开CMD，输入以下命令创建git ssh公钥，一路回车下去便可    

```java  
ssh-keygen -t rsa -C "key_name"  
```

接着讲ssh公钥添加到github中。ps:如果你碰到ssh-keygen不是内部命令，那么你需要将git安装目录下的`Git\usr\bin`
路径设置在环境变量中。

## 3. Hexo 和 Github关联
如果你已经完成了以上的步骤，那么你离成功只剩下一步之遥。还记得我们的hexo工程，以及*.github.io的软件仓吗？打开该文件，找到_config.yml文件，用文本编辑器打开，找到deploy
字段，添加以下信息     

```java
deploy:   
type: git    
repository:   git@github.com:AriaLyy/AriaLyy.github.io.git # 这个配置为你*.github.io软件仓库clone地址   
branch: master #分支为master分支  
```

这里有几点需要注意：
- 每个参数，冒号后面，必须要有一个空格！！！！！！
- repository 参数需要 ssh 的clone地址
- branch 分支需要是master分支

如果你已经配置好以上步骤，接下来在hexo文件夹下打开cmd，输入以下命令，将hexo工程部署到github上。    

```java  
hexo d -g  
```

**注意：部署这个命令一定要用git bash**   
如果一切顺利，那么将会出现类似于下面的成功提示信息



![](http://upload-images.jianshu.io/upload_images/4048192-e76cf2b0189cdcad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么在浏览器地址上输入:你的*.github.io软件仓库的名称，将会打开你的blog。


![](http://upload-images.jianshu.io/upload_images/4048192-230814e70840de19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果一切顺利，请尽情享受你的喜悦吧。
[这是我在github上创建的个人blog，点击我](http://arialyy.com/)
顺便给我的github开源项目Aria打下广告，[Aria，致力于让下载傻瓜化](https://github.com/AriaLyy/Aria) 

[Demo地址](https://github.com/helen-x/helen-x.github.io) 