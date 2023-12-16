Display(DPU)显示接口驱动和应用
===========================
[TOC]
<!-- toc -->

----
# 1. 模块简介

略

# 2. 内核空间

驱动代码位置：

```
drivers/video/fbdev/ingenic/fb_v12
├── displays        /*添加屏设备目录*/
│   ├── Kconfig
│   ├── Makefile
│   └── panel-y88249.c
├── dpu_reg.h
├── ingenicfb.c   /*控制器驱动*/
├── ingenicfb.h
├── jz_mipi_dsi   /*mipi dsi控制器驱动*/
│   ├── built-in.o
│   ├── jz_mipi_dsi.c
```

## 2.1. 设备树配置

在arch/mips/boot/dts/ingenic/板级.dts下配置：

### 2.1.1 dpu 控制器配置
```
dpu 控制器配置：
&dpu {
    status = "okay";
    port {
        dpu_out_ep: endpoint {
            remote-endpoint = <&panel_y88249_ep>;
            /*remote-endpoint = <&panel_kd050hdfia019_ep>;*/
        };
    };
};
```

### 2.1.2 显示屏配置
```
display-dbi {
        compatible = "simple-bus";
        #interrupt-cells = <1>;
        #address-cells = <1>;
        #size-cells = <1>;
        ranges = <>;
        panel_y88249@0 {
                compatible = "ingenic,y88249";
                status = "okay";                /*默认配置*/
                pinctrl-names = "default";
                pinctrl-0 = <&tft_lcd_pb>;  /*gpio功能配置*/
                ingenic,vdd-en-gpio = <&gpc 12 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                ingenic,rst-gpio = <&gpc 1 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                ingenic,pwm-gpio = <&gpc 15 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
                port {
                        panel_y88249_ep: endpoint {
                                remote-endpoint = <&dpu_out_ep>;
                        };
                };
        };
};
```

### 2.1.3 pwm背光配置：
```
&pwm {
        pinctrl-names = "default";
        pinctrl-0 = <&pwm1_pc>;  /*按照实际情况配置pwm的gpio*/
        status = "okay";
};
backlight {
        compatible = "pwm-backlight";
        pwms = <&pwm 1 1000000>; /* 选择pwm1控制背光，设置period 1000000ns */
        brightness-levels = <0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15>; /* 背光等级，可根据需求调整*/
        default-brightness-level = <4>;
};
```
## 2.2. 驱动配置

在SDK 顶层输入make kernel-menuconfig， 然后配置：


### 2.2.1. dpu控制器和显示屏驱动配置
```
Frame buffer Devices  --->
        <*> Support for frame buffer devices  --->
        <*> Ingenic Framebuffer Driver  ----
        (9) Vsync skip ratio[0..9]
        (2) how many frames support  /*配置驱动支持的frame数*/
        (4) how many layers support  /*配置驱动支持的layer数，帧内存 = frames * layers * 4，为了节省内存按需要选择支持的层数*/
        [*] fb test for displaying color bar  /*测试彩条，选择logo时应该去掉该选项*/
        [*] ingenic mipi dsi interface  /*如果是mipi dsi屏需选择*/
        <*> Ingenic Framebuffer Driver for Version 12  --->
                -*-   Supported lcd panels  --->
                        <*>   lcd panel y88249
```

### 2.2.2 pwm背光驱动配置
```
  │ Symbol: PWM_INGENIC_V2 [=y]
  │ Type  : tristate
  │ Prompt: Ingenic PWM V2 support
  │   Location:
  │     -> Device Drivers
  │       -> Pulse-Width Modulation (PWM) Support (PWM [=y])
  │   Defined at drivers/pwm/Kconfig:202
  │   Depends on: PWM [=y] && MACH_XBURST2 [=y]

Symbol: BACKLIGHT_PWM [=y]
  │ Type  : tristate
  │ Prompt: Generic PWM based Backlight Driver
  │   Location:
  │     -> Device Drivers
  │       -> Graphics support
  │         -> Backlight & LCD device support (BACKLIGHT_LCD_SUPPORT [=y])
  │           -> Lowlevel Backlight controls (BACKLIGHT_CLASS_DEVICE [=y])
  │   Defined at drivers/video/backlight/Kconfig:261
  │   Depends on: HAS_IOMEM [=y] && BACKLIGHT_LCD_SUPPORT [=y] && BACKLIGHT_CLASS_DEVICE [=y] && PWM [=y]
```

