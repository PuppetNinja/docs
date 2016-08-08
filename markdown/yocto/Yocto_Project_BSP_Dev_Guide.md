# Yocto_Project_BSP_Dev_Guide

tags: yocto 

----------------------------

[BSP](http://blog.csdn.net/qsj8362234/article/details/6325761)即Board Support Package，板级支持包。它来源于嵌入式操作系统与硬件无关的设计思想，操作系统被设计为运行在虚拟的硬件平台上。对于具体的硬件平台，与硬件相关的代码都被封装在BSP中，由BSP向上提供虚拟的硬件平台，BSP与操作系统通过定义好的接口进行交互。BSP是所有与硬件相关的代码体的集合。

## 1. BSP Layers

> A BSP consists of a file structure inside a base directory. Collectively,   you can think of the base directory, its file structure, and the contents   as a BSP Layer. 

在Yocto Project中, BSP有确定的命名规则:
> mate-bsp_name 

即我们要在自己的`bsp_name`之前加上"mate-"前缀. 

你可以在 http://git.yoctoproject.org/cgit/cgit.cgi 中查看Yoctoo Project支持的Layers. 比如, 我们可以在上述页面的 "Yocto Matedata Layers" 列表中查找到如 "meta-raspberrypi" 和 "meta-intel". 但是很显然, 没有 "meta-lemaker_guitar", 所以这个需要我们自己定制了. 

`meta-bsp_name`目录作为'层'的 base directory, 被认为是 BSP layer 的根(root). 在我们的 Build Directory 下可找到配置文件`conf/bblayers.conf`, 变量 BBLAYERS 决定了在使用OpenEmbedded构建环境时(i.e. `oe-init-build-env` and `oe-init-build-env-memres`) 加载的Layer. 添加 *the root* 可以让 OpenEmbedded 根据 BSP 定义构建镜像, 如: 
```
     BBLAYERS ?= " \
       /usr/local/src/yocto/meta \
       /usr/local/src/yocto/meta-poky \
       /usr/local/src/yocto/meta-yocto-bsp \
       /usr/local/src/yocto/meta-mylayer \
       "
```                
一些BSPs为了实现更多的功能会需要依赖额外的layers, 依赖关系会在它的**README**文件中的**Dependencies段**中说明.  如上面说的 mate-intel, 下图截取了它的README相关内容: 
<center>![mate-intel_README][pic-1]</center>

同样的, 也有layer包含了sub-layers, 如meta-intal中就包含有这些特殊的layer, 它们统一位于common/目录中. 

更详细的信息阅读官方文档: [Understanding and Creating Layers][mk-layer]. 

## 2. 文件系统布局

- License Files
- README File
- README.sources File
- Pre-built User Binaries
- Laer Configuration File
- HardWare Configuration Options 
- Miscellaneous BSP-Secific Recipe Files 
- Display Support Files 
- Linux Kernel Configuration

定义一个标准化的BSP目录结构有助于用户理解并熟悉整个工程. 而OpenEmbedded就可以根据这种标准的格式直接构建出我们想要的镜像. 
BSP会把所有硬件相关的细节以标准的格式统一放在一个位置, 而且它只关心硬件相关的内容, 并不包含如何镜像或一些工具.  这样用户在制作image时就不用考虑硬件问题而可以专注于制作系统. 

下面展示一般BSP Layer的目录结构, 可以用这个基础结构为标准, 在实际应用中也会有一些特殊的BSP的目录结构与这个有点差异:

    meta-bsp_name/
    meta-bsp_name/bsp_license_file
    meta-bsp_name/README
    meta-bsp_name/README.sources
    meta-bsp_name/binary/bootable_images
    meta-bsp_name/conf/layer.conf
    meta-bsp_name/conf/machine/*.conf
    meta-bsp_name/recipes-bsp/*
    meta-bsp_name/recipes-core/*
    meta-bsp_name/recipes-graphics/*
    meta-bsp_name/recipes-kernel/linux/linux-yocto_kernel_rev.bbappend

如 Raspberry Pi BSP 的目录结构, 与上述的通用标准就有一定的差别: 

     meta-raspberrypi/COPYING.MIT
     meta-raspberrypi/README
     meta-raspberrypi/classes
     meta-raspberrypi/classes/linux-raspberrypi-base.bbclass
     meta-raspberrypi/classes/sdcard_image-rpi.bbclass
     meta-raspberrypi/conf/
     meta-raspberrypi/conf/layer.conf
     meta-raspberrypi/conf/machine/
     meta-raspberrypi/conf/machine/raspberrypi.conf
     meta-raspberrypi/conf/machine/raspberrypi0.conf
     meta-raspberrypi/conf/machine/raspberrypi2.conf
     meta-raspberrypi/conf/machine/raspberrypi3.conf
     meta-raspberrypi/conf/machine/include
     meta-raspberrypi/conf/machine/include/rpi-base.inc
     meta-raspberrypi/conf/machine/include/rpi-default-providers.inc
     meta-raspberrypi/conf/machine/include/rpi-default-settings.inc
     meta-raspberrypi/conf/machine/include/rpi-default-versions.inc
     meta-raspberrypi/conf/machine/include/rpi-tune-arm1176jzf-s.inc
     meta-raspberrypi/files
     meta-raspberrypi/files/custom-licenses
     meta-raspberrypi/files/custom-licenses/Broadcom
     meta-raspberrypi/recipes-bsp
     meta-raspberrypi/recipes-bsp/bootfiles
     meta-raspberrypi/recipes-bsp/bootfiles/bcm2835-bootfiles.bb
     meta-raspberrypi/recipes-bsp/bootfiles/rpi-config_git.bb
     meta-raspberrypi/recipes-bsp/common
     meta-raspberrypi/recipes-bsp/common/firmware.inc
     meta-raspberrypi/recipes-bsp/formfactor_00.bbappend
     meta-raspberrypi/recipes-bsp/formfactor/raspberrypi/machconfig
     meta-raspberrypi/recipes-bsp/rpi-mkimage_git.bb
     meta-raspberrypi/recipes-bsp/rpi-mkimage/License
     meta-raspberrypi/recipes-bsp/rpi-mkimage/open-files-relative-to-script.patch
     meta-raspberrypi/recipes-bsp/u-boot/u-boot-rpi_git.bb
     meta-raspberrypi/recipes-core
     meta-raspberrypi/recipes-core/images
     meta-raspberrypi/recipes-core/images/rpi-basic-image.bb
     meta-raspberrypi/recipes-core/images/rpi-hwup-image.bb
     meta-raspberrypi/recipes-core/images/rpi-test-image.bb
     meta-raspberrypi/recipes-core/packagegroups
     meta-raspberrypi/recipes-core/packagegroups/packagegroup-rpi-test.bb
     meta-raspberrypi/recipes-core/psplash
     meta-raspberrypi/recipes-core/psplash/files
     meta-raspberrypi/recipes-core/psplash/psplash_git.bbappend
     meta-raspberrypi/recipes-core/psplash/files/psplash-raspberrypi-img.h
     meta-raspberrypi/recipes-devtools
     meta-raspberrypi/recipes-devtools/bcm2835
     meta-raspberrypi/recipes-devtools/bcm2835/bcm2835_1.46.bb
     meta-raspberrypi/recipes-devtools/pi-blaster
     meta-raspberrypi/recipes-devtools/pi-blaster/files
     meta-raspberrypi/recipes-devtools/pi-blaster/*.patch
     meta-raspberrypi/recipes-devtools/pi-blaster/pi-blaster.inc
     meta-raspberrypi/recipes-devtools/pi-blaster/pi-blaster_git.bb
     meta-raspberrypi/recipes-devtools/python
     meta-raspberrypi/recipes-devtools/python/python-rtimu
     meta-raspberrypi/recipes-devtools/python/python-rtimu/*.patch
     meta-raspberrypi/recipes-devtools/python/python-rtimu_git.bb
     meta-raspberrypi/recipes-devtools/python/python-sense-hat_2.1.0.bb
     meta-raspberrypi/recipes-devtools/python/rpi-gpio
     meta-raspberrypi/recipes-devtools/python/rpi-gpio/*.patch
     meta-raspberrypi/recipes-devtools/python/rpi-gpio_0.6.1.bb
     meta-raspberrypi/recipes-devtools/python/rpio
     meta-raspberrypi/recipes-devtools/python/rpio/*.patch
     meta-raspberrypi/recipes-devtools/python/rpio_0.10.0.bb
     meta-raspberrypi/recipes-devtools/wiringPi
     meta-raspberrypi/recipes-devtools/wiringPi/files
     meta-raspberrypi/recipes-devtools/wiringPi/files/*.patch
     meta-raspberrypi/recipes-devtools/wiringPi/wiringpi
     meta-raspberrypi/recipes-devtools/wiringPi/wiringpi/*.patch
     meta-raspberrypi/recipes-devtools/wiringPi/wiringpi_git.bb
     meta-raspberrypi/recipes-graphics
     meta-raspberrypi/recipes-graphics/eglinfo
     meta-raspberrypi/recipes-graphics/eglinfo/eglinfo-fb_%.bbappend
     meta-raspberrypi/recipes-graphics/eglinfo/eglinfo-x11_%.bbappend
     meta-raspberrypi/recipes-graphics/userland
     meta-raspberrypi/recipes-graphics/userland/userland
     meta-raspberrypi/recipes-graphics/userland/userland/*.patch
     meta-raspberrypi/recipes-graphics/userland/userland_git.bb
     meta-raspberrypi/recipes-graphics/vc-graphics
     meta-raspberrypi/recipes-graphics/vc-graphics/files
     meta-raspberrypi/recipes-graphics/vc-graphics/files/egl.pc
     meta-raspberrypi/recipes-graphics/vc-graphics/files/vchiq.sh
     meta-raspberrypi/recipes-graphics/vc-graphics/vc-graphics-hardfp.bb
     meta-raspberrypi/recipes-graphics/vc-graphics/vc-graphics.bb
     meta-raspberrypi/recipes-graphics/vc-graphics/vc-graphics.inc
     meta-raspberrypi/recipes-graphics/wayland
     meta-raspberrypi/recipes-graphics/wayland/weston_%.bbappend
     meta-raspberrypi/recipes-graphics/weston
     meta-raspberrypi/recipes-graphics/weston/weston_%.bbappend
     meta-raspberrypi/recipes-graphics/xorg-xserver
     meta-raspberrypi/recipes-graphics/xorg-xserver/xserver-xf86-config
     meta-raspberrypi/recipes-graphics/xorg-xserver/xserver-xf86-config/rpi
     meta-raspberrypi/recipes-graphics/xorg-xserver/xserver-xf86-config/rpi/xorg.conf
     meta-raspberrypi/recipes-graphics/xorg-xserver/xserver-xf86-config/rpi/xorg.conf.d
     meta-raspberrypi/recipes-graphics/xorg-xserver/xserver-xf86-config/rpi/xorg.conf.d/10-evdev.conf
     meta-raspberrypi/recipes-graphics/xorg-xserver/xserver-xf86-config/rpi/xorg.conf.d/99-pitft.conf
     meta-raspberrypi/recipes-graphics/xorg-xserver/xserver-xf86-config_0.1.bbappend
     meta-raspberrypi/recipes-kernel
     meta-raspberrypi/recipes-kernel/linux-firmware
     meta-raspberrypi/recipes-kernel/linux-firmware/linux-firmware
     meta-raspberrypi/recipes-kernel/linux-firmware/linux-firmware/LICENSE.broadcom_brcm80211
     meta-raspberrypi/recipes-kernel/linux-firmware/linux-firmware/brcmfmac43430-sdio.bin
     meta-raspberrypi/recipes-kernel/linux-firmware/linux-firmware/brcmfmac43430-sdio.txt
     meta-raspberrypi/recipes-kernel/linux-firmware/linux-firmware_git.bbappend
     meta-raspberrypi/recipes-kernel/linux
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi-3.14
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi-3.14/*.patch
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi-3.18
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi-3.18/*.patch
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi-4.1
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi-4.1/*.patch
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi.inc
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi/defconfig
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi_3.14.bb
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi_3.18.bb
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi_4.1.bb
     meta-raspberrypi/recipes-kernel/linux/linux-raspberrypi_4.4.bb
     meta-raspberrypi/recipes-kernel/linux/linux.inc
     meta-raspberrypi/recipes-multimedia
     meta-raspberrypi/recipes-multimedia/gstreamer
     meta-raspberrypi/recipes-multimedia/gstreamer/gstreamer1.0-omx
     meta-raspberrypi/recipes-multimedia/gstreamer/gstreamer1.0-omx/*.patch
     meta-raspberrypi/recipes-multimedia/gstreamer/gstreamer1.0-omx_%.bbappend
     meta-raspberrypi/recipes-multimedia/gstreamer/gstreamer1.0-plugins-bad_%.bbappend
     meta-raspberrypi/recipes-multimedia/omxplayer
     meta-raspberrypi/recipes-multimedia/omxplayer/omxplayer
     meta-raspberrypi/recipes-multimedia/omxplayer/omxplayer/*.patch
     meta-raspberrypi/recipes-multimedia/omxplayer/omxplayer_git.bb
     meta-raspberrypi/scripts
     meta-raspberrypi/scripts/lib
     meta-raspberrypi/scripts/lib/image
     meta-raspberrypi/scripts/lib/image/canned-wks
     meta-raspberrypi/scripts/lib/image/canned-wks/sdimage-raspberrypi.wks

下面的章节详细介绍下这种结构.

### 2.1. License File
目录位置: 

    meta-bsp_name/bsp_license_file
    
这是个可选文件, 如Raspberry Pi使用的是COPYING.MIT.

### 2.2. README File

目录位置:

    meta-bsp_name/README
    
**注意:** README 文件中至少要有**dependences**段, 如:

<center>![mate-intel_README][pic-1]</center>

把所依赖的BSP layer描述清楚, 并要有它的地址. 

### 2.3. README.source File

目录位置:

    meta-bsp_name/README.sources
     
这个目录中的文件提供获取BSP源文件的信息, 可以看到Raspberry BSP的目录中没有这个文件, 而在mate-intel中有这个文件. 
NOTE: 如果没有binary目录或binary目录中没有image, 单独的一个README.source文件没有意义；

- README ? README.source

> The README file should be present in the BSP Layer and it will explain how to use the images with the target hardware. Additionally, the README.sources file should be present to locate the sources used to build the images and provide information on the Metadata. 

### 2.4. Pre-built User Binaries

目录位置:

    meta-bsp_name/binary/bootable_images
    
这也是一个可选的内容: 官方说明:
> This optional area contains useful pre-built kernels and user-space filesystem images released with the BSP that are appropriate to the target system. This directory typically contains graphical (e.g. Sato) and minimal live images when the BSP tarball has been created and made available in the Yocto Project website. You can use these kernels and images to get a system running and quickly get started on development tasks. 

### 2.5. Layer Configuration File

目录位置:

    meta-bsp_name/conf/layer.conf

这个配置文件把文件结构当作一个layer并指定该layer所包含的内容和我们构建的系统应该如何使用它. 

### 2.6 - 2.9 

pass

## 3. 使用Yocto Project的BSP工具

使用工程自带的BSP工具可以快速的制作一个BSP.
在这之前要先source环境:
    
    source oe-init-build-env Build_Dir
    
使用`yocto-bsp help`查看帮助. 
例: 用`yocto-bsp create my_guitar qemu`可以开始自动构建, 在这个过程中会提示选择何种硬件架构, 选择内核等；
在选内核阶段, 我们要使用guitar的linux-actions, 可以使用本地目录或远程git连接. 

**注:**
1. 从git上clone下一个本地内容时, 该本地分支要为master. 
2. 先在linux-actions目录下执行`make defconfig`命令, 否则在构建BSP时会出错. 

----------------------------------

参考: [Board Support Packages (BSP) - Developer's Guide][bsp]

  [bsp]: http://www.yoctoproject.org/docs/2.1/bsp-guide/bsp-guide.html#bsp "BSP"
  [mk-layer]: http://www.yoctoproject.org/docs/2.1/dev-manual/dev-manual.html#understanding-and-creating-layers "create layers"
  [pic-1]: http://static.zybuluo.com/tab/ug1b6irpajnjvvawgca1r6ac/2016-05-25-153202_438x267_scrot.png "mate-intel README"
  