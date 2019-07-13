---
layout:     post
title:      Install and Set Up a Usable Ubuntu Desktop
subtitle:   打造一个可用的Ubuntu18.04桌面
date:       2018-04-26
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Linux
    - Ubuntu
---

>以下实践经验总结，错误之处难免。

# 虚拟机：VMware安装Ubuntu
1) 在VMware中依次点击文件-->新建虚拟机，调出新建虚拟机向导。

2) 指定系统镜像ISO文件，填写主机名和用户名，指定虚拟机名称，确认创建。

3)创建好后在我的计算机下查看刚刚创建的虚拟机，右击可进一步设置内存、硬盘等参数。

4)设置好后点击开启此虚拟机开始安装操作系统，也可通过安装步骤左边的小三角选择skip跳过。

5)安装vmware tools，注销并重新登陆生效，有时无法和windows互相复制粘贴，就需要手动执行/usr/bin/vmware-user.

# 双系统：Win10+Ubuntu
1) Win10下按win+x快捷键依次点击计算机管理-->磁盘管理，右击某磁盘选择压缩卷分出一块空间，保持空闲，或通过diskgenieus删除某分区使其空闲。

2) 用Ubuntu推荐的rufus工具将ISO写入U盘，重启按F12选择从上述U盘启动，在首界面选择安装Ubuntu。

3) 在Installation Type这一步选择Something else手动为Ubuntu分区，对上述空闲空间进行分区并指定挂载点，例如/boot 200M足够，最好放在前面，/swap来个4G，/根目录20G，/home要尽量大一点，尤其对用户较多的系统，1G按1024M计，安装启动引导器的设备选/boot。

4) 安装完成后，在Win10下用EasyBCD添加新入口，重启即可选择要进入的系统。

# 配置：打造一个可用的Linux操作环境
1) 安装vim

2) 编辑/etc/apt/sources.list更换源，执行apt update && apt upgrade更新升级

3) 激活自带中文输入法：设置里点击添加输入源，然后展开Chinese一项选择Intelligent Pinyin

4) 编辑~/.config/user-dirs.dirs文件，去掉多余的文件夹，同时新建user-dirs.conf文件并写入enabled=false这一行以屏蔽系统级配置

5) 编辑/etc/default/grubGrub修改启动超时GRUB_TIMEOUT=0.01(设为0似乎代表默认值不起作用)，sudo update-grub2更新GRUB使配置生效

6) 多版本jdk安装和切换，以及环境变量的配置  
修改/etc/profile全局配置，或~/.bashrc用户配置)
```bash
export JAVA_HOME=/usr/lib/jdk6/jdk1.6.0_45
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
#此时$PATH附加在JAVA_HOME后边，update-alternatives命令切换程序版本无效，可可将$PATH放在JAVA_HOME前面，然后执行
export PATH=${JAVA_HOME}/bin:$PATH

#为软链接/usr/bin/java和/usr/bin/javac添加对应的替代程序
update-alternatives --install
#回车选择要切换到的默认版本
update-alternatives --config java/javac
#类似地，可选择系统默认编辑器
sudo update-alternatives --config editor
```

7) 参照source.android.com安装源码编译所需的程序包

8) 安装make3.8.1并添加到PATH环境变量，据aosp官网说3.82版本会有问题

9) 安装交叉编译工具arm-2009q3(可从codesourcery.com网站获得最新版本)，并将其添加到PATH环境变量，也可在/usr/bin下建立arm-linux-gcc软链接

10) 安装p7zip-full软件，7z命令可解压大部分压缩格式

11) 为root用户配置密码 sudo passwd root，这将启用被锁定的root用户

12) 安装openssh-server、samba等常用服务

13) 安装apt install QImsg tree mailutils cifs-utils

14) 安装snap install notepadqq GitKraken Chromium Tusk

15) 修改/etc/network/interfaces设置静态ip，重启或执行sudo /etc/init.d/networking restart使设置生效（所有服务的启动脚本都位于/etc/init.d目录下）

16) 使用ADT Bundle时，若采用最新版eclipse，所需的jre版本(可在eclipse.ini中查看)应拷贝到eclipse根目录下，并安装ADT Plugin

17) 下载并解压android studio，将android-studio/bin加入环境变量，安装一些必要的32位库，其中lib32bz2-1.0已废弃，应使用libbz2-1.0:i386代替，然后执行studio.sh启动程序

18) 关闭ssh登录时MOTD (message of the day)，把/etc/update-motd.d下的00-10-90-开头的三个文件重命个名，并编辑/etc/ssh/sshd_config文件设置PrintLastLogin为no

