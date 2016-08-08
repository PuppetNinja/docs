# Get_Start_with_Yocto_Project

tags: yocto 

------

## 1. 概述
Yocto Project 是一个开源协作项, 它提供了一些模, 工具和方法来支持面向嵌入式产品的自定义Linux系统, 不管硬件架构是什么. 术语yocto是最小的SI单元, 作为一个前缀, yocto表示 ${10}^{-24}$ , 

## 2. 环境与配置
主机系统: mint 17.3 (base on ubuntu 14.04)

### 2.1. 安装需要的依赖包
以下基于ubuntu/debian系Linux, 其它发行版参考[这里][packages].
```
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo \
  gcc-multilib build-essential chrpath socat libsdl1.2-dev xterm
```

### 2.2. 获取工程
```
$ git clone -b krogoth git://git.yoctoproject.org/poky.git 
```

### 2.3. 初始化构建环境

运行环境配置脚本`source oe-init-build-env [build_dir]`在当前主机定义OpenEmbedded构建环境.

这会在目录创建目录**build**做为*Build Directory*, 并且当前工作目录也切换到了这个*Build  Directory*. 在构建过程中的所有文件都会保存到该目录下.

一般的用法是为不同的目标创建不同的build_dir. 如: ../build/x86 or ~/build/arm
本例:
```
$ mkdir ../build/x86
$ source oe-init-build-env ../build/x86
```
之后工作目录会自动切换到*../build/x86*下.

### 2.4. 检查本地配置文件
**build/conf/local.conf** 文件为本地配置文件. 在使用 *BitBake* 开始构建你的工程之前, 请先检查该配置是否符合你的要求.

**\*** 默认情况下会构建一个 qemux86 镜像, 即模拟 32 位 x86 处理器的一个 QEMU 实例. 
```
# This sets the default machine to be qemux86 if no other machine is selected:  
MACHINE ??= "qemux86"
```

**\*** 另一个要注意的地方: local.conf 配置文件默认选择RPM包管理机制, 这里我们要制作ubuntu系镜像, 要改为DEB方式. 
```
# We default to rpm:
PACKAGE_CLASSES ?= "package_rpm"
```
**将package_rpm改为package_deb.**

## 3. 构建与使用

### 3.1. 构建镜像

参考: 
[BitBake User Manual][bitbake command]
[bitbake images][bitbake images] 

使用如下命令开始构建镜像:
```
$ bitbake core-image-stao
```

**注意:**
- *Do not use bitbake as root.*
- 项目所有者不要是root；chown -R yocto usrname:usrename 
- 这个步骤所花的时间受当前主机的处理器数量/核心数/内存/网速等影响. 时间十分漫长. 

### 3.2. 使用QEMU运行镜像

```
$ runqemu qemux86
```
在仿真器的Terminal中执行`poweroff`可退出QEMU.

More Info: 
[Using the Quick EMUlator(QEMU)][use QEMU]
[QEMU Website][QEMU Website]
[QEMU User Manual][QEMU Documentation]

## 4. new image

下面的步骤展示了如何为MinnowBord MAX开发板build一个镜像的, Yocto Project默认支持该板型, 并提供*mate-intel intel-corei7-64* 和 *inter-core2-32* Board Support Packages (BSPs).

### 4.1. 获得本地mate-intel库

在为MinnowBord MAX制作镜像之前, 首先要获得*mate-intel*层(layer).
```
$ cd <your_path>/poky
$ git clone git://git.yoctoproject.org/meta-intel
```

**NOTE:** 
默认情况下, 我们git到的*mate-intel*库会在主分支上, 但是我们要用这个*mate-intel*库构建自己的镜像的话, 它**必须**与*poky*代码库使用相同的release(分支名相同). 所以:
```
$ cd <your_path>/poky/meta-intel
$ git checkout krogoth
```

### 4.2. 配置
参考2.3节设置[build_dir].
编辑文件*bblayers.conf* 和 *local.conf*, 位于目录< Build Direcroty>/conf. 
另一种快捷配置方式为使用`bitbake-layers`和`add-layer`命令分别用于添加*mate-intel* layer(包含了inter-core\* BSPs) 和设置MACHINE选择BSP.
```
$ cd <Build_DIR>
$ bitbake-layers add-layer "$HOME/poky/meta-intel"
$ echo 'MACHINE = "intel-corei7-64"' >> conf/local.conf
```
### 4.3. 构建最小镜像 

```
$ bitbake core-image-minimal
```
构建完成后, 可以在Build Directory下找到构建的镜像: 
`tmp/deploy/images/intel-corei7-64/core-image-minimal-intel-corei7-64.hddimg`

------

**参考** : 
[Yocto Project Quick Start][yocto quick start]
[Yocto Project 快速入门][yocto 快速入门] but是1.8版本的
[Getting Start with the Yocto Project][yocto get start]

---------------------------------------------------------------------

  [yocto quick start]: http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#yocto-project-qs-intro "yocto Quick Start"
  [yocto 快速入门]: https://www.zybuluo.com/XiangZhou/note/166790 "yoct project 快速入门"
  [yocto get start]:http://www.yoctoproject.org/docs/2.1/mega-manual/mega-manual.html#dev-manual-start "yocto get start"
  [packages]: http://www.yoctoproject.org/docs/2.1/ref-manual/ref-manual.html#required-packages-for-the-host-development-system "packages"
  [bitbake command]: http://www.yoctoproject.org/docs/2.1/bitbake-user-manual/bitbake-user-manual.html#bitbake-user-manual-command "bitbake"
  [bitbake images]: http://www.yoctoproject.org/docs/1.8/ref-manual/ref-manual.html#ref-images "images"
  [use QEMU]: http://www.yoctoproject.org/docs/2.1/dev-manual/dev-manual.html#dev-manual-qemu "use QEMU"
  [QEMU Website]: http://wiki.qemu.org/Main_Page "QEMU Main Page//"
  [QEMU Documentation]: http://wiki.qemu.org/Manual "QEMU Manual"