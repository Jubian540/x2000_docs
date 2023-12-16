SDIO-wifi驱动和应用
=================
[TOC]
<!-- toc -->

----
# 1.wifi模块简介
X2000和M300使用的ap6256芯片， AP6256是基于BCM4345C5方案的集成wifi和bluetooth的功能模块，它包括用于WiFi的SDIO接口，和用于蓝牙的UART/PCM接口。

* GPIO功能描述

| Name          | I/O  | Description                                            |
| :------------ | :--- | :----------------------------------------------------- |
| WL_REG_ON     | I    | Power up/down internal regulators used by WiFi section |
| WL_HOST_WAKE  | O    | WLAN to wake-up HOST                                   |
| GND           | -    | Ground connections                                     |
| SDIO_DATA_1   | I/O  | SDIO data line 1                                       |
| SDIO_DATA_2   | I/O  | SDIO data line 2                                       |
| SDIO_DATA_3   | I/O  | SDIO data line 3                                       |
| SDIO_DATA_4   | I/O  | SDIO data line 4                                       |
| SDIO_DATA_CMD | I/O  | SDIO command line                                      |
| SDIO_DATA_CLK | I/O  | SDIO clock line                                        |

* GPIO接口

```c
SDIO GPIO-PD08～GPIO-PD13 FUNCTION0
WL_REG_ON GPIO-PD19     
WL_WAKE_HOST GPIO-PD20
```
* MSC控制器命名对应关系
  
| 控制器  |   Base   | bootrom | kernel | pm-spec | hardware-pcd |
| :-----: | :------: | :-----: | :----: | :-----: | :----------: |
| 控制器0 | 13450000 |  msc0   |  msc0  |  msc0   |     msc0     |
| 控制器1 | 13460000 |  msc1   |  msc1  |  sdio   |     sdio     |
| 控制器2 | 13490000 |  msc4   |  msc2  |  msc1   |     msc1     |



# 2.内核空间

## 2.1.驱动相关的源码

```c
drivers/mmc/host/
├── sdhci.c
├── sdhci-ingenic.c
├── ingenic_sdio.c
```

## 2.2.设备树配置

```c
&msc1 {
        status = "okay";                  /*配置“okay”编译SDIO驱动，“disable”反之*/
        pinctrl-names ="default","enable", "disable";
        pinctrl-0 = <&msc1_4bit>;
        pinctrl-1 = <&rtc32k_enable>;
        pinctrl-2 = <&rtc32k_disable>;

        sd-uhs-sdr104;                    /*配置支持sdr104模式*/
        max-frequency = <150000000>;
        bus-width = <4>;
        voltage-ranges = <1800 3300>;
        non-removable;

        ingenic,sdio_clk = <1>;
        keep-power-in-suspend;

        /* special property */
        ingenic,wp-gpios = <0>;
        ingneic,cd-gpios = <0>;
        ingenic,rst-gpios = <0>;
        ingenic,removal-manual; /*removal-dontcare, removal-nonremovable, removal-removable, removal-manual*/

        bcmdhd_wlan: bcmdhd_wlan {
                 compatible = "android,bcmdhd_wlan";
                 ingenic,sdio-irq = <&gpd 20 IRQ_TYPE_LEVEL_HIGH INGENIC_GPIO_NOBIAS>;
                 ingenic,sdio-reset = <&gpd 19 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
        };
};
```

## 2.3.驱动配置

```c
Device Drivers
--- Network device support 
[*]   Network core driver support
[*]   Wireless LAN  --->
        <*>   Broadcom FullMAC wireless cards support
        (/firmware/fw_bcm43456c5_ag.bin) Firmware path  /*固件路径*/
        (/firmware/nvram_ap6256.txt) NVRAM path         /*固件路径*/
                Enable Chip Interface (SDIO bus interface support)  --->
                Interrupt type (Out-of-Band Interrupt)  --->
```

# 3.用户空间

