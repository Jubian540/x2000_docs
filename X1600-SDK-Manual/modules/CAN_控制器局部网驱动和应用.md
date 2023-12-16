CAN 控制器局域网驱动和应用
===========================
[TOC]
<!-- toc -->

----
# 1. 模块硬件简介

X1600芯片内集成2组独立的CAN控制器；每组CAN控制器支持的功能如下：
1. 支持CAN 2.0A和CAN 2.0B协议
2. 支持最高1Mbps位速率
3. 具有4个硬件接收滤波器
4. 支持监听模式
5. 支持self-loop自测模式


# 2. 内核空间

驱动代码位置：

```
drivers/net/can/ingenic
 ├── ingenic_can.c
```


## 2.1. 设备树配置

在arch/mips/boot/dts/ingenic/板级.dts下配置：

### 2.1.1 can 控制器配置
```
can 控制器配置：

&can0 {
        status = "okay";
        pinctrl-names = "default";          
        pinctrl-0 = <&can0_pd>;         /*配置CAN TX RX 引脚*/
        ingenic,clk-freq = <12000000>;  /*配置CAN 总线时钟12M*/
};

&can1 {
        status = "okay";
        pinctrl-names = "default";
        pinctrl-0 = <&can1_pd>;         /*配置CAN TX RX 引脚*/
        ingenic,clk-freq = <12000000>;  /*配置CAN 总线时钟12M*/
};

```

## 2.2. 驱动配置

在SDK 顶层输入make kernel-menuconfig， 然后配置：
```

  │ Symbol: INGENIC_CAN [=y]
  │ Type  : tristate
  │ Prompt: ingenic on-chip CAN support
  │   Location:
  │     -> Networking support (NET [=y])
  │       -> CAN bus subsystem support (CAN [=y])
  │           -> CAN Device Drivers
  │               -> Platform CAN drivers with Netlink support (CAN_DEV [=y])
  |                   -> INGENIC CAN support
  |    Defined at drivers/net/can/ingenic/Kconfig:3
  |    Depends on: NET [=y] && CAN [=y] && CAN_DEV [=y]

```


# 3. 用户空间

## 3.1. 查看CAN设备

执行ifconfig -a 命令查看支持的网络设备

```
# ifconfig -a
can0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          NOARP  MTU:16  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:10 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
          Interrupt:48 

can1      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          NOARP  MTU:16  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:10 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
          Interrupt:49 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```


## 3.2. 测试方法

### 3.2.1. can连接方式

```
芯片内集成CAN控制器，但还不能进行通讯。CAN控制器需要连接转发器，然后发送给另一个CAN设备的转发器；
转发器连接时。CAN_H连接CAN_H,CAN_L连接CAN_L。当连线出现问题时，会报can bus error错误。
```
### 3.2.2. 正常模式测试

1. 设置位速率

设置位速率需要在CAN设备使能之前进行；

设置1Mbps位速率
```
ip link set can0 type can bitrate 1000000
ip link set can1 type can bitrate 1000000
```
该方法内核会自动计算要设置的位速率，也可手动设置位速率，参考4.2章节

2. 使能can设备

使能can0 和 can1 设备

```
ifconfig can0 up
ifconfig can0 up
```

3. 设置can1接收数据

```
candump can1 &
```

4. 设置can0发送数据

分别发送标准帧，扩展帧和远程帧

```
cansend can0 -i 0x123 0x11 0x22 0x33 0x44 0x55 0x66 0x77 0x88
cansend can0 -i 0x12345 -e 0x11 0x22 0x33 0x44 0x55 0x66 0x77 0x88
cansend can0 -i 0x123  -r 0x11 0x22 0x33 0x44 0x55 0x66 0x77 0x88
```

5. 关闭can设备

关闭can0 和 can1 设备

```
ifconfig can0 down
ifconfig can0 down
```

### 3.2.3. 查看can状态

查询 can0 设备的参数设置
```
ip -details link show can0
```

在设备工作中，查询can0的工作状态：
```
ip -details -statistics link show can0
```

### 3.2.4. 使用self-loop方式测试

该测试会在控制器内部将tx和rx进行短接，将tx的数据直接送到rx上。
测试不需要连接can设备。

1. 设置can位速率并启动loop模式
```
ip link set can0 type can bitrate 1000000 loopback on
```

2. 启动can
```
ifconfig can0 up
```

3. 设置can接收
```
candump can0 &
```

4. can发送数据
```
cansend can0 -i 0x123 0x11 0x22 0x33 0x44 0x55 0x66 0x77 0x88
```
5. 关闭can设备
```
ifconfig can0 down
```

### 3.2.5. 使用监听模式

监听模式需要最少3个CAN设备在总线上连接；其中2个正常通讯，1个设置为监听。

1. 设置can位速率并启动listen模式
```
ip link set can0 type can bitrate 1000000 listen-only on
```

2. 启动can
```
ifconfig can0 up
```

3. 设置can接收
当can总线上有数据发送时，该监听设备会接收到数据

```
candump can0 &
```

4. 关闭can设备
```
ifconfig can0 down
```

# 4. CAN 位速率计算

## 4.1 CAN 位速率计算

```
rate = freq / ((tseg1 + tseg2 + 1) × brp)
rate:CAN要设置的速率
freq：CAN总线的时钟频率（在设备树中配置默认CAN总线时钟频率为12M）
tseg1:相位缓冲段1,对应CAN寄存器CANBTR.TSEG1[3-0] + 1
tseg2:相位缓冲段2,对应CAN寄存器CANBTR.TSEG2[2-0] + 1
brp:传播时间段,对应CAN寄存器CANBTR.CANCS[5-0] + 1
```

## 4.2 CAN 位速率手动设置

手动设置can位速率需要确定 brp, tseg1, tseg2, sjw 等参数的值
其中，sjw通常设置1或者2；
brp, tseg1, tseg2 根据4.1章节计算得出。计算时总体配置保持 tseg1 >= tseg2 tseg2 >= 2sjw

配置方法如下(以设置1Mbps为例)
```
ip link set can0 type can tq 250 prop-seg 0 phase-seg1 2 phase-seg2 1 sjw 1

```
其中 tq 根据 brp 得出，计算方法为：tq = (brp x 1G) / freq



# 5. 常见错误

## 5.1. bus error 

错误的打印如下
```
[ 9537.660906] ingenic_can 13560000.can can0: can0 Bus Error
[ 9537.670627] ingenic_can 13560000.can can0: can0 ingenic_can stop !!!, please reset can.
[ 9537.678980] ingenic_can 13560000.can can0: can0 Bus Off
[ 9537.684580] ingenic_can 13560000.can can0: can0 Error Warning
```
该问题通常是由于can的接线有问题；解决方法如下；
1. 检查can的接线是否松动，确保接线正确且无松动。
2. 检查终端120Ω电阻使用是否正确，只有最远端的2个can设备需要使用120Ω终端电阻，其他的不需要。
3. 检查连线的长度是否合理，一般can位速率越高，传输距离越短，传输线过长时可能会出问题。
4. 排查问题后，重启can设备进行测试。





