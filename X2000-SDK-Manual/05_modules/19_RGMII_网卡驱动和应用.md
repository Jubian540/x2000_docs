RGMII 网卡驱动和应用
==================
[TOC]
<!-- toc -->

----

# 1. 模块简介
X2000和M300芯片有2个完全独立的gmac控制器， 控制器支持RGMII和RMII接口。

RGMII（Reduced Gigabit Media Independent Interface）是Reduced GMII（吉比特介质独立接口）。RGMII均采用4位数据接口，工作时钟125MHz，并且在上升沿和下降沿同时传输数据，因此传输速率可达1000Mbps。RGMII数据结构符合IEEE以太网标准，接口定义见IEEE 802.3-2000, RGMII支持10/100/1000兆的总线接口速度。

RMII \(Reduced Media Independent Interface\) 简化媒体独立接口,是IEEE 802.3u标准中除MII接口之外的另一种实现。RMII支持10兆和100兆的总线接口速度。

# 2. u-boot中使用网络

uboot只支持一个gmac控制器工作，需要在对应板级头文件（include/configs/板级.h）中配置选择的控制器和对应控制器的接口模式。

配置文件include/configs/板级.h 如下：
```
/* Select GMAC Controller */
#define CONFIG_GMAC1          /*根据实际测试，配置对应的GMAC1、GMAC0*/

/* Select GMAC Interface mode */
#define CONFIG_RGMII          /*根据实际测试，配置对应RGMII、RMII*/

CONFIG_GMAC_PHY_RESET        /* 按照电路图配置phy的reset gpio */

CONFIG_GMAC_TX_CLK_DELAY     /* RGMII模式时 设置tx_clk delay */
CONFIG_GMAC_RX_CLK_DELAY     /* RGMII模式时 设置rx_clk delay */
```

如果kernel配置双网络控制器，当通过uboot传参启动网络文件系统时在设置bootargs时需要指定启动网络文件系统的的设备，设置形式如下：
```
setenv bootargs 'console=ttyS2,115200 mem=128M@0x0 ip=192.168.4.43:192.168.4.1:192.168.4.1:255.255.255.0::eth0:off nfsroot=192.168.4.93:/home/user/x2000-v12/system rw'  /*根据选择的网口，修改eth0:off或eth1:off*/
```

# 3. 内核空间

驱动代码位置：

```
drivers/net/ethernet/ingenic
 ├── ingenic_mac.c
 ├── synopGMAC_Dev.c
 ├── synopGMAC_plat.c
 ```

## 3.1. 设备树配置

RGMII在arch/mips/boot/dts/ingenic/板级.dts下配置

### 3.1.1. 设备mac0的rgmii配置

```
&mac0 {
        pinctrl-names = "default", "reset";
        pinctrl-0 = <&mac0_rgmii_p0_normal>, <&mac0_rgmii_p1_normal>, <&mac0_phy_clk>; /*如果使用芯片内部时钟作为phy芯片的工作时钟需要配置mac0_phy_clk*/
        pinctrl-1 = <&mac0_rgmii_p0_rst>, <&mac0_rgmii_p1_normal>;
        status = "okay";        /*根据需求配置okay 或 disable*/
        ingenic,rst-gpio = <&gpb 0 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
        ingenic,rst-ms = <10>;   /*复位时间*/
        ingenic,rst-delay-ms = <15>; /*复位后延时时间，延时期间不对phy进行任何操作*/
        ingenic,mac-mode = <RGMII>;
        ingenic,mode-reg = <0xb00000e4>;
        ingenic,rx-clk-delay = <0x2>;
        ingenic,tx-clk-delay = <0x3f>;
        ingenic,phy-clk-freq = <25000000>; /*如果使用芯片供给phy的工作时钟, 需要配置phy芯片工作时钟频率*/
};
```

### 3.1.2. 设备mac1的rgmii配置:

