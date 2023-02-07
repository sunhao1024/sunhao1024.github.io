# Android
## Android startup process
1.BIOS
2.grub
3.boot/kernel
	init(启动SElinux安全上下文,启动zygote用以支持DVM/ART)
	mingetty->login
	shell

根目录结构
/bin可执行
/boot启动vmlinuz
/dev设备(硬盘hdabcd,cdrom hdc,mouse psaux)
/etc(etcetera附加物)系统配置信息(/etc/passwd,/etc/hosts)
/mnt mount文件系统
/var 记录数据
/proc
/sbin 启动时工具，配置，修复分区fsck，安装引导lilo，内核首进程init
/usr 用户应用和文件


应用程序->文件系统接口VFS->EXT3，NTFS->驱动->硬件
app嵌入system.img system/app/xxx.apk

## 一段简单的 Android 代码实例
```java
package com.example.helloworld;

import android.content.Context;
import androidx.test.ext.junit.runners.AndroidJUnit4;
import org.junit.Test;
import org.junit.runner.RunWith;
import static org.junit.Assert.*;

/**
 * Instrumented test, which will execute on an Android device.
 *
 * @see <a href="http://d.android.com/tools/testing">Testing documentation</a>
 */
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Test
    public void useAppContext() {
        // Context of the app under test.
        Context appContext = InstrumentationRegistry.getInstrumentation().getTargetContext();
        gettime（）
        assertEquals("com.example.helloworld", appContext.getPackageName());
    }
}
```

## Android Framework
1.APP
2.JAVA FW
	API+Components
		Activity Manager活动管理器
		Service服务
		Content Provider内容提供
		BroadcastReceiver广播接收器
3.ART + C/C++Lib
	Android
4.HAL
5.Linux Kernel(增加Binder IPC进程间通信,Power Manager电源管理)

Binder
进程间通信机制，基于开源的 OpenBinder
https://www.cnblogs.com/baronzhang/p/8784458.html

Android的子系统主要包含：
1)Android RIL子系统：
Radio Interface Layer（简称：RIL）子系统，即：无线电接口系统用于管理用户的电话、短信、数据通信等相关功能，它是每个移动通信设备必备的系统。
2)Android Input子系统：
Input输入子系统用来处理所有来自己用户的输入数据，如：触摸屏，声音控制物理按键等。
3)Android GUI子系统：
GUI即：图形用户接口，也就是所谓的图形界面，它用来负责显示系统图形化界面，形象让用户和系统操作及信息进行交互。Android的 GUI系统和其它各子系统关系密切相关，是Android中最重要的子系统之一，如：绘制一个2D图形、通过OpenGL库处理3D游戏、通过SurfaceFlinger来重叠几个图形界面。
SurfaceFlinger
Android中图形混合器，用于将屏幕上显示的多个图形进行混合显示
4)Android Audio子系统：
Android的音频处理子系统，主要用于音频方面的数据流传输和控制功能，也负责音频设备的管理。Android的Audio系统和多媒体处理紧密相连，如：视频的音频处理和播放、电话通信及录音等。
5)Android Media子系统：
Android的多媒体子系统，它是Android系统中最庞大的子系统，与硬件编解码、OpenCore多媒体框架、Android多媒体框架等相关，如：音频播放器，视频播放器，Camera摄像预览等。
6)Android Connectivity子系统：
Android连接子系统是智能设备的重要组成部分，它除了一般所谓的网络连接，如：以太网、WI-FI②外，还包含：蓝牙连接、GPS③定位连接、NFC④等。
7)Android Sensor子系统：
Android的传感器子系统为当前智能设备大大提高了交互性，它在一新创新的应用程序和应用体验里发挥了重要作用，传感器子系统和手机的硬件设备紧密相关，如：gyroscope陀螺仪、accelerometer加速度计、proximity距离感应器、magnetic磁力传感器等。

### Android四大组件
AndroidManifest.xml描述了应用程序的每个组件，以及他们如何交互。
1.Activity
	四种基本状态
			Active/Running
				相当于一个页面，Task栈存储不同的Activity,用户方法
				finish()
				CallBack
					onCreate()
					onStart()
					onResume()
					onPause()
					onStop()
					onDestory()
					onRestare()
			Paused
			Stopped
			Killed
	状态转换
2.Service服务
	后台，执行长时间操作，非阻塞
	项目内Server包 右键 --> New --> Service --> Service
	或者直接创建Class类，继承Service并重写onBind,onCreate,onStartCommand,onDestroy方法
3.Content Provider内容提供
	借助ContentResolver类 访问内容提供器中共享的数据
	通过Context中的getContentResolver() 方法获取该类的实例
4.BroadcastReceiver广播接收器
	Android操作系统和应用程序之间的通信,简单响应从其他应用程序或者系统发来的广播消息
	android 广播：
		1）用于不同组件间的通信（含：应用内/不同应用之间）
		2）用于多线程通信
		3）与android系统的通信
	广播分为两个角色：广播发送者、广播接收者

### Android 五大存储
1.SharedPreferences 方式
	使用键值对的方式进行存储数据
	保存少量的数据，格式简单：字符串型、基本类型值
	保存基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息
