v4l2rtspserver使用说明文档
=======
[TOC]
<!-- toc -->

----

# 应用简介

v4l2rtspserver 应用支持从标准v4l2设备获取H264/JPEG 视频数据，通过标准alsa设备获取音频数据，通过live555开源服务，提供rtsp server/httpserver的功能

在X2000 平台实现的方案是，

Camera + H264 encoder  --> v4l2loopback --> v4l2rtspserver。

依赖的驱动：
* camera驱动
* h264/jpeg编码
* v4l2loopback 驱动
* alsa音频驱动
* 网络/wifi/adb


# 支持功能

# 源码结构

# 扩展功能

# 使用方法

## 开发板端运行
```
ffmpeg -s 1280*720 -pix_fmt nv12 -i /dev/video4 -vcodec h264_v4l2m2m -f v4l2 /dev/video11
v4l2rtspserver /dev/video11  #开启视频服务器
```
其中，ffmpeg命令用于将camera节点(/dev/video4)数据压缩成h264数据，给到v4l2loopback(/dev/video11)节点。

具体使用的节点需要根据实际情况更改。

## rtspclient测试方法

网络配置大致有三种方式，任选一可以实现功能测试。

### 1. adb 端口转发
为了方便测试，可以
```
adb forward tcp:8554 tcp:8554
```
打开浏览器，输入
```
rtsp://127.0.0.1:8554/unicast
```
可以看到预览画面即正常。

### 2. eth 网络配置


### 3. wifi 网路配置

# 程序缺陷

* 1 ~ 2秒延时
* 