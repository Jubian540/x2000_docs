PulseAudio声音系统支持
[TOC]
<!-- toc -->

----

注：*本文中出现的'$'表示pc端执行，'#'表示开发板运行。*

# 1. PulseAudio简介

## 1.1. PulseAudio声音系统

**PulseAudio**（以前叫Polypaudio）是一个跨平台的、可通过网络工作的低延迟声音服务。

## 1.2. PulseAudio主要特点

* 可对每一个应用程序进行音量控制Per-application volume controls
* 可扩展的插件与支持可装载模块架构
* 兼容性许多流行的音频应用程序
* 支持多重音源和多重输出
* 低延时操作和支持延迟测量
* 一个对处理器资源效率零拷贝内存架构
* 能够发现本地网络上使用PulseAudio的其他计算机并通过其扬声器直接播放声音
* 能够改变一个应用程序的声音输出设备，就算这个应用程序在播放声音（程序不需要支持这特性，而事实上，程序甚至没有意识到改变）
* 带有脚本功能的命令行界面
* 一个功能完善且带有命令行重新配置功能的守护进程
* 内置采样转换和重采样功能
* 能够合并多块声卡成一个声卡
* 能够同步播放多个音频流
* 动态检测蓝牙音频设备
* 使全系统均衡的能力

# 2. 搭建PulseAudio环境

## 2.1.buildroot添加PulseAudio相关配置

* PulseAudio 库基础配置

```txt
Symbol: BR2_PACKAGE_PULSEAUDIO [=y]
Type  : bool
  Location:
    -> Target packages
      -> Audio and video applications
  Defined at package/pulseaudio/Config.in:10
  Depends on: BR2_PACKAGE_PULSEAUDIO_HAS_ATOMIC [=y] &&  
  ... ....
  BR2_USE_MMU [=y]
```

* PulseAudio 作为守护进程时选择（本例测试未选）

```txt
Symbol: BR2_PACKAGE_PULSEAUDIO_DAEMON [=y]
Type  : bool
Prompt: start as a system daemon
  Location:
    -> Target packages
      -> Audio and video applications
        -> pulseaudio (BR2_PACKAGE_PULSEAUDIO [=y])
  Defined at package/pulseaudio/Config.in:34
  Depends on: BR2_PACKAGE_PULSEAUDIO [=y]

```

## 2.3.自定义配置

### 2.3.1. pulseaudio

#### 2.3.1.1.# pulseaudio --help

```txt
pulseaudio [options]

COMMANDS:
  -h, --help                            Show this help
      --version                         Show version
      --dump-conf                       Dump default configuration
      --dump-modules                    Dump list of available modules
      --dump-resample-methods           Dump available resample methods
      --cleanup-shm                     Cleanup stale shared memory segments
      --start                           Start the daemon if it is not running
  -k  --kill                            Kill a running daemon
      --check                           Check for a running daemon (only returns exit code)

OPTIONS:
      --system[=BOOL]                   Run as system-wide instance
  -D, --daemonize[=BOOL]                Daemonize after startup
      --fail[=BOOL]                     Quit when startup fails
      --high-priority[=BOOL]            Try to set high nice level
                                        (only available as root, when SUID or
                                        with elevated RLIMIT_NICE)
      --realtime[=BOOL]                 Try to enable realtime scheduling
                                        (only available as root, when SUID or
                                        with elevated RLIMIT_RTPRIO)
      --disallow-module-loading[=BOOL]  Disallow user requested module
                                        loading/unloading after startup
      --disallow-exit[=BOOL]            Disallow user requested exit
      --exit-idle-time=SECS             Terminate the daemon when idle and this
                                        time passed
      --scache-idle-time=SECS           Unload autoloaded samples when idle and
                                        this time passed
      --log-level[=LEVEL]               Increase or set verbosity level
  -v  --verbose                         Increase the verbosity level
      --log-target={auto,syslog,stderr,file:PATH,newfile:PATH}
                                        Specify the log target
      --log-meta[=BOOL]                 Include code location in log messages
      --log-time[=BOOL]                 Include timestamps in log messages
      --log-backtrace=FRAMES            Include a backtrace in log messages
  -p, --dl-search-path=PATH             Set the search path for dynamic shared
                                        objects (plugins)
      --resample-method=METHOD          Use the specified resampling method
                                        (See --dump-resample-methods for
                                        possible values)
      --use-pid-file[=BOOL]             Create a PID file
      --no-cpu-limit[=BOOL]             Do not install CPU load limiter on
                                        platforms that support it.
      --disable-shm[=BOOL]              Disable shared memory support.
      --enable-memfd[=BOOL]             Enable memfd shared memory support.

STARTUP SCRIPT:
  -L, --load="MODULE ARGUMENTS"         Load the specified plugin module with
                                        the specified argument
  -F, --file=FILENAME                   Run the specified script
  -C                                    Open a command line on the running TTY
                                        after startup

  -n                                    Don't load default script file
```

