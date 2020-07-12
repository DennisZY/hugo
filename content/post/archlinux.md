---
title: "ArchLinux安装"
date: 2020-07-12T14:56:19+08:00
draft: false
tags: 
- linux
- archlinux
categories: 
- 折腾笔记
---

# 安装准备

iso: archlinux-2020.07.01-x86_64.iso

## 下载iso文件

- [腾讯云](https://mirrors.cloud.tencent.com/archlinux/iso/2020.07.01/archlinux-2020.07.01-x86_64.iso)
- [阿里云](https://mirrors.aliyun.com/archlinux/iso/2020.07.01/archlinux-2020.07.01-x86_64.iso)
- [清华源](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/2020.07.01/archlinux-2020.07.01-x86_64.iso)
- [中科大源](https://mirrors.ustc.edu.cn/archlinux/iso/2020.07.01/archlinux-2020.07.01-x86_64.iso)

你甚至可以检查一下iso完整性(doge)

## 写U盘

- [Etcher](https://www.balena.io/etcher/)

- [Ventoy](https://www.ventoy.net/)

## 连接网络

通过`iwd`连接无线网络，有线的可以跳过。输入`iwctl`进入交互提示符。

查看你机器的无线设备名

``` 
[iwd]# device list 
```

扫描无线网络，将device替换为你的设备名，下同

``` 
[iwd]# station device scan
```

获取你扫描到的网络

```
[iwd]# station device get-networks
```

连接网络，如果有密码会提示你输入密码

```
[iwd]# station device connect SSID
```

其他的使用方法，请浏览[iwd](https://wiki.archlinux.org/index.php/Iwd)

通过ping检查网络是否设置好

``` sh
ping archlinux.org
```

## 设置系统时间

```
timedatectl set-ntp true
```

检查设置情况

``` sh
timedatectl status
```

## 分区

通过cfdisk进行分区

分好区后要格式化，swap空间要开启

## 挂载分区

```
mount /dev/sdX1 /mnt
mkdir /mnt/efi
mount /dev/sdX3 /mnt/efi
```

# 安装

## 设置更新源

编辑/etc/pacman.d/mirrorlist，先注释掉里面的所有行，然后在文件的最顶端添加

``` 
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
Server = https://mirrors.cloud.tencent.com/archlinux/$repo/os/$arch
```

 更新软件包缓存：

```
sudo pacman -Syy
```

## 安装必要的软件包

``` sh
pacstrap /mnt base base-devel linux linux-firmware vim dhcpcd networkmanager man-db man-pages
```

如果你要lts的内核
``` sh
pacstrap /mnt base base-devel linux-lts linux-firmware vim dhcpcd networkmanager man-db man-pages
```

# 配置系统

## fstab

``` sh
genfstab -U /mnt >> /mnt/etc/fstab
```

## Chroot

从u盘系统转到硬盘中的新系统

``` sh
arch-chroot /mnt
```

## 时区

``` sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

## 本地化

编辑`/etc/locale.gen`，将`en_US.UTF-8`和`zh_CN.UTF-8`前的#去掉

``` sh
locale-gen
```

新建`/etc/locale.conf`,添加下面一行

```
LANG=en_US.UTF-8
```

## 网络设置

创建主机名文件

```
/etc/hostname
myhostname
```

添加hosts配置

```
/etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

## 设置root用户密码

``` 
passwd
```

## 设置开机自启NetworkManager

``` 
systemctl enable NetworkManager
```

## 设置开机自启dhcpcd

``` 
systemctl enable dhcpcd
```

## 安装CPU微码

自己根据自己的CPU安装对应的微码

``` sh
sudo pacman -S intel-ucode
sudo pacman -S amd-ucode
```

## 配置引导

我用的引导程序是GRUB，首先安装必要的软件包：

```text
pacman -S grub efibootmgr
```

这里详细介绍一下UEFI系统如何安装配置GRUB：

1. 首先使用以下命令安装到系统：

```text
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=ArchLinux
```

note: 因为我的EFI分区在`/efi`目录下，所以上述命令的`--efi-directory`参数就设置为`/efi` 2. 使用`grub-mkconfig`生成grub配置文件：`grub-mkconfig -o /boot/grub/grub.cfg`



到这里就可以重启进入系统

# 进一步安装

## 显示管理器

``` sh
pacman -S xorg
```

## 显卡驱动

### intel

```reStructuredText
pacman -S xf86-video-intel
```

### NVIDIA

``` sh
pacman -S nvidia nvidia-utils nvidia-prime
```

lts

``` sh
pacman -S nvidia-lts nvidia-utils nvidia-prime
```

## 触摸板驱动

``` sh
sudo pacman -S xf86-input-synaptics
```

## KDE

### sddm

``` sh
sudo pacman -S sddm
```

### Plasma桌面

``` sh
sudo pacman -S kde
```

### KDE全家桶

``` sh
sudo pacman -S kde-applications
```

### 开机自启sddm

```text
systemctl enable lightdm.service
```

## 添加用户

``` sh
useradd -m -G wheel "用户名"
```

修改`/etc/sudoers`，将%wheel前的#号去掉#

# 后续

装得差不多了，自己要什么软件再安装吧