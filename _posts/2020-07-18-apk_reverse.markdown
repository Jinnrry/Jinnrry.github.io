Android app 反编译与无源码断点调试

### 前置技能

1、熟悉Android开发

2、熟悉java编程语言，了解jvm运行原理，了解java字节码

3、熟练使用xposed、Magisk

4、熟练使用adb工具

### 工具准备

1、Android3.4(4.0之后的版本smalidea插件用不了) [下载地址](https://developer.android.com/studio/archive?hl=zh-cn)

2、Smalidea-0.05 [下载地址](https://bitbucket.org/JesusFreke/smali/downloads/)

3、Apktool [下载地址](https://ibotpeaches.github.io/Apktool/)

4、BDOpener [下载地址](https://github.com/riusksk/BDOpener)

5、jd-gui [下载地址](https://github.com/java-decompiler/jd-gui)

6、dex2jar [下载地址](https://github.com/pxb1988/dex2jar)


### 反编译

> 我这里是反编译拼多多，apk文件为pinduoduo.apk，拼多多app的包名为com.xunmeng.pinduoduo

1、将apk文件负责一个副本，命名为pinduoduo2.apk，然后将这个文件直接改名为pinduoduo2.zip，修改文件后缀以后就可以使用winrara等解压工具解压了，将压缩包解压后取出其中的class[x].dex文件，拼多多安装包里面一共有5个

2、使用dex2jar将dex文件反编译为jar包。执行命令为

```bash

sh d2j-dex2jar.sh classes.dex
sh d2j-dex2jar.sh classes2.dex
sh d2j-dex2jar.sh classes3.dex
sh d2j-dex2jar.sh classes4.dex
sh d2j-dex2jar.sh classes5.dex

```

执行以后就能得到5个jar包了，然后打开jd-gui工具，直接把jar包拖入窗口内就能看到java代码了。

![jd-gui](/img/articleImg/apk_1.png)

### 无源码调试

目前你已经能够看到app里面所有代码了，但是由于代码都是被混淆过的，想直接通过这些代码还原出以前的业务逻辑还是很难的，毕竟人脑编译能力有限。

因此需要进行单步调试，进一步分析程序业务逻辑。

当我们正向开发和调试的时候会在自己的设备上运行 debug 版本的安装包，而正式发版的时候是 release 版的安装包。一般而言，debug 版本的安装包是可以调试的，而 release 版本的安装包是无法调试的，区别在于打包出的安装包的 AndroidManifest.xml 文件的 application 节点下是否有 android:debuggable="true" 这个属性。所以我们想要无源码调试 release 应用(一般是别人的应用)的时候，第一步需要做的就是反编译别人的应用，添加或修改这个 debuggable 属性，然后打包、重新签名、安装。

但是，上述方法有个弊端，就是我们修改并重新签名后的应用较原应用签名发生了变化（因为我们重新签名时使用的是自己的签名），如果原应用在 Java 代码或者打包的 so 类库里校验了应用的签名，我们修改完的应用安装后很可能无法正常使用，更别提调试了。

但是，还有另一种方法，那就是让android不校验这个属性，方法就是把android系统的`default.prop ro.debuggable`属性设置为1，设置方法有很多，我这里是使用BDOpener修改的，修改方法很简单，这是一个xposed插件，在xposed里面启用就可以了。


#### 反编译smali文件

使用 apktool d pinduoduo.apk 反编译并生成 Smali 源码到当前目录。

反编译后，会生成pinduoduo文件夹

![pinduoduo_res](//img/articleImg/pinduoduo_res.png)

#### 构建Android Studio项目

进入 Android Studio 后，通过 File -> Open… 选择上面生成的源码目录，并切换到 Project 视图，右键工程主目录，选择 Mark Directory As -> Sources Root

![src_path](//img/articleImg/res_path.png)

设置完成后Android Stuido会开始索引文件，索引完成后就可以开始调试了。

#### 设置断点和调试

![bk_point](//img/articleImg/bk_point.png)

设置完断点以后在手机上启动拼多多应用，然后点击Android Studio右上角的Attach Debug按钮，选择拼多多应用，当程序执行到断点位置时程序即可停下。后续的调试方法就和平时普通开发中的调试过程一致了

![attach.png](//img/articleImg/attach.png)





参考文章：

https://dysaniazzz.github.io/2019/06/25/debug/

https://www.usmacd.com/2020/03/19/apk_debug/

https://www.usmacd.com/2020/03/19/apk_debug/






