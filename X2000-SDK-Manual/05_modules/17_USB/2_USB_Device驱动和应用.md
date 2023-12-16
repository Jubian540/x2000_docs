# 2 USB Device驱动和应用
[TOC]
<!-- toc -->

----

### 3.2控制器作为device

#### 3.2.1 usb device 功能验证情况

* 支持device功能：`mass storage`、`adb`、`hid`、`uvc`、`uac1.0`、`printer`、`rndis`、`serial`。

* composite功能：
  * 默认支持mass storage、adb、uvc、uac1.0、printer、rndis可以同时使用。
  * hid、serial不支持复合设备，需要手动修改配置。
  * usb composite和相关功能描述符配置路径如下：

```c
    buildroot/package/ingenic/system_config/usb-device
    ├── S90usb
    └── usb
        ├── adb
        ├── hid
        ├── mass_storage
        ├── printer
        ├── rndis
        ├── serial
        ├── uac1
        ├── udc_daemon
        └── uvc
```

#### 3.2.2 device mass storage

驱动配置：

```txt
Symbol: USB_CONFIGFS_MASS_STORAGE [=y]                                             
Type  : boolean                                                                    
Prompt: Mass storage                                                               
  Location:                                                                        
    -> Device Drivers                                                              
      -> USB support (USB_SUPPORT [=y])                                            
        -> USB Gadget Support (USB_GADGET [=y])                                    
          -> USB Gadget Drivers (<choice> [=y])                                    
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])     
  Defined at drivers/usb/gadget/Kconfig:338                                        
  Depends on: <choice> && USB_CONFIGFS [=y] && BLOCK [=y]                          
  Selects: USB_F_MASS_STORAGE [=y]
```

使用方法：

1. 制作盘符

    > #dd if=/dev/zero of=fat32.img bs=1k count=2048

    > #mkfs.vfat fat32.img

2. 将盘符加入

    > #echo fat32.img > /sys/kernel/config/usb_gadget/demo/functions/mass_storage.0/lun.0/file

3. 完成后会在PC端弹出盘符并且支持热插拔

#### 3.2.3 device adb

驱动配置：

```txt
Symbol: USB_CONFIGFS_F_FS [=y]                                                     
Type  : boolean                                                                    
Prompt: Function filesystem (FunctionFS)                                           
  Location:                                                                        
    -> Device Drivers                                                              
      -> USB support (USB_SUPPORT [=y])                                            
        -> USB Gadget Support (USB_GADGET [=y])                                    
          -> USB Gadget Drivers (<choice> [=y])                                    
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])     
  Defined at drivers/usb/gadget/Kconfig:362                                        
  Depends on: <choice> && USB_CONFIGFS [=y]                                        
  Selects: USB_F_FS [=y]
```

使用方法：

1. 启动后无需任何配置直接开启adb功能

2. adb不支持热插拔，插拔后会将设备清除，再次插入时候需要重新绑定

    > #echo 13500000.otg > /sys/kernel/config/usb_gadget/demo/UDC

#### 3.2.4 device hid

驱动配置：

```txt
Symbol: USB_CONFIGFS_F_HID [=y]                                                   
Type  : boolean                                                                   
Prompt: HID function                                                              
  Location:                                                                       
    -> Device Drivers                                                             
      -> USB support (USB_SUPPORT [=y])                                           
        -> USB Gadget Support (USB_GADGET [=y])                                   
          -> USB Gadget Drivers (<choice> [=y])                                   
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])    
  Defined at drivers/usb/gadget/Kconfig:419                                       
  Depends on: <choice> && USB_CONFIGFS [=y]                                       
  Selects: USB_F_HID [=n]
```