```
&mac1 {
        pinctrl-names = "default", "reset";
        pinctrl-0 = <&mac1_rgmii_p0_normal>, <&mac1_rgmii_p1_normal>， <&mac1_phy_clk>;  /*如果使用芯片内部时钟作为phy芯片的工作时钟需要配置mac1_phy_clk*/
        pinctrl-1 = <&mac1_rgmii_p0_rst>, <&mac1_rgmii_p1_normal>;
        status = "okay";         /*根据需求配置okay 或 disable*/
        ingenic,rst-gpio = <&gpb 26 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
        ingenic,rst-ms = <10>;   /*复位时间*/
	ingenic,rst-delay-ms = <15>; /*复位后延时时间，延时期间不对phy进行任何操作*/
        ingenic,mac-mode = <RGMII>;
        ingenic,mode-reg = <0xb00000e8>;
        ingenic,rx-clk-delay = <0x2>;
        ingenic,tx-clk-delay = <0x3f>;
        ingenic,phy-clk-freq = <25000000>; /*如果使用芯片供给phy的工作时钟, 需要配置phy芯片工作时钟频率*/
};
```

**2 、RMII在arch/mips/boot/dts/ingenic/板级.dts下配置：**

### 3.1.3. 设备mac0的rmii配置: 
```
设备mac0的rmii配置:
&mac0 {
        pinctrl-names = "default", "reset";
        pinctrl-0 = <&mac0_rmii_p0_normal>, <&mac0_rmii_p1_normal>，<&mac0_phy_clk>; /*如果使用芯片供给phy芯片工作时钟，需要添加mac0_phy_clk gpio配置*/;
        pinctrl-1 = <&mac0_rmii_p0_rst>, <&mac0_rmii_p1_normal>;
        status = "okay";       /*根据需求配置okay 或 disable*/
        ingenic,rst-gpio = <&gpb 0 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
        ingenic,rst-ms = <10>;   /*复位时间*/
	ingenic,rst-delay-ms = <15>; /*复位后延时时间，延时期间不对phy进行任何操作*/
        ingenic,mac-mode = <RMII>;
        ingenic,mode-reg = <0xb00000e4>;
        ingenic,phy-clk-freq = <25000000>; /*如果使用芯片供给phy的工作时钟, 需要配置phy芯片工作时钟频率*/
};
```

### 3.1.4. 设备mac1的rmii配置 
```
设备mac1的rmii配置:
&mac1 {
        pinctrl-names = "default", "reset";
        pinctrl-0 = <&mac1_rmii_p0_normal>, <&mac1_rmii_p1_normal>， <&mac1_phy_clk>;  /*如果使用芯片内部时钟作为phy芯片的工作时钟需要配置mac1_phy_clk*/
        pinctrl-1 = <&mac1_rmii_p0_rst>, <&mac1_rmii_p1_normal>;
        status = "disable";      /*根据需求配置okay 或 disable*/
        ingenic,rst-gpio = <&gpb 26 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
        ingenic,rst-ms = <10>;   /*复位时间*/
	ingenic,rst-delay-ms = <15>; /*复位后延时时间，延时期间不对phy进行任何操作*/
        ingenic,mac-mode = <RMII>;
        ingenic,mode-reg = <0xb00000e8>;
        ingenic,phy-clk-freq = <25000000>; /*如果使用芯片供给phy的工作时钟, 需要配置phy芯片工作时钟频率*/
};
```

## 3.2. 需要根据具体设备更改的配置选项

依据开发板设计更改mac设备对应phy的复位gpio,复位有效电平,复位需要的保持的时间

如果工作在RGMII模式下需要配置TXCLK和RXCLK的delay,配置的精度是19.5ps,配置值范围0-128,delay的时间范围0-2.5ns.具体配置值需要由phy来确定,TXCLK和RXCLK和data保证2ns左右的delay,所以如果phy端已经做了delay,mac控制器端就不需要设置或补足差值

# 3.3. 驱动配置

