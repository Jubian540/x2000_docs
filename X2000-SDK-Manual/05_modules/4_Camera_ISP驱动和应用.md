Camera isp驱动和应用
==================
[TOC]
<!-- toc -->

----
# 1. 模块简介

isp(Image Signal Processing)模块的主要作用是对前端传感器输出的信号做后期处理。主要功能有白平衡、自动曝光控制、自动增益等。支持mipi、dvp接口raw数据输入。NV12/NV21格式输出。

isp模块包含csi、vic、isp-core、mscaler四个子模块，其驱动框架如下图所示。

![](/assets/isp-drv.png)

# 2. 内核空间

isp 驱动目录：

```
drivers/media/platform/ingenic-isp
├── csi.c
├── csi-regs.h
├── isp.c
├── isp-core
│   ├── inc
│      ├── system_sensor_drv.h
│      ├── tiziano_core.h
│      ├── tiziano_core_tuning.h
│      ├── tiziano_isp.h
│      └── tiziano_sys.h
│   ├── isp-core.a
│   ├── Makefile
├── isp-drv.c
├── isp-drv.h
├── isp-regs.h
├── isp-video.c
├── isp-video-mplane.c
├── Makefile
├── mscaler.c
├── mscaler-regs.h
├── sensor.c
├── vic.c
└── vic-regs.h
```


## 2.1. 设备树配置

**在arch/mips/boot/dts/ingenic/x2000-v12.dtsi(m300.dtsi) ahb0下配置：**
* 以x2000-v12.dtsi为例：

```
    ispcam0:
        isp-camera@0 {
        compatible = "ingenic,x2000-isp-camera";
        #address-cells = <1>;
        #size-cells = <1>;
        ranges = <>;

        port {
            isp0_ep:endpoint@0 {

            };
        };


        csi0: csi@0x10074000 {
            compatible = "ingenic,x2000-csi";
            reg = <0x10074000 0x1000>;
            interrupt-parent = <&core_intc>;
            interrupts = <IRQ_MIPI_CSI_4>;  // 4lane csi
            clocks = <&clock CLK_GATE_MIPI_CSI>;
            clock-names = "gate_csi";
        };

        vic0: vic@0x13710000 {
            compatible = "ingenic,x2000-vic";
            reg = <0x13710000 0x10000>;
            interrupt-parent = <&core_intc>;
            interrupts = <IRQ_VIC0>;
            status = "ok";
        };
        isp0: isp@0x13700000 {
            compatible = "ingenic,x2000-isp";
            reg = <0x13700000 0x2300>;
            interrupt-parent = <&core_intc>;
            interrupts = <IRQ_ISP0>;
            clocks = <&clock CLK_DIV_ISP>, <&clock CLK_GATE_ISP0>;
            clock-names = "div_isp", "gate_isp0";
            status = "ok;
            ingenic,cpm_reset = <0xb00000c4>;
            ingenic,bit_sr = <25>;
            ingenic,bit_stp = <24>;
            ingenic,bit_ack = <23>;
        };
        mscaler0: mscaler@0x13702300 {
            compatible = "ingenic,x2000-mscaler";
            reg = <0x13702300 0x400>;
            status = "ok";
        };
    };

    ispcam1: isp-camera@1 {
        compatible = "ingenic,x2000-isp-camera";
        #address-cells = <1>;
        #size-cells = <1>;
        ranges = <>;

        port {
            isp1_ep:endpoint@0 {

            };
        };

        csi1: csi@0x10073000 {
            compatible = "ingenic,x2000-csi";
            reg = <0x10073000 0x1000>;
            interrupt-parent = <&core_intc>;
            interrupts = <IRQ_MIPI_CSI2>;   // 2lane csi
            clocks = <&clock CLK_GATE_MIPI_CSI>;
            clock-names = "gate_csi";
        };

        vic1: vic@0x13810000 {
            compatible = "ingenic,x2000-vic";
            reg = <0x13810000 0x10000>;
            interrupt-parent = <&core_intc>;
            interrupts = <IRQ_VIC1>;
            status = "ok";
        };

        isp1: isp@0x13800000 {
            compatible = "ingenic,x2000-isp";
            reg = <0x13800000 0x2300>;
            interrupt-parent = <&core_intc>;
            interrupts = <IRQ_ISP1>;
            clocks = <&clock CLK_DIV_ISP>, <&clock CLK_GATE_ISP1>;
            clock-names = "div_isp", "gate_isp1";
            status = "ok";
            ingenic,cpm_reset = <0xb00000c4>;
            ingenic,bit_sr = <22>;
            ingenic,bit_stp = <21>;
            ingenic,bit_ack = <20>;
        };

        mscaler1: mscaler@0x13802300 {
            compatible = "ingenic,x2000-mscaler";
            reg = <0x13802300 0x400>;
            status = "ok";
        };
    };
```