## 3.1.驱动加载成功
```c
[    1.124644] Dongle Host Driver, version 1.363.59.144.11 (r)
[    1.124644] Compiled from 
[    1.125120] Register interface [wlan0]  MAC: 00:90:4c:11:22:33
[    1.125120] 
[    1.125195] dhd_module_init: Exit err=0
```
## 3.2./etc/wpa_supplicant配网方法

### 3.2.1.配置/etc/wpa\_supplicant.conf

```c
# cat /etc/wpa_supplicant.conf 
ctrl_interface=/var/run/wpa_supplicant
update_config=1
country=GB"

network={
    ssid="Guest"        /*连接WiFi账户*/
    psk="ingenic*123"   /*连接Wifi密码*/
    bssid=
    priority=1
}
```

### 3.2.2.执行wifi\_up.sh

```c
# wifi_up.sh 
[   13.285811] dhd_open: Enter 843c4800
[   13.290329] dhd_conf_read_config: Ignore config file /firmware/config.txt
[   13.297356] Final fw_path=/firmware/fw_bcm43456c5_ag.bin
[   13.302873] Final nv_path=/firmware/nvram_ap6256.txt
[   13.308002] Final clm_path=/firmware/clm_bcmdhd.blob
[   13.313152] Final conf_path=/firmware/config.txt
[   13.317926] dhd_set_bus_params: set use_rxchain 0
[   13.322804] dhd_set_bus_params: set txglomsize 36
[   13.328047] dhd_os_open_image: /firmware/fw_bcm43456c5_ag.bin (579388 bytes) open success
[   13.409630] dhd_os_open_image: /firmware/nvram_ap6256.txt (2440 bytes) open success
[   13.418211] NVRAM version: AP6256_NVRAM_V1.3_10092019.txt
[   13.500573] random: nonblocking pool is initialized
[   13.505678] dhd_bus_init: enable 0x06, ready 0x06 (waited 0us)
[   13.511910] bcmsdh_oob_intr_register: Enter
[   13.516230] bcmsdh_oob_intr_register: HW_OOB enabled
[   13.521376] bcmsdh_oob_intr_register OOB irq=72 flags=0x4
[   13.527009] bcmsdh_oob_intr_register: enable_irq_wake
[   13.532703] Disable tdls_auto_op failed. -1
[   13.537028] dhd_conf_set_intiovar: set WLC_SET_BAND 142 0
[   13.542795] dhd_preinit_ioctls: Set tcpack_sup_mode 0
[   13.548281] dhd_apply_default_clm: Ignore clm file /firmware/clm_bcmdhd.blob
[   13.556873] Firmware up: op_mode=0x0005, MAC=c0:84:7d:31:c8:c8
[   13.562917] dhd_conf_set_country: set country CN, revision 38
[   13.590855] Country code: CN (CN/38)
[   13.594898] dhd_conf_set_intiovar: set roam_off 1
[   13.608835] Firmware version = wl0: Sep 27 2019 15:21:10 version 7.45.96.53 (5a84613@shgit) 
                                (r745790) FWID 01-54faa385 es7.c5.n4.a3
[   13.621079]   Driver: 1.363.59.144.11 (r)
[   13.621079]   Firmware: wl0: Sep 27 2019 15:21:10 version 7.45.96.53 (5a84613@shgit) 
                                (r745790) FWID 01-54faa385 es7.c5.n4.a3 
[   13.637339]   clm = 9.2.9
[   13.640263] dhd_txglom_enable: enable 1
[   13.644226] dhd_conf_set_txglom_params: swtxglom=0, txglom_ext=0, txglom_bucket_size=0
[   13.652431] dhd_conf_set_txglom_params: txglomsize=36, deferred_tx_len=36, bus_txglom=-1
[   13.660809] dhd_conf_set_txglom_params: tx_in_rx=1, txinrx_thres=-1, dhd_txminmax=1
[   13.668723] dhd_conf_set_txglom_params: tx_max_offset=0, txctl_tmo_fix=1
[   13.675669] sdioh_set_mode: set txglom_mode to multi-desc
[   13.681259] dhd_preinit_ioctls: not define PROP_TXSTATUS
[   13.688554] dhd_conf_set_intiovar: set ampdu_hostreorder 0
[   13.694884] dhd_pno_init: Support Android Location Service
[   13.701057] tsk Enter, tsk = 0x84461498
[   13.705033] tsk->terminated = 0
[   13.719718] wl_host_event: Invalid ifidx 0 for wl0
[   13.724797] wl_host_event: Invalid ifidx 0 for wl0
[   13.748733] dhd_open: Exit ret=0
# udhcpc: started, v1.26.2
Successfully initialized wpa_supplicant
udhcpc: sending discover
[   16.320669] Connecting with 8c:0c:90:98:47:cd ssid "Guest", len (5) channel=153
[   16.320669] 
[   16.367807] wl_bss_connect_done succeeded with 8c:0c:90:98:47:cd
[   16.401524] wl_bss_connect_done succeeded with 8c:0c:90:98:47:cd
udhcpc: sending discover
udhcpc: sending select for 10.4.30.146
udhcpc: lease of 10.4.30.146 obtained, lease time 86400
deleting routers
adding dns 219.141.136.10
adding dns 202.106.0.20
```

