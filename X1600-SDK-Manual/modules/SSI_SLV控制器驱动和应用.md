SSI_SLV 控制器驱动和应用
===========================
[TOC]
<!-- toc -->

----
# 1. 模块硬件简介

SSI-SLV是一个全双工同步串行从设备接口，可以连接到多种外部模拟-数字（A/D）转换器、音频和电信编解码器以及其他使用串行传输数据的协议。支持SSI摩托罗拉的串行外设接口（SPI）协议。

* GPIO功能描述

|Name|I/O|Description|
|:-|:-|:-|
|SSI_SLV_CLK |Output |Serial bit-rate clock|
|SSI_SLV_CE0 |Output |First slave select enable|
|SSI_SLV_DT |Output |Transmit data (serial data out)|
|SSI_SLV_DR |Input |Receive data (serial data in)|

SSI0 PA28-31 FUNCTION1

# 2. 内核空间

驱动代码位置：

```
drivers/spi/
 ├── ingenic_slv.c
 ├── ingenic_slv.h

```


## 2.1. 设备树配置

在arch/mips/boot/dts/ingenic/板级.dts下配置：

```
&spi_slv0 {
        status = "okay";
        pinctrl-names = "default";
        pinctrl-0 = <&spi_slv_pa>;
        ingenic,has_dma_support = <1>;
};
```

## 2.2. 驱动配置

在SDK 顶层输入make kernel-menuconfig， 然后配置：
```

 Device Drivers  --->
    [*] SPI support  --->
        <*>   Ingenic series SPI slave driver
        [*]     Ingenic SoC SSI_SLV controller 0 for SPI SLV Host driver
```
# 3. 用户空间

## 3.1.驱动加载成功

```c
  INGENIC SSI slaver Controller ok!!
```

## 3.2.设备节点

```c
/dev/spi_slv
```

## 3.3.测试方法

***内核的 ssi_slv 驱动程序作为从设备驱动，测试时需要主设备提供ce片选与时钟**

* 应用程序可以使用spi_demo进行测试
* 可以在内核空间通过spi驱动操作 spi 接口
* 可以在用户空间通过 spi_ioc_transfer 操作 spi 接口
* 可以选择ssi模块作为主设备提供片选与时钟

### 3.3.1.测试程序

1. 测试spi读写

* 源码位置

```c
packages/example/Sample/spi_slv/spislaver_test.c
```

* 测试方法

> (1) 配置ssi主设备

**设备树配置
```c
&spi0 {
        &spi0 {
        status = "okay";
        pinctrl-names = "default";
        pinctrl-0 = <&spi0_pb>;
        spi-max-frequency = <54000000>;
        num-cs = <2>;
        ingenic,chnl = <2>;
        ingenic,allow_cs_same = <1>;
        ingenic,bus_num = <0>;
        ingenic,has_dma_support = <0>;
        ingenic,spi-src-clk = <0>;
        spidev: spidev@0 {
                compatible = "rohm,dh2228fv";
                reg = <0>;
                spi-max-frequency = <10000000>;
        };
};
**驱动配置
```c
Device Drivers  --->
    [*] SPI support  --->
        <*>   Ingenic SPI Controller    /*选择ssi驱动*/
        [*]     Ingenic SoC SSI controller 0 for SPI Host driver
        <*>   User mode SPI device driver support   /*向用户提供设备设备节点*/
```
**驱动相关的源码

```c
kernel/drivers/spi
├── ingenic_spi.c
├── ingenic_spi.h
├── spidev.c
├── spi.c
├── spi-bitbang.c
```
**设备节点
```c
/dev/spidev0.0
```
**测试程序源码位置

```c
kernel/Documentation/spi/spidev_test.c
```

(2) 将SSI_SLV_DT和SSI_SLV_DR引脚短接,SSI_SLV_CLK连接SSI0_CLK，SSI_SLV_CE连接SSI0_CE。

```c
# ./slvtest -help
./slvtest: invalid option -- 'h'
Usage: ./slvtest [-DsbdlHOLC3]
  -p --path   device to use (default /dev/spidev1.1)
  -b --bpw      bits per word 
  -l --loop     loopback
  -i            Send data (e.g. "1234\xde\xad")
  -d --dma      use dma
