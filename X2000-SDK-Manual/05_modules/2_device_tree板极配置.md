device tree板级配置
================
[TOC]
<!-- toc -->

----
# 1. 设备树

* 默认方式为设备树编译进内核
* 可以通过修改配置支持外部加载设备树
  
## 1.1. 使用内核Builtin设备树
 默认配置

## 1.2.使用外部加载设备树

### 1.2.1.添加uboot支持
  
* 需要修改配置文件，如：u-boot/include/configs/halley5.h

1. uboot依赖配置:

```c
    /* Device Tree Configuration*/
   #define CONFIG_OF_LIBFDT 1
   #define IMAGE_ENABLE_OF_LIBFDT  1
   #define CONFIG_LMB
```

2. 修改 **BOOTCMD** 和 **BOOTARGS**

**NAND**

```c
    #define CONFIG_BOOTARGS BOOTARGS_COMMON "ip=off init=/linuxrc ubi.mtd=3 root=ubi0:rootfs ubi.mtd=4 rootfstype=ubifs rw
```

```c
    #define CONFIG_BOOTCOMMAND "set uImage 0x80600000; set dtb 0x83000000; sfcnand read 0x100000 0x400000 $(uImage); sfcnand read 0x900000 0x20000 $(dtb); bootm $(uImage) - ${dtb}"
```

**MMC**

```c
    #define CONFIG_BOOTARGS BOOTARGS_COMMON " rootfstype=ext4 root=/dev/mmcblk0p7 rootdelay=3 rw"
```

```c
    #define CONFIG_BOOTCOMMAND "set dtb 0x83000000; set uImage 0x80f00000; mmc dev 0;mmc read ${uImage} 0x1800 0x2000; mmc read ${dtb} 0x5800 0x100; bootm ${uImage} - ${dtb}"
```

**注：** *需要根据实际分区情况进行修改*

### 1.2.2.添加kernel支持

* make menuconfig修改配置：**去掉INGNEIC_BUILTIN_DTB配置**

```c
There is no help available for this option.
Symbol: INGENIC_BUILTIN_DTB [=n]
Type  : boolean
Prompt: Ingenic Device Tree build into Kernel.  
    Location:
        -> Machine selection
            -> SOC Type Selection
    Defined at arch/mips/xburst2/Kconfig:49
    Depends on: MACH_XBURST2 [=y]
    Selects: BUILTIN_DTB [=n]
```

### 1.2.3.编译dtb文件

* 以halley5开发板为例：

```c
/*kernel选过配置可以直接make dtbs，也可以直接指定目标设备树make dtbs CONFIG_DT_HALLEY5_V20=y*/
$ make dtbs
  DTB     arch/mips/boot/dts/ingenic/halley5_v20.dtb.S
  AS      arch/mips/boot/dts/ingenic/halley5_v20.dtb.o
  LD      arch/mips/boot/dts/ingenic/built-in.o
rm arch/mips/boot/dts/ingenic/halley5_v20.dtb.S
  LD      arch/mips/boot/dts/built-in.o

#### make completed successfully (1 seconds) ####


/*arch/mips/boot/dts/ingenic/下生成*.dtb目标文件*/
$ ls arch/mips/boot/dts/ingenic/halley5_v20.dt
halley5_v20.dtb    halley_v20.dtb.o  halley5_v20.dts
```

### 1.2.4.添加dtb分区与烧录

**NAND**

* 修改分区和烧录都是通过烧录工具完成．

注：　*将dtb文件烧录到设备树预留的分区上，要和bootcmd和bootargs配置的保持一致*
![cloner dtb nand](/assets/cloner-dtb-nand.png "Optional title")

**MMC**

* dtb存放位置：推荐使用默认地址0xb00000(11MB)，无需做其它修改，直接烧录即可。

