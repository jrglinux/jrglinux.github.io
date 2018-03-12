---
layout: post
category: linux
title: Add re-pppoe to Openwrt
---
**主要思路：**
1. 了解openwrt的目录框架结构，往其中添加所需软件功能xxx模块
2. 编写package/xxx/目录下的Makefile
3. 生成package/xxx/src/目录下Makefile（与GNU Make语法相同）
4. 编译生成img文件，烧写到路由器，配置pppoe-server-options


**openwrt目录框架**
![openwrt目录](http://upload-images.jianshu.io/upload_images/5737261-7a86df56b858742c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**图中第一行为原始目录，第二行为编译后多生成的目录**
**tools**：存放一些Makefile。编译固件时需要工具，这些Makefile则定义如何获得这些工具的源码包以及如何编译/安装这些命令工具
**toolchain**：包含一些Makefile，这些Makefile定义了如何获得kernel headers、C library、bin-utils、compiler、debugger以及如何编译它们。这些源码用来制作交叉编译工具，如果需要定义一个新的系统架构，需要在这里添加一些配置
**include**：包含一些公用的Makefile，会被其他Makefile所包含
**scripts**：存放一些用于openwrt软件包管理的perl脚本
**target**：各个硬件平台在这里定义了编译固件、内核的步骤和方法
**package**：包含针对各个软件包的Makefile以及补丁，这些Makefile定义了如何获得和编译安装它们

**bin**：存放编译完成后生成的firmware及ipk软件包
**build_dir**：所有软件包都解压到该目录，并在该目录下编译
**staging_dir**：最终安装的目录，tool、cross-compilation-toolchain、rootfs都在这里
**dl**：存放从网上下的用户空间软件包的源代码
**feeds**：openwrt环境所需要的软件包套件 

以r7800-beta2-buildroot.git中的led-control模块为例学习
r7800-beta2-buildroot.git/package/led-control/Makefile:
`include $(TOPDIR)/rules.mk`

`PKG_NAME :=led-control`

`PKG_NAME :=led-control`
`PKG_RELEASE :=1`

`PKG_NAME :=led-control`
`PKG_RELEASE :=1`
`PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)`

`include $(INCLUDE_DIR)/package.mk`

`define Package/led-control`

`define Package/led-control`
`SECTION :=utils`

`define Package/led-control`
`SECTION :=utils`
`CATEGORY :=Base system`

`define Package/led-control`
`SECTION :=utils`
`CATEGORY :=Base system`
`DEFAULT :=y`

`define Package/led-control`
`SECTION :=utils`
`CATEGORY :=Base system`
`DEFAULT :=y`
`TITLE :=Update utility for LED control`

`define Package/led-control`
`SECTION :=utils`
`CATEGORY :=Base system`
`DEFAULT :=y`
`TITLE :=Update utility for LED control`
`endef`

`define Build/Prepare mkdir -p $(PKG_BUILD_DIR) $(CP) ./src/* $(PKG_BUILD_DIR)`

`define Build/Prepare mkdir -p $(PKG_BUILD_DIR) $(CP) ./src/* $(PKG_BUILD_DIR)`
`endef`

`define Package/led-control/install install -d -m0755 $(1)/sbin install -m0755 $(PKG_BUILD_DIR)/ledcontrol $(1)/sbin/ endef`

`$(eval $(call BuildPackage,led-control))`

这个Makefile主要分为几个部分：
**1. 首先需要包含几个文件，它们定义了一些变量、规则、函数**
`include $(TOPDIR)/rules.mk`
`include $(INCLUDE_DIR)/package.mk`
`include $(INCLUDE_DIR)/kernel.mk`定义内核模块时才需要
`include $(INCLUDE_DIR)/cmake.mk`使用cmake构建软件时用

**2.  一个典型的openwrt软件包目录下没有源码目录，构建openwrt的第一步是从指定的URL下载指定的源码软件包，定义如下**
`PKG_SOURCE := XXXX.tar.gz`
`PKG_SOURCE_URL :=http://www.example.com/`
`PKG_MD5SUM :=e06......askdfjkx`
这三行的意思是从http://www.example.com/XXXX.tar.gz下载源码包，一般存放在dl文件夹里，然后解压到<xxx>/build_dir/target-xxx/$(PKG_BUILD_DIR)

PKG_\*定义了和软件包相关的一些信息：
PKG_NAME   ——软件包名称
PKG_VERSION  ——下载或自编的软件包的版本（**必须有**）
PKG_RELEASE ——当前编译版本
PKG_BUILD_DIR ——在哪个目录下编译该软件包，默认为$(BUILD_DIR)/$(PKG_NAME)-$(PKG-VERSION)
PKG_SOURCE ——要从网上下载源码包的文件名
PKG_SOURCE_URL ——下载路径
PKG_MD5SUM ——验证下载源码包的完整性
PKG_CAT ——怎样解压源码包
PKG_BUILD_DEPENDS ——定义需要在该软件包之前编译的软件包或者一些版本的依赖

**3. package/xxx**描述软件包在menuconfig和ipkg中的条目（其中xxx即出现在menuconfig中的标签以及.config中的CONFIG_PACKAGE_xxx=y）
![package/xxx](http://upload-images.jianshu.io/upload_images/5737261-168e085134ae528e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**4. kernelpackage/xxx**
![kernelpackage/xxx](http://upload-images.jianshu.io/upload_images/5737261-5a4bd10f63757a07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**5. Build/Prepare 定义一组指令，比如用于给源码打补丁，创建编译目录，拷贝自己的源码到编译目录等**

**6. Package/xxx/install 定义如何安装软件包，定义一组命令，用来在嵌入式文件系统中创建目录或者拷贝相关文件到嵌入式文件系统或ipk**
![package/xxx/install](http://upload-images.jianshu.io/upload_images/5737261-e545a4d49a5401c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**7. Package/xxx/preinst定义预先执行的脚本**
![Package/xxx/preinst](http://upload-images.jianshu.io/upload_images/5737261-3ecf795f751f639d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**8. call eval函数调用**
`$(eval $(call BuildPackage,xxx))`
或者
`$(eval $(call KernelPackage,xxx))`
前者用于应用程序，后者用于内核模块
这个(call BuildPackage,xxx)里面大有文章，值得深究


###遇到的问题
从网上下载rp-pppoe源码包，网址：
rp-pppoe源码
https://github.com/cmtsij/OpenWrt_AttitudeAdjustment_Packages/tree/master/net/rp-pppoe
rp-pppoe-3.10源码包
http://download.csdn.net/detail/gududesiling/3468420

在编译rp-pppoe的时候，系统提醒需要ppp（主要是需要pppd）的支持，因此下载ppp源码包进行编译安装
ppp源码：https://github.com/openwrt/openwrt/tree/master/package/network/services/ppp
ppp-2.4.7源码包
http://linux.softpedia.com/get/Communications/Telephony/ppp-14015.shtml

**编译ppp时遇到的问题**
问题：
1. `cannot stat  xxx/build_dir/linux-ipq806x/linux-3.4.103/include/linux/compiler.h (还有atm*.h)`
2. `Strip：Unable to recognise the format of the imput file xxx/sbin/chat`

解决方法：
1. 网上下载linux-3.4.113.tar.xz的源码包，将其`include/linux/compiler.h`以及`include/linux/atm*.h`拷贝到openwrt包的`xxx/build_dir/linux-ipq806x/linux-3.4.103/include/linux/`下即可
2. 网上找了两种解决方案
   （1）修改PATH环境变量，使得gcc和strip命令路径一致
   （2）修改config.mk文件中`INSTALLSTRIP=-s`,删除“-s”参数
   这里，两种方法都无效。不过参照方法（2），修改package/ppp/路径下的各层Makefile.linux中的install命令，将`install -s -c ...`中的`-s`参数去除即可，也即安装过程中不用strip命令

**编译rp-pppoe时遇到的问题**

1. 如何编写package/rp-pppoe/路径下的Makefile，使得其在src文件夹下利用configure文件自动生成Makefile。

  解决方法：
Makefile中添加：
       define Build/configure
       $(call Build/Configure/Default)
       endef
其过程中，可能会遇到问题
`$error: no defaults for cross-compiling `
这是由于rp-pppoe/src/下的configure文件中有这么一句话
`if test "$cross_compiling"=yes,then:
    $ECHO "no defaults for cross-compiling"; exit 0;`
将其中的`exit 0`删除即可

1. 安装rp-pppoe套件时，报错
        cannot overwrite non-directory ... /etc/ppp
   这是因为原油的openwrt中/etc/ppp是一个快捷方式，因此在安装rp-pppoe时，其Makefile定义软件和配置文件安装路径为/etc/ppp，不可写入。因此重新建立一个文件夹/etc/pppa，将pppoe-server的配置文件安装到此目录，然后到开发板上再移动到ppp文件夹里面去

2. 将生成的固件刷入路由器Netgear/R7800，但是拨号时候没有什么效果
   通过抓包分析知道，是路由器向PC端发送了一个PADT终止包。
   修改package/rp-pppoe/src/pppoe-server.c
   其中，它在处理childHandler的函数中，有一段话
        if(!(session->flags & FLAG_SENT_PADT))
        {if(sesssion->flages&FLAG_RECVD-PADT
        sendPATD();
        }
        else{
        sendPADT();
        }
        xxxxxxx
        }
   将其中的else语段注释掉就行

3. 最后拨号时，还是连不上，是pppoe-server运行还需要pppoe软件的支持，在package/rp-pppoe/Makefile中添加pppoe软件包的安装即拨号成功