使用方法：
1. pc端识别设备

    > $dmesg

    ```txt
    [2180880.531376] usb 2-1.5: new high-speed USB device number 115 using ehci-pci
    [2180880.628285] usb 2-1.5: New USB device found, idVendor=18d1, idProduct=d002
    [2180880.628292] usb 2-1.5: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [2180880.628304] usb 2-1.5: Product: composite-demo
    [2180880.628307] usb 2-1.5: Manufacturer: ingenic
    [2180880.628309] usb 2-1.5: SerialNumber: 0123456789ABCDEF
    [2180880.635882] input: ingenic composite-demo as /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.5/2-1.5:1.0/0003:18D1:D002.006C/input/input126
    [2180880.691853] hid-generic 0003:18D1:D002.006C: input,hidraw3: USB HID v1.01 Keyboard [ingenic composite-demo] on usb-0000:00:1d.0-1.5/input0
    ```

2. 开发板执行测试程序

    > #cd /testsuite/usb_test/usb_gadget/hid_gadget_test

    模拟usb键盘
    > #./hid_gadget_test /dev/hidg0 keyboard

    模拟usb鼠标
    > #./hid_gadget_test /dev/hidg1 mouse

#### 3.2.5 device uvc gadget

功能描述：

* 支持mjpeg、yuyv格式输出。
* 支持360p、480p、720p、1080p动态调整。
* 支持静态图像测试，支持camera动态获取测试。

驱动配置：

```txt
(1)USB Video Class (UVC)
Symbol: USB_VIDEO_CLASS [=y]                                                                                            
Type  : tristate                                                                                                        
Prompt: USB Video Class (UVC)                                                                                           
  Location:                                                                                                             
    -> Device Drivers                                                                                                   
      -> Multimedia support (MEDIA_SUPPORT [=y])                                                                        
        -> Media USB Adapters (MEDIA_USB_SUPPORT [=y])                                                                  
  Defined at drivers/media/usb/uvc/Kconfig:1                                                                            
  Depends on: USB [=y] && MEDIA_SUPPORT [=y] && MEDIA_USB_SUPPORT [=y] && MEDIA_CAMERA_SUPPORT [=y] && VIDEO_V4L2 [=y]  
  Selects: VIDEOBUF2_VMALLOC [=n]                                                                                       

 (2)USB Webcam function
Symbol: USB_CONFIGFS_F_UVC [=y]                                                    
Type  : boolean                                                                    
Prompt: USB Webcam function                                                        
  Location:                                                                        
    -> Device Drivers                                                              
      -> USB support (USB_SUPPORT [=y])                                            
        -> USB Gadget Support (USB_GADGET [=y])                                    
          -> USB Gadget Drivers (<choice> [=y])                                    
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])     
  Defined at drivers/usb/gadget/Kconfig:429                                        
  Depends on: <choice> && USB_CONFIGFS [=y] && VIDEO_DEV [=y]                      
  Selects: VIDEOBUF2_VMALLOC [=y] && USB_F_UVC [=n]      

  (3)Enable usb dwc2 highwidth fifo
  Symbol: USB_DWC2_HIGHWIDTH_FIFO [=y]                             
Type  : boolean                                                  
Prompt: Enable usb dwc2 highwidth fifo                           
  Location:                                                      
    -> Device Drivers                                            
      -> USB support (USB_SUPPORT [=y])                          
        -> DesignWare USB2 DRD Core Support (USB_DWC2 [=y])      
  Defined at drivers/usb/dwc2/Kconfig:64                         
  Depends on: USB_SUPPORT [=y] && USB_DWC2 [=y]
```

测试方法：

1. pc端识别设备

    > $dmesg

    ```txt
    [2187560.274157] usb 2-1.5: new high-speed USB device number 19 using ehci-pci
    [2187560.371029] usb 2-1.5: New USB device found, idVendor=18d1, idProduct=d002
    [2187560.371033] usb 2-1.5: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [2187560.371035] usb 2-1.5: Product: composite-demo
    [2187560.371037] usb 2-1.5: Manufacturer: ingenic
    [2187560.371039] usb 2-1.5: SerialNumber: 0123456789ABCDEF
    [2187560.394857] uvcvideo: Found UVC 1.00 device composite-demo (18d1:d002)
    [2187560.403005] input: composite-demo as /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.5/2-1.5:1.0/input/input141
    ```