### 2.3.1. pacmd

#### 2.3.1.1.# pacmd --help

```txt
pacmd exit
pacmd help
pacmd list-(modules|sinks|sources|clients|cards|samples)
pacmd list-(sink-inputs|source-outputs)
pacmd stat
pacmd info
pacmd load-module NAME [ARGS ...]
pacmd unload-module NAME|#N
pacmd describe-module NAME
pacmd set-(sink|source)-volume NAME|#N VOLUME
pacmd set-(sink-input|source-output)-volume #N VOLUME
pacmd set-(sink|source)-mute NAME|#N 1|0
pacmd set-(sink-input|source-output)-mute #N 1|0
pacmd update-(sink|source)-proplist NAME|#N KEY=VALUE
pacmd update-(sink-input|source-output)-proplist #N KEY=VALUE
pacmd set-default-(sink|source) NAME|#N
pacmd kill-(client|sink-input|source-output) #N
pacmd play-sample NAME SINK|#N
pacmd remove-sample NAME
pacmd load-sample NAME FILENAME
pacmd load-sample-lazy NAME FILENAME
pacmd load-sample-dir-lazy PATHNAME
pacmd play-file FILENAME SINK|#N
pacmd dump
pacmd move-(sink-input|source-output) #N SINK|SOURCE
pacmd suspend-(sink|source) NAME|#N 1|0
pacmd suspend 1|0
pacmd set-card-profile CARD PROFILE
pacmd set-(sink|source)-port NAME|#N PORT
pacmd set-port-latency-offset CARD-NAME|CARD-#N PORT OFFSET
pacmd set-log-target TARGET
pacmd set-log-level NUMERIC-LEVEL
pacmd set-log-meta 1|0
pacmd set-log-time 1|0
pacmd set-log-backtrace FRAMES

  -h, --help                            Show this help
      --version                         Show version
When no command is given pacmd starts in the interactive mode.

```

### 2.3.1. 配置文件

> #ls etc/pulse/

```txt
client.conf  daemon.conf  default.pa   system.pa
```

# 3. 快速测试方法

## 3.1.配置音频通路

**注**:更多音频通路配置请参照:  [3_Audio_音频子系统驱动和应用.md](../05_modules/3_Audio_音频子系统驱动和应用.md)

1. 配置speaker通路

    > #amixer cset name='LO0_MUX' LI8

      ```txt
        numid=16,iface=MIXER,name='LO0_MUX'
        ; type=ENUMERATED,access=rw------,values=1,items=17
        ; Item #0 'UNUSED'
        ; Item #1 'LI0'
        ; Item #2 'LI1'
        ; Item #3 'LI2'
        ; Item #4 'LI3'
        ; Item #5 'LI4'
        ; Item #6 'LI5'
        ; Item #7 'LI6'
        ; Item #8 'LI7'
        ; Item #9 'LI8'
        ; Item #10 'LI9'
        ; Item #11 'LI10'
        ; Item #12 'LI11'
        ; Item #13 'LI12'
        ; Item #14 'LI13'
        ; Item #15 'LI14'
        ; Item #16 'LI15'
        : values=9
        ```

