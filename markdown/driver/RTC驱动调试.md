# RTC驱动调试

tags: driver

---

## 代码

**ds1307的I2C地址为: 0x68**

首先在dts文件中添加相应的信息. 设备使用引脚3和5. 如下:

![40Pin GPIO definition][1]

设备是接在I$^2$C2上的, 所以下面的信息要加在节点2下: 
- dts  

```
		ds1307@68{
			compatible = "ds1307";
			reg = <0x68>;
		};
```

Linux内核中默认有ds1307的代码, 在内核配置选项中打开对ds1307的支持: 

![kernel-config][2]

由于 I$^2$C 和 SPI 外设驱动和设备树中设备节点的兼容性属性中有一种弱式匹配方法, 即'别名'匹配, 所以驱动源码无需修改:  
```
static const struct i2c_device_id ds1307_id[] = {
	{ "ds1307", ds_1307 },
	{ "ds1337", ds_1337 },
	{ "ds1338", ds_1338 },
	{ "ds1339", ds_1339 },
	{ "ds1388", ds_1388 },
	{ "ds1340", ds_1340 },
	{ "ds3231", ds_3231 },
	{ "m41t00", m41t00 },
	{ "mcp7941x", mcp7941x },
	{ "pt7c4338", ds_1307 },
	{ "rx8025", rx_8025 },
	{ }
};
```

## 加载驱动错误

错误信息: 

> rtc-ds1307: probe of 2-0068 failed with error -5

解: 

这个提示是由于设备没有匹配上引起的, 检查设备是否正确连接, 引脚连接是否出错. 

## 使用

设备正确连接后, 加载驱动没有错误, 并发现设备`/dev/rtc1`. 
使用命令`hwclock -f /dev/rtc1`即可查看ds1307内的时间.使用`-f`参数用于指定rtc设备, 系统默认使用的是rtc0:
```
root@Lemuntu:~# ls -al /dev/rtc*
lrwxrwxrwx 1 root root      9 Jul  4 16:42 /dev/rtc -> /dev/rtc0
crw------- 1 root root 254, 0 Jan  1  2011 /dev/rtc0
crw------- 1 root root 254, 1 Jan  1  2011 /dev/rtc1
```

可以通过更改**dev/rtc**来使/dev/rtc1做为默认设备:
```
ln -sf /dev/rtc1 /dev/rtc
```   

 使用`hwclock -h`查看命令的详细用法, 以下示例用做参考:
 
 - 查看rtc设备时间
 ```
 hwclock [-f /dev/rtc1]
 ```

 - 修改rtc设备时间
 
```
hwclock --set --date="07/04/16 16:52:25" [-f /dev/rtc1]
# 修改时间为 2016年07月04日 16:52:25
```

- 同步rtc设备时间到系统

```
hwclock -s [-f /dev/rtc1]
```
之后输入`date`查看系统时间是否修改. 

-----------------------------------------------------------


  [1]: http://static.zybuluo.com/tab/ret2i3rwzrhgon46qxag7b14/image_1amqebp50lrb1ki615h0uso6obn.png
  [2]: http://static.zybuluo.com/tab/e5uyitq9v90i8zwpjuz93lvb/image_1amt5dr3t17ih17t23paavnsid9.png