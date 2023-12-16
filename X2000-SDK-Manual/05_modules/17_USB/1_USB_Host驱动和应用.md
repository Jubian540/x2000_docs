# 1 USB Host驱动和应用
[TOC]
<!-- toc -->

----

### 3.1控制器作为host

#### 3.1.1 usb host 功能验证情况

* 支持host功能：`mass storage`、`usb camera`、`hid`。

#### 3.1.2 USB Mass Storage support

驱动配置：

```
(1)VFAT (Windows-95) fs support
Symbol: VFAT_FS [=y]                     
Type  : tristate                         
Prompt: VFAT (Windows-95) fs support     
  Location:                              
    -> File systems                      
      -> DOS/FAT/NT Filesystems          
  Defined at fs/fat/Kconfig:60           
  Depends on: BLOCK [=y]                 
  Selects: FAT_FS [=y]                   


(2)Codepage 437 (United States, Canada) 
Symbol: NLS_CODEPAGE_437 [=y]                             
Type  : tristate                                          
Prompt: Codepage 437 (United States, Canada)              
  Location:                                               
    -> File systems                                       
      -> Native language support (NLS [=y])               
  Defined at fs/nls/Kconfig:39                            
  Depends on: NLS [=y]                                    


(3)NLS ISO 8859-1  (Latin 1; Western European Languages) 
Symbol: NLS_ISO8859_1 [=y]                                        
Type  : tristate                                                  
Prompt: NLS ISO 8859-1  (Latin 1; Western European Languages)     
  Location:                                                       
    -> File systems                                               
      -> Native language support (NLS [=y])                       
  Defined at fs/nls/Kconfig:318                                   
  Depends on: NLS [=y]                                            

(4)SCSI disk support
Symbol: BLK_DEV_SD [=y]                        
Type  : tristate                               
Prompt: SCSI disk support                      
  Location:                                    
    -> Device Drivers                          
      -> SCSI device support                   
  Defined at drivers/scsi/Kconfig:73           
  Depends on: SCSI [=y]

(5)
Symbol: USB_STORAGE [=y]                                        
Type  : tristate                                                
Prompt: USB Mass Storage support                                
  Location:                                                     
    -> Device Drivers                                           
      -> USB support (USB_SUPPORT [=y])                         
        -> Support for Host-side USB (USB [=y])                 
  Defined at drivers/usb/storage/Kconfig:8                      
  Depends on: USB_SUPPORT [=y] && USB [=y] && SCSI [=y]
```

使用方法：

1. 插入Ｕ盘弹出打印信息

    ```txt
    [   46.070034] usb 1-1: new high-speed USB device number 2 using dwc2
    [   46.296250] usb 1-1: New USB device found, idVendor=0951, idProduct=1666
    [   46.303202] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [   46.310592] usb 1-1: Product: DataTraveler 3.0
    [   46.315185] usb 1-1: Manufacturer: Kingston
    [   46.319508] usb 1-1: SerialNumber: 60A44C413C4EB211A98A00A0
    [   46.325821] usb-storage 1-1:1.0: USB Mass Storage device detected
    [   46.342363] scsi host0: usb-storage 1-1:1.0
    [   47.726378] scsi 0:0:0:0: Direct-Access     Kingston DataTraveler 3.0 PMAP PQ: 0 ANSI: 6
    [   47.736215] sd 0:0:0:0: [sda] 30277632 512-byte logical blocks: (15.5 GB/14.4 GiB)
    [   47.752538] sd 0:0:0:0: [sda] Write Protect is off
    [   47.759990] sd 0:0:0:0: [sda] No Caching mode page found
    [   47.765648] sd 0:0:0:0: [sda] Assuming drive cache: write through
    [   47.780215]  sda: sda1
    [   47.789136] sd 0:0:0:0: [sda] Attached SCSI removable disk
    ```

2. 挂载Ｕ盘到文件系统
  
    > #mount -t vfat /dev/sda1 mnt/

3. 数据交换进入到/mnt即可操作完成

#### 3.1.3 USB host camera

驱动配置：

```txt
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
```

使用方法：

