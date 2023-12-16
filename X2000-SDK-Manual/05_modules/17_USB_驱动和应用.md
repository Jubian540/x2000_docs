USB 驱动和应用
=============
[TOC]
<!-- toc -->

----
# 1. 模块简介

通用串行总线 \(Universal Serial Bus，USB\) 是一种新兴的并逐渐取代其他接口标准的数据通信方式，由 Intel、Compaq、Digital、IBM、Microsoft、NEC 及 Northern Telecom 等计算机公司和通信公司于 1995 年联合制定，并逐渐形成了行业标准。

USB 总线作为一种高速串行总线，其极高的传输速度可以满足高速数据传输的应用环境要求，且该总线还兼有供电简单（可总线供电）、安装配置便捷（支持即插即用和热插拔）、 扩展端口简易（通过集线器最多可扩展 127 个外设）、传输方式多样化（4 种传输模式），以及兼容良好（产品升级后向下兼容）等优点。

USB OTG控制器实现了USB主机和多种可同时访问的便携式外围设备之间进行串行数据交换。

# 2. 内核空间

控制器代码结构:

```txt
drivers/usb/dwc2/
├── core.c
├── core.h
├── core_intr.c
├── debugfs.c
├── debug.h
├── gadget.c
├── hcd.c
├── hcd_ddma.c
├── hcd.h
├── hcd_intr.c
├── hcd_queue.c
├── hw.h
├── Kconfig
├── Makefile
├── pci.c
└── platform.c
```

## 2.1 设备树配置

**１、arch/mips/boot/dts/ingenic/x2000-v12.dtsi(m300.dtsi)**

```txt
 otg_phy: otg_phy {
                        compatible = "ingenic,innophy";
                };
```

```
otg: otg@0x13500000 {
                        compatible = "ingenic,dwc2-hsotg";
                        reg = <0x13500000 0x40000>;
                        interrupt-parent = <&core_intc>;
                        interrupts = <IRQ_OTG>;
                        ingenic,usbphy=<&otg_phy>;
                        status = "disabled";
                };
```

**２、arch/mips/boot/dts/ingenic/zebra.dts**

```txt
&otg {
        g-use-dma;
        dr_mode = "otg";
        status = "okay";
};
```

```txt
&otg_phy {
        dr_mode = "otg";
        compatible = "ingenic,innophy", "syscon";
        ingenic,id-dete-gpio = <&gpc 27 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;
        ingenic,vbus-dete-gpio = <&gpd 17 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;
        ingenic,drvvbus-gpio = <&gpe 22 GPIO_ACTIVE_HIGH INGENIC_GPIO_NOBIAS>;
        status = "okay";
};

```

# 3. 控制器OTG使用说明

## 3.1 Host和Device切换说明
### 3.1.1 软件切换方式
#### a. Host模式切换
```txt
        echo host > /sys/devices/platform/apb/10000000.otg_phy/sw_switch_hsotg
```
#### b. Device模式切换
```txt
	echo device > /sys/devices/platform/apb/10000000.otg_phy/sw_switch_hsotg
```

### 3.1.2 硬件切换方式
```txt
	通过改变USB_ID电平来切换Host和Device模式
```

## 3.2 [USB_Host驱动和应用](/05_modules/17_USB/1_USB_Host驱动和应用.md)
## 3.3 [USB_Device驱动和应用](/05_modules/17_USB/2_USB_Device驱动和应用.md)