2.文件存储方式
3 SQLite数据库存储数据
4 使用ContentProvider存储数据
5 网络存储数据

### Android 六大布局
LinearLayout 线性布局

1.线性布局，
	在线性方向上一次排列，水平和竖直
	android:orientation = "vertical" | "horizontal" 竖直或水平，默认水平
	android:layout_gravity = "top" | "center" | "bottom" 内部的布局方式
	android:gravity = "top"|"center"|"bottom" 相对于父容器的对齐方式
	android:layout_weidht 使用比例方式执行控件的大小，在手机屏幕适配方面起到非常重要的作用
2.TableLayout 表格布局
3.FrameLayout 帧布局
4.RelativeLayout 相对布局
5.GridLayout 网格布局
6.AbsoluteLayout 绝对布局

### Android分区
splash1.img开机启动画面区
boot.img内核区
system.img系统区
userdata.img用户数据区
cache.img数据缓存区
recovery.img数据恢复区

boot.img
mkimage命令给zImage文件加上了64个字节的数据头得到uImage文件
后被u-boot识别并正确引导
虚拟的根文件系统->寻址真正的根文件系统镜像
然后执行init完成系统启动
解析init.rc脚本，挂载系统其他分区、开启各个进程和服务等
system.img
Android操作系统核心部分
包含Android系统的firmware、用户界面、预编译应用等
userdata.img
用户出厂数据,挂载在文件系统的/data分区
用户新存储的数据、安装的程序均会被放置到这个分区
cache.img缓存最频繁访问的数据和应用

out/target/product/generic/system目录:
out/target/product/generic/system/apk:Android自带apk
out/target/product/generic/system/bin:可执行文件如C编译执行
out/target/product/generic/system/lib:动态链接库
out/targer/product/generic/system/lib/hw硬件抽象层文件

ramdisk.img，system.img，userdata.img
对应的目录树root，system，data

### Android 主要驱动
1. 显示驱动 display driver：常用于基于linux的帧缓冲frame buffer 驱动程序。
2. flash内存驱动flash memory driver :基于MTD的flash驱动程序。
3. 照相机驱动camera driver :基于linux的v4l video for linux驱动。
4. 音频驱动 audio driver ：基于ALSA advanced linux sound architechure驱动。
5. wifi驱动：基于IEEE801.31标准的驱动程序。
6. 键盘驱动keyboard driver：作为输入设备的键盘驱动。
7. 蓝牙驱动 bluetooth diver :基于IEEE801.35.1标准的无限传输技术。
8. binder IPC驱动：android一个特殊的驱动程序，具有单独的设备节点，提供进程间通信的功能。
9. power management能源管理：管理电池电量等信息。

### Android 主要库
1. C库，基于linux系统调用实现的库，C语言标准库，也是系统最底层的库。
2. 多媒体框架 media framwork
3. SGL:2D图像引擎
4. SSL secure socket layer:为数据通信提供安全支持
5. open GL:对3D提供支持
6. 界面管理工具 surface management
7. SQLite：一种通用的嵌入式数据库
8. Webkit：网络浏览器的核心
9. free type位图和矢量字体的功能

HAL libmodule.so
新的架构module stub
Stub是根，桩
上层应用层或框架层代码加载so库代码
so库代码我们称为module
HAL层注册每个硬件对象的存根stub
当上层需要访问硬件的时候，就从当前注册的硬件对象stub里查找，通过这个module操作接口来访问硬件
APP->加载本地库(module.so)->查找(*.Stub)->driver->hardware

### Android应用程序
XXX_Demo.java
2.基于HAL封装的Java框架层的API
XXX_Service.java
3.Java代码不能访问HAL层，通过JNI调用该库实现Service本地代码部分
libXXX_runtime.so
在.so中查找注册为XXX硬件设备的module
找到后保存操作接口指针在库中等Service调用
返回XXX硬件module对应的device操作结构体中封装的函数指针
4.针对XXX硬件的HAL代码
XXX.default.so


目录结构
Android.mk
freamworks
	Android.mk
		services
			jni
				com_XXX.cpp								#本地服务代码(编译为so共享库)
hardware
	Android.mk
	XXX.c														#HAL .C
	XXX.h														#HAL .H
XXX_app
	AndroidManifest.xml
		Android.mk
	default.properties
		res
			drawable
				icon.png
			layout
				main.xml
			values
				strings.xml
		src
			com
				XXX
					XXX_Demo.java							#APP
					XXX_Service.java					#框架层服务代码



HAL Stub框架
321架构,三个结构体、两个常量、一个函数
一个硬件对象通过hw_module_t描述，可继承hw_module_t后扩展自己的属性
硬件对象固定名字：HMI(Hardware Module Information)
对象里封装一个函数指针open,调用后返回硬件对应的Operation interface
struct hw_module_t｛
	uint32_t tag;           		// 该值必须声明为HARDWARE_MODULE_TAG
	uint16_t version_major; 		// 主版本号
	uint16_t version_minor;     // 次版本号
	const char *id;         		//硬件id名，唯一标识module
	const char *name;       		// 硬件module名字
	const char * author;        // 作者
	struct hw_module_methods_t* methods;    //指向封装有open函数指针的结构体
	void* dso;              		// module’s dso
	uint32_t reserved[32-7];    // 128字节补齐