2. 开发板执行测试程序

    webcam_gadget使用方法：

        ```txt
        Usage: webcam_gadget [options]
        Available options are
        -b             Use bulk mode
        -d             Do not use any real V4L2 capture device
        -h             Print this help screen and exit
        -i             images dir for [uvc-WxH.jpg uvc-WxH.yuv]
        -m             Streaming mult for ISOC (b/w 0 and 2)
        -n             Number of Video buffers (b/w 2 and 32)
        -o <IO method> Select UVC IO method:
                0 = MMAP
                1 = USER_PTR
        -s <speed>     Select USB bus speed (b/w 0 and 2)
                0 = Full Speed (FS)
                1 = High Speed (HS)
                2 = Super Speed (SS)
        -t             Streaming burst (b/w 0 and 15)
        -u device      UVC Video Output device
        -v device      V4L2 Video Capture device
        -e device      HELIX Video Capture device
        ```

    camera动态测试：

      format:

      ```c
        # ./webcam_gadget -u /dev/video<uvc video node #> -v /dev/video<camera video node #> -e /dev/video<helix video node #>
      ```

      example:
      > #./webcam_gadget -u /dev/video7 -v /dev/video4 -e /dev/video0

    静态图片测试：

      format:
      **注：** *需要提前将测试图片放到-i 指令的路径下, 命名规则：uvc-(weight)x(height).yuv 和 uvc-(weight)x(height).jpg

      example:
      > #./webcam_gadget -u /dev/video<uvc video node #> -v /dev/video<camera video node #> -e /dev/video<helix video node #> -d -i /mnt

    pc端测试工具：

      ```txt
      linux os:
              Video tools: xawtv
              Video tools: cheese webcam booth

              If you use cheese webcam booth, please set the video resolution or photo resolution;

      windows os:
              Video tools: amcap

      手机端:
              APP: usb 摄像头
      ```

      **注**: *华为手机,应使用legacy配置*

#### 3.2.6 device remote NDIS

驱动配置：

```txt
Symbol: USB_CONFIGFS_RNDIS [=y]                                                        
Type  : boolean                                                                        
Prompt: RNDIS                                                                          
  Location:                                                                            
    -> Device Drivers                                                                  
      -> USB support (USB_SUPPORT [=y])                                                
        -> USB Gadget Support (USB_GADGET [=y])                                        
          -> USB Gadget Drivers (<choice> [=y])                                        
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])         
  Defined at drivers/usb/gadget/Kconfig:297                                            
  Depends on: <choice> && USB_CONFIGFS [=y] && NET [=y]                                
  Selects: USB_U_ETHER [=n] && USB_F_RNDIS [=n]
```

使用方法：

1. pc端识别设备
    pc端识设备信息：

    >$ dmesg

    ```txt
    识别信息：
    [2188325.480982] usb 2-1.5: new high-speed USB device number 21 using ehci-pci
    [2188325.577912] usb 2-1.5: New USB device found, idVendor=18d1, idProduct=d002
    [2188325.577919] usb 2-1.5: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [2188325.577930] usb 2-1.5: Product: composite-demo
    [2188325.577932] usb 2-1.5: Manufacturer: ingenic
    [2188325.577934] usb 2-1.5: SerialNumber: 0123456789ABCDEF
    [2188325.586180] rndis_host 2-1.5:1.0 usb0: register 'rndis_host' at usb-0000:00:1d.0-1.5, RNDIS device, 52:ea:19:19:5f:69
    [2188325.650205] rndis_host 2-1.5:1.0 enp0s29u1u5: renamed from usb0
    [2188325.670351] IPv6: ADDRCONF(NETDEV_UP): enp0s29u1u5: link is not ready
    ```

    pc端识别到usb网卡设备：

    > $ ifconfig -a

    ```txt
    enp0s29u1u5 Link encap:Ethernet  HWaddr 52:ea:19:19:5f:69  
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:0 errors:0 dropped:0 overruns:0 frame:0
              TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
    ```

  2. pc端配置usb网卡设备ip：

      > $ sudo ifconfig enp0s29u1u5 192.168.4.250 up

      ```txt
      enp0s29u1u5 Link encap:Ethernet  HWaddr 52:ea:19:19:5f:69  
                inet addr:192.168.4.250  Bcast:192.168.4.255  Mask:255.255.255.0
                inet6 addr: fe80::50ea:19ff:fe19:5f69/64 Scope:Link
                UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
                RX packets:0 errors:0 dropped:0 overruns:0 frame:0
                TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
                collisions:0 txqueuelen:1000 
                RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
      ```

