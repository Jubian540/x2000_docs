Hash 驱动和应用
=============
[TOC]
<!-- toc -->

----

# 1. 模块简介

DTRNG（数字真随机数发生器）用于为系统提供真随机数加密种子。
SC_HASH（信息摘要计算器）用于计算数据的摘要信息。

# 2. 内核空间

**驱动位置**

```
drivers/crypto/ingenic-hash.c
```

## 2.1. 设备树配置

**设备树代码位置arch/mips/boot/dts/ingenic/x2000-v12.dtsi(m300.dtsi)**

```
                hash: hash@0x13470000 {                                                                                     
                        compatible = "ingenic,hash";
                        reg = <0x13470000 0x10000>;
                        interrupt-parent = <&core_intc>;
                        interrupts = <IRQ_HASH>;
                        status = "ok";
                };

```

## 2.2. 驱动配置

```
Symbol: CRYPTO_USER_API_HASH [=y]                                        Type  : tristate                                                      Prompt: User-space interface for hash algorithms                        Location:                                                            
(9) -> Cryptographic API (CRYPTO [=y])                                  Defined at crypto/Kconfig:1607                                        Depends on: CRYPTO [=y] && NET [=y]                                    Selects: CRYPTO_HASH [=y] && CRYPTO_USER_API [=y]  

Symbol: CRYPTO_DEV_INGENIC_SHA [=y]                                    Type  : tristate                                                      Prompt: Support for Ingenic SHA hw accelerator                          Location:                                                                -> Cryptographic API (CRYPTO [=y])                                
(1)   -> Hardware crypto devices (CRYPTO_HW [=y])                        Defined at drivers/crypto/Kconfig:493                                  Depends on: CRYPTO [=y] && CRYPTO_HW [=y] && (MARCH_XBURST1 || MACH_XBURST2 [=y])                                                          Selects: CRYPTO_ALGAPI [=y] 
```

# 3. 用户空间
## 3.1. 测试方法
* 源码路径

```c
/home/zdhuang/linphone/Manhatta-halley611/packages/example/Sample/hash
```
* 测试方法
```c
hash_test in_file out_file
```

