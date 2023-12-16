SPI Flash(SFC)-uboot驱动和配置
===========================
[TOC]
<!-- toc -->

----
# 1. SFC uboot驱动和配置

## 1.1.源码位置

* spl

```c
u-boot/common/spl/
├── spl_sfc_nand_v2.c
└── spl_sfc_nor_v2.c

u-boot/tools/ingenic-tools/nand_device/ /*各厂商的参数配置文件和一些通用接口*/
```

* uboot

```C
u-boot/drivers/mtd/devices/jz_sfc_v2/
├── jz_sfc_common.c
├── jz_sfc_nand.c
├── jz_sfc_nor.c
├── jz_sfc_ops.c
└── nand_device/    /*各厂商的参数配置文件和一些通用接口*/
```

## 1.2.配置gpio function

* x2000 SFC共支持两组GPIO:

```c
        PD1.8V:
            SFC0：PD17-22 FUNCTION1
            SFC1：PD23-26 FUNCTION1
        PE3.3V:
            SFC0：PE16-21 FUNCTION0
```

注: *PD组只可以单独配置SFC0或者同时配置SFC0、SFC1为8线，SFC1不可单独使用。*

* spl和uboot配置SFC的gpio function，需要修改板级配置文件:

```c
例如:　u-boot/include/configs/halley5.h

修改宏:
    /* sfc gpio */
    /* #define CONFIG_JZ_SFC_PD_4BIT */
    /* #define CONFIG_JZ_SFC_PD_8BIT */
    /* #define CONFIG_JZ_SFC_PD_8BIT_PULL */
    #define CONFIG_JZ_SFC_PE
```

## 1.3.SFC频率

* spl和uboot配置SFC的频率，需要修改板级配置文件:

```c
例如:　u-boot/include/configs/halley5.h

修改宏:
    /*
     * NOR: SFC外部时钟频率　＝　CONFIG_SFC_NOR_RATE/4
     * NAND: SFC外部时钟频率　＝　CONFIG_SFC_NAND_RATE/4
     **/
    #define CONFIG_SFC_NOR_RATE     296000000　　/* value <= 296000000(sfc 74Mhz)*/
    #define CONFIG_SFC_NAND_RATE    296000000　　/* value <= 296000000(sfc 74Mhz)*/
```

## 1.4.配置单线四线

* spl配置单线四线，需要修改板级配置文件

```c
例如:　u-boot/include/configs/halley5.h

修改宏:
    #define CONFIG_SFC_QUAD  /*定义则配置为４线*/
```

* uboot NOR:　配置单线四线，需要配置cloner烧录工具

> `Boot quad`勾选后，NOR Flash uboot启动为四线，否则为单线。

![ ](/assets/cloner-quad-boot.png "cloner select quad boot.")

* uboot NAND:　配置单线四线，需要修改各厂商参数配置文件源码

```c
例如:　u-boot/drivers/mtd/devices/jz_sfc_v2/nand_device/dosilicon_nand.c

修改基础参数:
            [1] = {
        /*DS35Q2GAXXX*/
                .pagesize = 2 * 1024,
                .blocksize = 2 * 1024 * 64,
                .oobsize = 64,
                .flashsize = 2 * 1024 * 64 * 2048,

                .tHOLD  = THOLD,
                .tSETUP = TSETUP,
                .tSHSL_R = TSHSL_R,
                .tSHSL_W = TSHSL_W,

                .tRD = 90,
                .tPP = TPP,
                .tBE = TBE,

                .plane_select = 1,
                .ecc_max = 0x4,
                //.need_quad = 1,  /*配置为４线模式*/
                .need_quad = 0,  /*配置为单线模式*/
        },

```

## 1.5.NAND配置BBP和PPB

* BPP: bytes per page，需要修改配置文件
* PPB: pages per block，需要修改配置文件

```c
例如:　u-boot/include/configs/halley5.h

修改宏:
/* sfc nand config */
#define CONFIG_SPI_NAND_BPP (2048 +64)      /*Bytes Per Page*/
#define CONFIG_SPI_NAND_PPB (64)            /*Page Per Block*/
```

## 1.6.flash中参数位置

* spl和uboot默认将参数烧录到flash的固定位置，启动时去指定位置读参数

```c
例如:　u-boot/include/configs/halley5.h

修改宏:
#define CONFIG_SPIFLASH_PART_OFFSET 0x5800　　/*参数的偏移地址*/

```

# 2.添加一款新的flash参数(依赖cloner烧录工具)

## 2.1. 添加NOR的spl和uboot参数(同kenrel)

* 通过cloner烧录工具添加spi nor flash参数:

  1. 打开cloner中的`Config`，在`INFO`菜单下，选择`Board`为“x2000_v12_sfc_nor_lpddr3_linux.cfg”。
  2. 在`SFC`菜单下，选择二级菜单`norinfo`，选择`ADD`。
  3. 在弹出的对话框中按照spi nor flash的参数填上保存即可。

    注: *烧录工具中`id`号为红色时，表示存在多个相同ID的flash参数，需要手动删除冲突的flash参数*