### 3.2.3.执行ping命令连网

```c
# ping www.baidu.com
PING www.baidu.com (220.181.38.150): 56 data bytes
64 bytes from 220.181.38.150: seq=0 ttl=53 time=14.347 ms
64 bytes from 220.181.38.150: seq=1 ttl=53 time=9.873 ms
64 bytes from 220.181.38.150: seq=2 ttl=53 time=12.889 ms
```

## 3.3 airkiss配网测试
AirKiss是微信硬件平台为Wi-Fi设备提供的微信配网、局域网发现和局域网通讯的技术。

### 3.3.1 微信配网
**以下操作必须在WIFI(外网)环境下配置**

使用AirkissDebugger软件（手机端下载）配置wifi

### 3.3.2 执行airkiss命令
```c
# airkiss

[  113.669340] dhd_open: Enter 842f1000
[  113.673116] dhd_open : no mutex held. set lock
[  113.677706] 
[  113.677706] Dongle Host Driver, version 1.579.77.41.26 (r-20200429-2)
[  113.686146] [dhd-wlan0] wl_android_wifi_on : in g_wifi_on=0
[  113.691950] wifi_platform_set_power = 1
[  113.695902] ======== PULL WL_REG_ON(-1) HIGH! ========
[  113.720040] ingenic,sdhci 13460000.msc: card insert manually
[  114.030026] sdio_reset_comm():
[  114.054045] mmc0: queuing unknown CIS tuple 0x80 (2 bytes)
[  114.060212] mmc0: queuing unknown CIS tuple 0x80 (3 bytes)
[  114.066344] mmc0: queuing unknown CIS tuple 0x80 (3 bytes)
[  114.072997] mmc0: queuing unknown CIS tuple 0x80 (7 bytes)
[  114.079686] mmc0: queuing unknown CIS tuple 0x81 (9 bytes)
[  114.109986] sdioh_start: set sd_f2_blocksize 256
[  114.114973] 
[  114.114973] 
[  114.114973] dhd_bus_devreset: == WLAN ON ==
[  114.122604] F1 signature read @0x18000000=0x15294345
[  114.130573] F1 signature OK, socitype:0x1 chip:0x4345 rev:0x9 pkg:0x2
[  114.137706] DHD: dongle ram size is set to 819200(orig 819200) at 0x198000
[  114.144983] dhd_bus_set_default_min_res_mask: Unhandled chip id
[  114.153611] [dhd] dhd_conf_read_config : Ignore config file /firmware/config.txt
[  114.161413] [dhd] dhd_conf_set_path_params : Final fw_path=/firmware/fw_bcm43456c5_ag.bin
[  114.169851] [dhd] dhd_conf_set_path_params : Final nv_path=/firmware/nvram_ap6256.txt
[  114.177960] [dhd] dhd_conf_set_path_params : Final clm_path=/firmware/clm_bcm43456c5_ag.blob
[  114.186695] [dhd] dhd_conf_set_path_params : Final conf_path=/firmware/config.txt
[  114.194942] dhd_os_open_image: /firmware/fw_bcm43456c5_ag.bin (579388 bytes) open success
[  114.266737] dhd_os_open_image: /firmware/nvram_ap6256.txt (2440 bytes) open success
[  114.277110] NVRAM version: AP6256_NVRAM_V1.3_10092019.txt
[  114.283245] dhdsdio_write_vars: Download, Upload and compare of NVRAM succeeded.
[  114.369136] dhd_bus_init: enable 0x06, ready 0x06 (waited 0us)
[  114.375340] bcmsdh_oob_intr_register: HW_OOB irq=74 flags=0x4
[  114.382140] Disable tdls_auto_op failed. -1
[  114.386461] dhd_tcpack_suppress_set: TCP ACK Suppress mode 0 -> mode 1
[  114.393469] dhd_apply_default_clm: Ignore clm file /firmware/clm_bcm43456c5_ag.blob
[  114.402597] Firmware up: op_mode=0x0005, MAC=c0:84:7d:6a:92:c1
[  114.411197] dhd_preinit_ioctls Set scancache failed -23
[  114.423672]   Driver: 1.579.77.41.26 (r-20200429-2)
[  114.423672]   Firmware: wl0: Sep 27 2019 15:21:10 version 7.45.96.53 (5a84613@shgit) (r745790) FWID 01-54faa385 es7.c5.n4.a3
[  114.423672]   CLM: 9.2.9 (2016-02-03 04:34:31) 
[  114.445131] dhd_txglom_enable: enable 1
[  114.449091] [dhd] dhd_conf_set_txglom_params : txglom_mode=multi-desc
[  114.455763] [dhd] dhd_conf_set_txglom_params : txglomsize=36, deferred_tx_len=0
[  114.463324] [dhd] dhd_conf_set_txglom_params : txinrx_thres=128, dhd_txminmax=-1
[  114.470971] [dhd] dhd_conf_set_txglom_params : tx_max_offset=0, txctl_tmo_fix=300
[  114.478702] [dhd] dhd_conf_get_disable_proptx : fw_proptx=1, disable_proptx=-1
[  114.486988] dhd_wlfc_hostreorder_init(): successful bdcv2 tlv signaling, 64
[  114.495449] dhd_pno_init: Support Android Location Service
[  114.501131] dhd_preinit_ioctls: SensorHub diabled 0
[  114.506677] dhd_preinit_ioctls failed to set ShubHub disable
[  114.513478] dhd_wl_ioctl_get_intiovar: get int iovar wnm_bsstrans_resp failed, ERR -23
[  114.521658] failed to get wnm_bsstrans_resp
[  114.526479] failed to set WNM capabilities
[  114.550403] [dhd] CFG80211-ERROR) wl_cfg80211_event : Event handler is not created
[  114.550406] [dhd] dhd_conf_map_country_list : CN/38
[  114.550413] [dhd] dhd_conf_set_country : set country CN, revision 38
[  114.570318] [dhd] dhd_conf_set_country : Country code: CN (CN/38)
[  114.580570] [dhd-wlan0] wl_android_wifi_on : Success
[  114.633169] dhd_open : the lock is released.
[  114.637581] dhd_open: Exit ret=0
Easy setup target library v4.0.0
state: 0 --> 1
state: 1 --> 3
state: 3 --> 5
ssid: JZ_SW            /**软件端配置的网络账号**/
password: jz_sw%#!135  /**软件端配置的网络密码**/
/etc/wpa_supplicant.conf create successfully!
random: 0xc1
time elapsed: 0s
# udhcpc: started, v1.31.1
udhcpc: sending discover
Successfully initialized wpa_supplicant
[  118.985525] [dhd-wlan0] wl_run_escan : LEGACY_SCAN sync ID: 0, bssidx: 0
wlan0: Trying to associate with c4:01:7c:78:c3:dd (SSID='JZ_SW' freq=5765 MHz)
[  121.338272] [dhd-wlan0] wl_cfg80211_connect : Connecting with c4:01:7c:78:c3:dd ssid "JZ_SW", len (5), sec=wpa2psk/mfpn/tkipaes, channel=153
[  121.338272] 
[  121.402046] [dhd-wlan0] wl_ext_iapsta_event : [S] Link UP with c4:01:7c:78:c3:dd
[  121.409697] [dhd-wlan0] wl_notify_connect_status : wl_bss_connect_done succeeded with c4:01:7c:78:c3:dd 
wlan0: Associated with c4:01:7c:78:c3:dd
wlan0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
wlan0: WPA: Key negotiation completed with c4:01:7c:78:c3:dd [PTK=CCMP GTK=TKIP]
wlan0: CTRL-EV[  121.451231] [dhd-wlan0] wl_notify_connect_status : wl_bss_connect_done succeeded with c4:01:7c:78:c3:dd vndr_oui: 00-90-4C 00-13-92 
ENT-CONNECTED - Connection to c4:01:7c:78:c3:dd completed [id=0 id_str=]
udhcpc: sending discover
udhcpc: sending select for 10.10.30.203
udhcpc: lease of 10.10.30.203 obtained, lease time 86400
deleting routers
adding dns 192.168.1.2
```
### 3.3.3.执行ping命令连网

