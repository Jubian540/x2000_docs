SD Card接口驱动和应用
===================
[TOC]
<!-- toc -->

----
# 1.Emmc控制器简介

DWC_mshc是一种可高度配置和可编程的高性能移动存储主控制器，其数据传输的总接口位AXI。

* GPIO功能描述

|Name|I/O|Funtion|
|:-|:-|:-|
|MSC0_CLK|GPIO-PE00|FUNTION0|
|MSC0_CMD|GPIO-PE01|FUNTION0|
|MSC0_DATA|GPIO-(PE02~PE05)|FUNTION0|


* MSC控制器命名对应关系
  
|控制器|Base|bootrom|kernel|pm-spec|hardware-pcd|
|:-:|:-:|:-:|:-:|:-:|:-:|
|控制器0|13450000|msc0|msc0|msc0|msc0|
|控制器1|13460000|msc1|msc1|sdio|sdio|
|控制器2|13490000|msc4|msc2|msc1|msc1|

# 2.内核空间

## 2.1.驱动相关的源码

```c
drivers/mmc/host
 ├── sdhci.c                
 ├── sdhci-ingenic.c        /*控制器通过这个配置*/

```

## 2.2.设备树配置

```c
&msc2 {
        status = "okay";                 /*配置“okay”编译sd驱动，“disable”反之*/           
        pinctrl-names ="default";
        pinctrl-0 = <&msc2_4bit>;
        sd-uhs-sdr104;                   /*配置sd卡支持SDR104传输模式*/
        /*cap-mmc-highspeed;*/
        max-frequency = <200000000>;     /*通过配置该参数来指定当前模式下最大频率*/
        cd-inverted;                     /*配置支持SD卡热插拔*/ 
        bus-width = <4>;                 /*配置数据总线宽度，支持1bit、4bit*/
        voltage-ranges = <1800 3300>;

        /* special property */
        ingenic,sdr-gpios = <&gpc 0 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;   /*通过GPIO配置SD卡外部电路3.3V到1.8V电压切换*/
        ingenic,wp-gpios = <0>;
        ingneic,cd-gpios = <&gpc 12 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;   /*配置CD pin用于SD卡检测*/
        ingenic,rst-gpios = <0>;
};
```

## 2.3.驱动配置

```c
Device Drivers  --->
  <*> MMC/SD/SDIO card support  --->
            --- MMC/SD/SDIO card support
      [ ]   MMC debugging
            *** MMC/SD/SDIO Card Drivers ***
      <*>   MMC block device driver
      (8)     Number of minors per block device
      [*]     Use bounce buffer for simple hosts
      < >   SDIO UART/GPS class support
      < >   MMC host test driver
            *** MMC/SD/SDIO Host Controller Drivers ***
      <*>   Ingenic(XBurst2)  MMC/SD Card Controller(MSC) support
      -*-   Secure Digital Host Controller Interface support
```

# 3.用户空间

## 3.1.驱动加载成功

```c
[    1.575803] mmc1: new ultra high speed SDR104 SDHC card at address aaaa
[    1.593001] mmcblk0: mmc1:aaaa SC16G 14.8 GiB 
[    1.601920] Alternate GPT is invalid, using primary GPT.
[    1.607432]  mmcblk0: p1 p2 p3 p4 p5 p6 p7
```

## 3.2.设备节点

```c
/dev/mmcblk0
/dev/mmcblk0p1~p7 /*对应sd的分区*/
```

## 3.3.测试方法
1. 测试SD卡读写
* 写测试方法
      dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=100 conv=fsync
* 读测试方法
      sync; echo 3 > proc/sys/vm/drop_caches
      time dd if=/dev/mmcblk0 of=/dev/null bs=1M count=100