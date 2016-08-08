# Note_of_Yocto_Project

tags: yocto 

---

## Toaster

在2.1版本的Yocto Project中, Hob已被弃用, 参考[官方说明][hob].
现新的工具为[Toaster][], 使用方法: [toaster user manual][].

-------

## 使用Yocto Project的BSP工具

使用工程自带的BSP工具可以快速的制作一个BSP.
在这之前要先source环境:
    
    source oe-init-build-env Build_Dir
    
使用`yocto-bsp help`查看帮助. 
例: 用`yocto-bsp create guitar qemu`可以开始自动构建, 在这个过程中会提示选择何种硬件架构, 选择内核等；
在选内核阶段, 我们要使用guitar的linux-actions, 可以使用本地目录或远程git连接. 

**注:**
1. 从git上clone下一个本地内容时, 该本地分支要为master. 
2. 先在linux-actions目录下执行`make defconfig`命令, 否则在构建BSP时会出错. 

------

## 尝试构建镜像

参考Get_Start_With_Yocto_Project一文中使用meta-intel的方法. 

问题: 
1. 在创建BSP时使用的`yocto-bsp create guitar qemu`, 镜像构建完成后没有*.ddimg镜像. ()
2. 在guitar目录下执行`runqemu qemux86`模拟时, `uname -r`发现内核为yocto-4.4 :( 
3. 使用`yocto-bsp create my_guitar arm`新构建一个BSP, 再次build镜像时提示MACHINE还是qemux86.
3.1. 在构建前设置MACHINE为guitar, 构建失败. 

思考:
1. 没有*ddimg镜像文件的问题 >> 有其它配置选项? 
2. 镜像MACHINE选项问题 >> 在制作BSP时的配置? >> build镜像时的配置(参数应该是参考BSP哪个文件)?
3. 关于问题2, 手动添加的BSP好像没生效? 为何`uname -r`还是默认的yocto默认的4.4内核?

------

http://blog.hackroad.com/operations-engineer/linux_server/12856.html
这个问题...
> Error: No space left on device


------
下面为Raspberry Pi使用Yocto的一个例子: 
[12MB Minimal Image for Raspberry Pi using the Yocto Project][rpi]

-------------------------------
2016.06.23: error

```
ERROR: u-boot-v2016.03+gitAUTOINC+df61a74e68-r0 do_compile: oe_runmake failed
ERROR: u-boot-v2016.03+gitAUTOINC+df61a74e68-r0 do_compile: Function failed: do_compile (log file is located at /home/tab/work/yocto/build/guitar/tmp/work/guitar-poky-linux-gnueabi/u-boot/v2016.03+gitAUTOINC+df61a74e68-r0/temp/log.do_compi
le.16108)
ERROR: Logfile of failure stored in: /home/tab/work/yocto/build/guitar/tmp/work/guitar-poky-linux-gnueabi/u-boot/v2016.03+gitAUTOINC+df61a74e68-r0/temp/log.do_compile.16108
Log data follows:
| DEBUG: Executing shell function do_compile
| NOTE: make -j 4 CROSS_COMPILE=arm-poky-linux-gnueabi- CC=arm-poky-linux-gnueabi-gcc  --sysroot=/home/tab/work/yocto/build/guitar/tmp/sysroots/guitar V=1 HOSTCC=gcc  -isystem/home/tab/work/yocto/build/guitar/tmp/sysroots/x86_64-linux/usr/
include -O2 -pipe -L/home/tab/work/yocto/build/guitar/tmp/sysroots/x86_64-linux/usr/lib -L/home/tab/work/yocto/build/guitar/tmp/sysroots/x86_64-linux/lib -Wl,-rpath-link,/home/tab/work/yocto/build/guitar/tmp/sysroots/x86_64-linux/usr/lib -
Wl,-rpath-link,/home/tab/work/yocto/build/guitar/tmp/sysroots/x86_64-linux/lib -Wl,-rpath,/home/tab/work/yocto/build/guitar/tmp/sysroots/x86_64-linux/usr/lib -Wl,-rpath,/home/tab/work/yocto/build/guitar/tmp/sysroots/x86_64-linux/lib -Wl,-O
1 6666
| make -f ./Makefile silentoldconfig
| make -f ./scripts/Makefile.build obj=scripts/basic
| rm -f .tmp_quiet_recordmcount
| make -f ./scripts/Makefile.build obj=scripts/kconfig silentoldconfig
| mkdir -p include/config include/generated
| scripts/kconfig/conf  --silentoldconfig Kconfig
| ***
| *** Configuration file ".config" not found!
| ***
| *** Please run some configurator (e.g. "make oldconfig" or
| *** "make menuconfig" or "make xconfig").
| ***
| scripts/kconfig/Makefile:46: recipe for target 'silentoldconfig' failed
| make[2]: *** [silentoldconfig] Error 1
| Makefile:477: recipe for target 'silentoldconfig' failed
| make[1]: *** [silentoldconfig] Error 2
| make: *** No rule to make target '6666'.  Stop.
| ERROR: oe_runmake failed
| ERROR: Function failed: do_compile (log file is located at /home/tab/work/yocto/build/guitar/tmp/work/guitar-poky-linux-gnueabi/u-boot/v2016.03+gitAUTOINC+df61a74e68-r0/temp/log.do_compile.16108)
ERROR: Task 77 (/home/tab/work/yocto/poky/meta/recipes-bsp/u-boot/u-boot_2016.03.bb, do_compile) failed with exit code '1'
NOTE: Tasks Summary: Attempted 1775 tasks of which 1769 didn't need to be rerun and 1 failed.
Waiting for 0 running tasks to finish:

Summary: 1 task failed:
  /home/tab/work/yocto/poky/meta/recipes-bsp/u-boot/u-boot_2016.03.bb, do_compile
Summary: There was 1 WARNING message shown.
Summary: There were 2 ERROR messages shown, returning a non-zero exit code.
```

--------------------

- How do I add my package to the Yocto Project?
	
To add a package, you need to create a BitBake recipe. For information on how to create a BitBake recipe, see the "Writing a New Recipe" in the Yocto Project Development Manual. http://www.yoctoproject.org/docs/2.1/mega-manual/mega-manual.html#new-recipe-writing-a-new-recipe


-------------------------------------------------------------
  [rpi]: http://www.tuicool.com/articles/fQ3MNf "R-Pi yocto"
  [hob]: https://www.yoctoproject.org/tools-resources/projects/hob "hob"
  [Toaster]: https://www.yoctoproject.org/tools-resources/projects/toaster "toaster" 
  [OpenEmbedded]: http://www.openembedded.org/wiki/Main_Page "OpenEmbedded"
  [toaster user manual]: https://www.yoctoproject.org/docs/1.8/toaster-manual/toaster-manual.html "taster user manual"
