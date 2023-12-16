Security 安全子系统驱动和应用
=========================
[TOC]
<!-- toc -->

----

# 1. rsa模块
## 1.1 模块简介

RSA公开密钥密码体制是一种使用不同的加密密钥与解密密钥

## 1.2 内核空间
**驱动位置**

```
$ drivers/misc/ingenic_rsa.c
```

### 1.2.1. 设备树配置

**设备树代码位置arch/mips/boot/dts/ingenic/x2000-v12.dtsi(m300.dtsi)**

```
555                 rsa: rsa@0x13480000 {
556                         compatible = "ingenic,rsa";
557                         reg = <0x13480000 0x10000>;
558                         interrupt-parent = <&core_intc>;
559                         interrupts = <IRQ_RSA>;
560                         status = "ok";
561                 };  
```
### 1.2.2.驱动配置

```
this driver is used to Encrypt/Decrypt by RSA.
Symbol: INGENIC_RSA [=y]
Type  : boolean     
Prompt: JZ RSA Driver  
    Location:
        -> Device Drivers 
    Defined at drivers/misc/Kconfig:536
    Depends on: MACH_XBURST2 [=y]
```
## 1.3. 用户空间
### 1.3.1 测试方法
* 源码路径

```c
$ packages/example/Sample/security_test/rsa
```
* 测试方法
```c
rsa_test in_file out_file rsa_key
```
    file:为需要运算的数据
    key_file:为key数据文件

相关测试数据：
```
$ packages/example/Sample/security_test/test_data/rsa_test
```
rsa_key:为rsa密钥
rsa_result.txt：该数据为结果数据
rsa_test_data.txt：该数据为待测试数据

```
# rsa_test rsa_test_data.txt rsa_key
```
执行路径下会生成 rsa_test_result.txt文件，该文件为运算结果数据输出。

# 2. hash模块
## 2.1. 模块简介

SC_HASH（信息摘要计算器）用于计算数据的摘要信息。

## 2.2. 内核空间
**驱动位置**

```
$ drivers/crypto/ingenic-hash.c
```

### 2.2.1.设备树配置

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
### 2.2.2.驱动配置

```
Symbol: CRYPTO_USER_API_HASH [=y]                                        
Type  : tristate                                                      
Prompt: User-space interface for hash algorithms                        Location:                                                            
(9) -> Cryptographic API (CRYPTO [=y])                                  
Defined at crypto/Kconfig:1607                                        
Depends on: CRYPTO [=y] && NET [=y]                                    
Selects: CRYPTO_HASH [=y] && CRYPTO_USER_API [=y]  

Symbol: CRYPTO_DEV_INGENIC_SHA [=y]                                    
Type  : tristate                                                      
Prompt: Support for Ingenic SHA hw accelerator                          Location:                                                                
-> Cryptographic API (CRYPTO [=y])                                
(1)   -> Hardware crypto devices (CRYPTO_HW [=y])                        
Defined at drivers/crypto/Kconfig:493                                  
Depends on: CRYPTO [=y] && CRYPTO_HW [=y] && (MARCH_XBURST1 || MACH_XBURST2 [=y])                                                          
Selects: CRYPTO_ALGAPI [=y] 
```

## 2.3 用户空间
### 2.3.1. 测试方法
* 源码路径

```c
$ packages/example/Sample/security_test/hash
```
* 测试方法
```c
hash_test in_file out_file
```
相关测试数据：
```
$ packages/example/Sample/security_test/test_data/hash_test
```
hash_result.txt：该数据为结果数据
hash_test_data.txt：该数据为待测试数据

```
# hash_test hash_test_data.txt hash_result.txt
```
执行路径下会生成 hash_test_result.txt文件，该文件为运算结果数据输出。

# 3. aes模块
## 3.1. 模块简介

aes对称加解密硬件模块

## 3.2. 内核空间
**驱动位置**

```
$ drivers/crypto/ingenic-aes.c
```

### 3.2.1.设备树配置

**设备树代码位置arch/mips/boot/dts/ingenic/x2000-v12.dtsi（m300.dtsi）**

```
539                 aes: aes@0x13430000 {
540                         compatible = "ingenic,aes";
541                         reg = <0x13430000 0x10000>;
542                         interrupt-parent = <&core_intc>;
543                         interrupts = <IRQ_AES>;
544                         status = "ok";
545                 };   
```
### 3.2.2.驱动配置

```
Symbol: CRYPTO_USER_API_SKCIPHER [=y]
Type  : tristate
Prompt: User-space interface for symmetric key cipher algorithms
  Location:
      -> Cryptographic API (CRYPTO [=y])
   Defined at crypto/Kconfig:1616
   Depends on: CRYPTO [=y] && NET [=y] 
   Selects: CRYPTO_BLKCIPHER [=y] && CRYPTO_USER_API [=y]

Symbol: CRYPTO_DEV_INGENIC_AES [=y] 
Type  : tristate
Prompt: Support for INGENIC AES hw engine
  Location:
    -> Cryptographic API (CRYPTO [=y])
     -> Hardware crypto devices (CRYPTO_HW [=y])
  Defined at drivers/crypto/Kconfig:311
  Depends on: CRYPTO [=y] && CRYPTO_HW [=y] && (MARCH_XBURST1 || MACH_XBURST2 [=y])
  Selects: CRYPTO_AES [=y] && CRYPTO_BLKCIPHER2 [=y]
```

## 3.3 用户空间
### 3.3.1. 测试方法
* 源码路径

```c
$ packages/example/Sample/security_test/aes
```
* 测试方法
```c
aes_test in_file out_file key_file op(en:1, de:0)
```
相关测试数据：
```
$ packages/example/Sample/security_test/test_data/aes_test
```
key.txt:key数据
aes_result.txt：该数据为结果数据
aes_test_data.txt：该数据为待测试数据

```
# aes_test aes_test_data.txt aes_test_result.txt key.txt 1
```
执行路径下会生成 aes_test_result.txt文件，该文件为运算结果数据输出。

# 4. dtrng模块
## 4.1 模块简介

DTRNG（数字真随机数发生器）用于为系统提供真随机数加密种子。

## 4.2 内核空间
**驱动位置**

```
drivers/char/hw_random/ingenic-rng.c
```

### 4.2.1.设备树配置

**设备树代码位置arch/mips/boot/dts/ingenic/x2000-v12.dtsi(m300.dtsi)**

```
dtrng: dtrng@0x10072000 {
                        compatible = "ingenic,dtrng";
                        reg = <0x10072000 0x100>;
                        interrupt-parent = <&core_intc>;
                        interrupts = <IRQ_DTRNG>;
                        status = "disable";
                };  
```
### 4.2.2.驱动配置
* 以x2000_v12配置为例：
```
Symbol: HW_RANDOM_INGENIC [=y]                                              
Type  : tristate                                                            
Prompt: Ingenic HW Random Number Generator support                          
  Location:                                                                 
    -> Device Drivers                                                       
      -> Character devices                                                  
(1)     -> Hardware Random Number Generator Core support (HW_RANDOM [=y])   
  Defined at drivers/char/hw_random/Kconfig:384                             
  Depends on: HW_RANDOM [=y] && MIPS [=y] && (SOC_X2000_V12 [=y] || SOC_M300 [=n])
```

## 4.3. 用户空间
### 4.3.1. 测试方法
* 源码路径

```c
$ packages/example/Sample/security_test/dtrng
```
* 测试方法
```c
dtrng_test out_file
```
执行路径下会生成 dtrng_test_result.txt文件，该文件为运算结果数据输出。
