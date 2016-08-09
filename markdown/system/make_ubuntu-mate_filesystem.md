# make_ubuntu-mate_filesystem

tags: system 

--------

## 最小ubuntu系统

以下参考：[wiki.pro][1]

### install the required packages

    apt-get install qemu-user-static binfmt-support

### install Ubuntu Mate using debootstrap

    mkdir ubuntu-mate-fs
    debootstrap --arch armhf --foreign xenial ubuntu-mate-fs
    
    cp /usr/bin/qemu-arm-static ubuntu-mate-fs/usr/bin
    cp /etc/resolv.conf ubuntu-mate-fs/etc

    #后面加个bash是因为host主机用的是zsh,在ubuntu-mate最小系统里没有zsh,指定为bash;
    chroot ubuntu-mate-fs bash
    export LANG=C
    # We can now continue with the debootstrap process using:
    debootstrap/debootstrap --second-stage  

    #add source.list for ubuntu
    cat <<EOT > etc/apt/sources.list
    deb http://ports.ubuntu.com/ xenial main universe
    deb-src http://ports.ubuntu.com/ xenial main universe
    deb http://ports.ubuntu.com/ xenial-security main universe
    deb-src http://ports.ubuntu.com/ xenial-security main universe
    deb http://ports.ubuntu.com/ xenial-updates main universe
    deb-src http://ports.ubuntu.com/ xenial-updates main universe
    EOT

    # set up apt
    cat <<EOT > etc/apt/apt.conf.d/71-no-recommends
    APT::Install-Recommends "0";
    APT::Install-Suggests "0";
    EOT
    # update the latest database from the servers:
    apt-get update
    # And you should set your locales (Debian) with dpkg scripts or they tend to complain otherwise:
    apt-get install locales
    dpkg-reconfigure locales
    # Choose en_US.UTF-8 or whatever you want:
    export LANG=en_US.UTF-8

    # Set a root password so you can login
    passwd
    # Set the hostname
    echo mate > etc/hostname
    
    # add this line to /etc/hosts 
    127.0.1.1   mate
    
    # Exit the chroot session
    etc/init.d/ssh stop
    exit

### Clean up the support files

```
rm ubuntu-mate-fs/usr/bin/qemu-arm-static ubuntu-mate-fs/etc/resolv.conf
```

## Mate桌面环境安装

### 安装tasksel工具

```
apt-get install tasksel
```    

### 安装mate

```
tasksel --task-packages ubuntu-mate-desktop | xargs apt-get -y install
```

## 桌面环境优化

### 安装IDE开发工具

```
# apt-get install codeblocks
apt-get install build-essential
```    

### python 开发环境

默认已安装. 

```
# apt-get install idle-python2.7
# apt-get install idle-python3.5
```

## 系统优化

### 开机加载的驱动

`vim /etc/modules`

    ethernet
    wlan_8723bs
    rfkill-actions_8723bs
    
### 屏蔽不要的驱动

删除原文件系统**/etc/modprobe.d**目录下*.conf文件。
新建**/etc/modprobe.d/blacklist-guitar.conf**:

    # The file blacklist some drivers which don't use on the Guitar.
    ###GPU Driver
    #blacklist pvrsrvkm

    ###WiFi Driver
    blacklist wlan_8723bs_vq0

    ###CTP Driver
    blacklist ctp_gslX680
    blacklist ctp_gsl3680
    blacklist ctp_gt9xx

    ###Gsensor Driver
    blacklist gsensor_mir3da
    blacklist gsensor_stk8313
    blacklist gsensor_bma222
    blacklist lightsensor_ltr301

### 更换桌面背景

将需要替换的桌面复制到`/usr/share/backgrounds/ubuntu-mate-common`中.
创建软链接: 
```
#更改登陆窗口背景
ln -sf Ubuntu-Mate-Lemaker.jpg Ubuntu-Mate-Cold-lightdm.jpg
#更改桌面背景
ln -sf Ubuntu-Mate-Lemaker.jpg Ubuntu-Mate-Cold.jpg
```

### 允许root登陆ssh

先安装openssh-server: `apt-get install openssh-server` 
编辑文件: `vim /etc/ssh/sshd_cnfig`

    ...
    #PermitRootLogin prohibit-password  #注释该选项
    PermitRootLogin yes  #新增选项
    ...
   
### 添加lemaker帐户

```
useradd -s /bin/bash -m lemaker
passwd lemaker
```

### 添加lemaker到sudoer files

`vim /etc/sudoers`  ... ...

### 桌面系统自动登陆

```
# /usr/share/lightdm/lightdm.conf.d# cat 50-ubuntu-mate.conf
[SeatDefaults]
autologin-user=lemaker
autologin-timerout=0
user-session=mate
```    