**在arch/mips/boot/dts/ingenic/板级.dts下配置：**
* 若选用驱动支持的camera小板无需配置，若不选用驱动支持的camera小板请参照arch/mips/boot/dts/ingenic/"板级"_cameras自行配置。

## 2.2. 驱动配置

**make menuconfig， 然后配置：**

### 2.2.1 通用配置:

```
    Symbol: VIDEO_INGENIC_ISP [=y]
    Type  : tristate
    Prompt: V4L2 Driver for ingenic isp
        Location:
            -> Device Drivers
                -> Multimedia support (MEDIA_SUPPORT [=y])
                     -> V4L platform devices (V4L_PLATFORM_DRIVERS [=y])Kconfig:117
        Depends on: MEDIA_SUPPORT [=y] && V4L_PLATFORM_DRIVERS [=y] && VIDEO_DEV [=y] && VIDEO_V4L2 [=y]
        Selects: VIDEOBUF2_DMA_CONTIG [=y]

        Symbol: VIDEO_V4L2_SUBDEV_API [=y]
        Type  : boolean
        Prompt: V4L2 sub-device userspace API
        Location:
            -> Device Drivers
                -> Multimedia support (MEDIA_SUPPORT [=y])
        Defined at drivers/media/Kconfig:117
        Depends on: MEDIA_SUPPORT [=y] && VIDEO_DEV [=y] && MEDIA_CONTROLLER [=y]
```


### 2.2.2 扩展配置

#### 2.2.2.1 选择不同的Camera子板驱动
```
---halley5_camera_board  
[ ]   halley5 camera driver for RD_X2000_HALLEY5_CAMERA_2V1  
[ ]   halley5 camera driver for RD_X2000_HALLEY5_CAMERA_4V2  
[ ]   halley5 camera driver for RD_X2000_HALLEY5_CAMERA_3V2  
[ ]   halley5 camera driver for RD_X2000_HALLEY5_CAMERA_5V0

--- gewu_camera_board
[ ]   gewu camera driver for CAMERA_1.0
```

* 选用RD_X2000_HALLEY5_CAMERA_3V2和gewu camera时需选择sensor驱动支持的分辨率及帧率

* 若不使用以上几款camera小板请根据所连接的sensor选择驱动


#### 2.2.2.2 选择sensor驱动 

```
 < > ov4689 camera support  
 < > sc2232h camera support  
 < > ov2735 camera DVP interfac  
 < > ov2735 camera MIPI interface support
 < > ar0144 camera support  
```

#### 2.2.2.3 配置VIC BYPASS输出功能
```
    Symbol: VIC_DMA_OUT [=y]
    Type  : boolean
    Prompt: vic dma out bebug device node
    Location:
     -> Device Drivers
        -> Multimedia support (MEDIA_SUPPORT [=y])
            -> V4L platform devices (V4L_PLATFORM_DRIVERS [=y])
               -> V4L2 Driver for ingenic isp (VIDEO_INGENIC_ISP [=y])
                    -> vic dma out route enable (VIC_DMA_ROUTE [=y])
                        -> vic dma out route select (<choice> [=y])
      Defined at drivers/media/platform/ingenic-isp/Kconfig:13
      Depends on: <choice> && VIC_DMA_ROUTE [=y]
```

