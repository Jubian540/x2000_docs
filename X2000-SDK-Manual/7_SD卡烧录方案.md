SD卡烧录方案
===========

[TOC]

<!-- toc -->

----

注：*本文中出现的'$'表示pc端执行，'#'表示开发板运行。*

# 1. sd卡烧录方案简介

---

* 使用sd卡启动，将制作好的镜像烧录到flash。
* 当前支持X2000和M300平台nand flash烧录。

# 2. 制作nand烧录镜像以及配置文件

---

**U-BOOT.bin**:

* 制作builtin参数uboot镜像，参考sfc-uboot文档，＂`4.uboot内置flash参数实现（不使用cloner烧录工具)`＂章节。

  链接：  [sfc-uboot文档](/modules/sfc-uboot.md)

* 使用`add_chip_info_host`工具，处理编译生成的uboot镜像，以区分不同芯片

  该工具位于Manhattan工程/prebuilts/sd-burntools/add_chip_info_host

  ~~~c
  $ add_chip_info_host uboot 128 X2000E		/*以x2000e为例*/
  ~~~

* $ mv uboot U-BOOT.bin

**kernel**:

* 根据板级选择

**system.ubi**:

* Manhattan编译生成的system.ubifs为裸镜像，需要制作ubi卷标

* 制作ubinize配置文件

  参考文件：`buildroot/package/ingenic/system_config/sd_burn/ubinize.ini`

  ```c
  [rootfs]
  mode=ubi
  image=system.ubifs  /*源文件*/
  vol_id=0            /*卷序号*/
  vol_size=100MiB     /*卷大小*/
  vol_tpye=dynamic    /*动态卷*/
  vol_name=rootfs     /*卷名*/
  vol_flags=autoresize
  vol_alignment=1
  ```

* 开始制作

  ```c
  # ubinize -o system.ubi -p 131072 -m 2048 -s 2048 ubinize.ini
  
    ==============================
    ubinize 参数：
        -o：输出文件名。
        -m：最小输入输出大小为2KiB(2048bytes)，一般为页大小。
        -p：物理可擦出块大小为128KiB=每块的页数*页大小=64*2KiB=128KiB。
        -s：用于UBI头部信息的最小输入输出单元，一般与最小输入输出单元(-m参数)大小一样。
    ==============================
  ```

**nand_burn.ini**

- 具体可参考Manhattan工程/prebuilts/sd-burntools/nand_burn.ini

- 配置文件语法说明如下，如有需要可自行更改：

  ~~~c
  ==============================
  [option]
  erase=0/1
  烧录文件到分区之前是否需要擦除该分区
  
  [partition]
  part0=nand-boot,0x0,0x600000,mtd0
  part0:     第一个分区
  nand-boot: 分区名
  0x0:       分区偏移地址
  0x600000:  分区大小
  mtd0:      mtd分区号
  
  [file]
  file0=BOOT.bin,0x0
  file0:     第一个文件
  BOOT.bin:  文件名
  0x0:       文件在nand中的偏移(不是分区内偏移，可以一个分区烧录多个文件)
  ==============================
  ~~~

# 3. 制作SD卡烧录镜像

---

## 3.1. 修改device tree

由于卡烧需要通过device tree获取nand分区信息，所以需要修改device tree，将ingenic,use_ofpart_info  = /bits/ 8 <0>中的/bits/ 8 <0>改为/bits/ 8 <1>

*修改参考文件：kernel/arch/mips/boot/dts/ingenic/“板级”.dts*

```c
    配置设备树：
    &sfc {
        ingenic,use_ofpart_info  = /bits/ 8 <1>; /*支持设备树MTD分区*/
        ... ...

            nandflash@1 {
                partitions {
                        compatible = "fixed-partitions";
                        #address-cells = <1>;
                        #size-cells = <1>;

                        /* spi nand flash partition */
                        partition@0 {
                                label = "uboot";
                                reg = <0x0000000 0x100000>;
                        };  

                        partition@100000 {
                                label = "kernel";
                                reg = <0x100000 0x800000>;
                        };

                        partition@900000 {
                                label = "rootfs";
                                reg = <0x900000 0xf700000>;
                        };
                };
        };

        ... ...
};
```

## 3.2. 修改u-boot配置文件

由于X2000使用LPDDR3，而X2000E使用的是LPDDR2，在不使用烧录工具的情况下，需要根据芯片修改u-boot的配置选择对应的DDR类型。

*修改参考文件：u-boot/include/configs/“板级”.h*