## 2.3. 屏幕配置
x2000和m300 dpu 支持的屏幕种类：

* **smart lcd**: dbi硬件接口，液晶屏有自己的ram，防泪屏需要有te功能
* **tft lcd**: dpi硬件接口，没有ram，需要按照一定的帧率刷新屏幕
* **mipi smart lcd**: dsi硬件接口，液晶屏有自己的ram
* **mipi tft lcd**: dsi硬件接口，没有ram，需要按照一定的帧率刷新屏幕

屏幕驱动配置解释：

### 2.3.1. 屏幕配置
```
struct lcd_panel {
        const char *name;
        unsigned int num_modes;  /*显示模式支持的数量，固定值1*/      
        struct fb_videomode *modes; /*显示模式*/
        struct jzdsi_data *dsi_pdata;  /*如果屏幕是mipi dsi接口需要实现*/

        enum ingenic_lcd_type lcd_type; /*屏幕种类，如果是mipi tft lcd需要配置LCD_TYPE_TFT，其它的按功能配置*/
        unsigned int bpp;     /*不需要填充*/
        unsigned int width;  /*屏幕实际物理宽度，单位mm*/
        unsigned int height;  /*屏幕实际物理高度，单位mm*/

        struct smart_config *smart_config; /*如果屏幕是smart lcd需要实现*/
        struct tft_config *tft_config; /*如果屏幕是tft lcd需要实现*/

        unsigned dither_enable:1; /*打开dither功能*/
        struct {
                unsigned dither_red;
                unsigned dither_green;
                unsigned dither_blue;
        } dither;

        struct lcd_panel_ops *ops; /*不需要实现*/
};

struct fb_videomode {
        const char *name;       /* optional */
        u32 refresh; /*配置帧率，驱动会根据配置参数和帧率计算pixclock*/
        u32 xres; /*显示有效宽度，按照屏手册填写*/
        u32 yres; /*显示有效高度，按照屏手册填写*/
        u32 pixclock;
        u32 left_margin; /*tft屏时按照手册填写，当smart lcd时赋值0*/
        u32 right_margin; /*tft屏时按照手册填写，当smart lcd时赋值0*/
        u32 upper_margin; /*tft屏时按照手册填写，当smart lcd时赋值0*/
        u32 lower_margin; /*tft屏时按照手册填写，当smart lcd时赋值0*/
        u32 hsync_len; /*tft屏时按照手册填写，当smart lcd时赋值0*/
        u32 vsync_len; /*tft屏时按照手册填写，当smart lcd时赋值0*/
        u32 sync;  /*当mipi dsi tft lcd时需要赋值（FB_SYNC_HOR_HIGH_ACT & FB_SYNC_VERT_HIGH_ACT)，其他的屏幕不关注*/
        u32 vmode;  /*默认值：FB_VMODE_NONINTERLACED*/
        u32 flag;
};
```

### 2.3.2 smart lcd 配置
```
struct smart_config {
        unsigned int te_switch;    /*smart lcd te功能控制*/
        unsigned int te_mipi_switch; /*设置 mipi dsi smart lcd te功能控制*/
        unsigned int te_md;  /*0:te前沿有效，1:后沿有效*/
        unsigned int te_dp;  /*0:te低电平有效，1:高电平有效*/
        unsigned int te_anti_jit; /*0:te信号保持1个pixclk有效，1:te信号保持3个pixclk有效*/
        unsigned int dc_md;  /*0:DC高电平数据，低电平命令, 1:DC高电平命令，低电平数据*/
        unsigned int wr_md;  /*0:下降沿采样，1:上升沿采样 */
        enum smart_lcd_type smart_type; /*smart slcd种类，支持6800/8080/spi-3/spi-4*/
        enum smart_lcd_format pix_fmt;  /*总线数据格式，支持565,666等*/
        enum smart_lcd_dwidth dwidth;  /*数据总线宽度*/
        enum smart_lcd_cwidth cwidth;  /*命令总线宽度*/
        unsigned int bus_width;

        unsigned long write_gram_cmd;  /*发送数据前需要发送的命令，默认0x2c*/
        unsigned int length_cmd; /*不需要实现*/
        struct smart_lcd_data_table *data_table; /*配置屏幕命令表*/
        unsigned int length_data_table; /*配置屏幕命令数量*/
        int (*init) (void); /*不需要实现*/
        int (*gpio_for_slcd) (void); /*不需要实现*/
};
```

