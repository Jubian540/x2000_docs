linphone使用说明文档
==================
[TOC]
<!-- toc -->

----

# 1. 编译
```
#make linphone 
```
# 2. 使用

## 2.1. 开发板端使用
获取开发板的IP地址作为设备号码。

启动linphone相关命令
```
1.linphonec -C 使能摄像头，但不将图像显示到LCD上
2.linphonec -D 使能LCD显示
3.linphonec -V 使能LCD显示和摄像头功能，类似于linphone -C -D的组合
4.answer   接听当前默认的等待电话
```
## 2.2. 手机端使用
应用商店下载，或者登录http://www.linphone.org/ 官网下载。
在拨号界面进行拨号，例如：root@192.168.x.x

## 2.3. 暂不支持开发板到开发板端间的视频通信