* 如需要自己指定dtb分区，需要做如下修改:
    1. 以halley5为例，需要参考文件`u-boot/board/ingenic/"板级"/partitions.tab`
    2. 内核需要修改分区个数，与上面partitions.tab配置相同.
    3. 修改烧录工具dtb位置．

    *u-boot/board/ingenic/"板级"/partitions.tab*

    ```c
    property:
        disk_size = 4096m
        gpt_header_lba = 512
        custom_signature = 0

    partition:
            #name     =  start,   size, fstype
            xboot     =     0m,     3m,
            boot      =     3m,     8m, EMPTY
            recovery  =    12m,    16m, EMPTY
            pretest   =    28m,    16m, EMPTY
            reserved  =    44m,    52m, EMPTY
            misc      =    96m,     4m, EMPTY
            cache     =   100m,   100m, LINUX_FS
            system    =   200m,   1800m, LINUX_FS
            data      =   2000m,  2048m, LINUX_FS

    #fstype could be: LINUX_FS, FAT_FS, EMPTY
    ```

    *内核 make menuconfig*

    ```c
    Symbol: MMC_BLOCK_MINORS [=9]　　/*上面partitions.tab配置分区数为9*/
    Type  : integer
    Range : [4 256]
    Prompt: Number of minors per block device
        Location:
            -> Device Drivers 
                -> MMC/SD/SDIO card support (MMC [=y]
                    -> MMC block device driver (MMC_BLOCK [=y])
        Defined at drivers/mmc/card/Kconfig:17
        Depends on: MMC [=y] && MMC_BLOCK [=y]  
    ```

* 烧录

注：　*dtb推荐使用地址0xb00000，为MMC的11MB位置．*

![cloner dtb mmc](/assets/cloner-dtb-mmc.png "mmc dtb")

### 1.2.5.启动

* uboot识别设备树，如下打印

```c
## Flattened Device Tree blob at 83000000
   Booting using the fdt blob at 0x83000000
   Uncompressing Kernel Image ... OK
   Loading Device Tree to 87ff6000, end 87fff7c4 ... OK
```

### 1.2.6.备注

* uboot提供fdt命令，可以对设备树进行相关操作

```c
# fdt
fdt - flattened device tree utility commands

Usage:
fdt addr [-c]  <addr> [<length>]   - Set the [control] fdt location to <addr>
fdt move   <fdt> <newaddr> <length> - Copy the fdt to <addr> and make it active
fdt resize                          - Resize fdt to size + padding to 4k addr
fdt print  <path> [<prop>]          - Recursive print starting at <path>
fdt list   <path> [<prop>]          - Print one level starting at <path>
fdt get value <var> <path> <prop>   - Get <property> and store in <var>
fdt get name <var> <path> <index>   - Get name of node <index> and store in <var>
fdt get addr <var> <path> <prop>    - Get start address of <property> and store in <var>
fdt get size <var> <path> [<prop>]  - Get size of [<property>] or num nodes and store in <var>
fdt set    <path> <prop> [<val>]    - Set <property> [to <val>]
fdt mknode <path> <node>            - Create a new node after <path>
fdt rm     <path> [<prop>]          - Delete the node or <property>
fdt header                          - Display header info
fdt bootcpu <id>                    - Set boot cpuid
fdt memory <addr> <size>            - Add/Update memory node
fdt rsvmem print                    - Show current mem reserves
fdt rsvmem add <addr> <size>        - Add a mem reserve
fdt rsvmem delete <index>           - Delete a mem reserves
fdt chosen [<start> <end>]          - Add/update the /chosen branch in the tree
                                        <start>/<end> - initrd start/end addr
NOTE: Dereference aliases by omiting the leading '/', e.g. fdt print ethernet0.
```

## 1.3.使用设备树传递启动参数(Command Line)

### 1.3.1.修改设备树

* 定义如下`chosen`节点，节点中的`bootargs`作为向内核传递的`cmdline`.

    *例：arch/mips/boot/dts/ingenic/halley5_v20.dts*

```c
/ {
        compatible = "ingenic,halley5", "ingenic,x2000-v12";

        chosen {
                bootargs = "console=ttyS1,115200 mem=128M@0x0ip=off init=/linuxrc ubi.mtd=3 root=ubi0:rootfs ubi.mtd=4 rootfstype=ubifs rw";
        };
};
```

### 1.3.2.配置kernel选项

* 选择`MIPS_CMDLINE_FROM_DTB`配置，内核会去解析dtb镜像中的`chosen`节点

    *注: 启动过程中，uboot解析dtb中的chosen节点，如果没有定义，则uboot会创建默认的chosen节点．*

```c
Symbol: MIPS_CMDLINE_FROM_DTB [=y]
Type  : boolean
Prompt: Dtb kernel arguments if available
    Location:
        -> Kernel type
            -> Kernel command line type (<choice> [=y])
    Defined at arch/mips/Kconfig:2858
    Depends on: <choice> && USE_OF [=y]
```