### 2.3.3 tft lcd配置
```
struct tft_config {
        unsigned int pix_clk_inv; /*0:pixclk默认输出， 1:反转pixclk*/
        unsigned int de_dl; /*0:DE引脚高电平输出有效数据， 1:低电平输出有效数据*/
        unsigned int sync_dl; /*0:vsync和hsync引脚高电平输出有效数据， 1:低电平输出有效数据*/
        enum tft_lcd_color_even color_even; /*偶数行时总线RGB顺序*/
        enum tft_lcd_color_odd color_odd;  /*奇数行时总线RGB顺序*/
        enum tft_lcd_mode mode; /*总线数据格式，支持888/666/565*/
};
```

### 2.3.4 mipi dsi lcd配置
```
struct jzdsi_data jzdsi_pdata = {
        .modes = &panel_modes, /*显示模式*/
        .video_config.no_of_lanes = 2,  /*按照硬件连接填写，支持1、2lane*/
        .video_config.virtual_channel = 0, /*默认值:0*/
        .video_config.color_coding = COLOR_CODE_24BIT, /*RGB888:COLOR_CODE_24BIT, RGB565:COLOR_CODE_16BIT_CONFIG1，注：当smart lcd时需要配置COLOR_CODE_24BIT*/
        .video_config.video_mode = VIDEO_BURST_WITH_SYNC_PULSES, /*默认值:VIDEO_BURST_WITH_SYNC_PULSES*/
        .video_config.receive_ack_packets = 0, /*默认值:0*/
        .video_config.is_18_loosely = 0,  /*默认值:0*/
        .video_config.data_en_polarity = 1, /*默认值:1*/
        .video_config.byte_clock = 0, /*默认值:0,驱动根据配置参数自动计算*/
        .video_config.byte_clock_coef = MIPI_PHY_BYTE_CLK_COEF_MUL6_DIV5, /*byte_clock系数，需要根据实际情况变动系数，保证正常显示情况下系数越小越好，例：MUL6_DIV5=1.2(乘6除5)*/

        .dsi_config.max_lanes = 2, /*固定值:2*/
        .dsi_config.max_hs_to_lp_cycles = 100, /*默认值:100*/ 
        .dsi_config.max_lp_to_hs_cycles = 40, /*默认值:40*/
        .dsi_config.max_bta_cycles = 4095, /*默认值:4095*/
        .dsi_config.color_mode_polarity = 1, /*默认值：1*/
        .dsi_config.shut_down_polarity = 1,  /*默认值：1*/
        .dsi_config.max_bps = 2750,     /*默认值:2.75Gbps*/
        .bpp_info = 24, /*RGB888:24 RGB565:16，注：当smart lcd时需要配置24*/
};
```

#### 2.3.4.1 mipi dst tft lcd配置

当屏幕是mipi dsi tft lcd时，赋值tft_config需按照下面配置
```
static struct tft_config tft_cfg = {
        .pix_clk_inv = 0, /*固定值：0*/
        .de_dl = 0, /*固定值：0*/
        .sync_dl = 0, /*固定值：0*/
        .color_even = TFT_LCD_COLOR_EVEN_RGB, /*固定值*/
        .color_odd = TFT_LCD_COLOR_ODD_RGB, /*固定值*/
        .mode = TFT_LCD_MODE_PARALLEL_888, /*RGB888：TFT_LCD_MODE_PARALLEL_888， RGB565:TFT_LCD_MODE_PARALLEL_565*/
};
```

#### 2.3.4.2 mipi dsi smart lcd配置

当屏幕是mipi dsi smart lcd时，配置smart_confg时需要按照下面配置
```
struct smart_config smart_cfg = {
        .dc_md = 0, /*固定值：0*/
        .wr_md = 1, /*固定值：1*/
        .smart_type = SMART_LCD_TYPE_8080, /*固定值:SMART_LCD_TYPE_8080*/
        .pix_fmt = SMART_LCD_FORMAT_888,  /*固定值:SMART_LCD_FORMAT_888*/
        .dwidth = SMART_LCD_DWIDTH_24_BIT, /*固定值:SMART_LCD_DWIDTH_24_BIT*/
        .te_mipi_switch = 1,  /*1:打开te功能，0:关闭te功能，使用PB27的func2做te引脚，检测到高电平就输出数据*/
        .te_switch = 1, /*1:打开te功能，0：关闭te功能*/
};
```

