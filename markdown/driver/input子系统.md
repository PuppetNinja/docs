# input子系统

tags: driver

---

## Linux输入子系统(Input Subsystem)

在Linux中, 输入子系统是由输入子系统设备驱动层, 输入子系统核心层(Input Core)和输入子系统事件处理层(Event Handler)组成. 一个输入事件, 如鼠标移动, 键盘按键按下等都是通过 Driver -> InputCore -> Eventhandler -> userspace 的顺序到达用户空间传给应用程序. 所以这使得我们输入设备的驱动部分不在用关心对设备文件的操作,  而是要关心对各硬件寄存器的操作和提交的输入事件. 

- 驱动层功能：主要实现对硬件设备的读写访问，中断设置， 将底层硬件设备对用户输入的响应转换为标准的输入事件以后再向上发送给输入子系统核心层（Input Core）. 

- Input系统核心层：Input Core即Input Layer, 由driver/input/input.c及相关头文件实现, 它对下提供了设备驱动层的接口, 对上提供了事件处理层（Event Handler）的编程接口. 

- 事件处理层将硬件设备上报的事件分发到用户空间和内核. 


![Linux输入子系统结构][pic1]

另一幅图来说明Linux输入子系统的结构，可能更加形象容易理解。如下：

![Linux输入子系统结构2][pic2]


## 设备驱动层

作为输入设备的驱动开发者，需要做以下几步：

- 在驱动加载模块中，设置你的input设备支持的事件类型，类型参见表1设置
- 注册中断处理函数，例如键盘设备需要编写按键的抬起、放下，触摸屏设备需要编写按下、抬起、绝对移动，鼠标设备需要编写单击、抬起、相对移动，并且需要在必要的时候提交硬件数据（键值/坐标/状态等等）
- 将输入设备注册到输入子系统中 

Linux输入子系统提供了设备驱动层上报输入事件的函数, 在`include/linux/input.h`中:

```
voidinput_report_key(struct input_dev *dev, unsigned int code, int value);      //上报按键事件
voidinput_report_rel(struct input_dev *dev, unsigned int code, int value);      //上报相对坐标事件
voidinput_report_abs(struct input_dev *dev, unsigned int code, int value);      //上报绝对坐标事件
……
```

当提交输入设备产生的输入事件之后, 需要调用下面的函数来通知输入子系统, 以处理设备产生的完整事件:  

```
void input_sync(struct input_dev *dev); 
```


----------------------------------------------------
参考: 
http://blog.csdn.net/ylyuanlu/article/details/6704744
http://blog.csdn.net/lbmygf/article/details/7360084
http://www.bkjia.com/Androidjc/936637.html
http://blog.chinaunix.net/uid-23381466-id-3875934.html


------------------------------

  [pic1]: http://www.linuxidc.com/upload/2011_10/111018063567091.png 
  [pic2]: http://www.linuxidc.com/upload/2011_10/111018063567092.png 
