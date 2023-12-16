Rotater 驱动和应用
================
[TOC]
<!-- toc -->

----
# 1. 模块简介

输入格式支持：RGB888 ARGB88 RGB565 RGB555 ARGB1555 YUV422

输出格式支持：ARGB8888,RGB565,RGB555,YUV422

旋转角度：0°, 90°, 180°, 270°, Horizontal mirror,Vertical mirror

不支持RGB和YUV422格式互相转换，支持RGB格式相互转换

# 2. 内核空间

内核驱动路径：

drivers/media/platform/ingenic-rotate/

## 2.1. 设备树配置

**在arch/mips/boot/dts/ingenic/x2000-v12.dtsi(m300.dtsi)下配置：**
* 以x2000-v12.dtsi为例：

```
rotate: rotate@0x13070000 {
    compatible = "ingenic,x2000-rotate";
    reg = <0x13070000 0x10000>;
    interrupt-parent = <&core_intc>;
    interrupts = <IRQ_ROTATE>;
    status = "okay";
};
```

## 2.2.  驱动配置

**在SDK 顶层目录执行 make kernel-menuconfig， 然后配置：**
* 以x2000-v12为例：
```
CONFIG_VIDEO_INGENIC_ROTATE:
Symbol: VIDEO_INGENIC_ROTATE [=y]
Type  : tristate
Prompt: Ingenic rotate driver
Location:
-&gt; Device Drivers

  -&gt; Multimedia support \(MEDIA\_SUPPORT \[=y\]\)

    -&gt; Memory-to-memory multimedia devices \(V4L\_MEM2MEM\_DRIVERS \[=y\]\)
Defined at drivers/media/platform/Kconfig:174
Depends on: MEDIA_SUPPORT [=y] && V4L_MEM2MEM_DRIVERS [=y] && VIDEO_DEV [=y] && VIDEO_V4L2 [=y] && (SOC_X2000 [=n] || SOC_X2000_V12 [=y] || SOC_M300 [=n])
Selects: VIDEOBUF2_DMA_CONTIG [=y] && V4L2_MEM2MEM_DEV [=y]
```

# 3. 用户空间
## 3.1. 设备节点
```
/dev/video0
```
## 3.2. 应用程序
```
manhatton工程
packages/example/Sample/rotate/
```
## 3.3. 测试方法
测试应用在testsuite/rotate,　测试了在不同分辨率下进行水平镜像，垂直镜像，旋转0度，旋转90度，旋转180度，旋转270度。