~~~c
......
#define CONFIG_DDR_INNOPHY
#define CONFIG_DDR_DLL_OFF
#define CONFIG_DDR_PARAMS_CREATOR
#define CONFIG_DDR_HOST_CC
/*#define CONFIG_DDR_TYPE_DDR3*/
#define CONFIG_DDR_TYPE_LPDDR3		/*X2000使用LPDDR3*/
#define CONFIG_DDR_TYPE_LPDDR2		/*X2000E使用LPDDR2*/
......
~~~
~~~c
......
#elif defined(CONFIG_JZ_MMC_MSC2)
	#define MSC_BOOTARGS " rootfstype=ext4 root=/dev/mmcblk2p7 rootdelay=3 rw flashtype=nand"
#endif
......
~~~

## 3.3. 编译生成SD卡烧录镜像

* 进入Manhattan目录下，执行如下

```c
$ source build/envsetup.sh
$ lunch
```

* 选择如下配置

```c
25. ”板级“_msc_4.4.94_burn-eng
26. “板级”_msc_4.4.94_burn-user
27. “板级”_msc_4.4.94_burn-userdebug
```

* 编译

```c
$ make -j4
```

* 生成如下镜像

```c
$ ls out/product/”板级“/image/
kernel  system.ext2  uboot
```

# 4. 烧录执行

1. 将SD卡插到pc上，按照下述表格中的烧录命令将编译生成的镜像文件烧录到SD卡指定位置（其中SD卡的设备节点/dev/sd*具体需根据pc确定）

|    镜像     |  偏移  |                 烧录命令                 |
| :---------: | :----: | :--------------------------------------: |
|    uboot    |   0    |      ./burn_sd.sh 0 uboot /dev/sd*       |
|   kernel    |  6144  |    ./burn_sd.sh 6144 kernel /dev/sd*     |
| system.ext2 | 409600 | ./burn_sd.sh 409600 system.ext2 /dev/sd* |

  也可使用烧录工具进行烧录。

2. 烧录完成之后，可以看到SD卡被分为了八个分区，按照如下步骤格式化第八分区并将要烧录的nand镜像以及配置文件nand_burn.ini放到该分区下：

```c
     sudo mkfs.ext2 /dev/"sd*8"	    /*格式化*/
     sudo mount /dev/"sd*8" /mnt	/*挂载*/
     cd /mnt						/*将烧录配置文件放到/mnt目录下*/
     mkdir image			        /*将做好的nand启动镜像文件(U-BOOT.bin kernel system.ubi)放到/mnt/image目录下*/
```

​	  放好之后/mnt目录结构参考如下：

~~~c
/mnt/
├── image			/*镜像文件*/
│   ├── kernel
│   ├── system.ubi
│   └── U-BOOT.bin
├── lost+found
└── nand_burn.ini	/*配置文件*/
~~~

3. 制作完成，可将SD卡插入板子进行卡烧操作，如下打印，则烧录成功：

```c
=============== Start Burn =================
=============== Nand info ===============
EraseBlockSize: 131072
MinIOSize: 2048
SubPageSize: 2048
=========================================
Partition:nand-uboot,   MTD:mtd0,       Offset:0x0,     Size:0x100000
Eraseing /dev/mtd0...
Erasing 128 Kibyte @ e0000 -- 100 % complete 
Write /burn/image/U-BOOT.bin to /dev/mtd0 at offset 0
Writing data to block 0 at offset 0x0
Writing data to block 1 at offset 0x20000
Writing data to block 2 at offset 0x40000
Partition:nand-kernel,  MTD:mtd1,       Offset:0x100000,        Size:0x800000
Eraseing /dev/mtd1...
Erasing 128 Kibyte @ 7e0000 -- 100 % complete 
Write /burn/image/kernel to /dev/mtd1 at offset 0
Writing data to block 0 at offset 0x0
Writing data to block 1 at offset 0x20000

    ... ...

Writing data to block 274 at offset 0x2240000
handleSuccess
=============== Burn Success =================
```
4. 若打印如下内容：
```c
Initializing random number generator: OK
Saving random seed: OK
Starting telnetd: OK
=============== Start Burn =================
No "/usr/mmcdata/mmcblk2p8/image" folder!
handleError
============== Burn Error ====================
```
将文件 /etc/init.d/S99burn 中的 mmcblk0p8 修改为 mmcblk2p8


# 5. 注意事项

---

## 5.1. 工具依赖

```c
1. nandwrite
2. flash_erase
3. mtdinfo
4. bash
```

## 5.2.确保nand分区一致

1. sd卡烧时的mtd设备分区
2. sd卡烧配置文件nand_burn.ini的分区
3. nand flash的uboot内置分区