｝;

JDK,NDK(支持C/C++，用以管理原生activity，访问硬件),


URL:
BSP案例 .so
https://blog.csdn.net/lionfire/article/details/6725379
https://blog.csdn.net/suofeng1234/article/details/51890982
HAL案例 .c .h
https://blog.csdn.net/flappy_boy/article/details/81150290
https://blog.csdn.net/suofeng1234/article/details/51890925
Linux模块
https://blog.csdn.net/finewind/article/details/47165513
Android系统驱动与平台开发
https://blog.csdn.net/mr_raptor/article/details/8940414


---------------------------------------
.so
1.so文件是什么？
也是ELF格式文件，共享库（动态库），类似于DLL。节约资源，加快速度，代码升级简化。
2.怎么生成以及使用一个so动态库文件?
//C文件：s.c
 #include <stdio.h>
 int count;
 void out_msg(const char *m)
 {//2秒钟输出1次信息，并计数
  for(;;) {printf("%s %d\n", m, ++count); sleep(2);}
 }

编译：得到输出文件libs.o
gcc -fPIC -g -c s.c -o libs.o
链接：得到输出文件libs.so
gcc -g -shared -Wl,-soname,libs.so -o libs.so libs.o -lc

//头文件:s.h
#ifndef _MY_SO_HEADER_
 #define _MY_SO_HEADER_
 void out_msg(const char *m);
 #endif

编译链接这个文件：得到输出文件ts
gcc -g ts.c -o ts -L. -ls
执行./ts:error while loading shared libraries: libs.so: cannot open shared object file: No such file or directory
系统不能找到我们自己定义的libs.so
修改环境变量LD_LIBRARY_PATH

#shell脚本123
#!/bin/sh
export LD_LIBRARY_PATH=${pwd}:${LD_LIBRARY_PATH}
./ts

执行./123 & 屏幕上就开始不停有信息输出了

3.地址空间,线程安全
./e &开始后再 ./e&，
会是两个进程交叉输出信息，全局变量count互不干扰，虽然他们引用了同一个so文件。
只有代码是否线程安全一说，没有进程安全

4.库的初始化，解析
初始化函数,卸载函数来完成库的初始化,资源回收
ELF文件本身执行时就会执行一个_init()函数以及_fini()函数来完成这个，我们只要把自己的函数能让系统在这个时候执行
就可以了。
修改s.c
#include <stdio.h>
void my_init(void)__attribute__((constructor));//告诉gcc把这个函数扔到init section
void my_fini(void) __attribute__((destructor)); //告诉gcc把这个函数扔到fini section
void out_msg(const char *m) { printf(" Ok!\n"); } int i; //仍然是个计数器
void my_init(void) { printf("Init ... ... %d\n", ++i); }
void my_fini(void) { printf("Fini ... ... %d\n", ++i); }

5.使用我们自己库里的函数替换系统函数：
创建一个新的文件b.c,替换系统函数malloc以及free(可以自己写个内存泄露检测工具了)

#include <stdio.h>
 void* malloc(int size)
 {
  printf("My malloc\n");
  return NULL;
 }
 void free(void* ad)
 {
  printf("My free\n");
 }

编译链接成一个so文件：得到libb.so
gcc -fPIC -g -c b.c -o libb.o
gcc -g -shared -Wl,-soname,libb.so -o libb.so -lc
修改s.c:重新生成libs.so

void out_msg()
 {
  int *p;
  p = (int*)malloc(100);
  free(p);
  printf("Stop Ok!\n");
 }

 修改脚本文件e:
#!/bin/sh
export LD_PRELOAD=${pwd}libb.so:${LD_PRELOAD}
export LD_LIBRARY_PATH=${pwd}:${LD_LIBRARY_PATH}
./ts
关键就在LD_PRELOAD上了，这个路径指定的so将在所有的so之前加载，并且符号会覆盖后面加载的so文件中的符号。如果可执行文件的权限不合适（SID），这个变量会被忽略。
执行：./e &
嗯，可以看到我们的malloc,free工作了。

## Android hal
Android Framework进程和Hal分离，每个Hal独立运行在自己的进程地址空间

hal接口定义
hardware/libhardware/include/hardware/ *.h

gralloc and fb
gralloc对应FramebufferNativeWindows(OpenGL ES的本地窗口)

hardware/libhardware/modules/gralloc/framebuffer.cpp
int mapFrameBufferLocked(struct private_module_t* module)
char const * const device_template[] = {
        "/dev/graphics/fb%u",
        "/dev/fb%u",
        0 };