### cpu动态频率设置

`vim etc/rc.local`
```
echo ondemand > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

echo 45 > /sys/devices/system/cpu/cpufreq/ondemand/up_threshold
echo 15 > /sys/devices/system/cpu/cpufreq/ondemand/sampling_down_factor
echo 1 > /sys/devices/system/cpu/cpufreq/ondemand/io_is_busy

```    
    
### 安装GPU
参考 [wiki:硬件加速][2]

#### 下载并解压文档

```
wget http://mirror.lemaker.org/GPU_For_LeMaker_Guitar_201603.tar.gz
sudo tar -xvf GPU_For_LeMaker_Guitar_201603.tar.gz
```

#### 安装软件包

```
cd GPU_For_LeMaker_Guitar_201603
sudo chown -R root:root ./*
sudo ./install.sh
```

#### 替换 X, Xorg

```
cd /usr/bin
sudo mv X X.original
sudo mv Xorg Xorg.original
sudo ln -sf /usr/local/XSGX/bin/X X
sudo ln -sf /usr/local/XSGX/bin/Xorg Xorg
sudo cp /usr/local/XSGX/etc/xorg.conf  /etc/
```

#### 检查依赖模块

```
sudo depmod
```

> 注意： 使用depmod时提示没有/lib/modules/* 目录，手动创建之。
> ```
> mkdir -p /lib/modules/`uname -r`
> mkdir -p /lib/modules/`uname -r`/modules.order
> mkdir -p /lib/modules/`uname -r`/modules.builtin
> ```

#### 加载路径库

需要将/usr/local/XSGX/lib及 /usr/lib目录添加到系统默认的库搜索路径，在/etc/ ld.so.conf.d目录下新增文件sgx544.conf, 内容如下: 

    /usr/local/XSGX/lib
    /usr/lib
    # ldconfig  #更新缓存文件/etc/ld.so.cache

系统默认安装了mesa库，为使用SGX544提供的OpenGL/OpenGL-ES库，需先屏蔽系统原生的mesa库，具体操作如下:

```
rm arm-linux-gnueabihf_GL.conf
rm arm-linux-gnueabihf_EGL.conf
ldconfig
```
    
> 安装GPU驱动后桌面启不来, 要删除以下文件即可: 
> `sudo rm /usr/share/X11/xorg.conf.d/10-amdgpu.conf` 

## Clear

1、清除/var/log/目录下所有的log

    cd /var/log/ && find ./ -type f | xargs rm -rf {}
    
2、删除/var/cache/apt/archives目录下的*.deb*缓存文件

    cd /var/cache/apt/archives && rm -rf *.deb
    
    
## 问题

### NTFS写入问题

    apt-get install ntfs-3g 

### 网络

新镜像不能上网，补上这个软链接：[参考][3]

    ln -s  /run/resolvconf/resolv.conf /etc/resolv.conf
    
### 关于本地环境
在使用apt-get时发现报错如下:  

    perl: warning: Setting locale failed.
    perl: warning: Please check that your locale settings:
            LANGUAGE = (unset),
            LC_ALL = (unset),
            LC_PAPER = "zh_CN.UTF-8",
            LC_NUMERIC = "zh_CN.UTF-8",
            LC_IDENTIFICATION = "zh_CN.UTF-8",
            LC_MEASUREMENT = "zh_CN.UTF-8",
            LC_NAME = "zh_CN.UTF-8",
            LC_TELEPHONE = "zh_CN.UTF-8",
            LC_ADDRESS = "zh_CN.UTF-8",
            LC_CTYPE = "en_US.UTF-8",
            LC_MONETARY = "zh_CN.UTF-8",
            LANG = "en_US.UTF-8"
        are supported and installed on your system.
    perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
    /usr/bin/locale: Cannot set LC_ALL to default locale: No such file or directory

解:
`vim /etc/default/local` 并添加以下两行:

    LANGUAGE=en_US.ETF-8
    LC_ALL=C


[1]: http://wiki.lemaker.org/BananaPro/Pi:Setting_up_the_Linux_distribution_root_file_system#Using_debootstrap "ubuntu最小文件系统"
[2]: http://wiki.lemaker.org/LeMaker_Guitar:GPU_hardware_acceleration/zh-hans#.E5.9C.A8LeMaker_Guitar.E4.B8.8A.E5.AE.89.E8.A3.85.E7.A1.AC.E4.BB.B6.E5.8A.A0.E9.80.9F.E5.BA.93 "wiki:硬件加速"
[3]: http://www.cnblogs.com/hiccup/p/4486686.html "网络修复"