![ ](/assets/cloner_nor_add_params.png "Add nor params.")

## 2.2. 添加NAND的spl和uboot参数

* 添加NAND spl参数

```c
相关代码位置:
u-boot/tools/ingenic-tools/nand_device/
├── ato_nand.c
├── dosilicon_nand.c
├── foresee_nand.c
├── gd_nand.c
├── mxic_nand.c
├── sfc_params.c
├── tc_nand.c
├── winbond_nand.c
├── xtx_mid0b_nand.c
├── xtx_nand.c
└── zetta_nand.c

参照已有的配置添加新型号的flash参数，例如:
u-boot/tools/ingenic-tools/nand_device/dosilicon_nand.c
```

* 添加NAND uboot参数（同kernel）

```c
相关代码位置:
u-boot/drivers/mtd/devices/jz_sfc_v2/nand_device/
├── ato_nand.c
├── dosilicon_nand.c
├── foresee_nand.c
├── gd_nand.c
├── mxic_nand.c
├── nand_common.c
├── xtx_mid0b_nand.c
├── xtx_nand.c
└── zetta_nand.c

参照已有的配置添加新型号的flash参数，例如:
u-boot/drivers/mtd/devices/jz_sfc_v2/nand_device/dosilicon_nand.c
```

# 3. 通过spl传参数为bootrom升频

* 在boards.cfg中添加SPL_PARAMS_FIXER配置

```c
例如:
zebra_uImage_sfc_nor   mips    xburst2 zebra   ingenic x2000_v12   zebra:SPL_SFC_NOR,ENV_IS_IN_SFC,MTD_SFCNOR,SPL_PARAMS_FIXER
```

* 可以通过配置PLL，来配置bootrom频率，源码位置如下:

```c
u-boot/tools/ingenic-tools/spl_params_fixer_x2000_v12.c
```

# 4.uboot内置flash参数实现（不使用cloner烧录工具)

**NOR** :

* 需要修改uboot配置文件

   *例：u-boot/include/configs/halley5.h*

```c
    /* 打开如下配置宏　*/
    #define CONFIG_NOR_BUILTIN_PARAMS
```

* 添加一款flash参数，配置文件路径如下：

```c
    u-boot/tools/ingenic-tools/sfc_builtin_params/nor_device.c
```

* 每次只能支持一款flash参数，编译进镜像，需要通过宏来控制，具体如下所示：

```c
    #define CONFIG_INGENIC_GD25Q127C

    /* spi nor info params */
    struct spi_nor_info builtin_spi_nor_info = {
    #ifdef CONFIG_INGENIC_GD25Q127C
            /* GD25Q127C */
            .name = "GD25Q127C",
            .id = 0xc84018,

            .read_standard    = CMD_INFO(0x03, 0, 3, 0),
            .read_quad        = CMD_INFO(0x6b, 8, 3, 5),
            .write_standard   = CMD_INFO(0x02, 0, 3, 0),
            .write_quad       = CMD_INFO(0x32, 0, 3, 5),
            .sector_erase     = CMD_INFO(0x52, 0, 3, 0),
            .wr_en            = CMD_INFO(0x06, 0, 0, 0),
            .en4byte          = CMD_INFO(0, 0, 0, 0),
            .quad_set         = ST_INFO(0x31, 1, 1, 1, 1, 0),
            .quad_get         = ST_INFO(0x35, 1, 1, 1, 1, 0),
            .busy             = ST_INFO(0x05, 0, 1, 0, 1, 0),

            .tCHSH = 5,
            .tSLCH = 5,
            .tSHSL_RD = 20,
            .tSHSL_WR = 50,

            .chip_size = 16777216,
            .page_size = 256,
            .erase_size = 32768,

            .quad_ops_mode = 1,
            .addr_ops_mode = 0,
    #endif
    };

    /* partitions params */
    struct norflash_partitions builtin_norflash_partitions = {

            /* max 10 partitions*/
            .num_partition_info = 3,

            .nor_partition = {

                    [0].name = "uboot",
                    [0].offset = 0x0,
                    [0].size =   0x40000,
                    [0].mask_flags = NORFLASH_PART_RW,

                    [1].name = "kernel",
                    [1].offset = 0x40000,
                    [1].size =   0x300000,
                    [1].mask_flags = NORFLASH_PART_RW,

                    [2].name = "rootfs",
                    [2].offset = 0x360000,
                    [2].size = 0xca0000,
                    [2].mask_flags = NORFLASH_PART_RW,
            },
    };

    /* private params */
    private_params_t builtin_private_params = {
            .fs_erase_size = 32768,
            .uk_quad = 1,
    };
```

**NAND** :

* 需要修改uboot配置文件

   *例：u-boot/include/configs/halley5.h*