#### 2.2.2.4 配置Debug调试接口功能

```
    Symbol: VIC_DMA_DEBUG [=y]
    Type  : boolean
    Prompt: vic dma bebug sys node
    Location:
     -> Device Drivers
        -> Multimedia support (MEDIA_SUPPORT [=y])
            -> V4L platform devices (V4L_PLATFORM_DRIVERS [=y])
               -> V4L2 Driver for ingenic isp (VIDEO_INGENIC_ISP [=y])
                    -> vic dma out route enable (VIC_DMA_ROUTE [=y])
                        -> vic dma out route select (<choice> [=y])
      Defined at drivers/media/platform/ingenic-isp/Kconfig:16
      Depends on: <choice> && VIC_DMA_ROUTE [=y]
```

# 3. 用户空间

## 3.1. 设备节点

驱动加载成功，生成的相关设备节点

```
/sys/devices/platform/ahb0/ahb0:isp-camera@0/10074000.csi csi0文件节点
/sys/devices/platform/ahb0/ahb0:isp-camera@0/13710000.vic vic0文件节点
/sys/devices/platform/ahb0/ahb0:isp-camera@0/13700000.isp isp0文件节点
/sys/devices/platform/ahb0/ahb0:isp-camera@0/13702300.mscaler mscaler0 文件节点
/sys/devices/platform/ahb0/ahb0:isp-camera@1/10073000.csi csi1文件节点
/sys/devices/platform/ahb0/ahb0:isp-camera@1/13810000.vic vic1文件节点
/sys/devices/platform/ahb0/ahb0:isp-camera@1/13800000.isp isp1文件节点
/sys/devices/platform/ahb0/ahb0:isp-camera@1/13802300.mscaler mscaler1 文件节点
```
```
/dev/video3         vic0设备节点
/dev/video4         mscaler0-ch0设备节点
/dev/video5         mscaler0-ch1设备节点
/dev/video6         mscaler0-ch2设备节点
/dev/video7         vic0设备节点
/dev/video8         mscaler0-ch0设备节点
/dev/video9         mscaler0-ch1设备节点
/dev/video10        mscaler0-ch2设备节点
```

## 3.2. 应用程序

测试应用为v4l2-ctl 源码位置:buildroot/output/build/libv4l-1.80.0/utils/v4l2-ctl

测试应用cimutils 源码位置：packages/example/App/cimutils

## 3.3. 测试方法

### 3.3.1. v4l2-ctl
* 测试isp通路并保存图像数据到文件
```
v4l2-ctl -v width=640,height=480,pixelformat="NV12" --stream-mmap=3 --stream-to="test-ch0.yuv" -d /dev/video3
```
* 其中，-d为mscaler或vic对应的设备节点 --stream-to = 图像文件名

    使用mscaler设备节点时，width、height为缩放后的宽高，pixelformat支持NV12、NV21

    使用vic设备节点时，width、height为图像的原始宽高，pixelformat支持BG10，GB10， BA10等，请根据sensor输出的格式选择

### 3.3.2. ffmpeg
* 配合LCD屏测试isp通路并预览
```
ffmpeg -pix_fmt nv12 -s 640*480 -i /dev/video3 -f fbdev /dev/fb0
```
* 其中， -i为相应的mscaler对应的设备节点 fbdev为显示屏对应的节点
        -s为缩放后的图像宽高
        -pix_fmt为格式，支持nv12格式

### 3.3.3. cimutils
* 测试isp通路并对图像进行硬件编码

