SPI 接口驱动和应用
================
[TOC]
<!-- toc -->

----
# 1.SSI控制器简介

SSI是一个全双工同步串行接口，可以连接到多种外部模拟-数字（A/D）转换器、音频和电信编解码器以及其他使用串行传输数据的协议。X2000和M300支持SSI摩托罗拉的串行外设接口（SPI）协议。

* GPIO功能描述

|Name|I/O|Description|
|:-|:-|:-|
|SSI_CLK |Output |Serial bit-rate clock|
|SSI_CE0 |Output |First slave select enable|
|SSI_DT |Output |Transmit data (serial data out)|
|SSI_DR |Input |Receive data (serial data in)|

* GPIO接口

```c
SSI0 PB28-31 FUNCTION1
SSI1 PC09-12 FUNCTION2
SSI0 PD08-10 PD13 FUNCTION1
SSI1 PD17-19 PD22 FUNCTION2
SSI1 PE16-18 PE21 FUNCTION1
```

# 2.内核空间

## 2.1.驱动相关的源码

```c
kernel/drivers/spi
├── ingenic_spi.c
├── ingenic_spi.h
├── spidev.c
├── spi.c
├── spi-bitbang.c
```

## 2.2. 设备树配置

```c
&spi0 {
        status = "ok";
        pinctrl-names = "default";
        pinctrl-0 = <&spi0_pb>;

        spi-max-frequency = <54000000>;
        num-cs = <2>;
        cs-gpios = <0>, <0>;
        /*cs-gpios = <&gpa 27 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>, <&gpa 27 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;*/
        ingenic,chnl = <0>;
        ingenic,allow_cs_same = <1>;
        ingenic,bus_num = <0>;
        ingenic,has_dma_support = <0>; /*选择dma，需要配置pdma通道*/
        ingenic,spi-src-clk = <1>;/*0.ext; 1.ssi*/

        /* Add SPI interface device */
        spidev: spidev@0 {
                compatible = "rohm,dh2228fv";
                reg = <0>;
                spi-max-frequency = <10000000>;
        };  
};
```

## 2.3 驱动配置

```c
Device Drivers  --->
    [*] SPI support  --->
        <*>   Ingenic SPI Controller    /*选择ssi驱动*/
        [*]     Ingenic SoC SSI controller 0 for SPI Host driver
        [ ]     Use GPIO CE on Ingenic SSI controller 0  /*当使用spi特定协议时，选用gpio模拟CS时选上*/
        <*>   User mode SPI device driver support   /*向用户提供设备设备节点*/
```

# 3.用户空间

## 3.1.驱动加载成功

```c
[    0.298196] INGENIC SSI Controller for SPI channel 0 driver register
```

## 3.2.设备节点

```c
/dev/spidev0.0
```

## 3.3.测试方法

***内核的 spi 驱动程序是基于 spi 子系统架构编写的**

* 应用程序可以使用spi_demo进行测试
* 可以在内核空间通过spi驱动操作 spi 接口
* 可以在用户空间通过 spi_ioc_transfer 操作 spi 接口
  
### 3.3.1.测试程序

1. 测试spi读写

* 源码位置

```c
kernel/Documentation/spi/spidev_test.c
```

* 测试方法

> (1) 将SSI0_DT和SSI0_DR引脚短接

```c
# ./spi_test --help
./spi_test: unrecognized option '--help'
Usage: ./spi_test [-DsbdlHOLC3]
  -D --device   device to use (default /dev/spidev1.1)
  -s --speed    max speed (Hz)
  -d --delay    delay (usec)
  -b --bpw      bits per word
  -l --loop     loopback
  -H --cpha     clock phase
  -O --cpol     clock polarity
  -L --lsb      least significant bit first
  -C --cs-high  chip select active high
  -3 --3wire    SI/SO signals shared
  -v --verbose  Verbose (show tx buffer)
  -p            Send data (e.g. "1234\xde\xad")
  -N --no-cs    no chip select
  -R --ready    slave pulls low to pause
  -2 --dual     dual transfer
  -4 --quad     quad transfer
```

>　(2) spidev_test -D /dev/spidev0.0

* 测试结果

```c
# ./spi_test -D /dev/spidev0.0
spi mode: 0x0
bits per word: 8
max speed: 500000 Hz (500 KHz)
RX | FF FF FF FF FF FF 40 00 00 00 00 95 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0 0D  | ......@....𕕕..................�.
```

２. 测试spi nor flash

```c
源码位置:
packages/example/Sample/spi/
```

# 4.使用gpio模拟spi协议

  为满足一些特殊的协议要求，也可以采用基于bitbang的gpio模拟spi功能．

## 4.1.修改设备树

* 将如下spi-gpio节点，添加设备树根节点下：

  *例: arch/mips/boot/dts/ingenic/halley5_v20.dts*

```c
/ {
        spi_gpio {
                status = "okay";
                compatible = "spi-gpio";
                #address-cells = <0x1>;
                ranges;
                gpio-sck = <&gpb 28 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                gpio-miso = <&gpb 29 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                gpio-mosi = <&gpb 30 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                cs-gpios = <&gpb 31 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                num-chipselects = <1>;
                /* clients */
                spidev1: spidev1@0 {
                        compatible = "rohm,dh2228fv";
                        reg = <0>;
                        spi-max-frequency = <500000>;
                };
        };
}
```

## 4.2．修改内核配置

```c
Device Drivers  --->
    [*] SPI support  --->
        < >   Ingenic SPI Controller    /*去掉ingenic ssi控制器配置*/
        -*-   Utilities for Bitbanging SPI masters
        <*>   GPIO-based bitbanging SPI Master
        <*>   User mode SPI device driver support   /*向用户提供设备设备节点*/
```

## 4.3.产生设备节点

* 产生设备节点`spidev%d.%d*`

```c
例：/dev/spidev32766.0
````
