---
layout:     post
title:      "深入探究按键精灵（Android）的实现原理"
subtitle:   "逆向工程"
date:       2019-05-15 12:00:00
author:     "木木的木头"
header-img: "img/17.jpg"
catalog: true
tags:
    - Android
	- 逆向工程
---
## 深入探究按键精灵（Android）的实现原理

> 起因：本来想用按键精灵写点挂机脚本，但广告过于恶心，于是就想自己写一个模拟人工操作的app，但是写的时候发现各种权限问题，于是便好奇想探究一下按键精灵是如何做到Root后后台截屏并模拟手动点击的


##  Step 1

首先是在网上查了按键精灵的相关讨论，发现挺多人其实都在研究按键精灵的实现，但是大家都是从结果讨论的，基本上想办法做到能够模拟出点击就结束了，并没有证明按键精灵究竟是如何实现的。

## Step 2
根据网上相关的资料，大致有了几个未经验证的猜想。

猜想1：模拟点击实现都是使用Root权限模拟发送ADB的input命令。

猜想2：后台截图使用ADB的screencap命令。

猜想3：Android 5.0后使用MediaProjectionManager 的录屏接口进行截屏，但是这样的话会系统弹窗让用户确认授权。

## Step 3

动手实践，上述3种方案我都写代码进行了尝试，最终证明上面3种方案都能实现相关功能，但是猜想2有验证的性能问题，猜想3无法解决弹窗授权问题，而按键精灵明明没有弹窗，只需要授予Root权限即可。

screencap在java代码中并不能直接像命令行那样拿到图片的二进制数据返回，而只能将截图保存到闪存中，然后再从闪存读取图片数据。这样虽然能够实现，但是性能显然太差，而且伤硬件。

猜想3使用了Google官方的Demo测试，性能没问题，但是授权必须用户确认。

## Step 4

为了最终找到按键精灵的底层原理，我反编译了安卓版的按键精灵，一探究竟。工具：Luyten,Dex2Jar。另外，我刚开始读jar文件用的 jd-gui，但是部分文件发现读取失败，于是换了Luyten，Luyten效果好一点。反编译后全局搜索ScreenShot关键字，找到了一个ForScreenShotActivity类，这个就是按键精灵截图实现的Activity，很明显能够看到其中
```
 private void a() {
        if (Build$VERSION.SDK_INT >= 21) {
            this.startActivityForResult(((MediaProjectionManager)this.getSystemService("media_projection")).createScreenCaptureIntent(), 32896);
            return;
        }
        Log.e("ForScreenShotActivity", "The API version is too low,required is >= 21.");
        this.finish();
    }
    
    protected void onActivityResult(final int n, final int n2, final Intent intent) {
        super.onActivityResult(n, n2, intent);
        if (n != 32896) {
            return;
        }
        if (n2 == -1 && intent != null) {
            this.setResult(-1);
            ScreenShoterV3.getInstance().init((Context)this, intent);
            this.finish();
            return;
        }
        this.a();
        this.setResult(0);
    }
    
    protected void onCreate(final Bundle bundle) {
        super.onCreate(bundle);
        this.requestWindowFeature(1);
        this.getWindow().setBackgroundDrawable((Drawable)new ColorDrawable(0));
        this.getWindow().setDimAmount(0.0f);
        this.a();
    }

```

其中，"media_projection"刚好对于安卓的Context.MEDIA_PROJECTION_SERVICE常量值，而-1刚好对应Activity.RESULT_OK常量值。到此，如果是Android开发者肯定很清楚了，按键精灵的后台截图就是使用了录屏接口。不过为什么没有出现弹窗要求授权我猜测是他使用Root权限，直接修改了安卓的记录文件，相当于你点击了授权弹窗那里的“下次不再提醒”。