```c
# ping www.baidu.com
PING www.baidu.com (220.181.38.150): 56 data bytes
64 bytes from 220.181.38.150: seq=0 ttl=53 time=14.347 ms
64 bytes from 220.181.38.150: seq=1 ttl=53 time=9.873 ms
```

## 3.4 AP模式使用方法
AP模式下的配置文件信息(路径：/etc/hostapd.conf)：
```c
interface=wlan0
driver=nl80211
ssid=ingenic
channel=1
hw_mode=g
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
#wpa=2
#wpa_passphrase=12345678
#wpa_key_mgmt=WPA-PSK
#wpa_pairwise=TKIP
#rsn_pairwise=CCMP
```
### 3.4.1 执行以下命令进入AP模式
```c
# wifi_ap_mode_start.sh
```

* 出现如下打印说明AP模式使能成功：
```c
wlan0: interface state UNINITIALIZED->ENABLED
wlan0: AP-ENABLED 
```
### 3.4.2 执行ifconfig命令查看连接信息
```c
# ifconfig 
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

wlan0     Link encap:Ethernet  HWaddr C0:84:7D:6A:F1:CD  
          inet addr:192.168.1.1  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:46 errors:0 dropped:21 overruns:0 frame:0
          TX packets:3 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:6509 (6.3 KiB)  TX bytes:744 (744.0 B)
```
### 3.4.3 使用手机（或电脑）连接WIFI

连接SSID为“ingenic”对应配置文件ssid=ingenic，密码可以使用wpa_passphrase=12345678指定(本测试未设置密码)，连接成功会打印如下信息：
```c
[  515.831824] [dhd-wlan0] wl_ext_iapsta_event : [A] connected device 00:26:c6:58:50:54
[  515.839828] [dhd-wlan0] wl_notify_connect_status_ap : connected device 00:26:c6:58:50:54
[  515.851919] [dhd] CFG80211-ERROR) wl_cfg80211_change_station : WLC_SCB_AUTHORIZE sta_flags_mask not set 
udhcpd: sending OFFER to 192.168.1.2
udhcpd: sending ACK to 192.168.1.2
```

### 3.3.4 连接成功
连接成功后进入adb shell，ping手机端（或pc端）IP,打印如下信息则连接成功：
```c
# ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: seq=0 ttl=64 time=8.148 ms
64 bytes from 192.168.1.2: seq=1 ttl=64 time=15.411 ms
64 bytes from 192.168.1.2: seq=2 ttl=64 time=12.505 ms
64 bytes from 192.168.1.2: seq=3 ttl=64 time=7.654 ms
64 bytes from 192.168.1.2: seq=4 ttl=64 time=7.505 ms
```