#### 2.3.4.3 mipi dsi屏幕寄存器通用配置方式
```
方式一:
发送大于等于2个参数:
        struct dsi_cmd_packet cmd = {0x39, 0x05, 0x00, {0x2A, 0x00, 0x00, 0x02, 0xCF}}
        0x39: 命令种类，发送大于等于2个参数
        0x05: 参数个数，5个参数
        0x00: 无意义，默认填充0x00
        {0x2A, 0x00, 0x00, 0x02, 0xCF}: 5个参数值
发送2个参数:
        struct dsi_cmd_packet cmd = {0x15, 0xC2, 0x08}
        0x15: 命令种类，发送2个参数
        0xC2: 第一个参数
        0x08: 第二个参数
发送1个参数
        struct dsi_cmd_packet cmd = {0x05, 0x10, 0x00}
        0x05: 命令种类，发送1个参数
        0x10: 第一个参数
        0x00: 无意义，默认填充0x00
```

# 3. 用户空间

## 3.1. 设备节点

```
/dev/fb0
```

## 3.2. 测试程序

```
manhatton工程
packages/example/Sample/dpu
```

## 3.3. 测试方法

测试应用为testsuite/dpu

```
参数设置：
        -n 设置刷新的帧数
        -F 配置１，２层显示的格式，支持rgb888,rgb565,nv12,yuv422，3,4层默认格式rgb888
        -o 配置每帧显示的图像打开的层数
        -t 打开tlb功能，不使用kernel申请预留的内存，使用应用层临时申请的用户空间内存

具体使用方式：
        ./dpu -n 1000                         /*　使用-n 配置刷新1000帧后结束应用程序　*/
        ./dpu -n 1000 -o 1                    /*　使用-o 配置打开的layer数，要求等于小于kernel配置的最大layer数 */
        ./dpu -n 1000 -o 1 -F rgb565          /*　使用-F 配置１，２层显示格式，为rgb565 */
        ./dpu -n 1000 -o 1 -F rgb565  -t      /*　使用-t 打开tlb功能 */
```

# 4. 测试程序介绍


## 4.1 驱动内存分配管理
```
        内核申请内存总量：
        内存总量= width * height * MAX_BYTES_PER_PIX * CONFIG_FB_INGENIC_NR_FRAMES * CONFIG_FB_INGENIC_NR_LAYERS
        CONFIG_FB_INGENIC_NR_FRAMES /*kernel使用make menuconfig配置FRAMES*/
        CONFIG_FB_INGENIC_NR_LAYERS /*kernel使用make menuconfig配置LAYERS*/
        MAX_BYTES_PER_PIX = 4  /*每个像素点占用的最大bytes, RGB888格式时每个像素占用4个bytes*/
        width 显示屏幕宽度
        height 显示屏高度

        内核内存分配方式：
        当CONFIG_FB_INGENIC_NR_FRAMES = 3， CONFIG_FB_INGENIC_NR_LAYERS = 4内核内存分配方式：
        -------------------------------------------------------------------------------------------------------------
        |               frame 0             |               frame 1             |               frame 2             |
        -------------------------------------------------------------------------------------------------------------
        | layer0 | layer1 | layer2 | layer3 | layer0 | layer1 | layer2 | layer3 | layer0 | layer1 | layer2 | layer3 |
        -------------------------------------------------------------------------------------------------------------

        当CONFIG_FB_INGENIC_NR_FRAMES = 2， CONFIG_FB_INGENIC_NR_LAYERS = 2内核内存分配方式：
        -------------------------------------
        |     frame 0     |      frame 1    |
        -------------------------------------
        | layer0 | layer1 | layer0 | layer1 |
        -------------------------------------

        用户分配内存方法：
        使用mmap获得数据内存用户空间的地址，按照内存分配方式获取对应帧和层的数据地址
```


