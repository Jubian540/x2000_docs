VPU 驱动和应用
=============
[TOC]
<!-- toc -->

----
# 1. VPU控制器简介
X2000和M300将vpu分为felix和helix两部分：
1. felix是h264解码,包括:流解析器、运动补偿、反量化、IDCT和De-block engines的功能。
2. helix是h264编码、JPEG压缩和解压缩。

# 2. 内核空间

## 2.1. 驱动相关的源码：

```
drivers/media/platform/ingenic-vcodec
├── felix
│   ├── felix_drv.c
│   ├── felix_drv.h
│   ├── felix_ops.c
│   ├── felix_ops.h
│   ├── libh264
├── helix
│   ├── api
│   ├── default_sliceinfo.c
│   ├── h264e.c
│   ├── h264e.h
│   ├── h264enc
│   ├── helix_buf.h
│   ├── helix_drv.c
│   ├── helix_drv.h
│   ├── helix_ops.c
│   ├── helix_ops.h
│   ├── jpgd.c
│   ├── jpgd.h
│   ├── jpge
│   ├── jpge.c
│   ├── jpge.h
│   ├── Makefile
│   └── README
```

## 2.2. 设备树配置

```
&felix {
        status = "okay";
};

&helix {
        status = "okay";
};
```

## 2.3. 驱动配置

```
--- Multimedia support
    *** Multimedia core support ***
[*]   Cameras/video grabbers support
[*]   Memory-to-memory multimedia devices  --->
    --- Memory-to-memory multimedia devices
    <*>   V4L2 driver for ingenic Video Codec /*选择vpu驱动*/
    < >   SuperH VEU mem2mem video processing driver
[ ]   Autoselect ancillary drivers (tuners, sensors, i2c, frontends)
```

# 3. 用户空间

## 3.1. 驱动加载成功

```
helix-venc 13200000.helix: encoder(helix) registered as /dev/video1 /*helix加载成功*/
felix-vdec 13300000.felix: h264decoder(felix) registered as /dev/video2 /*felix加载成功*/
```

## 3.2. 设备节点

```
/dev/video1    /*helix的设备节点*/
/dev/video2    /*felix的设备节点*/
```

## 3.3. 测试方法

### 3.3.1. 测试h264解码

* 源码路径

```
packages/example/App/v4l2-h264dec
```

* 测试方法

```
v4l2_h264dec -t nv12/nv21/tile420 -v /dev/video2 -f video.mp4.dump.h264 -w 1280 -h 720

注意：
    -v 指定felix的video节点
    -f 指定要解码的码流
    -w 指定图片width
    -h 指定图片height
    -s 指定解码后的数据存为图片
    -p 指定解码后的数据在屏幕预览
```

* 标准：h264解码后lcd显示的图像完整无数据丢失

### 3.3.2. 测试h264编码

* 源码路径

```c
packages/example/App/v4l2-h264enc
```

* 测试方法

```
v4l2_h264enc -t nv12 -v /dev/video1 -f video-1280x720_nv12.yuv -w 1280 -h 720
注意：
    -v 指定felix的video节点
    -f 指定要编码的文件
    -w 指定图片width
    -h 指定图片height
```

* 标准：执行完成后查看生成的output.h264文件是否正常

### 3.3.3. 测试jpeg解码

* 源码路径

```
packages/example/App/v4l2-jpegdec
```

* 测试方法

```
v4l2_jpegdec -v /dev/video1 -f test.jpg
```

* 标准：执行完成后查看生成output.raw文件是否正常

### 3.3.4. 测试jpeg编码

* 源码路径

```
packages/example/App/v4l2-jpegenc
```

* 测试方法

```
v4l2_jpegenc -v /dev/video1 -f video-1280x720_nv12.yuv
```

* 标准：执行完成后查看生成的output.jpg文件是否正常
  
## 3.4. ffmpeg使用说明

 ### 3.4.1. ffmpeg视频流播放命令(软件解码)

   ```
   echo 6 > sys/devices/platform/ahb0/13050000.dpu/debug/color_modes /*指定LCD显示格式*/
   ffmpeg -i test-480p.mp4 -pix_fmt nv12 -f fbdev /dev/fb0 /*视频流播放*/
   ```

### 3.4.2. ffmpeg音频流播放命令

   ```
   ffmpeg -i test-480p.mp4 -f alsa default
   ```

### 3.4.3. ffmpeg视频流播放命令（硬件解码)

   ```
   ffmpeg -c:v h264_v4l2m2m -i test-480p.mp4 -pix_fmt nv12 -f fbdev /dev/fb0
   ```
### 3.4.4. ffmpeg h264 硬件编码
```
# 从摄像头获取数据，编码成h264码流
ffmpeg -s 1280*720 -pix_fmt nv12 -i /dev/video3 -f h264 -vcodec h264_v4l2m2m -b:v 400k -f h264 test.h264

ffmpeg -s 1280*720 -pix_fmt nv12 -i /dev/video3 -f h264 -vcodec h264_v4l2m2m -b:v 400k -f mp4 test.mp4

-b 指定码率400k

```

## 3.5. ffmpeg音视频同时播命令

   ```
   ffmpeg -c:v h264_v4l2m2m -i test-480p.mp4 -pix_fmt nv12 -f fbdev /dev/fb0 -f alsa default
   ```
