# docker安装与使用

tags: docker

---

[TOC]

---

## 安装
通过Docker源安装最新版本  
要安装最新的 Docker 版本，首先需要安装 apt-transport-https 支持，之后通过添加源来安装。

    sudo apt-get install apt-transport-https
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
    sudo bash -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
    sudo apt-get update
    sudo apt-get install lxc-docker
## 确保docker就绪

> hange @ nuc ॐ  ~:
7 ◯  sudo docker info
[sudo] password for hange: 
Containers: 0
Images: 0
Server Version: 1.9.1
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: true
Execution Driver: native-0.2
Logging Driver: json-file
Kernel Version: 3.19.0-32-generic
Operating System: Ubuntu 14.04.4 LTS
CPUs: 4
Total Memory: 15.56 GiB
Name: nuc
ID: 2AIA:NROB:6KMW:VDNR:5TFR:QGA6:QOPZ:5YGB:SWD5:GAKG:4PFM:LZ2F
WARNING: No swap limit support

如上所示, 调用docker的info命令可以查看所有容器和镜像的数量,docker使用的执行驱动和存储驱动(execution and storage driver), 以及docker的基本配置.

## 运行一个容器
使用`docker run`命令来创建一个容器, 它提供了docker容器从创建到启动的功能:

> hange @ nuc ॐ  ~:
51 ◯  sudo docker run -it ubuntu /bin/bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
487bffc61de6: Pulling fs layer 
acb8e44f43fa: Download complete 
202e40f8bb3a: Pulling fs layer 
b0c2dfa2701f: Pulling fs layer 
17b6a9e179d7: Pulling fs layer 
Pulling repository docker.io/library/ubuntu
686477c12982: Download complete 
dd25ab30afb3: Download complete 
a83540abf000: Download complete 
630aff59a5d5: Download complete 
cdc870605343: Download complete 
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu: this image was pulled from a legacy registry.  Important: This registry version will not be supported in future versions of docker.
root@694a92d96381:/# 
    
解析:
`sudo run docker run -it ubuntu /bin/bash`

* 两个参数:  **-i** 和 **-t** 
 - '-i'保证容器中STDIN是开启的, 持久的标准输入是shell的半边天.
 - '-t'则是另外半边天,它告诉docker要为创建的容器分配一个伪tty终端.

这样, 新创建的容器才可以提供一个交互式shell. 如果不是创建一个运行后台服务器的容器的话, 这两个参数是最基本的.
使用`man docker-run`查看`docker run`的所有参数.

接下来, 告诉docker基于什么镜像去创建容器, 这里以ubuntu基础镜像为例(还有其它的fedora, debian, centos等), 它由Docker公司提供并保存在Docker Hub Registry上. 在这里, 我们基于此基础镜像创建并启动了一个容器, 并且没有对它进行任何改动. 

在这个过程背后发生了什么?
首先, docker会检查本地是否有ubuntu镜像,如果本地没有ubuntu镜像的话, docker会连接官方维护的Docker Hub Registry查看Docker Hub上是否有该镜像, 一旦找到就会把它下载并保存到本地宿主机中.

之后, docker在自己的文件系统内部创建了一个新的容器, 它有自己的网络, ip地址和一个用来和宿主机进行通信的桥接网络接口, 最后, 告诉docker在新的容器中要运行什么命令, 本例在容器中运行/bin/bash命令启动一个Bash shell. (省略该参数, 默认运行/bin/bash)

## 使用容器

例如: 运行一个容器: `sudo docker run --name hange -it ubuntu`
这里用**-name**参数是给创建的容器指定一个名称, 否则会自动生成一个名称, 不利于管理. 

> root@63e97ef8b640:/# hostname 
  63e97ef8b640

可以看到, 容器的主机名就是它的ID. 同样的也可以查看容器中的进程: 

> root@63e97ef8b640:/# ps -aux 
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  18244  3136 ?        Ss   05:35   0:00 /bin/bash
root        54  0.0  0.0  34424  2708 ?        R+   05:48   0:00 ps -aux

安装一个软件: 

> root@63e97ef8b640:/# apt-get update && apt-get install vim 

当所有的工作完成时, 可以输入`exit`回到宿主机的命令提示符. 这时, 容器已经停止运行了, 但是它仍然是存在的, 用`sudo docker ps -a`可以查看系统中所有容器, 包括已经停止的和正在运行的. 

![2016-05-08-142030_1077x78_scrot.png-18.1kB][pic1]

**NOTE** 
`docker ps -a`中的*-a*选项表示列出所有容器, **-l**表示列出上一次运行的容器. 

启动已经停止的容器: `sudo docker start name/id`
附着到容器上: `docker attach name/id`
后台运行一个容器, 例: 


[pic1]: http://static.zybuluo.com/tab/bvb4zyfv5idsx8zmwc8goohx/2016-05-08-142030_1077x78_scrot.png "ps -a "