```c
    /* 打开如下配置宏　*/
    #define CONFIG_NAND_BUILTIN_PARAMS
```

* 添加一款flash参数，配置文件路径如下：

```c
    u-boot/tools/ingenic-tools/sfc_builtin_params/nand_device.c
```

* nand flash设备参数默认编译进镜像，因此只需要配置partitions参数，具体如下所示：

```c
    /*params: nand flash partitions*/
    nand_partition_builtin_params_t nand_builtin_params = {

            .magic_num = SPINAND_MAGIC_NUM,

            /* max 10 partitions*/
            .partition_num = 4,

            .partition = {

                    [0].name = "uboot",
                    [0].offset = 0x0,
                    [0].size =   0x100000,
                    [0].mask_flags = NORFLASH_PART_RW,
                    [0].manager_mode = MTD_MODE,

                    [1].name = "kernel",
                    [1].offset = 0x100000,
                    [1].size =   0x800000,
                    [1].mask_flags = NORFLASH_PART_RW,
                    [1].manager_mode = MTD_MODE,

                    [2].name = "rootfs",
                    [2].offset = 0x900000,
                    [2].size = 0x2800000,
                    [2].mask_flags = NORFLASH_PART_RW,
                    [2].manager_mode = UBI_MANAGER,

                    [3].name = "userdata",
                    [3].offset = 0x3100000,
                    [3].size = 0xcf00000,
                    [3].mask_flags = NORFLASH_PART_RW,
                    [3].manager_mode = UBI_MANAGER,
            },
    };
```

# 5.注意事项

* Nand Flash ECC方案
  
```c
    使用Ｎand Ｆlash自带的硬件ECC, 基于MTD框架，在内存中建立BBT．
```

* 已经支持过的参数位置

```c
    NAND:
        u-boot/tools/ingenic-tools/nand_device/             /*spl*/
        u-boot/drivers/mtd/devices/jz_sfc_v2/nand_device/   /*uboot*/
    NOR:
        见烧录工具．
```

* 支持多Die的SPI Nor Flash

```c
    spl:
        暂不支持多Die切换
    参数:
        当flash参数使用烧录方式时，必须存放于Die0
    启动:
        支持重启异常时reset flash切换Die0
```

* u-boot相关命令

  * NAND

    ```c
    Usage:
            nand read - addr off|partition size
            nand write - addr off|partition size
                        read/write 'size' bytes starting at offset 'off'
                        to/from memory address 'addr', skipping bad blocks.
            nand erase[.spread] [clean] off size
                        - erase 'size' bytes from offset 'off' With '.spread',
                        erase enough for given file size, otherwise,'size' 
                        includes skipped bad blocks.
    例:
            /* 内存地址， flash偏移 ,  size */
            nand read 0x80806000 0x000 0x100

            /* 内存地址， flash偏移 ,  size */
            nand erase 0x0 1

            /* 内存地址， flash偏移 ,  size （写之前必须先擦除）*/
            nand write 0x80808080 0x0 0x100
    ```

  * NOR

    ```c
    Usage:
            sfcnor sfcnor read   [src:nor flash addr] [bytes:0x..] [dst:ddr address]
            sfcnor write  [dst:nor flash addr] [bytes:0x..] [src:ddr address] [force erase:1, no erase:0]
            sfcnor erase  [src:nor flash addr] [bytes:0x..]

    例:
            /* flash偏移 ,  size ,  内存地址 */
            sfcnor read 0x0000 0x100 0x80800000

            /* flash偏移 ,  size */
            sfcnor erase 0x0000 0x40000

            /* flash偏移 ,  size ,  内存地址 (写之前先擦除)*/
            sfcnor write 0x0000 0x100 0x80800000
    ```

* 使用SFC的PD组GPIO时，要确保PE组GPIO配置成其它function

    注：
    1. *当PD组和PE组同时配置为sfc function时，优先级PE>PD*
    2. *boot select 选择usb启动或者sfc 1.8v启动*
    3. 选择sfc pD GPIO，修改配置文件如下：

        *例：u-boot/include/configs/halley5.h*

    ```c
    #define CONFIG_JZ_SFC_PD_4BIT
    /* #define CONFIG_JZ_SFC_PD_8BIT */
    /* #define CONFIG_JZ_SFC_PD_8BIT_PULL */
    /* #define CONFIG_JZ_SFC_PE */
    ```

* 支持page size 4k的NAND　flash

    注：
    1. *sfc启动默认配置参数，支持bytes per page为2KB的nand flash*
    2. 支持4K page size的NAND　Flash需要修改配置文件如下：

        *例：u-boot/include/configs/halley5.h*

    ```c
        #define CONFIG_SPI_NAND_BPP     (4096 +256)      /*Bytes Per Page*/
        #define CONFIG_SPI_NAND_PPB     (64)             /*Page Per Block*/  
    ```