1）指定设备
```
cimutils  -v isp -E helix -C -f 50.jpg -t nv12 -x 320 -y 240
```
  * 其中-v 指定isp控制器 -E 指定helix硬件编码

2）指定设备节点编号
```
cimutils  -I 1 -I 3 -C -f 40.jpg -t nv12 -x 320 -y 240
```
  * 其中-I 指定helix和isp对应的设备节点编号，具体按实际节点情况配置 

### 3.3.4. mjpeg-streamer
* 测试isp通路并通过pc预览图像
```
mjpg_streamer -i "/usr/lib/mjpg-streamer/input_syncframes.so -r 640x480 -m -camera_device0 /dev/video3" -o "/usr/lib/mjpg-streamer/output_http.so -w /usr/share/mjpg-streamer/www"
```
  * 其中 -camera_device0 指定mscaler对应的设备节点
        -r 指定图像缩放后的大小
  * 预览方法：开发板连接网络，确保开发板pc互ping成功，启动pc浏览器，浏览网址 http://“开发板IP”: 8080/stream.html， 例如 http://192.168.1.101:8080/stream.html
### 3.3.5. v4l2-sysfs-path  查询设备节点

可以通过v4l2-sysfs-path命令，查询/dev/videoX对应的具体设备，例如:

```
# v4l2-sysfs-path -d

Device platform:
        hw:0(sound card, dev 0:0) hw:0,5(pcm capture, dev 116:8) hw:0,6(pcm capture, dev 116:9) hw:0,7(pcm capture, dev 116:10) hw:0,8(pcm capture, dev 116:11) hw:0,9(pcm capture, dev 116:12) hw:0,0(pcm o
utput, dev 116:3) hw:0,1(pcm output, dev 116:4) hw:0,2(pcm output, dev 116:5) hw:0,3(pcm output, dev 116:6) hw:0,4(pcm output, dev 116:7) hw:0(mixer, dev 116:2) 
Device platform/ahb0/13070000.rotate:
        video0(video, dev 81:0) 
Device platform/ahb0/13200000.helix:
        video1(video, dev 81:1) 
Device platform/ahb0/13300000.felix:
        video2(video, dev 81:2) 
Device platform/ahb0/ahb0:isp-camera@0:
        video3(video, dev 81:3) v4l-subdev0(v4l subdevice, dev 81:4) v4l-subdev1(v4l subdevice, dev 81:5) v4l-subdev2(v4l subdevice, dev 81:6) v4l-subdev3(v4l subdevice, dev 81:7) v4l-subdev4(v4l subdevic
e, dev 81:8) 
Device platform/ahb0/ahb0:isp-camera@1:
        video4(video, dev 81:9) v4l-subdev5(v4l subdevice, dev 81:10) v4l-subdev6(v4l subdevice, dev 81:11) v4l-subdev7(v4l subdevice, dev 81:12) v4l-subdev8(v4l subdevice, dev 81:13) v4l-subdev9(v4l subd
evice, dev 81:14) 
Device virtual0:
        timer(sound timer, dev 116:33) 

```

# 4. ISP 效果调试接口

## 4.1. API
ISP提供亮度、锐度、饱和度、对比度调节的API，亮度、锐度、饱和度、对比度的最小值为0，最大值为255，默认值为128。

API使用方法详见packages/example/App/v4l2-isp-tuning/isp-tuning.c中182-224行示例。

## 4.2. 使用示例

    源码位置:packages/example/App/v4l2-isp-tuning/isp-tuning.c

    使用方法: ./v4l2-isp-tuning -w 640 -h 480 -i 3 -s 128 -c 128 -S 128 -b 128
    其中: -w 置顶图像宽度
          -h 指定图像高度
          -i 指定video节点编号
          -s 指定锐度
          -c 指定对比度
          -S 指定饱和度
          -b 指定亮度
    可通过生成的/tmp/picturex.yuv观察调节效果，或通过mscaler其他通道节点预览调节效果。