20) Windows下安装Git可以获得类Linux环境(babun更专业，适用于Windows的Linux子系统最专业)，安装TDM-GCC-64可以获得gcc和make，安装opensshd可获得ssh server

21) 加速Android开发者文档sdk/docs/index.html离线查看速度，找到火狐浏览器菜单中的“开发者”选项，并勾选“脱机工作”即可

22) 欢迎界面/登录界面/greeter screen隐藏用户：在/var/lib/AccountService/users目录下新建文件，以要隐藏的用户名命名该文件，并写入以下两行，注意大小写
```
[User]
SystemAccount=true
```

23) Samba来源于windows的SMB协议，表示server message block，微软后将其改名为cifs，即通用网络文件系统Common Internet File System

SMB协议基于IBM的NETBIOS协议，即网络基本输入输出协议，后者可以运行在多种传输协议上，如NETBIOS over TCP/IP

```bash
smbclient -L server 查看
smbclient //server/file 连接
#编辑文件/etc/samba/smb.conf可设置共享，应该用smbpasswd为samba设置单独密码
sudo smbpasswd -a existing_linux_user
sudo service smbd restart 使生效
```
24) Linux命令行挂载smb共享
```
#回车要求输入密码
#其中-o为option的缩写，挂载点必须存在，挂载之后里面的内容无法使用，卸载后恢复可用
sudo mount -t cifs //server <mount-point> -o user=xxx[,pass=xxx]
sudo umonut <mount-point>
```


# 注意事项

1)「权限」无小事

2) 图形界面文件浏览器想要使用root权限，可以命令行执行sudo nautilus

3) 虚拟机要加载iso或usb等新设备，有时需要手动连接，可以通过虚拟机-->可移动设备选择相应类型的设备，也可通过点击标题栏/状态栏中的图标来连接

4) 虚拟机要想从U盘启动，应该将U盘添加为物理硬盘

5) vmware网络连接模式有
1. 虚拟网络vmnet子网
2. 桥接vmnet0网桥，采用之，若桥接连不上外网可临时改用NAT
3. nat网络地址转换：包括端口转发(隧道)和端口映射vmnet8
4. host-only：vmnet1


6) 连接vmware共享虚拟机时，无法拖拽文件，也无法访问共享文件夹，可以mount挂载本地windows共享文件夹，或connect to server直接通过url访问

7) EXT4格式可能会有问题，可改用EXT3比较好

8) 分区设为主分区还是逻辑分区没有太大关系，只需要注意一块硬盘主分区加扩展分区最多只能有四个，其中扩展分区最多一个，且不可直接格式化存储数据，需要将其划分为多个逻辑分区，逻辑分区设备文件名从sda5开始，分区指的是以柱面为单位的连续磁盘空间

9) Ubuntu17开始将显示管理器从lightDM换成了GDM，相应地，登录界面的配置文件也不一样了

10) 在recovery维护模式下，根目录默认挂载为只读，所以要mount -o remount,rw /重新挂载为可读写才可进行操作

11) Windows配置开机启动虚拟机  
运行 > gpedit.msc > 用户配置 > windows设置 > 脚本(登录/注销)
```
#开机脚本如下
:: vmrun.bat
:: vmware installation dir should be added to the PATH variable
vmrun -T ws start "**/Ubuntu16.04.vmx" nogui > nul
```

12) 磁盘的第一个扇区(512字节，目前扇区的大小主要有512Bytes与4K两种格式)存放的是MBR(里面存放的bootloader[Grub]在安装操作系统时装入, 446)、分区表(64)和标志位(2)，标志位为空表示未分区，磁盘将不可用

13) MBR分区格式无法处理大于2.2TB的磁盘，于是GPT磁盘分区表应运而生

14) bootloader既可以安装在MBR中，也可以安装在每个分区的启动扇区中。并且，bootloader认识内核的同时也认识其他bootloader，MBR中的bootloader既可以直接载入内核，也可以将控制权移交给其他bootloader。

15) 先安装windows再安装linux的逻辑在于: linux在安装时可以选择将bootloader安装在MBR或其他分区的启动扇区，而windows在安装时将直接覆盖MBR和所在分区的启动扇区，即windows的bootloader默认不具有转交开机管理功能给其他loader的功能。

16) BIOS是硬件自带的，bootloader是安装操作系统时装到磁盘MBR或启动扇区上的，bootloader的主要功能是提供菜单+载入内核+将开机管理功能转交给其他bootloader，当然最终是为了载入内核

17) 双系统造成的windows时间错乱
```bash
#先在ubuntu下更新一下时间，确保时间无误：
sudo apt install ntpdate
sudo ntpdate time.windows.com
#然后将时间更新到硬件上
sudo hwclock --localtime --systohc
```
