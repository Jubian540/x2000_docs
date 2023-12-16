
## X1000 hw-jpeg


## hw-jpeg source code

X1000_SDK/development/source/jpg_api/jpeg_encode2.c
``` c
// save jpg to file
int yuv422_to_jpeg(void *handle,unsigned char *input_image, FILE *fp, int width, int height, int quality)

// save jpg to memory
int jz_put_jpeg_yuv420p_memory(void *handle,unsigned char *dest_image, int image_size,
				      unsigned char *input_image, int width, int height, int quality)
```

## linux kernel driver

kernel/arch/mips/configs/halley2_linux_nor_defconfig
>CONFIG_JZ_IMEM=y
CONFIG_IMEM_MAX_ORDER=10
CONFIG_VIDEO_INGENIC_VPU_FOR_V4L2=y
CONFIG_VIDEO_INGENIC_X1000_JPEG=y

device node after bootup:

>/dev/video0
/proc/jz/mem/imem


## hw-jpeg sample

X1000_SDKpackages/example/App/cimutils/jpeg_hw_encoder/test-jpeg.c
```c
                      jz_jpeg = jz_jpeg_encode_init(width, height);
                      yuv422_to_jpeg(jz_jpeg, frame, dst_jpeg, width, height, quality);
                      jz_jpeg_encode_deinit(jz_jpeg);
```


## camera jpeg demo

X1000_SDK/packages/example/App/cimutils/process_picture/savejpeg.c
```c
int yuv422_packed_to_jpeg(void *handle, const __u8 *frm, FILE *fp, struct camera_info *camera_inf, int quality)
retval = yuv422_to_jpeg (handle, frm, fp, width, height, quality);    //defined in libj
```


## usb camera grab

X1000_SDK/packages/example/App/grab/
```c
// allocate buffer, grab/v4l2uvc.c
		vd->framebuffer = (unsigned char *)malloc((size_t) vd->framesizeIn);

// save yuv422 buffer to jpg, grab/camera.c
	    get_pictureYUYV(videoIn->framebuffer,videoIn->width,videoIn->height, filename);
```

### grab for hw-jpg

```c
// allocate buffer, grab/v4l2uvc.c
		vd->framebuffer = (unsigned char *)JZMalloc(128, (size_t) vd->framesizeIn);

// save yuv422 buffer to jpg, grab/camera.c
	    get_pictureYUYV(videoIn->framebuffer,videoIn->width,videoIn->height, filename);
change to  yuv422_to_jpeg();
or 
jz_put_jpeg_yuv420p_memory()
    
```