3. pc端测试网卡

      > $ ping 192.168.4.250

      ```txt
      PING 192.168.4.250 (192.168.4.250) 56(84) bytes of data.
      64 bytes from 192.168.4.250: icmp_seq=1 ttl=64 time=0.034 ms
      64 bytes from 192.168.4.250: icmp_seq=2 ttl=64 time=0.034 ms
      64 bytes from 192.168.4.250: icmp_seq=3 ttl=64 time=0.031 ms
      ```

#### 3.2.7 device Serial Gadget

驱动配置：

```txt
 Symbol: USB_CONFIGFS_SERIAL [=y]                                              
 Type  : boolean                                                               
 Prompt: Generic serial bulk in/out                                            
   Location:                                                                   
     -> Device Drivers                                                         
       -> USB support (USB_SUPPORT [=y])                                       
         -> USB Gadget Support (USB_GADGET [=y])                               
           -> USB Gadget Drivers (<choice> [=y])                               
             -> USB functions configurable through configfs (USB_CONFIGFS [=y])
   Defined at drivers/usb/gadget/Kconfig:241                                   
   Depends on: <choice> && USB_CONFIGFS [=y] && TTY [=y]                       
   Selects: USB_U_SERIAL [=n] && USB_F_SERIAL [=n]                             

Symbol: USB_CONFIGFS_ACM [=y]                                                 
Type  : boolean                                                               
Prompt: Abstract Control Model (CDC ACM)                                      
  Location:                                                                   
    -> Device Drivers                                                         
      -> USB support (USB_SUPPORT [=y])                                       
        -> USB Gadget Support (USB_GADGET [=y])                               
          -> USB Gadget Drivers (<choice> [=y])                               
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])
  Defined at drivers/usb/gadget/Kconfig:250                                   
  Depends on: <choice> && USB_CONFIGFS [=y] && TTY [=y]                       
  Selects: USB_U_SERIAL [=n] && USB_F_ACM [=n]                                

Symbol: USB_CONFIGFS_OBEX [=y]                                                
Type  : boolean                                                               
Prompt: Object Exchange Model (CDC OBEX)                                      
  Location:                                                                   
    -> Device Drivers                                                         
      -> USB support (USB_SUPPORT [=y])                                       
        -> USB Gadget Support (USB_GADGET [=y])                               
          -> USB Gadget Drivers (<choice> [=y])                               
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])
  Defined at drivers/usb/gadget/Kconfig:260                                   
  Depends on: <choice> && USB_CONFIGFS [=y] && TTY [=y]                       
  Selects: USB_U_SERIAL [=n] && USB_F_OBEX [=n]                               
```

使用方法：

1. pc端识别设备

    >$ dmesg

    ```txt
    设备信息：
    [927759.973606] usbserial_generic 2-1.8:1.0: The "generic" usb-serial driver is only for testing and one-off prototypes.
    [927759.973611] usbserial_generic 2-1.8:1.0: Tell linux-usb@vger.kernel.org to add your device to a proper driver.
    [927759.973615] usbserial_generic 2-1.8:1.0: generic converter detected
    [927759.973820] usb 2-1.8: generic converter now attached to ttyUSB1
    ```

2. pc端配置usb serial的VID和PID

    >$ echo 0x18d1 0xd002 >  /sys/bus/usb-serial/drivers/generic/new_id

