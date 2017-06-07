#简介
Nuwa热补丁是基于下文中**手Q热补丁轻量级方案**的具体实现，大致是通过“插桩”方式提前加载需要“打补丁”的类，以避免bug 类被加载。想要了解具体原理的可以参考本文参考。这里有两部分，**Nuwa实现了补丁类的加载动作,NuwaGradle 则实现了补丁类的生成。**这里我们重点关注如何生成补丁。
#预备知识
在制作补丁的时候，首先要对一个apk的生成过程有一个大致的了解。了解了如何生成apk，才能知道我们应该在哪一步去获取制作补丁的“原材料”。

![apk的生成过程](http://upload-images.jianshu.io/upload_images/22193-5cbeeee20568a6ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大致步骤如下：
1. 使用aapt生成R.java类文件 
2. 使用android SDK提供的aidl.exe把.aidl转成.java文件
3.  javac编译.java类文件生成class文件
4. 使用android SDK提供的dx.bat命令行脚本生成classes.dex文件
5. 使用Android SDK提供的aapt.exe生成资源包文件
6. apkbuilder  生成未签名的apk安装文件
7. 使用jdk的jarsigner对未签名的包进行apk签名
8. zipAlign 对齐

#思路
我们制作补丁时，**必须防止类被打上ISPREVERIFIED这个标记。**
原理**一个类直接引用到的类不在同一个dex中即可。**这样，就能防止类被打上ISPREVERIFIED标记并能进行热更新。

简单来说，就是将所有类的构造函数中，引用另一个hack.dex中的类，这个类叫Hack.class，然后在加载补丁patch.dex前动态加载这个hack.dex，但是有一个类的构造函数中不能引用Hack.class，这个类就是Application类的子类，一旦这个类的构造函数中加入Hack.class这个类，那么程序运行时就会找不到Hack.class这个类，因为还没有被加载。

生成补丁有几个需要注意的地方：
1.  寻找插入task的点
 在gradle1.5以下的版本中，我们可以找到dex task ，而gradle1.5以上，dex task 无法找到。
这里便用 transformClassesWithDexForXXX 这个task。在其之前执行生成改造类的工作。
没有开启Multidex的情况，存在一个preDex的Task。**preDex会在dex任务之前把所有的库工程和第三方jar包提前打成dex，下次运行只需重新dex被修改的库，以此节省时间。**dex任务会把preDex生成的dex文件和主工程中的class文件一起生成class.dex，这样就需要针对有无preDex，做不同的修改字节码策略即可。
2. 改造字节类，在类的构造函数中插入另一个dex中的类
对于java的.class的改造，我们一般会用到asm或者javasist 这两类工具，而NuwaGradle则采用了asm，和同事沟通，发现使用javasist时，有些类的无参构造函数可能无法找到。这里就贴下asm的具体实现：
```groovy
class NuwaProcessor {
    /**
     * 处理jar
     * @param hashFile
     * @param jarFile
     * @param patchDir
     * @param map
     * @param includePackage
     * @param excludeClass
     * @return
     */
    public static processJar(File hashFile, File jarFile, File patchDir, Map map, HashSet includePackage, HashSet excludeClass) {
        if (jarFile) {
            /**
             * classes.jar dex后的文件
             */
            def optJar = new File(jarFile.getParent(), jarFile.name + ".opt")
 
            def file = new JarFile(jarFile);
            Enumeration enumeration = file.entries();
            JarOutputStream jarOutputStream = new JarOutputStream(new FileOutputStream(optJar));
 
            /**
             * 枚举jar文件中的所有文件
             */
            while (enumeration.hasMoreElements()) {
                JarEntry jarEntry = (JarEntry) enumeration.nextElement();
                String entryName = jarEntry.getName();
                ZipEntry zipEntry = new ZipEntry(entryName);
 
                InputStream inputStream = file.getInputStream(jarEntry);
                jarOutputStream.putNextEntry(zipEntry);
                /**
                 * 以class结尾的文件并且在include中不在exclude中,并且不是cn/jiajixin/nuwa/包中的文件
                 */
                if (shouldProcessClassInJar(entryName, includePackage, excludeClass)) {
                    /**
                     * 构造函数中注入字节码
                     */
                    def bytes = referHackWhenInit(inputStream);
                    /**
                     * 写入子杰
                     */
                    jarOutputStream.write(bytes);
 
                    /**
                     * hash校验
                     */
                    def hash = DigestUtils.shaHex(bytes)
                    /**
                     * 加入hash值
                     */
                    hashFile.append(NuwaMapUtils.format(entryName, hash))
                    /**
                     * hash值与上一release版本不一样则拷到对应的目录,作为patch的类
                     */
                    if (NuwaMapUtils.notSame(map, entryName, hash)) {
                        NuwaFileUtils.copyBytesToFile(bytes, NuwaFileUtils.touchFile(patchDir, entryName))
                    }
                } else {
                    /**
                     * 否则直接输出文件不处理
                     */
                    jarOutputStream.write(IOUtils.toByteArray(inputStream));
                }
                jarOutputStream.closeEntry();
            }
            jarOutputStream.close();
            file.close();
            /**
             * 删除jar文件
             */
            if (jarFile.exists()) {
                jarFile.delete()
            }
            /**
             * dex后的文件重命名为jar文件
             */
            optJar.renameTo(jarFile)
        }
 
    }
 
    //refer hack class when object init
    private static byte[] referHackWhenInit(InputStream inputStream) {
        ClassReader cr = new ClassReader(inputStream);
        ClassWriter cw = new ClassWriter(cr, 0);
        ClassVisitor cv = new ClassVisitor(Opcodes.ASM4, cw) {
            @Override
            public MethodVisitor visitMethod(int access, String name, String desc,
                                             String signature, String[] exceptions) {
 
                MethodVisitor mv = super.visitMethod(access, name, desc, signature, exceptions);
                mv = new MethodVisitor(Opcodes.ASM4, mv) {
                    @Override
                    void visitInsn(int opcode) {
                        /**
                         * 如果是构造函数
                         */
                        if ("".equals(name) & opcode == Opcodes.RETURN) {
                            /**
                             * 注入代码
                             */
                            super.visitLdcInsn(Type.getType("Lcn/jiajixin/nuwa/Hack;"));
                        }
                        super.visitInsn(opcode);
                    }
                }
                return mv;
            }
 
        };
        cr.accept(cv, 0);
        return cw.toByteArray();
    }
 
    /**
     * 是否需要在preDex前处理
     * @param path
     * @return
     */
    public static boolean shouldProcessPreDexJar(String path) {
        return path.endsWith("classes.jar") & !path.contains("com.android.support") && !path.contains("/android/m2repository");
    }
 
    /**
     * jar中的文件是否需要处理
     * @param entryName
     * @param includePackage
     * @param excludeClass
     * @return
     */
    private static boolean shouldProcessClassInJar(String entryName, HashSet includePackage, HashSet excludeClass) {
        return entryName.endsWith(".class") & !entryName.startsWith("cn/jiajixin/nuwa/") && NuwaSetUtils.isIncluded(entryName, includePackage) && !NuwaSetUtils.isExcluded(entryName, excludeClass) && !entryName.contains("android/support/")
    }
 
    /**
     * 处理class
     * @param file
     * @return
     */
    public static byte[] processClass(File file) {
        def optClass = new File(file.getParent(), file.name + ".opt")
 
        FileInputStream inputStream = new FileInputStream(file);
        FileOutputStream outputStream = new FileOutputStream(optClass)
        /**
         * 对class注入字节码
         */
        def bytes = referHackWhenInit(inputStream);
        outputStream.write(bytes)
        inputStream.close()
        outputStream.close()
        if (file.exists()) {
            file.delete()
        }
        optClass.renameTo(file)
        return bytes
    }
}
```
3. 对于混淆时，应有mapping文件
我们一般发布的apk，都是混淆过的，所以可能需要对某些混淆的类来“打补丁”，这里就需要在执行progurad这个task的时候指定mapping文件，具体如下：
```groovy
  def proguardTask = project.tasks.findByName("proguard${variant.name.capitalize()}")
  if (oldNuwaDir) {
            def mappingFile = NuwaFileUtils.getVariantFile(oldNuwaDir, variant, MAPPING_TXT)
            NuwaAndroidUtils.applymapping(proguardTask, mappingFile)
   }
```
NuwaAndroidUtils中的具体实现：
 ```groovy
   //使用mapping文件做proguard
    public static applymapping(DefaultTask proguardTask, File mappingFile) {
        if (proguardTask) {
            if (mappingFile.exists()) {
                proguardTask.applymapping(mappingFile)
            } else {
                println "$mappingFile does not exist"
            }
        }
    }
```
4. 生成dex
```
/**
 * 对jar进行dex操作
 * @param project
 * @param classDir
 * @return
 */
public static dex(Project project, File classDir) {
    if (classDir.listFiles().size()) {
        def sdkDir
        /**
         * 获得sdk目录
         */
        Properties properties = new Properties()
        File localProps = project.rootProject.file("local.properties")
        if (localProps.exists()) {
            properties.load(localProps.newDataInputStream())
            sdkDir = properties.getProperty("sdk.dir")
        } else {
            sdkDir = System.getenv("ANDROID_HOME")
        }
        if (sdkDir) {
            /**
             * 如果是windows系统,加入后缀.bat
             */
            def cmdExt = Os.isFamily(Os.FAMILY_WINDOWS) ? '.bat' : ''
            def stdout = new ByteArrayOutputStream()
            /**
             * 拼接命令
             * dx --dex --output=patch.jar classDir
             * classDir是注入字节码后的补丁目录
             */
            project.exec {
                commandLine "${sdkDir}/build-tools/${project.android.buildToolsVersion}/dx${cmdExt}",
                        '--dex',
                        "--output=${new File(classDir.getParent(), PATCH_NAME).absolutePath}",
                        "${classDir.absolutePath}"
                standardOutput = stdout
            }
            def error = stdout.toString().trim()
            if (error) {
                println "dex error:" + error
            }
        } else {
            throw new InvalidUserDataException('$ANDROID_HOME is not defined')
        }
    }
}
```

#实现
**具体实现可以看[Android 热修复Nuwa的原理及Gradle插件源码解析](http://android.jobbole.com/82552/)**，写的很不错，我这里就不再做无用功了。
#参考
* [QFix探索之路—手Q热补丁轻量级方案](http://www.cnblogs.com/bugly/p/5969129.html)
* [Nuwa](https://github.com/jasonross/Nuwa)
* [NuwaGradle](https://github.com/jasonross/NuwaGradle)
* [Android系列之Android 命令行手动编译打包详解](http://www.cnblogs.com/jk1001/archive/2010/08/05/1793216.html)
* [Android 热修复Nuwa的原理及Gradle插件源码解析](http://android.jobbole.com/82552/)