读写,内存映射
ioctl(fd, FBIOGET_FSCREENINFO, &finfo)
ioctl(fd, FBIOGET_VSCREENINFO, &info)
ioctl(fd, FBIOPUT_VSCREENINFO, &info)
void* vaddr = mmap(0, fbSize, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
module->framebuffer->base = intptr_t(vaddr);
memset(vaddr, 0, fbSize);
映射地址为module->framebuffer->base,module来自hw_get_module()

libhardware/modules/gralloc/gralloc.cpp
int gralloc_device_open(const hw_module_t* module, const char* name,
        hw_device_t** device)
if (!strcmp(name, GRALLOC_HARDWARE_GPU0)) {
    gralloc_context_t *dev;
    dev = (gralloc_context_t*)malloc(sizeof(*dev));
    dev->device.alloc   = gralloc_alloc;
    dev->device.free    = gralloc_free;
    *device = &dev->device.common;
图形缓冲区的分配与释放(gralloc.open->GPU0.alloc/free fb0)

hardware/libhardware/hardware.c
int hw_get_module(const char *id, const struct hw_module_t **module)


adb shell dumpsys -l
adb shell service list


activity 		ActivityManagerService 	AMS相关信息
package 		PackageManagerService 	PMS相关信息
window 			WindowManagerService 	WMS相关信息
input 			InputManagerService 	IMS相关信息
power 			PowerManagerService 	PMS相关信息
batterystats 	BatterystatsService 	电池统计信息
battery 		BatteryService 			电池信息
alarm 			AlarmManagerService 	闹钟信息
dropbox 		DropboxManagerService 	调试相关
procstats 		ProcessStatsService 	进程统计
cpuinfo 		CpuBinder 				CPU
meminfo 		MemBinder 				内存
gfxinfo 		GraphicsBinder 			图像
dbinfo 			DbBinder 				数据库
SurfaceFlinger 							图像相关
appops 			app使用情况
permission 								权限
processinfo 	进程服务
batteryproperties 	电池相关
audio 	查看声音信息
netstats 	查看网络统计信息
diskstats 	查看空间free状态
jobscheduler 	查看任务计划
wifi 	wifi信息
diskstats 	磁盘情况
usagestats 	用户使用情况
devicestoragemonitor 	设备信息

## Android bringup

rc启动脚本
android.hardware.keymaster@4.0-service
/vendor/bin/hw/android.hardware.keymaster@4.0-service.rc
编译后，会将该脚本文件copy到vendor/etc/init目录
开机init进程会读取并解析这个脚本，启动android.hardware.keymaster@4.0-service进程

init.rc文件
基本组成单位:section, section
三种类型,由三个关键字(每一行的第一列)来区分:on、service、import
on类型的section表示一系列命令的组合
service类型的section表示一个可执行程序
import类型的section表示引入另外一个.rc文件

参考init.rc:
```sh
# Copyright (C) 2012 The Android Open Source Project
#
# IMPORTANT: Do not create world writable files or directories.
# This is a common source of Android security bugs.
#

import /init.usb.rc
import /init.${ro.hardware}.rc
import /init.trace.rc

on early-init
    # Set init and its forked children's oom_adj.'
    write /proc/1/oom_adj -16
    # Set the security context for the init process.
    # This should occur before anything else (e.g. ueventd) is started.
    setcon u:r:init:s0
    start ueventd

# create mountpoints
    mkdir /mnt 0775 root system

on init
    chmod 666 /dev/ttymxc0
    chmod 666 /dev/ttymxc1
    chmod 666 /dev/i2c-0
    chmod 666 /dev/i2c-1
    ......
    export PATH /sbin:/vendor/bin:/system/sbin:/system/bin:/system/xbin
    export LD_LIBRARY_PATH /vendor/lib:/system/lib
    export ANDROID_BOOTLOGO 1
    export ANDROID_ROOT /system
    export ANDROID_ASSETS /system/app
    export ANDROID_DATA /data
    ......
......
service servicemanager /system/bin/servicemanager
    class core
    user system
    group system
    critical
    onrestart restart zygote
    onrestart restart media
    onrestart restart surfaceflinger
    onrestart restart drm

service vold /system/bin/vold
    class core
    socket vold stream 0660 root mount
    ioprio be 2
......
```

1. BoardConfig.mk
device/qcom/sdm845/BoardConfig.mk
BOARD_KERNEL_CMDLINE 后面加androidboot.selinux=permissive -> 关闭selinux
BOARD_VNDK_VERSION:=current 注释掉 -> 关闭VNDK

project LINUX/android/device/qcom/sdm845/
diff --git a/BoardConfig.mk b/BoardConfig.mk
index ece2066..1e51cc2 100644
--- a/BoardConfig.mk
+++ b/BoardConfig.mk
@@ -131,7 +131,7 @@ TARGET_USES_QCOM_BSP := false

 TARGET_USES_IOPHAL := true

-BOARD_KERNEL_CMDLINE := console=ttyMSM0,115200n8 earlycon=msm_geni_serial,0xA84000 androidboot.hardware=qcom androidboot.console=ttyMSM0 video=vfb:640x400,bpp=32,memsize=3072000 msm_rtb.filter=0x237 ehci-hcd.park=3 lpm_levels.sleep_disabled=1 service_locator.enable=1 swiotlb=2048 androidboot.configfs=true firmware_class.path=/vendor/firmware_mnt/image loop.max_part=7 androidboot.usbcontroller=a600000.dwc3
+BOARD_KERNEL_CMDLINE := console=ttyMSM0,115200n8 earlycon=msm_geni_serial,0xA84000 androidboot.hardware=qcom androidboot.console=ttyMSM0 video=vfb:640x400,bpp=32,memsize=3072000 msm_rtb.filter=0x237 ehci-hcd.park=3 lpm_levels.sleep_disabled=1 service_locator.enable=1 swiotlb=2048 androidboot.configfs=true firmware_class.path=/vendor/firmware_mnt/image loop.max_part=7 androidboot.usbcontroller=a600000.dwc3 androidboot.selinux=permissive

 BOARD_EGL_CFG := device/qcom/$(TARGET_BOARD_PLATFORM)/egl.cfg

@@ -212,4 +212,4 @@ TARGET_ENABLE_MEDIADRM_64 := true
 #Flag to enable System SDK Requirements.
 #All vendor APK will be compiled against system_current API set.
 BOARD_SYSTEMSDK_VERSIONS:=28
-BOARD_VNDK_VERSION:= current
+#BOARD_VNDK_VERSION:= current


2. manifest.xml
device/qcom/sdm845/manifest.xml
<!-- keymaster -->
<hal format="hidl">
   <name>android.hardware.keymaster</name>
   <transport>hwbinder</transport>
   <version>4.0</version>
   <interface>
       <name>IKeymasterDevice</name>
       <instance>default</instance>
   </interface>
</hal>

3. sdm845.mk
device/qcom/sdm845/sdm845.mk
diff --git a/sdm845.mk b/sdm845.mk
index 9b3caf1..dd35294 100644
--- a/sdm845.mk
+++ b/sdm845.mk
@@ -263,8 +263,8 @@ PRODUCT_PROPERTY_OVERRIDES += ro.vendor.qti.sys.fw.bg_apps_limit=60
 KMGK_USE_QTI_SERVICE := true

 #Enable KEYMASTER 4.0
-ENABLE_KM_4_0 := true
-
+#ENABLE_KM_4_0 := true
+ENABLE_KM_4_0 := false
 ifneq ($(strip $(TARGET_USES_RRO)),true)
 DEVICE_PACKAGE_OVERLAYS += device/qcom/sdm845/overlay
 endif

4. make/target/product/embedded.mk
project LINUX/android/build/make/target/product/embedded.mk
diff --git a/target/product/embedded.mk b/target/product/embedded.mk
index 69cf10227..8b37d2938 100644
--- a/target/product/embedded.mk
+++ b/target/product/embedded.mk
@@ -38,7 +38,7 @@ PRODUCT_PACKAGES += \
     fastboot \
     gralloc.default \
     healthd \
-    hwservicemanager \
+    #hwservicemanager \
     init \
     init.environ.rc \
     init.rc \

5. base.mk
project LINUX/android/device/qcom/common/base.mk
diff --git a/base.mk b/base.mk
index d79353be..1111310c 100644
--- a/base.mk
+++ b/base.mk
@@ -520,7 +520,7 @@ LIBMEMTRACK += memtrack.msm8998
 LIBMEMTRACK += memtrack.msmnile
 LIBMEMTRACK += memtrack.sdmshrike
 LIBMEMTRACK += memtrack.sdm660
-LIBMEMTRACK += memtrack.sdm845
+#LIBMEMTRACK += memtrack.sdm845
 LIBMEMTRACK += memtrack.apq8098_latv
 LIBMEMTRACK += memtrack.sdm710
 LIBMEMTRACK += memtrack.qcs605



### SEliunx
rc文件替换
1.SEliunx 文件替换
系统中SElinux文件的存放位置为/system/etc/selinux 和 /vendor/etc/selinux可以在编译出来的版本中只替换手机vendor或system下selinux文件夹（注意是整个文件夹）来达到修改目的
2.rc文件修改
/system/etc/init/和/vendor/etc/init/可以直接修改相应rc文件，无需下载img，对根目录下的rc文件不能采用该方法


* 相关文件介绍
device/qcom/sepolicy / common/device.te   ：定义设备相关的权限
device/qcom/sepolicy / common/file.te     ：定义文件相关的权限
device/qcom/sepolicy / common/file_contexts ： 分类相应文件使用哪些权限，然后就会调用到同名的 xxx.te 文件
device/qcom/sepolicy / common/genfs_contexts： 定义文件系统权限？
device/qcom/sepolicy / common/service.te ： 定义 service 服务权限
device/qcom/sepolicy / common/service_contexts ： 分类相应服务使用哪些权限
device/qcom/sepolicy / msm8937/property.te ： 定义属性权限
device/qcom/sepolicy / msm8937/property_contexts ： 分类相应属性使用什么权限

* 具体进程权限
device/qcom/sepolicy / common/kernel.te ： 内核进程
device/qcom/sepolicy / common/system_app.te ：系统 app 权限
device/qcom/sepolicy / common/init.te ：init 进程权限
device/qcom/sepolicy / msm8937/init_shell.te ： shell 进程？
device/qcom/sepolicy / msm8937/platform_app.te ：平台 app 权限？
device/qcom/sepolicy / common/untrusted_app.te ： 非平台签名的 apk

* 通用步骤：
一、首先验证是否是 Selinux 权限相关问题
    在 eng 版本中使用：
        setenforce 0
    临时关闭 selinux 后，再验证。(注：有时是权限问题，但也未必有效，这时可通过 log 确认)

二、抓取开机 log, 以内核搜索出如下关键字：
    E:\Kernel_Log\20180731-203625.kernel.txt (13 hits)
        [   26.660378] type=1400 audit(4308.789:13): avc: denied { module_load } for pid=1 comm="init" path="/lib/modules/texfat.ko" dev="rootfs" ino=5423 scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=system permissive=0
        [   26.686817] type=1400 audit(4308.789:13): avc: denied { module_load } for pid=1 comm="init" path="/lib/modules/texfat.ko" dev="rootfs" ino=5423 scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=system permissive=0
        [   26.686831] type=1400 audit(4308.809:14): avc: denied { module_load } for pid=1 comm="init" path="/lib/modules/tntfs.ko" dev="rootfs" ino=46 scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=system permissive=0
        [   28.231490] type=1400 audit(4308.809:14): avc: denied { module_load } for pid=1 comm="init" path="/lib/modules/tntfs.ko" dev="rootfs" ino=46 scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=system permissive=0

    参考示例添加：
        // [  342.204415] type=1400 audit(4504.179:161): avc: denied { search } for pid=5728 comm="unlockcheck" name="block" dev="tmpfs" ino=568 scontext=u:r:unlockcheck:s0 tcontext=u:object_r:block_device:s0 tclass=dir permissive=0
        //          scontext=u:r:unlockcheck:s0                 # 操作主体 unlockcheck ， 可通过 ls -Z  ps -Z 查看
        //          tcontext=u:object_r:block_device:s0         # 操作客体 block_device
        //          tclass=dir permissive=0                     # 操作客体所属类别  dir ， 相关权限可通过执行相关权限目录酌情添加
        // allow unlockcheck block_device:dir { search getattr read write};

三、重新编译安卓即可，需要更新 boot.img 和 system.img

* 给可执行文件添加权限：
给可执行程序添加权限：
    src\device\qcom\sepolicy\common\file_contexts
        # wangjun@wind-mobi.com 20180108 add unlock start
        /vendor/bin/unlockcheck u:object_r:unlockcheck_exec:s0
        # wangjun@wind-mobi.com 20180108 add unlock end


    # 新增的权限文件
    src\device\qcom\sepolicy\common\unlockcheck.te
        # 以下几个是可执行程序必要的权限声明
        type unlockcheck ,domain;
        type unlockcheck_exec , file_type, vendor_file_type, exec_type;
        init_daemon_domain(unlockcheck)

        # 以下权限是通过 kernel log 一条条添加的，报哪条添加哪条
        allow unlockcheck qdma_data_file:file create_file_perms;
        allow unlockcheck qdma_data_file:dir create_dir_perms;
        allow unlockcheck { proc sysfs }:file r_file_perms;
        allow unlockcheck { proc sysfs }:dir r_dir_perms;
        allow unlockcheck factory_data_file: file {read write open create getattr};
        allow unlockcheck factory_data_file: dir {search write read add_name};
        allow unlockcheck self:capability dac_override;
        allow unlockcheck diag_device:chr_file {read write open ioctl};

        // [  342.204415] type=1400 audit(4504.179:161): avc: denied { search } for pid=5728 comm="unlockcheck" name="block" dev="tmpfs" ino=568 scontext=u:r:unlockcheck:s0 tcontext=u:object_r:block_device:s0 tclass=dir permissive=0
        //          scontext=u:r:unlockcheck:s0                 # 操作主体 unlockcheck ， 可通过 ls -Z  ps -Z 查看
        //          tcontext=u:object_r:block_device:s0         # 操作客体 block_device
        //          tclass=dir permissive=0                     # 操作客体所属类别  dir ， 相关权限可通过执行相关权限目录酌情添加
        allow unlockcheck block_device:dir { search getattr read write};
        allow unlockcheck proinfo_block_device:blk_file {open read write};


重新编译安卓即可，需要更新 boot.img 和 system.img


* 给新挂载分区及文件添加权限：
1. 定义文件权限
    device/qcom/sepolicy / common/device.te
        type xrom_block_device, dev_type;

    device/qcom/sepolicy / common/file.te
        type xrom_file, file_type;

    device/qcom/sepolicy / common/file_contexts
        /dev/block/platform/soc/7824900.sdhci/by-name/xrom                 u:object_r:xrom_block_device:s0
        /xrom(/.*)?            u:object_r:xrom_file:s0


2. 使用者添加使用权限：
    device/qcom/sepolicy / common/fsck.te
        allow fsck xrom_block_device:blk_file           { read open write ioctl };

    device/qcom/sepolicy / common/init.te
        # xrom
        allow init xrom_file:dir { mounton };
        allow init xrom_block_device:blk_file { write };

* 给位于 dev 目录下的文件添加权限：
1. 定义权限
    device/qcom/sepolicy / common/device.te
        type dvt_isdbt, dev_type;

    device/qcom/sepolicy / common/file_contexts
        /dev/isdbt				u:object_r:dvt_isdbt:s0

2. 添加使用者权限
    device/qcom/sepolicy / common/system_app.te
        allow system_app dvt_isdbt:chr_file {read write ioctl open};


* 新建的 proc 目录下文件添加权限：
1. 分配权限
    device/qcom/sepolicy / common/file.te
        type proc_gestures_file, fs_type;
        type proc_gestures_item_file, fs_type;

    device/qcom/sepolicy / common/genfs_contexts  # 这个路径为 /proc/android_touch/
        genfscon proc /android_touch/SMWP u:object_r:proc_gestures_file:s0
        genfscon proc /android_touch/GESTURE u:object_r:proc_gestures_item_file:s0

2. 使用权限
    device/qcom/sepolicy / common/system_app.te
        allow system_app proc_gestures_file:file rw_file_perms;
        allow system_app proc_gestures_item_file:file rw_file_perms;

    device/qcom/sepolicy / common/system_server.te
        allow system_server proc_gestures_file:file rw_file_perms;
        allow system_server proc_gestures_item_file:file rw_file_perms;

* 新增文件系统，添加权限： texfat/tntfs
device/qcom/sepolicy / common/file_contexts
    # Tuxera exFAT/NTFS
    # Must be placed at the very end of the file, after all of the AOSP entries!
    /system/bin/exfatck	--	u:object_r:fsck_exec:s0
    /system/bin/ntfsck	--	u:object_r:fsck_exec:s0
    /system/bin/exfatlabel  u:object_r:blkid_exec:s0
    /system/bin/exfatvsn    u:object_r:blkid_exec:s0

device/qcom/sepolicy / common/genfs_contexts
    # Tuxera exFAT/NTFS labeled with the 'vfat' label since it will be used in the same context.
    genfscon texfat / u:object_r:vfat:s0
    genfscon tntfs / u:object_r:vfat:s0

device/qcom/sepolicy / common/init.te
    allow init system_file:system module_load;


* 修改文件系统路径 添加权限： texfat/tntfs
device\qcom\sepolicy\common\init.te
    allow init rootfs:system module_load;


### 正常模式和工厂模式的区分
工厂模式的adb好用，正常模式adb口不好用
如果打开了串口
工厂模式的属性ro.bootmode=ffbm-02  正常模式的属性ro.bootmode=unknown
正常模式->工厂模式的方法有是两种
方法一：
插入鼠标，拔掉type-c
在拨打电话的窗口输入”*#*#7664#*#*“，弹出QMMI界面
点击右下角reboot->reboot to QMMI
方法二：
进入fastboot模式（按下右侧是三个按键中的最下方的按键后上电）
输入fastboot flash misc misc.img
注：制作misc.img命令为“echo ffbm-02 > misc.img”
工厂模式->正常模式的方法有是三种
方法一：
在拨打电话的窗口输入”*#*#7664#*#*“，弹出QMMI界面
点击右下角reboot->reboot to Android
方法二：
进入fastboot模式（按下右侧三个按键中的最上方靠近IN-Car camera接口的按键后上电）
输入fastboot erase misc
方法三：
进入shell，输入如下命令
setprop sys.boot_mode normal
sleep 3
reboot

## Jekins整编
Update make_qfil_images_for_sdm845.sh
fastboot flash label imgfile

common/content.xml

/proc/kmsg
/proc/interrupts
/proc/meminfo


## QFIL
QPST sahara memory dump ramdump
QCAP
QXDM mpss modem pross sub sys adsp cdsp

Insmode lsmode modprobe

# ADB
ADB全称为Android Debug Bridge，是管理android模拟器或者设备的一个工具，简单的说它就是一个调试工具
adb devices 	显示当前运行的全部模拟器
adb root 		获取管理员权限
adb disable-verity		关闭分区Hash校验
adb connect ip:port 	连接设备
adb install -r 	123.apk安装应用程序
adb uninstall 123.apk
adb pull 		获取模拟器中的文件
adb push 		向模拟器中写文件
adb shell 		进入模拟器的shell模式
adb kill-server
adb start-server
adb devices
\adb shell pm disable-user "apk包名"
\adb shell pm disable-user com.android.mediacenter
\adb shell pm enable "apk包名"
adb shell pm list packages -3
adb shell getprop ro.build.version.release	查看 Android 版本

WiFi 代理
adb connect 192.168.1.20:5555
adb forward tcp:8118 tcp:8118

Adb shell getevent –l  //查看TP中断上报情况，显示input设备/type/code/value
Adb shell getevent –r //查看TP报点率
Adb shell getevent –I  //查看输入事件信息
使用adb 截图
adb shell /system/bin/screencap -p /sdcard/screenshot.png（执行该命令后截图保存在手机的sd卡中）

rmdir xxx  		删除xxx的文件夹
cd .. 返回上一级目录
cd ../.. 返回上两级目录
cd 进入个人的主目录
cd ~user1 进入个人的主目录
cd - 返回上次所在的目录

删除系统应用
adb remount （重新挂载系统分区，使系统分区重新可写）。
adb shell
cd system/app
rm 123.apk
查看应用列表
adb shell pm list package
启动/停止应用
adb shell am start -n [packageName/StartActivity]
adb shell am force-stop [packageName]

查看进程列表
adb shell ps
查看指定进程状态
adb shell ps -x [PID]
查看后台services信息
adb shell service list
查看IO内存分区
adb shell cat /proc/iomem

push
adb push apns-conf.xml /system/etc/
发生错误：failed to copy 'apns-conf.xml' to '/system/etc//apns-conf.xml': Read-only file system
adb shell之后su是可以的，但是
$ adb remount
remount of system failed: Permission denied
remount failed
退出adb shell（exit）
$ adb root
restarting adbd as root
$ adb remount
remount succeeded
$ adb push apns-conf.xml /system/etc/
2577 KB/s (271006 bytes in 0.102s)

端口转发
adb forward --list
adb forward tcp:8118 tcp:8118
adb reverse tcp:3000 tcp:3000
adb forward --remove tcp:8080
adb forward --remove-all


开机模式 				屏幕显示 			冷启动 						热启动 					按键退出 					命令退出
1.Android/Normal 		Android界面 		按Power键 					adb reboot 				手机短按，VR长按Power键 	adb shell reboot -p(关机)
2.Recovery/OTA/卡刷 	Recovery界面 	按住OK键(Vol+)，再按Power键 	adb reboot recovery 	长按Power键重启 			adb reboot
3.Fastboot/线刷 		Fastboot界面 	按住BACK键(Vol-)，再按Power键 adb reboot bootloader	长按Power键重启 			fastboot reboot fastboot continue(resuming boot)
4.FFBM/Fast Factory/厂测/半开机 	显示测试列表 	misc分区头部为ffbm时，按Power键 	misc分区头部为ffbm时，adb reboot 	长按Power键重启依然进入FFBM
唯一退出方式擦除misc分区
5.EDL/紧急下载/9008/砖头/裸板 	无显示,黑屏 	同时按住OK键(Vol+)和BACK键(Vol-)，再按Power键
adb reboot edl
fastboot reboot emergency
	长按Power键重启 	无



adb no permission
lsusb
Bus 003 Device 019: ID  05c6 : 9091  Qualcomm, Inc
UDEV=SUBSYSTEM==\"usb\", ATTR{idVendor}==\"2717\", ATTRS{idProduct}==\"ff68\", MODE=\"0666\", GROUP=\"plugdev\"
sudo echo $UDEV >> /etc/udev/rules.d/51-android.rules
sudo chmod a+x /etc/udev/rules.d/51-android.rules

sudo /etc/init.d/udev restart

sudo udevadm control --reload-rules
sudo udevadm trigger

useradd -G plugdev sunhao
sudo adb kill-server
sudo adb devices



# fastboot
fastboot erase {partition}擦除分区
fastboot erase boot

fastboot flash {partition} {*.img}烧写分区
fastboot flash boot boot.img

fastboot flashall烧写所有分区
在当前目录中查找所有img文件，烧写到对应的分区中并重启手机

fastboot update {*.zip}一次烧写boot，system，recovery分区
创建包含boot.img，system.img，recovery.img文件的zip包

fastboot flash splash1 烧写开机画面

fastboot reboot 重启

## 编译
LINUX/android
build/envsetup.sh
https://www.cnblogs.com/vincentcc-90/p/4615258.html

Android指令
hmm  			所有option
辅助命令
cgrep keyword 	在C，C++代码中搜索指定关键字
jgrep keyword 	在java代码中搜索指定关键字
resgrep keyword 在资源xml文件中搜索指定关键字
croot
cout
cproj
findmakefile    打印当前目录所在工程的Android.mk的文件路径
printconfig  	打印各种编译变量的值
print_lunch_menu打印lunch可选择的各种product
godir xxx		进入文件目录
repo diff

调试相关
gdbclient 		gdb调试
getlastscreenshot 获取最后一张截图
getscreenshotpath 获取屏幕截图的路径
key_back 		模拟按返回键
key_home 		模拟按Home键
key_menu 		模拟按菜单键
runtest 		调用development/testrunner/runtest.py，进行测试
smoketest 		利用SmokeTestApp.apk，SmokeTest.apk对系统进行一个smoke test
systemstack  	dump the current stack trace of all threads in the system process to the usual ANR traces file
tracedmdump 	调用q2dm将系统堆栈导出来，并利用dmtracedump将其转为可读的html文件



禁用app
.\adb shell pm disable-user "apk包名"
以华为音乐为例
.\adb shell pm disable-user com.android.mediacenter
备注:一些核心app请不要禁用,如需开启可以执行
.\adb shell pm enable "apk包名"


# ARChon
https://bitbucket.org/vladikoff/archon/src/master
在可以跨多端（ OS X、linux、windows）上使用 Android APP, 适用于在 chrome上提供了一个Android Runtime

npm install chromeos-apk -g
chromeos-apk xxx.apk -archon

Crx 文件夹下的 _locales\en\messages.json
"message":"包名"

chrome://extensions
chrome://apps
