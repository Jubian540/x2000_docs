Camera cim驱动和应用
===================
[TOC]
<!-- toc -->

----
# 1. 模块简介

cim(camera interface module)模块的主要作用是接收前端camera传感器发送的图像信号。支持8bit dvp、mipi接口数据输入。支持YUV422、RGB888等格式。支持snapshot功能。

# 2. 内核空间

 CIM 驱动目录：

 ```
drivers/media/platform/soc_camera/ingenic/cim-v2
├── ingenic_camera.c
├── ingenic_camera.h
├── Kconfig
├── Makefile
├── mipi_csi.c
└── mipi_csi.h
 ```

## 2.1. 设备树配置

**在arch/mips/boot/dts/ingenic/x2000-v12.dtsi(m300.dtsi) ahb0下配置：**
* 以x2000-v12.dtsi为例：
```
    cim: cim@0x13060000 {
        compatible = "ingenic,x2000-cim";
        reg = <0x13060000 0x10000>;
        interrupt-parent = <&core_intc>;
        interrupts = <IRQ_CIM>;
        clocks = <&clock CLK_DIV_CIM>, <&clock CLK_GATE_CIM>, <&clock CLK_GATE_MIPI_CSI>;
        clock-names = "div_cim", "gate_cim", "gate_mipi";
        status = "disable";
    };
```
**在arch/mips/boot/dts/ingenic/板级.dts下配置：**

```
    &i2c3 {                                                              
            status = "okay";
        clock-frequency = <100000>;
        timeout = <1000>;
        pinctrl-names = "default";
        pinctrl-0 = <&i2c3_pa>;

        ov7725: dac@0x21 {
                compatible = "ovti,ov772x";
                status = "okay";
                reg = <0x21>;
                pinctrl-names = "default","cim";
                pinctrl-0 = <&cim_vic_mclk_pe>, <&cim_pa>;
                ingenic,cim-interface-vol = <CIM_INTERFACE_VOL_1V8>;

                resetb-gpios = <&gpa 9 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                vcc-en-gpios = <&gpa 15 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;
                pwdn-gpios = <&gpa 11 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;

                port {
                        ov7725_0: endpoint {
                                remote-endpoint = <&cim_0>;
                        };
                };
        };
    };

    &cim {
        status = "okay";
        port {
                cim_0: endpoint@0 {
                        remote-endpoint = <&ov7725_0>;
                        bus-width = <8>;
                };
        };
    };
```

* 需根据camera板以及所连接的sensor进行配置，上例为DB_X2000_ZEBRA_LCD_CAMERA_V2.0 链接ov7725 sensor


## 2.2 驱动配置

**make menuconfig， 然后配置：**

通用配置:

```
    --- V4L platform devices
        < >   V4L2 Driver for ingenic isp
        <*>  SoC camera support
        <*>  platform camera support
        <*>  Ingenic Camera Sensor Interface driver
        <*>  Ingenic Soc Camera Driver for X2000 && M300
        < >  Ingenic Soc camera Driver for X1000
        [ ]  Sensor support snapshot function /*该项为snapshot功能开关*/
        < >  Xilinx Video IP (EXPERIMENTAL)
```

扩展配置，根据不同的sensor选择驱动

**soc_camera sensor drivers**

```
        < > ov5640 camera support
        < > gc2155 camera support
        < > imx074 support
        < > mt9m001 support
        < > mt9m111, mt9m112 and mt9m131 support
        < > mt9t031 support
        < > mt9t112 support
        < > mt9v022 and mt9v024 support
        < > ov2640 camera support
        < > ov5642 camera support
        < > ov5645 camera support
        < > ov9281 camera support
        < > ov6650 sensor support
        < > ov772x camera support
        < > ov7725 camera support
        < > ov9640 camera support
        < > ov9740 camera support
        < > rj54n1cb0c support
        < > tw9910 support
```

# 3. 用户空间

## 3.1. 设备节点

驱动加载成功，生成的相关设备节点
```
/dev/video12    sensor设备节点
/sys/devices/platform/ahb0/13060000.cim cim文件节点
```
## 3.2. 应用程序
测试应用为cimutils 源码位置: packages/example/App/cimutils

## 3.3. 测试方法
## 3.3.1 测试cim通路
```
cimutils -v cim -C -x 320 -y 240 -t grey -f 1.raw -l 0
```
* 其中 -v指定cim控制器，-x -y 指定图像的分辨率， -t指定图像的格式，-f指定图像文件名

测试snapshot模式时，除了要在驱动配置中打开snapshot功能外，还要在sensor的驱动中配置snapshot功能

## 3.3.2 测试cim通路并对图像进行硬件编码

1）指定设备
```
cimutils  -v cim -E helix -C -f 50.jpg -t nv12 -x 320 -y 240
```
  * 其中-v 指定cim控制器 -E 指定helix硬件编码

2）指定设备节点编号
```
cimutils  -I 1 -I 3 -C -f 40.jpg -t nv12 -x 320 -y 240
```
  * 其中-I 指定helix和cim对应的设备节点编号，具体按实际节点情况配置 