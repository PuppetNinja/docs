# ilitek230x-tp调试

tags: driver 

---

## dts 

如下为官方给出源码中的dts文件, 根据电路图修改相应引脚: 

- 中断引脚: touch-gpio为 24 (GPIOA24)
- 复位引脚: reset-gpio为 35 (GPIOB3)

```
ilitek@41 {
	status = "okay";
	compatible = "tchip,ilitek";
	reg = <0x41>;
	//touch-gpio = <&gpio7 GPIO_A6 IRQ_TYPE_EDGE_RISING>;
	//ilitek,irq-gpio = <&gpio8 GPIO_A7 IRQ_TYPE_EDGE_FALLING>;
	//ilitek,reset-gpio = <&gpio8 GPIO_A6 GPIO_ACTIVE_LOW>;
	touch-gpio =  <&gpio 24 1>;
	reset-gpio =  <&gpio 35 1>;
	//enable-gpio = <&gpio8 GPIO_B0 GPIO_ACTIVE_HIGH>;
	//max-x = <1600>;
	//max-y = <900>;
};
```

**NOTE:** 这段代码应插入到i2c2的节点中(硬件决定), 否则会造成i2c通信失败的问题. 
```
i2c@b0178000 {
	i2c_num = <2>;
	...
};
```


## 头文件ilitek_ts.h

在这个文件中还是要修改源码中要引用的中断引脚和复位引脚:
```
#define RESET_GPIO  35
#define IRQ_GPIO  24
```
## 问题 
i2c通信失败:
1. 硬件问题, TP线序不对. 
2. dts代码误插入到I2C<1>(实际I2C<2>)中, 导致驱动不能读取到TP信息.

### 解决过程
- 硬件问题

由于通过`cat /proc/interrupts`命令没有发现产生中断. i2c通信也不正常, 
用示波器看各引脚的电压与波形, 发现这些电压表现比较奇怪, 甚至有些电平不高不低的.
把TP取下之后再测量发现有的引脚表现与电路图有出入, 判断是硬件线序搞错了, TP也因此烧坏.

- dts问题

重新调整了硬件线序问题, 继续进行测试, 串口还是i2c通信不正常, 没有中断产生. 
这次测试中发现reset引脚为0.7V, 这明显不对. 因为正常工作时, reset引脚应保持高电平, 只有在驱动加载时才有一个拉低再拉高的过程. 

通过示波器抓到的波形发现, 驱动在加载过程中的确有一个拉低再拉高的过程, 只不之后又会被拉低到0.7V. 

手动控制引脚: 
```
cd /sys/class/gpio 
echo 35 > export 
cd gpio35 
echo out > direction  #设置为输出
# 控制gpio口输出电平
while true; do echo 1 > value; sleep 1; echo 0 > value; sleep 1; done
```

是可以正常控制io口的.
`dmesg | grep ilitek` 看结果, 发现有一错误:

> ilitek read tp info fail

把TP拔出后再次尝试加载驱动, 出现相同的错误, 看来驱动是没能够正常的读取到TP的信息.

以下命令:
```
ls /sys/bus/i2c/devices/
```
发现tp挂在了i2c1下, 应该挂在i2c2下的. 原来是在dts配置中出错了. 
修改dts, 把ilitek配置挂在i2c2节点下.

重新加载驱动, 一切正常. :)

## 其它

很幸运在驱动加载完后可以在linux下正常运行.