3. 开发板与pc端交互测试

    pc端测试：

    注：*ttyUSB1对应usb serial设备*

    >$ echo 1111111111 > /dev/ttyUSB1

    >$ cat /dev/ttyUSB1

    开发板测试：

    > #cat /dev/ttyGS0

    > #echo 222 > /dev/ttyGS0

#### 3.2.8 device printer Gadget

驱动配置：

```txt
Symbol: USB_CONFIGFS_F_PRINTER [=y]                                           
Type  : boolean                                                               
Prompt: Printer function                                                      
  Location:                                                                   
    -> Device Drivers                                                         
      -> USB support (USB_SUPPORT [=y])                                       
        -> USB Gadget Support (USB_GADGET [=y])                               
          -> USB Gadget Drivers (<choice> [=y])                               
            -> USB functions configurable through configfs (USB_CONFIGFS [=y])
  Defined at drivers/usb/gadget/Kconfig:469                                   
  Depends on: <choice> && USB_CONFIGFS [=y]                                   
  Selects: USB_F_PRINTER [=y]                                                 
```

使用方法：

1. pc端识别设备

    >$ dmesg

    ```txt
    设备信息：
    usblp 2-1.7:1.4: usblp0: USB Bidirectional printer dev 38 if 4 alt 0 proto 2 vid 0x18D1 pid 0xD002
    ```

2. 开发板与pc端交互测试
    开发板测试：

    > #cd /testsuite/usb_test/usb_gadget/prn_example/

    > #./prn_example -read_data

    > #cat data_file | ./prn_example -write_data

    pc端测试：

    > $ echo 111 > /dev/usb/lp0

    > $ cat /dev/usb/lp0

#### 3.2.9 device uac1.0 Gadget

驱动配置：

```txt
Symbol: USB_CONFIGFS_F_UAC1 [=y]                                               
Type  : boolean                                                                
Prompt: Audio Class 1.0                                                        
  Location:                                                                    
    -> Device Drivers                                                          
      -> USB support (USB_SUPPORT [=y])                                        
        -> USB Gadget Support (USB_GADGET [=y])                                
          -> USB Gadget Drivers (<choice> [=y])                                
            -> USB functions configurable through configfs (USB_CONFIGFS [=y]) 
  Defined at drivers/usb/gadget/Kconfig:402                                    
  Depends on: <choice> && USB_CONFIGFS [=y] && SND [=y]                        
  Selects: USB_LIBCOMPOSITE [=y] && SND_PCM [=y] && USB_F_UAC1 [=y]            
```

使用方法：

1. pc端识别设备

    >$ aplay -l

    ```txt
    声卡信息：
    card 1: compositedemo [composite-demo], device 0: USB Audio [USB Audio]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    ```

2. 开发板执行测试程序，pc端选择usb 声卡设备，播放声音

    注：*配置speaker音频通道，其中声卡1为usb声卡设备*

    > #amixer cset name='LO0_MUX' LI8

    > #arecord -f dat -t wav -D hw:1,0 | aplay -D hw:0,0 &

    pc端执行:

    > $aplay -D hw:1,0 test.wav

3. pc端选择usb 声卡设备，开发板录音，pc端播放声音

    > #amixer cset name='LO6_MUX' LI2

    > #arecord -f dat -t wav -D hw:0,6 | aplay -D hw:1,0 &

    pc端执行:

    > $ arecord -f dat -t wav -D hw:1,0 | aplay -D hw:0,0 


## 3.3 legacy配置

#### 3.3.1 device Serial Gadget

驱动配置：

```txt
Symbol: USB_G_SERIAL [=y]                                                                                            
Type  : tristate                                                                                                     
Prompt: Serial Gadget (with CDC ACM and CDC OBEX support)                                                            
  Location:                                                                                                          
    -> Device Drivers                                                                                                
      -> USB support (USB_SUPPORT [=y])                                                                              
        -> USB Gadget Support (USB_GADGET [=y])                                                                      
          -> USB Gadget Drivers (<choice> [=y])                                                                      
  Defined at drivers/usb/gadget/legacy/Kconfig:260                                                                   
  Depends on: <choice> && TTY [=y]                                                                                   
  Selects: USB_U_SERIAL [=y] && USB_F_ACM [=y] && USB_F_SERIAL [=y] && USB_F_OBEX [=y] && USB_LIBCOMPOSITE [=y]
```