2. 配置amic通路

    > #amixer cset name='LO6_MUX' LI2

    ```txt
    numid=22,iface=MIXER,name='LO6_MUX'
      ; type=ENUMERATED,access=rw------,values=1,items=17
      ; Item #0 'UNUSED'
      ; Item #1 'LI0'
      ; Item #2 'LI1'
      ; Item #3 'LI2'
      ; Item #4 'LI3'
      ; Item #5 'LI4'
      ; Item #6 'LI5'
      ; Item #7 'LI6'
      ; Item #8 'LI7'
      ; Item #9 'LI8'
      ; Item #10 'LI9'
      ; Item #11 'LI10'
      ; Item #12 'LI11'
      ; Item #13 'LI12'
      ; Item #14 'LI13'
      ; Item #15 'LI14'
      ; Item #16 'LI15'
      : values=3
      ```

## 3.2.配置DBUS_SESSION_BUS_ADDRESS环境变量（需要时配置）

1. 获取DBUS_SESSION_BUS_ADDRESS环境变量

    > #dbus-launch

    ```txt
    DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-hSDNBqPuxD,guid=5eecae77decc93c5c7d6cc985e6497deDBUS_SESSION_BUS_PID=2615
    ```

2. 设置为全局变量

    > #export DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-hSDNBqPuxD,guid=5eecae77decc93c5c7d6cc985e6497de

## 3.3.启动PulseAudio

1. 启动PulseAudio作为守护进程

    > #pulseaudio --start --load="module-alsa-sink device=hw:0,0" --load="module-alsa-source device=hw:0,6" --exit-idle-time=-1 --log-level=0 &

    **注：** *启动pulseAudio前，需要保证声卡设备没有被占用。（UAC可能会后台占用）*
    **`--exit-idle-time=-1`：** *表示禁止空闲时退出守护程序。*
    **`--log-level=0`：** *表示打印等级0~4，值为４时等级最高。*

## 3.4.查看当前加载的模块

1. 查看可用外设播放通道

    > #pacmd list-sinks

    ```txt
    2 sink(s) available.
       * index: 0
             name: <alsa_output.0.stereo-fallback>
             ... ...
             properties:
                     alsa.resolution_bits = "16"
                     device.api = "alsa"
                     device.class = "sound"
                     alsa.class = "generic"
                     alsa.subclass = "generic-mix"
                     alsa.name = ""
                     alsa.id = "DMA0 playback (*)"
                     alsa.subdevice = "0"
                     alsa.subdevice_name = "subdevice #0"
                     alsa.device = "0"
                     alsa.card = "0"
                     alsa.card_name = "halley5_v20"
                     alsa.long_card_name = "halley5_v20"
                     device.string = "hw:0"
                     device.buffering.buffer_size = "524288"
                     device.buffering.fragment_size = "131072"
                     device.access_mode = "mmap+timer"
                     device.profile.name = "stereo-fallback"
                     device.profile.description = "Stereo"
                     device.description = "halley5_v20 Stereo"
                     device.icon_name = "audio-card"
        index: 1
             ... ...
             ... ...
    ```

2. 查看可用外设录音通道
  
    > #pacmd list-sources

    ```txt
    4 source(s) available.
        index: 0
            ... ...
        index: 1
            ... ...
      * index: 2
            ... ...
        index: 3
              properties:
                    alsa.resolution_bits = "16"
                    device.api = "alsa"
                    device.class = "sound"
                    alsa.class = "generic"
                    alsa.subclass = "generic-mix"
                    alsa.name = ""
                    alsa.id = "DMA6 capture (*)"
                    alsa.subdevice = "0"
                    alsa.subdevice_name = "subdevice #0"
                    alsa.device = "6"
                    alsa.card = "0"
                    alsa.card_name = "halley5_v20"
                    alsa.long_card_name = "halley5_v20"
                    device.string = "hw:0,6"
                    device.buffering.buffer_size = "523776"
                    device.buffering.fragment_size = "130944"
                    device.access_mode = "mmap+timer"
                    device.description = "halley5_v20"
                    device.icon_name = "audio-input-microphone"
    ```

## 3.5.录放音通路测试

1. 录音通路测试，生成文件record.wav

    > #parecord --device=3 --channels=1 record.wav

    **注：** *device 3对应`#pacmd list-sources`的index 3，需要根据实际注册模块情况设置。*

2. 放音通路测试

    > #paplay --device=0 record.wav

    **注：** *device 0对应`#pacmd list-sinks`的index 0，需要根据实际注册模块情况设置。*