```
(3) ./slv_test -l

```c
>>>bits_prt_word : 8
>>>tranfer mode : cpu
>>>len: 32
[17880.956803] slv ready
[17880.962866] write data 0
[17880.966667] write data 1
[17880.969299] write data 2
[17880.972785] write data 3
[17880.975429] write data 4
[17880.978061] write data 5
[17880.981283] write data 6
[17880.983945] write data 7
[17880.986597] write data 8
[17880.989228] write data 9
[17880.992425] write data 10
[17880.995154] write data 11
[17880.997877] write data 12
[17881.000598] write data 13
[17881.003904] write data 14
[17881.006653] write data 15
[17881.009396] write data 16
[17881.012742] write data 17
[17881.015495] write data 18
[17881.018216] write data 19
[17881.021477] write data 20
[17881.024227] write data 21
[17881.026968] write data 22
[17881.029687] write data 23
[17881.033020] write data 24
[17881.035770] write data 25
[17881.038513] write data 26
[17881.041651] write data 27
[17881.044400] write data 28
[17881.047121] write data 29
[17881.049840] write data 30
[17881.053176] write data 31
```
（4）打开新终端,执行ssi主设备测试程序提供时钟与片选
```c
#adb shell
#cd
#./spi_test -D /dev/spidev0.0 -O -H
```
（5）SSI_SLV测试程序追加打印
```c
[18309.917179] write number: 32 ; reads number:32
[18309.922251] read_c[0] = 0
[18309.925002] read_c[1] = 1
[18309.927724] read_c[2] = 2
[18309.930444] read_c[3] = 3
[18309.933745] read_c[4] = 4
[18309.936498] read_c[5] = 5
[18309.939220] read_c[6] = 6
[18309.942455] read_c[7] = 7
[18309.945186] read_c[8] = 8
[18309.947886] read_c[9] = 9
[18309.950608] read_c[10] = 10
[18309.954085] read_c[11] = 11
[18309.957016] read_c[12] = 12
[18309.959918] read_c[13] = 13
[18309.963339] read_c[14] = 14
[18309.966249] read_c[15] = 15
[18309.969168] read_c[16] = 16
[18309.972594] read_c[17] = 17
[18309.975524] read_c[18] = 18
[18309.978446] read_c[19] = 19
[18309.981774] read_c[20] = 20
[18309.984703] read_c[21] = 21
[18309.987625] read_c[22] = 22
[18309.990526] read_c[23] = 23
[18309.993961] read_c[24] = 24
[18309.996891] read_c[25] = 25
[18309.999792] read_c[26] = 26
[18310.003169] read_c[27] = 27
[18310.006099] read_c[28] = 28
[18310.009000] read_c[29] = 29
[18310.012579] read_c[30] = 30
[18310.015492] read_c[31] = 31
[18310.018394] @@@@@@ SLV LOOP TEST OK  @@@@@@
```
２. 测试ssi与ssi_slv通信

(1) 将SSI_SLV_DT和SSI0_DR引脚连接,SSI_SLV_DR和SSI0_DT引脚连接，SSI_SLV_CLK连接SSI0_CLK，SSI_SLV_CE连接SSI0_CE。

(2) 执行命令顺序同上

(3) ssi_slv打印
```c
# ./slvtest 
>>>bits_prt_word : 8
>>>tranfer mode : cpu
>>>len: 32
0x02 0x03 0x04 0x05 0x06 0x07 0x07 0x08 0x09 0x10 0x11 0x12 0x13 0x14 0x15 0x16 
0x17 0x18 0x19 0x20 0x21 0x22 0x23 0x24 0x25 0x26 0x27 0x28 0x29 0x30 0x31 0x32 
```
(4) ssi打印
```c
 # ./spidev_test -D /dev/spidev0.0 -O -H 
spi mode: 0x3
bits per word: 8
max speed: 500000 Hz (500 KHz)
RX | FC FD FE FF 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10 11 12 13 14 15 16 17 18 19 1A 1B
```