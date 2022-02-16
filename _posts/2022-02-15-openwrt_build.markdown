---
layout:     post
title:      "OpenWRT定制化编译过程"
subtitle:   "OpenWRT"
date:       2022-02-15 12:00:00
author:     "Jinnrry"
header-img: "img/33.jpg"
catalog: true
tags:
    - OpenWRT
---
# 为啥要自己编译

1、路由器作为家里的网关核心，网络安全至关重要，网上随便下一个镜像刷进去实在是不放心。毕竟别人的镜像要设后门太容易了

2、自己家里有各种各样的需求，别人的镜像总有一些不满足的地方。如果别人都满足的话，那肯定又会多编译一堆没用的东西，因此只有自己编译的才是最合适的。

# 编译条件

OS: Ubuntu操作系统

安装依赖:

```
sudo apt-get update
sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync
```

# 编译过程


## 1、下载OpenWRT官方代码

`git clone https://www.github.com/openwrt/openwrt`



我选用的OpenWRT官方的源码，配置HelloWorld项目会麻烦一点，Lede的源码配置简单一些，但是我实际使用过程中发现Lede有一些Bug，还是官方代码好点

另外，也可以选用具体某个分支的代码，比如`openwrt-21.02`


## 2、更新&安装源

```
./scripts/feeds update -a 
./scripts/feeds install -a
```

## 3、添加HelloWorld项目源（科学上网插件）

编辑`feeds.conf.default`文件，在文件末尾加入

`src-git helloworld https://github.com/fw876/helloworld.git`

## 4、从[lede](https://github.com/coolsnowwolf/lede) 项目中拷出HelloWorld项目所需的依赖

```
mkdir -p package/helloworld
for i in "dns2socks" "microsocks" "ipt2socks" "pdnsd-alt" "redsocks2"; do \
  svn checkout "https://github.com/immortalwrt/packages/trunk/net/$i" "package/helloworld/$i"; \
done

svn checkout https://github.com/coolsnowwolf/lede/trunk/tools/ucl tools/ucl
svn checkout https://github.com/coolsnowwolf/lede/trunk/tools/upx tools/upx

sed -i 'N;24a\tools-y += ucl upx' tools/Makefile
sed -i 'N;40a\$(curdir)/upx/compile := $(curdir)/ucl/compile' tools/Makefile
```

## 5、再次更新&安装源

```
./scripts/feeds update -a 
./scripts/feeds install -a
```

## 6、配置镜像

`make menuconfig`

同配置Linux内核类似，几乎每一个设置都有三个选项:y / m / n，分别代表如下含义：
* `<*>` (按下`y`)这个包会被包含进固件镜像
* `<m>` (按下`m`)这个包会在生成刷新OpenWrt的镜像文件以后被编译，但是不会被包含进镜像文件
* `< >` (按下`n`)这个包不会被编译

下面是新3（newifi3 D2）的配置

```
# 选择处理器平台
Target System 
		MediaTek Ralink MIPS

# 选择处理器型号
Subtarget 
    MT7621 based boards

# 选择具体的设备
Target Profile
		Newifi D2
		
# 系统配置
Base System
    # 取消dnsmasq，使用dnsmasq-full （lua-ssr-plus必须使用dnsmasq-full，如果选择了dnsmasq将编译失败）
    < > dnsmasq
    <*> dnsmasq-full
# 内核配置
Kernel modules
    # 语言支持
    Native Language Support
        <*> kmod-nls-iso8859-1
        <*> kmod-nls-utf8

# 插件配置
LuCI
    Collections
        -*- luci
    # 管理界面中文支持
    Modules
        Translations
            <*> Chinese (zh_CN)
	# 这个模块一定得启用，不然很多Luci应用会报错`module 'luci.cbi' not found`
	<*> luci-compat
    Themes
        *- luci-theme-bootstrap
		Applications
				按需选择
		
```


## 7、编译

`make -j 9 V=s `

`-j 9`表示启用9个线程一起编译

`V=s`表示显示编译过程中的详细信息，如果出错了可以打印出详细的错误原因

# 最终输出

最终编译完成的镜像位于`bin/targets`中












