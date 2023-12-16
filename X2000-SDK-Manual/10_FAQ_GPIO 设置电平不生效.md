## 问题描述
拉低拉高芯片gpio, 芯片管脚没有发生变化。

## 解决办法

X2000/M300 系列芯片部分IO 是3.3v/1.8v 电压可变IO，主要涉及到的IO 有
* GPIOA组[0:17] 全部IO
* GPIOE组[0:5]

对于使用这两组IO进行电路设计时，硬件和软件上都需要特殊处理。

### 硬件处理
根据实际IO 电压情况
- VDDIO_CIM 接 3.3V 或者 1.8V
- VDDIO_SD 接 3.3v 或者 1.8v
![](/assets/vddio_cim_sd.png)

### 软件修改
找到kernel arch/mips/boot/dts/ingenic/ 对应dts文件，修改以下配置
- GPIO_VOLTAGE_1V8
- GPIO_VOLTAGE_3V3
```
&pinctrl {                                                                                                                    
        ingenic,gpa_voltage = <GPIO_VOLTAGE_1V8>;
        ingenic,gpe_msc_voltage = <GPIO_VOLTAGE_1V8>;
};
```
**注意: 通常gpe_msc_voltage 是专门用于sd卡 SDR104模式高速切换时使用，msc驱动会自动控制电压，只有当电路设计用于普通IO时，需要在dts中配置**