使用方法：

1. pc端识别设备
    >$ dmesg

    ```txt
    设备信息：
    [2189021.818872] usb 2-1.5: new high-speed USB device number 25 using ehci-pci
    [2189021.915645] usb 2-1.5: New USB device found, idVendor=0525, idProduct=a4a7
    [2189021.915651] usb 2-1.5: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [2189021.915655] usb 2-1.5: Product: Gadget Serial v2.4
    [2189021.915659] usb 2-1.5: Manufacturer: Linux 4.4.94+ with 13500000.otg
    [2189021.922892] cdc_acm 2-1.5:2.0: ttyACM0: USB ACM device
    ```

2. 互发信息

    pc端：

    > $echo 1111111111 > /dev/ttyACM0

    > $cat /dev/ttyACM0

    开发板端：

    > #cat ttyGS0

    > #echo 2222 > ttyGS0

#### 3.3.2 device uvc Gadget

驱动配置：

```txt
Symbol: USB_G_WEBCAM [=y]                                                                                   
Type  : tristate                                                                                            
Prompt: USB Webcam Gadget                                                                                   
  Location:                                                                                                 
    -> Device Drivers                                                                                       
      -> USB support (USB_SUPPORT [=y])                                                                     
        -> USB Gadget Support (USB_GADGET [=y])                                                             
          -> USB Gadget Drivers (<choice> [=y])                                                             
  Defined at drivers/usb/gadget/legacy/Kconfig:471                                                          
  Depends on: <choice> && VIDEO_DEV [=y]                                                                    
  Selects: USB_LIBCOMPOSITE [=y] && VIDEOBUF2_VMALLOC [=y] && USB_F_UVC [=y] && USB_DWC2_HIGHWIDTH_FIFO [=y]
```

使用方法：

1. pc端识别设备
    >$ dmesg

    ```txt
    设备信息：
    [1278801.857403] usb 2-1.7: new high-speed USB device number 25 using ehci-pci
    [1278801.954312] usb 2-1.7: New USB device found, idVendor=1d6b, idProduct=0102
    [1278801.954318] usb 2-1.7: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [1278801.954322] usb 2-1.7: Product: Webcam gadget
    [1278801.954326] usb 2-1.7: Manufacturer: Linux Foundation
    [1278801.976022] uvcvideo: Found UVC 1.00 device Webcam gadget (1d6b:0102)
    [1278801.983318] input: Webcam gadget as /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.7/2-1.7:1.0/input/input119
    ```

2. 手机端测试方法

   1. 手机通过转接线连接到开发板
   2. 手机打开APP: `USB摄像头`

      **注**: *如果手机识别到设别,请重启开发板*

3. 详细测试方法

    同上: [325-device-uvc-gadget](#325-device-uvc-gadget)

## 3.3 FAQ

* 修改usb gadget配置：

  * 修改默认支持的gadget设备，配置文件如下
  路径：*buildroot/package/ingenic/system_config/usb-device/S90usb*

```c
        /etc/init.d/usb/uvc     $1
        /etc/init.d/usb/adb     $1
        /etc/init.d/usb/mass_storage $1
        #/etc/init.d/usb/hid    $1        /*当使用hid时，需要将其它设备注释上*/
        /etc/init.d/usb/printer $1
        /etc/init.d/usb/rndis   $1
        /etc/init.d/usb/uac1    $1
        #/etc/init.d/usb/serial $1        /*当使用serial时，需要将其它设备注释上*/
```

* 华为手机使用uvc设备，otg无法识别问题

```txt
  使用legacy配置，可以解决。
```
