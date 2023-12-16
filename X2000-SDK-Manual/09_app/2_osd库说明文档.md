osd库说明文档
[TOC]
<!-- toc -->

----

该项目对layer（图层）进行叠加并将其显示在lcd屏中。使用IPC机制进行通信。该项目仅支持X2000和M300
# 1. 编译
```
#mkdir build && cd build 
#cmake .. 
#make
```

1.osdServer 可执行文件， 该可执行文件为OSD中的服务端。该可执行程序负责接收client端数据并进行叠加和显示。

2.libosdClient.so osd client端lib库。该库接口将为用户提供相应的OSD数据通讯接口。

# 2. 使用

1.启动服务器。 

#./osdServer

2.执行对应的client端代码

# 3. osd lib库接口说明 

# 3.1 API接口函数

### 3.1.1. 初始化函数

`int init_osd_client(char *name) `

参数说明： 
char *name：client客户端名 

### 3.1.2.销毁函数

`int uinit_client() `

### 3.1.3.获取内存地址

`int get_osd_package(struct osd_head_info *extp) `

功能说明：
申请内存空间，在server端会动态释放。

`int get_osd_reserved_package(struct osd_head_info *extp) `

功能说明：
申请内存空间，在server端会不被动态释放，直到client端退出，才会释放。 

## 3.1 结构体说明
```
struct osd_head_info {   
osd_lay_cfg *layer;   
/* Date memory address and len. */ 
uint8_t *shm_mem;   
Addr_t  shm_len;   
/* Date shared memory offset. */   
Addr_t  private_shm_off;   
/* Shared memory info. */   
struct osd_shm_info private_info; 
}; 
 
typedef struct osd_lay_cfg{   
    unsigned int lay_en;  // The state of the layer 1：en 0：disable.   
    unsigned int lay_scale_en; // Status of scaling 1：en 0：disable.   
    unsigned int lay_z_order; // Select the layer, a total of four layers.   
    unsigned int source_w; // The width of the original image data.   
    unsigned int source_h; //The height of the original image data   
    unsigned int disp_pos_x; // The displayed X-axis coordinate is relative to the 0'0 coordinate point   
    unsigned int disp_pos_y; // The displayed Y-axis coordinate is relative to the 0'0 coordinate point   
    unsigned int g_alpha_en; //State of transparency   
    unsigned int g_alpha_val; // The value of alpha. 0~0xFF   
    unsigned int color; //    
    unsigned int format; //Image data format.   
    unsigned int stride; // Memory size used by one line (byte).   
    unsigned int scale_w; // The value of the wide scale of the image data   
    unsigned int scale_h; // The value of the height scale of the image data   
    unsigned int uv_offset；
    }osd_lay_cfg; 
```
## 3.2. 例程
```
int set_layer_info(struct osd_head_info *osd_head,
		   int width, int height, int x, int y,
		   int fmt, int order)
{
  ingenicfb_lay_cfg *layer = osd_head->layer;

  layer->lay_en = 1;
  layer->lay_z_order = order;

  layer->lay_scale_en = 1;
  layer->scale_w = 720-450;
  layer->scale_h = 1280- 800;
  layer->source_w = width;
  layer->source_h = height;
  layer->disp_pos_x = 150;
  layer->disp_pos_y = 450;
  layer->g_alpha_en = 0;
  layer->g_alpha_val = 0xff;
  layer->color = 0;
  layer->format = fmt;
  layer->stride = width;
  return 0;
}
```
## 3.4.获取fb0节点的width值

`int get_osd_screen_width(uint32_t* width); `

参数说明： 
uint32_t* width ：width指针。 

## 3.5.获取fb0节点的height值

`int get_osd_screen_height(uint32_t* height); `

参数说明： 
uint32_t* height ：height指针。 

## 3.6.获取fb0支持的layer层数

`int get_osd_screen_layer_total(uint32_t *layer_total) `

参数说明： 
uint32_t *layer_total ：layer层数指针。 

## 3.7.Post 接口

`int post_osd_package(struct osd_head_info *extp) `

## 3.8 实例代码
```
#include <stdint.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <OsdApi.h>
#include <test.h>
#include <pthread.h>

static int gen_bgra_datax(uint8_t *dest,  uint32_t stride, uint32_t h,
		  uint8_t b, uint8_t g, uint8_t r, uint8_t a)
{
  int i, j;

  for (i = 0; i < h; i++)
    for (j = 0; j < stride; j += 4)
      {
	    dest[i * stride + j + 0] = b;
	    dest[i * stride + j + 1] = g;
	    dest[i * stride + j + 2] = r;
	    dest[i * stride + j + 3] = a;
      }
  for (i = 100; i < h - 100; i++)
    for (j = 150; j < stride - 150; j += 4)
      {
            dest[i * stride + j + 0] = 0;
            dest[i * stride + j + 1] = 0;
            dest[i * stride + j + 2] = 0;
            dest[i * stride + j + 3] = 0;
      }
  return 0;
}

int set_layer_info(struct osd_head_info *osd_head,
		   int width, int height, int scale_w, int scale_h, int x, int y,
		   int fmt, int order)
{
  ingenicfb_lay_cfg *layer = osd_head->layer;

  layer->lay_en = 1;
  layer->lay_z_order = order;

  layer->lay_scale_en = 1;
  layer->scale_w = scale_w;
  layer->scale_h = scale_h;
  layer->source_w = width;
  layer->source_h = height;
  layer->disp_pos_x = x;
  layer->disp_pos_y = y;
  layer->g_alpha_en = 1;
  layer->g_alpha_val = 0xff;
  layer->color = 0;
  layer->format = fmt;
  layer->stride = width;
  return 0;
}

int main(int argc, char **argv)
{

  int i, ret;
  struct osd_head_info *osd_head, *osd_head2;

  osd_head = (struct osd_head_info *)malloc(sizeof(struct osd_head_info));
  int width = 720, height = 1280, x = 0, y = 0, fmt, space = 0, layer = 3;
  fmt = LAYER_CFG_FORMAT_ARGB8888;
  int type = 0, time = 0, color = 0;

  init_osd_client((char*)("client"));

  {
    int scale_w = 720 - 150;
    int scale_h = 1280 - 400;

    int i = 50000;
    while(i) {

      get_osd_package (osd_head);
      gen_bgra_datax(osd_head->shm_mem,width *4,height,0xFF,0,0,0xFF);

      set_layer_info (osd_head, width, height, scale_w, scale_h, x, y, fmt, 1);
      post_osd_package (osd_head);
    i--;
    }
  }
  uinit_client();
  return 0;
}


```