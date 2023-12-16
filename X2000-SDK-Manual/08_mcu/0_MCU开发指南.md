MCU开发指南
===========
[TOC]
<!-- toc -->

----

![](/assets/X2000-arch.png)

MCU 为集成在SOC 内部的xburst@0 CPU， 运行频率300MHz。
主CPU 可以通过中断与MCU 进行通信。


MCU SDK 主要分为两个部分。

1. MCU 固件开发，运行在MCU上的程序
2. MCU 固件加载器(loader)
3. 主CPU调用MCU功能

# 1. MCU 固件介绍
libmcu-bare为 MCU 固件源码工程，在MCU上运行的程序可以在该库中进行开发。

**路径:**
工程目录顶层 development/libmcu-bare

```
.
├── app
├── config.mk
├── hw
├── include
├── lib
├── Makefile
└── scripts
```

主要分为 app, hw, lib, scripts目录

* app 目录为MCU应用程序
* hw 包含SOC 中各控制器驱动
* lib 包含mcu启动，中断处理，以及其他软件库
* scripts 包含常用工具，链接脚本配置



# 2. MCU 加载器介绍

MCU 加载器为内核驱动，所在目录

> drivers/remoteproc/ingenic/

可以通过以下配置选择:

```
CONFIG_INGENIC_RPROC:                             
                                                  
Say y or m here to support ingenic mcu remoteproc.
This can be either built-in or a loadable module. 
If unsure say N.                                  
                                                  
                                                  
Symbol: INGENIC_RPROC [=y]                        
Type  : tristate                                  
Prompt: ingenic remoteproc support                
  Location:                                       
    -> Device Drivers                             
      -> Remoteproc drivers                       
  Defined at drivers/remoteproc/Kconfig:80        
  Depends on: HAS_DMA [=y]                        
  Selects: REMOTEPROC [=n]                        

```

devicetree 配置

> arch/mips/boot/dts/ingenic/x2000-v12.dtsi

```
mcu: mcu@0x13420000 {
        compatible = "ingenic,x2000-mcu";
        reg = <0x13420000 0x10000>;
        interrupt-parent = <&core_intc>;
        interrupt-names = "pdmam";
        interrupts = <IRQ_PDMAM>;
        ingenic,tcsm_size = <16384>;
};

```


# 3. 加载MCU 固件
在开发板上执行:

> 1. 通过libmcu-bare编译好的固件放在 lib/firmware目录下面
> 2. echo 1 > /sys/devices/platform/ahb2/13420000.mcu/load_fw 加载固件。


关闭mcu:
> echo 1 >  /sys/devices/platform/ahb2/13420000.mcu/shutdown