## 4.2 测试程序格式配置方式
```
        /* 使用ioctl传递配参数，显示相关配置都包含在 jzfb_frm_cfg结构体中 */
        ioctl(int fd, JZFB_PUT_FRM_CFG, struct jzfb_frm_cfg *frm_cfg)

        /* jzfb_frm_cfg 包含4层的配置 */
        struct jzfb_frm_cfg {
                struct jzfb_lay_cfg lay_cfg[4];  /*硬件最大支持4层*/

        /* 每层的配置参数 */
        struct jzfb_lay_cfg {
    unsigned int lay_en;            /*对应层的使能开关, 1:打开，0:关闭*/
        unsigned tlb_en;                /*对应层的tlb 开关*/
    unsigned int lay_scale_en;      /*对应层缩放功能使能开关, 1:打开，0:关闭*/
    unsigned int lay_z_order;       /*本层相对其他层位置，设置值0-3, 0:最底层，3最顶层*/
    unsigned int source_w;          /*数据源宽度，设置值应小于4096，可大于或小于实际屏幕显示的宽度*/
    unsigned int source_h;          /*数据源高度，设置值应小于4096，可大于或小于实际屏幕显示的高度*/1
    unsigned int disp_pos_x;        /*本层最终显示在屏幕上对应的x坐标，设置值应大于等于0，小于等于实际屏幕宽度减去显示宽度（经过剪切或缩放后的宽度）*/
    unsigned int disp_pos_y;        /*本层最终显示在屏幕上对应的y坐标，设置值应大于等于0，小于等于实际屏幕宽度减去显示高度（经过剪切或缩放后的高度）*/
    unsigned int g_alpha_en;        /*对应层的全局透明度开关，1:打开，0:关闭*/
    unsigned int g_alpha_val;       /*对应层的全局透明度值，设置值0-0xff, 设置值0时本层相对于底层全透明，看不到本层图像，设置值0xff时看不到底层图像*/
    unsigned int color;             /*当显示格式为RGB时，设置RGB三原色在内存中实际排列顺序*/
    unsigned int format;            /*设置本层数据源格式，支持ARGB888、RGB888、RGB565、ARGB1555、RGB555、NV12、NV21、YUV422，其中只有1,2层支持YUV格式数据*/
    unsigned int stride;            /*设置数据源每行数据在内存中跨度，单位是像素，设置值应小于4096，大于等于数据源宽度，通过stride设置可以实现从原点位置剪切*/
    unsigned int scale_h;           /*数据源缩放后的宽度，设置值应大于等于20，小于等于实际显示屏宽度*/
    unsigned int scale_h;           /*数据源缩放后的高度，设置值应大于等于20，小于等于实际显示屏高度*/
    unsigned int addr[3];           /*当使用TLB功能时，传递对应层数据内存地址*/
    unsigned int uv_offset[3];      /*当数据格式是nv12或nv21时设置uv地址偏移，相对本层buffer首地址的偏移，单位是byte*/
        };

        缩放设置,800*480数据缩放到320*240的显示屏上
                   原始数据                     显示屏幕
                     800                        320
            -----------------------        --------------
           |                      |        |            |
        480|                      |        |            | 240
           |                      |        |            |
           |                      |        --------------
           -----------------------
         {
                 .lay_en = 1;
                 .lay_scale_en = 1;
                 .lay_z_order = 0x0;  /*选择最底层*/
                 .source_w = 800;
                 .source_h = 480;
                 .disp_pos_x = 0;
                 .disp_pos_y = 0;
                 .g_alpha_en = 0;
                 .g_alpha_val = 0;
                 .color = LAYER_CFG_COLOR_RGB;
                 .format = LAYER_CFG_FORMAT_RGB888；
                 .stride = 800;
                 .scale_w = 320;
                 .scale_h = 240;
                 .addr[0] = 0 /*不使用tlb功能*/
                 .uv_offset[0] = 800*480;  /*当数据格式是nv12或nv21时设置*/
         }

        缩放设置,800*480数据缩放到240*120后从屏幕的坐标（80， 120）开始显示
                   原始数据                     显示屏幕
                     800                        320
            -----------------------        --------------
           |                      |        |   |        |
        480|                      |        |   |        | 240
           |                      |        |   ---------|
           |                      |        --------------
           -----------------------
         {
                 .lay_en = 1;
                 .lay_scale_en = 1;
                 .lay_z_order = 0x0;  /*选择最底层*/
                 .source_w = 800;
                 .source_h = 480;
                 .disp_pos_x = 80;
                 .disp_pos_y = 120；
                 .g_alpha_en = 0;
                 .g_alpha_val = 0;
                 .color = LAYER_CFG_COLOR_RGB;
                 .format = LAYER_CFG_FORMAT_RGB888；
                 .stride = 800;
                 .scale_w = 240;
                 .scale_h = 120;
                 .addr[0] = 0 /*不使用tlb功能*/
                 .uv_offset[0] = 800*480;  /*当数据格式是nv12或nv21时设置*/
         }

        原点剪切设置,800*480数据从原点剪切240*240后从屏幕的坐标（80， 0）开始显示
                   原始数据                     显示屏幕
                     800                        320
            -----------------------        --------------
           |--------              |        |   |        |
           |   240  |             |        |   |        | 240
        480|     240|             |        |   |   240  |
           |        |             |        --------------
           -----------------------           （80,0）
        （0,0）
         {
                 .lay_en = 1;
                 .lay_scale_en = 0;
                 .lay_z_order = 0x0;  /*选择最底层*/
                 .source_w = 240;
                 .source_h = 240;
                 .disp_pos_x = 80;
                 .disp_pos_y = 0；
                 .g_alpha_en = 0;
                 .g_alpha_val = 0;
                 .color = LAYER_CFG_COLOR_RGB;
                 .format = LAYER_CFG_FORMAT_RGB888；
                 .stride = 800;
                 .scale_w = 0;
                 .scale_h = 0;
                 .addr[0] = 0 /*不使用tlb功能*/
                 .uv_offset[0] = 800*480;  /*当数据格式是nv12或nv21时设置*/
         }
```