1. 插入摄像头识别成功

    ```txt
    [   56.560037] usb 1-1: new high-speed USB device number 2 using dwc2
    [   56.910300] usb 1-1: New USB device found, idVendor=058f, idProduct=5608
    [   56.917231] usb 1-1: New USB device strings: Mfr=3, Product=1, SerialNumber=0
    [   56.924635] usb 1-1: Product: USB 2.0 Camera
    [   56.929114] usb 1-1: Manufacturer: Alcor Micro, Corp.
    [   56.939481] uvcvideo: Found UVC 1.00 device USB 2.0 Camera (058f:5608)
    [   56.949251] input: USB 2.0 Camera as /devices/platform/ahb2/13500000.otg/usb1/1-1/1-1:1.0/input/input0
    ```

2. 拍照测试

    ```txt
    usage: uvcview [-d <device>] [-c <count>]
    --help -H               print this message 
    --print_formats -F              print video device info
    --device -d             , video device, default is /dev/video0
    --width -w              grab width
    --height -h             grab height
    --count -c              set the count to grab
    --rate -r               frame sample rate(fps)
    --yuv -y                use yuyv input format
    --timeout -t            select timeout
    --match -m              do picture match test
    --perf -p               do performance test
    ```

    注：*/dev/vidio5是usb camera生成的标准video节点．*
    > #./grab -w 640 -h 480 -d /dev/video5 -y -c 3

3. 生成图像

    ```txt
    p-0.jpg　p-１.jpg　p-２.jpg
    ```

#### 3.1.4 USB  host hid mouse

驱动配置：

```txt
(1)USB HIDBP Mouse (simple Boot) support
Symbol: USB_MOUSE [=y]                                                         
Type  : tristate                                                               
Prompt: USB HIDBP Mouse (simple Boot) support                                  
  Location:                                                                    
    -> Device Drivers                                                          
      -> HID support                                                           
        -> USB HID support                                                     
          -> USB HID Boot Protocol drivers                                     
  Defined at drivers/hid/usbhid/Kconfig:66                                     
  Depends on: USB_HID [=n]!=y && EXPERT [=y] && USB [=y] && INPUT [=y]         

(2)Mouse interface
Symbol: INPUT_MOUSEDEV [=y]                                                          
Type  : tristate                                                                     
Prompt: Mouse interface                                                              
  Location:                                                                          
    -> Device Drivers                                                                
      -> Input device support                                                        
        -> Generic input layer (needed for keyboard, mouse, ...) (INPUT [=y])        
  Defined at drivers/input/Kconfig:95                                                
  Depends on: !UML && INPUT [=y]
```

使用方法：

1. 插入鼠标设备到控制器

    ```txt
    [  117.100038] usb 1-1: new low-speed USB device number 2 using dwc2
    [  117.313977] usb 1-1: New USB device found, idVendor=046d, idProduct=c077
    [  117.320929] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [  117.328305] usb 1-1: Product: USB Optical Mouse
    [  117.333098] usb 1-1: Manufacturer: Logitech
    [  117.338568] input: Logitech USB Optical Mouse as /devices/platform/ahb2/13500000.otg/usb1/1-1/1-1:1.0/input/input0
    ```

2. 执行捕获事件的程序后移动鼠标触发事件

    > #cd /testsuite/usb_test/usb_host/getevent_test/

    > #./getevent_test 0

    ```txt
    /dev/input/mouse0
        evdev version: 0.0.0
        name: 
        features: relative reserved unknown unknown unknown unknown unknown unknown unknown
    /dev/input/mouse0: open, fd = 3
    Sun Mar  1 16:38:18 2020.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    [  125.626285] random: nonblocking pool is initialized
    Wed Dec 25 06:26:16 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    Thu Dec 26 00:59:52 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    Mon Dec 23 18:27:20 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    Thu Dec 26 00:55:36 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    Wed Dec 25 06:47:36 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    Tue Dec 24 12:31:04 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    Wed Dec 25 06:47:36 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    Mon Dec 23 18:10:16 2019.000000 type 0x0011; code 0x000e; value 0x00000000; Led
    ....
    ....
    ....
    ```
