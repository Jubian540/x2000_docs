NCNN深度学习框架测试说明文档
[TOC]
<!-- toc -->

----

注：*ncnn测试例程默认使用opencv2环境*

注：*本文中出现的'$'表示pc端执行，'#'表示开发板运行。*

# 1.ncnn源码

## 1.1.下载ncnn源码

```c
    https://github.com/Tencent/ncnn
```

## 1.2.ncnn源码集成到Manhattan环境下

* ncnn源码放到Manhattan/packages/example/App/目录下
* 创建Build.mk文件，如下：

```c
    $ cat packages/example/App/ncnn/Build.mk

    include $(CLEAR_VARS)
    CMAKE_PATH=$(LOCAL_PATH)
    LOCAL_MODULE:= ncnn-test
    LOCAL_MODULE_TAGS :=optional
    include $(BUILD_CMAKE_DEVICE)
```

# 2.配置opencv2环境

## 2.1.buildroot添加opencv2和jpeg相关配置

```c
    Symbol: BR2_PACKAGE_OPENCV [=y]
    Type  : bool
    Prompt: opencv-2.4
    　　Location:
    　　　　-> Target packages
    　　　　　　-> Libraries
    　　　　　　　　-> Graphics
    　　Defined at package/opencv/Config.in:1
    　　Depends on: BR2_TOOLCHAIN_HAS_THREADS_NPTL [=y] && BR2_INSTALL_LIBSTDCPP [=y] && 　　　BR2_USE_WCHAR [=y]
    　　Selects: BR2_PACKAGE_ZLIB [=y]


    Symbol: BR2_PACKAGE_OPENCV_WITH_JASPER[=y]
    Type  :　bool
    Prompt: jpeg2000 support
    　　Location:
    　　　　-> Target packages
    　　　　　　-> Libraries
    　　　　　　　　-> Graphics
    　　　　　　　　　　-> opencv-2.4 (BR2_PACKAGE_OPENCV [=y])
    　　Defined at package/opencv/Config.in:225
    　　Depends on: BR2_PACKAGE_OPENCV [=y]
    　　Selects: BR2_PACKAGE_JASPER [=n]

    Symbol: BR2_PACKAGE_OPENCV_WITH_JPEG [=y]
    Type  : bool
    Prompt: jpeg support
    　　Location:
    　　　　-> Target packages
    　　　　　　-> Libraries
    　　　　　　　　-> Graphics
    　　　　　　　　　　-> opencv-2.4 (BR2_PACKAGE_OPENCV [=y])
    　　Defined at package/opencv/Config.in:235
    　　Depends on: BR2_PACKAGE_OPENCV [=y]
    　　Selects: BR2_PACKAGE_JPEG [=y]
```

## 2.2.编译Manhattan工程

* 进入工程目录，执行以下命令

```c
    $ source build/envsetup.sh
    $ lunch
```
* 选择相应的开发板

```c
    $ make -j4
```

* 生成镜像如下：

```c
    $ ls out/product/"板级"/image/
    kernel  system.*  uboot
```

# 3. 编译ncnn

* 到Manhattan目录下，编译ncnn-test(Build.mk中指定)

```c
    $ make ncnn-test -j4
```

* 生成ncnn例程的一些可执行文件

```c
    $ ls out/product/"板级"/obj/packages/example/App/ncnn/examples/
    squeezenet ...
```

# 4. 快速测试

* 启动文件系统，通过adb导入一些文件

    **squeezenet**:

    ```c
        squeezenet:
            ncnn例程测试程序。
        文件位置：
            out/product/"板级"/obj/packages/example/App/ncnn/examples/squeezenet
    ```

     **squeezenet_v1.1.bin** 、**squeezenet_v1.1.param**:

    ```c
        squeezenet_v1.1.bin:
            squeezenet神经网络模型权重参数。
        squeezenet_v1.1.param:
            squeezenet神经网络模型模型参数。
        文件位置：
            packages/example/App/ncnn/examples/squeezenet_v1.1.bin
            packages/example/App/ncnn/examples/squeezenet_v1.1.param
    ```

    **libgomp.so.1**:

    ```c
        libgomp.so.1:
            GNU编译器集合OpenMP运行时的共享库，在编译工具下。
        文件位置:
            /prebuilts/toolchains/mips-gcc720-glibc226/mips-linux-gnu/libc/usr/lib/libgomp.so.1
    ```

    **plane.jpg**:

    ```c
        plane.jpg用于进行图像分类测试的测试数据。
    ```

* 将共享库放到lib下
  
```c
    # mv libgomp.so.1 /lib/
```

* 执行测试程序

```c
    # ./squeezenet plane.jpg
    404 = 0.800639
    895 = 0.145950
    795 = 0.016727
```

注：*确保　`squeezenet_v1.1.param`、`squeezenet_v1.1.bin`、`squeezenet`　在同一目录下*