## 4.3 刷新图像数据方式
```
        驱动是以帧为单位更新数据，每帧对应多层，完成一次数据刷新就是把一帧对应多层数据刷新到屏幕上。
        当使用驱动申请的连续物理内存，每层的内存和帧对应的关系可以参照上一节的驱动内存分配管理。
        当使用用户空间地址需要打开tlb功能，tlb功能可以完成用户空间的虚拟地址到物理地址转换。
        刷新帧使用ioctl(int fd, FBIOPAN_DISPLAY, struct fb_var_screeninfo *var_info）函数
        通过更改var_info结构体成员变量yoffset，
        当刷新第0帧 yoffset = height * 0;  /*height 屏幕的高度，以像素点为单位 */
        当刷新第1帧 yoffset = height * 1;
        当刷新第2帧 yoffset = height * 2;
```

## 4.4 tlb功能使用方法
```
        (1)申请的数据buffer地址需要按4096对齐.
        (2)数据buffer虚拟内存到物理内存的映射和tlb表的制作函数
        unsigned int gtlb_base = ioctl(int fd, JZFB_DMMU_MAP, struct dpu_dmmu_map_info *di)
        如果使用的数据buffer是多次申请获得的，需要多次调用函返回tlb表地址，也可以一次完成buffer的申请并调用一次函数。
                int fd  /*fb设备文件描述符*/
                unsigned int gtlb_base /*tlb一级页表地址，地址值唯一，所以多次调用后返回值相同*/
                struct dpu_dmmu_map_info {
                        unsigned int addr;  /*buffer 地址*/
                        unsigned int len;   /*buffer地址的长度*/
                };
        (3)配置tlb一级页表地址函数
        ioctl(int fd, JZFB_USE_TLB, unsigned int gtlb_base)；
                int fd  /*fb设备文件描述符 */
                unsigned int gtlb_base
        (4)给驱动传递每层的buffer地址
        struct jzfb_lay_cfg {
                .addr[0] = 0x7xxxxxxx; /*0帧的地址*/
                .addr[1] = 0x7xxxxxxx; /*1帧的地址*/
                .addr[2] = 0x7xxxxxxx; /*2帧的地址*/
        }
        (5)更新图像数据后刷新cache函数
        ioctl(int fd, JZFB_DMMU_FLUSH_CACHE, struct dpu_dmmu_map_info *di)
```

# 5. 注意事项


* 最多支持2层缩放，缩放可以是任意2层
* 只有1,2层支持yuv422、nv12、nv21

# 6. 调试屏幕列表

```
屏幕                            种类                      分辨率
panel-ma0060                    mipi slcd               720*1280
panel-jd9161z                   mipi tft                480*800
panel-st7701s                   mipi tft                480*800
panel-kd050hdfia019             mipi tft                480*854
panel-tl040hds01ct              mipi tft                720*720
panel-ylym286a                  mipi tft                1920*1080
panel-tl040wvs03ct              tft                     480*480
panel-y88249                    tft                     640*480
panel-yts500xlai                tft                     720*1280
panel-kd035hvfbd037             slcd                    320*480
```