**1、在顶层执行 make kernel-menuconfig， 然后配置：**

```
--- Network device support
[*]   Network core driver support
[*]   Ethernet driver support  --->
    --- Ethernet driver support
    <*> ingenic on-chip MAC support
    [*] Ingenic mac dma interfaces
            Ingenic mac dma bus interfaces (MAC_AXI_BUS)  --->
<*>   PHY Device support and infrastructure  --->

可选配置选项：
(8192)  Ingenic gmac receive descriptor number[80..10240] /*gmac 控制器接收数据包的缓存数量，缓存越大可以减少满载时的丢包率，经过测试设置8192可以避免控制器层出现丢包, 这种情况运行时一个网卡的动态内存会使用16M-32M*/
[*] Dual core mutex transmission   /* 使用双网卡时配置， 配置后会把双网卡处接收数据协议栈的过程互斥处理，可以减少指令cache的miss率*/

```

# 4. 用户空间

## 4.1. 测试网络是否正常

执行ifconfig -a 命令查看支持的网络设备

```
# ifconfig -a
eth0      Link encap:Ethernet  HWaddr 1E:01:7F:E6:D3:26     /*设备树配置GMAC0后显示设备接口eth0*/
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

eth1      Link encap:Ethernet  HWaddr 3E:4C:9D:F4:74:4B     /*设备树配置GMAC1后显示设备接口eth
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          LOOPBACK  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

配置网络

```
ifconfig eth0 IP                /*根据实际选择的网口配置eth0或eth1*/
```

使用ping命令查看网络连接情况

```
ping IP -I eth0                /*通过-I 选项可以指定ping 命令使用的网口*/
```

## 4.2 网络性能测试

找一台有千兆以太网功能的电脑，使用网线直连，使用iperf3命令测试，电脑当做服务端，开发板做客户端

电脑端执行命令
```
iperf3  -s
```
测试单向发送性能
```
iperf3  -c 192.168.4.105 -u -b 1000M -l 65507 -t 10
```
测试单向接收性能
```
iperf3  -c 192.168.4.105 -u -b 1000M -l 65507 -t 10 -R
```
应用层提高网络性能的方法，以下方法根据具体应用场景使用，设置不合理反而会影响测试结果
```
sysctl -w net.core.rmem_default=10485760  /*增加网络核心层接收的缓存, 此设置会使用10M内存， 双网口桥接不使用*/
sysctl -w net.core.rmem_max=10485760 /*增加网络核心层接收的缓存*/

指定内核处理接收网络协议栈使用的cpu核，指定后可以避免内核动态切换过程造成双核使用不均衡造成的性能不稳定：
echo 0 > /proc/irq/61/smp_affinity_list   /*指定cpu0处理gmac1接收的数据*/
echo 1 > /proc/irq/63/smp_affinity_list   /*指定cpu1处理gmac0接收的数据， gmac0中断号63  gmac1中断号61*/

sysctl -w net.core.netdev_max_backlog=4000 /*增加转接过程中数据包缓存长度，可以减少数据抖动但消耗内存越大*/

echo 2 > /sys/class/net/eth0/queues/rx-0/rps_cpus /*指定cpu1处理eth0接收到数据包，输入1指定cpu0，2指定cpu1, 3指定cpu0-1*/
```

## 4.3. 1588硬件时间戳测试
使用2块带有以太网接口的开发板，使用网线直连。

使用CONFIG_INGENIC_GMAC_USE_HWSTAMP宏配置kernel的1588功能。

文件系统中需要有测试1588功能的测试程序，默认使用linuxptp测试，linuxptp测试工程在manhatton工程的buildroot中，关于linuxptp使用方法可以参照网络资料。

网络正常工作后：
一个开发板做slave执行命令：
```
ptp4l -E -4 -H -i eth0 -s -m
```

另一个开发板做master执行命令：
```
ptp4l -E -4 -H -i eth0 -m
```