# MCU使用方法


MCU 为集成在SOC 内部的xburst@I CPU， 运行频率300MHz。
主CPU 可以通过中断与MCU 进行通信。


MCU SDK 主要分为两个部分。

1. MCU 固件开发，运行在MCU上的程序
2. MCU 固件加载器(loader)和主CPU调用MCU功能

## 1. MCU 固件介绍
libmcu-bare为 MCU 固件源码工程，在MCU上运行的程序可以在该库中进行开发。

**路径:**
工程目录顶层 development/libmcu-bare

```
.
|-- Config.in
|-- Makefile
|-- README.md
|-- config.h
|-- configs
|   |-- x1000_test_defconfig
|   `-- x2000_test_defconfig
|-- drivers
|   |-- Config.in
|   |-- Makefile
|   `-- console
|       |-- Config.in
|       `-- uart_console.c
|-- include
|   |-- always_compile.h
|   |-- assert.h
|   |-- bit_field2.h
|   |-- bits_opt.h
|   |-- delay.h
|   |-- driver
|   |   |-- clk.h
|   |   |-- console.h
|   |   |-- gpio.h
|   |   |-- irq.h
|   |   |-- systick.h
|   |   |-- tcu.h
|   |   `-- uart.h
|   `-- ring_mem.h
|-- lib
|   |-- Config.in
|   |-- Makefile
|   |-- assert.c
|   |-- printf.c
|   |-- ring_mem.c
|   |-- simple_printf.c
|   |-- string.c
|   |-- xformat.c
|   `-- xformatc.h
|-- options
|   |-- Config.in
|   `-- Makefile
|-- soc
|   |-- Config.in
|   |-- Makefile
|   |-- x1000
|   |   |-- Config.in
|   |   |-- Makefile
|   |   |-- clk
|   |   |   |-- clk.c
|   |   |   `-- clk_gate.c
|   |   |-- config_uart.in
|   |   |-- gpio
|   |   |   `-- gpio.c
|   |   |-- include
|   |   |   |-- asm
|   |   |   |   |-- addrspace.h
|   |   |   |   |-- asm.h
|   |   |   |   |-- cacheops.h
|   |   |   |   |-- elf.h
|   |   |   |   |-- mipsregs.h
|   |   |   |   |-- mipsregs_others.h
|   |   |   |   |-- regdef.h
|   |   |   |   `-- sgidefs.h
|   |   |   |-- cpu
|   |   |   |   |-- ffs.h
|   |   |   |   |-- host_cpu.h
|   |   |   |   |-- io.h
|   |   |   |   |-- irq.h
|   |   |   |   `-- irqflags.h
|   |   |   `-- soc
|   |   |       |-- base.h
|   |   |       |-- clk.h
|   |   |       |-- cpm.h
|   |   |       `-- gpio.h
|   |   |-- init
|   |   |   |-- exception.S
|   |   |   |-- host_cpu.c
|   |   |   |-- init.c
|   |   |   |-- irq.c
|   |   |   `-- start.S
|   |   |-- intc
|   |   |   `-- intc.c
|   |   |-- libgcc.a
|   |   |-- uart
|   |   |   |-- uart.c
|   |   |   `-- uart_regs.h
|   |   `-- x1000.lds
|   `-- x2000
|       |-- Config.in
|       |-- Makefile
|       |-- clk
|       |   |-- clk.c
|       |   `-- clk_gate.c
|       |-- config_uart.in
|       |-- gpio
|       |   `-- gpio.c
|       |-- include
|       |   |-- asm
|       |   |   |-- addrspace.h
|       |   |   |-- asm.h
|       |   |   |-- cacheops.h
|       |   |   |-- elf.h
|       |   |   |-- mipsregs.h
|       |   |   |-- mipsregs_others.h
|       |   |   |-- regdef.h
|       |   |   `-- sgidefs.h
|       |   |-- cpu
|       |   |   |-- ffs.h
|       |   |   |-- host_cpu.h
|       |   |   |-- io.h
|       |   |   |-- irq.h
|       |   |   `-- irqflags.h
|       |   `-- soc
|       |       |-- base.h
|       |       |-- clk.h
|       |       |-- cpm.h
|       |       |-- gpio.h
|       |       `-- tcu.h
|       |-- init
|       |   |-- exception.S
|       |   |-- host_cpu.c
|       |   |-- init.c
|       |   |-- irq.c
|       |   `-- start.S
|       |-- intc
|       |   `-- intc.c
|       |-- libgcc.a
|       |-- systick
|       |   `-- systick.c
|       |-- tcu
|       |   |-- tcu.c
|       |   `-- tcu_regs.h
|       |-- uart
|       |   |-- uart.c
|       |   `-- uart_regs.h
|       `-- x2000.lds
|-- tools
|   |-- build_package.mk
|   |-- check_config.sh
|   |-- configparser
|   |-- include_package.mk
|   `-- msg.mk
`-- vendor
    |-- Config.in
    |-- Makefile
    `-- vendor.c
```
## 2. MCU 加载器介绍

### 1. 加载mcu固件的代码在工程目录顶层 development/mcu-host,编译生成mcu-host。

### 2. MCU 加载器内核驱动，所在目录

> drivers/remoteproc/ingenic/mcu_driver

可以通过以下配置选择:

```
CONFIG_INGENIC_MCU_RPROC:

Say y or m here to support ingenic mcu remoteproc.
This can be either built-in or a loadable module.
This module needs to be used with the mcu-host application.
If unsure say N.
Symbol: INGENIC_MCU_RPROC [=y]
Type  : tristate
Prompt: ingenic mcu remoteproc support
  Location:
    -> Device Drivers
      -> Remoteproc drivers
  Defined at drivers/remoteproc/Kconfig:90
  Depends on: HAS_DMA [=y]
  Selects: REMOTEPROC [=y]

```


## 3.MCU加载器使用方法


在开发板上执行:

> 1. 通过libmcu-bare编译好的固件放在 lib/firmware目录下面。
> 2. 通过mcu-host编译好的应用放在文件系统下。
> 3. ./mcu-host write_firmware /lib/firmware/libmcu-bare.bin加载固件
> 4. ./mcu-host write_data  主核往mcu写数据
> 5. ./mcu-host read_data   主核从mcu读数据

更多使用方法可以使用 mcu-host --help 查看
```
./mcu-host usage
Usage1:./mcu-host write_firmware <file>
Example:
        ./mcu-host write_firmware /usr/data/libmcu-bare.bin
Usage2:./mcu-host bootup
Example:
        ./mcu-host bootup
Usage3:./mcu-host write_str <str>
Example:
        ./mcu-host write_str test123456
Usage4:./mcu-host write_data <data...>
Example:
        ./mcu-host write_data 0x01 0x02 0x03 0x04
Usage5:./mcu-host write_mem <offset> <data...>
Example:
        ./mcu-host write_mem 0x400 0x01 0x02 0x03
Usage6:./mcu-host read_str <size>
Example:
        ./mcu-host read_str 11
Usage7:./mcu-host read_data <size>
Example:
        ./mcu-host read_data 11
Usage8:./mcu-host shutdown
Example:
        ./mcu-host shutdown
Usage9:./mcu-host reset
Example:
        ./mcu-host reset
```



