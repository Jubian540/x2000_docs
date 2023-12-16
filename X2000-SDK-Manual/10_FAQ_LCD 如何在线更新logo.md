

# 问题描述
如何通过烧录logo文件的方式或者将lcd logo文件放在文件系统中，来更新logo。

# 解决办法

## 原理

根据使用不同的主存储介质，从spi-flash 或者从eMMC 空间中划分一段空间出来，用来存储logo文件，改分区不使用文件系统管理，因为使用文件系统加载Logo时间会变长，影响体验.

### 2.1.1 准备logo文件

1. 编译kernel scripts/pic2logo.c
``` 
   gcc -lm pic2logo.c -o pic2logo
```

2. 执行命令
```
pic2logo logo.png 32 0xffffff0 logo.out
```
```
logo.png是要转换的图片；
32是logo.png的图像像素深度（32bpp）；
0xffffff0是背景色，当屏的分辨率大于logo.png分辨率时，大于的部分用背景色填充；
logo.out 是最终要使用的文件
```

### 2.1.2 通过烧录工具更新logo
该方式需要修改烧录工具，在烧录工具上添加一个logo分区；这里以linux烧录工具2.5.18版本为例，说明一下添加logo分区的流程；

1. 选择一个要烧录的配置，这里选择x2000_sfc_nand_lpddr3_linux.cfg进行说明；

2. 选择 配置-->SFC-->分区信息；

3. 添加一个分区，将其放在rootfs之后，名称为logo，偏移为rootfs的大小加上其偏移，大小填0x100000（1M）一般就够用了，可以根据自己的需求添加，分区类型为MTD_MODE；如下图。

   ![](/assets/cloner-sfc.png)

4. 切换到烧录工具POLICY界面，添加一个烧录选项；label为logo，type选择文件，ops选择SFC_NAND,offset为步骤3中计算出来的偏移。然后选择上面准备好的logo.out文件；如下图。

   ![](/assets/cloner-policy.png)

5. 第一次烧录logo时，需选择全擦烧录。之后正常烧录即可。


### 2.1.3 通过文件系统更新logo
将上面步骤中制作好的logo.out通过烧录工具烧录到文件系统中。


### 2.1.4 uboot 启动命令修改

1. 修改bootcmd，添加读logo分区到内存.
```
在bootcmd中添加指令，将logo.out文件读写到内存的指定位置。
如原来 bootcmd=bootcmd=sfcnand read 0x100000 0x500000 0x80600000 ;bootm 0x80600000
修改为 bootcmd=sfcnand read 0x100000 0x500000 0x80600000 ;sfcnand read 0x6900000 0x100000 0x81600000 ;bootm 0x80600000
即添加从地址0x6900000处读取0x100000的数据到内存的0x81600000处。其中地址0x6900000是烧录工具中设置的logo分区偏移地址。

```

2. 修改bootargs传参， 指定logo读到的位置
bootargs 添加 "logo=0x81600000", 其中0x81600000 是logo读到的位置。可以自行指定。但需要和bootcmd中设置的值保持一致。








