---
title: Arch Linux安装
author: Lin Han
date: 2021-09-10
categories:
  - Linux
tags:
  - Arch
pin: true
---

Arch是一个十分干净简洁的Linux Distro，日常使用占用的硬件资源很少，低配的电脑上也几乎不会卡。滚动更新方式意味着安装之后只需要更新，不需要费劲升级系统版本。Arch的社区可能是一众Linux发行版中最好的，在Arch Wiki里基本能找到所有与系统相关问题的答案，Arch社区甚至对一些使用基于Arch的Linux发行版用户(比如Manjaro)来说都很有用。在AUR(Arch User Respository)的帮助下基本上安装所有的软件都可以只用一行代码。总结起来就是简单而且强大。

# 硬件需求
- RAM：512M以上
- 硬盘：2G以上
- 网：插网线比较简单，wifi也可
- u盘：虚拟机不需要。大概8G就够用，质量好的U盘安装会快很多

# iso
国内可以从[清华源](https://mirrors.tuna.tsinghua.edu.cn/)下载Arch的iso，速度比较快。或者可以从[Arch官方镜像列表](https://archlinux.org/download/)里挑一个下载。如果是做虚拟机不需要做启动盘，直接用iso就可以。如果在物理机器上装，[Etcher](https://www.balena.io/etcher/)非常适合做任何Linux Distro的安装盘，使用简单而且一般都是一次成功。选盘的时候注意别选错了，否则可能把正在用的电脑的盘格掉。即使手上有之前的安装盘也尽量做一个新的，之前遇到过用一个月之前的安装盘进行安装，出了一些软件兼容的小问题，用最新的镜像可以避免麻烦。

# 开始安装
插入做好的启动盘，重启电脑选择U盘作为启动媒体，一般是按F1，F2, F5，F10，F12中的一个键。启动后选择Arch Linux install medium。

![image](https://user-images.githubusercontent.com/29757093/139161890-baedb7c9-3080-4c44-bd33-9d514bbe4ce3.png)

之后会进到一个命令行，开头的提示符是root@archiso，如下

![image](https://user-images.githubusercontent.com/29757093/139161941-9e1e6abc-4e50-4797-b15d-8216001e2866.png)

## 联网
安装过程中需要联网下软件。可以ping一个网站检查网络连接
```shell
ping baidu.com
# ctrl c 停止
```
如果看到 Name or service not know之类的报错就是目前没网。

![image](https://user-images.githubusercontent.com/29757093/139162328-17fe7acf-ddf2-4c69-a537-e9afaba1e19d.png)

如果有网线直接插上是最简单的方法，不需要进行额外操作。连接wifi可以用[iwctl](https://wiki.archlinux.org/title/Iwd#Connect_to_a_network)。
```shell
iwctl device list
ip link set wlan0 up # 先打开硬件，关闭硬件是down
iwctl # 进入iwctl
device list # 列出所有网络设备，一般会有一个lo是环回的，不是这个。需要用的设备大概叫wlan0
station wlan0 scan
station wlan0 get-networks # 获取所有wifi
station wlan0 connect [wifi name] # 之后输密码
quit # 退出iwctl
```
连接完成重新ping一下，这时候应该看到能ping通。

连接8021x校园网[1](https://unix.stackexchange.com/questions/145366/how-to-connect-to-an-802-1x-wireless-network-via-nmcli/334675) [2](https://www.reddit.com/r/archlinux/comments/pb3r0f/cannot_connect_to_college_wifi_using/)

![image](https://user-images.githubusercontent.com/29757093/139162642-68f2027f-d4cb-4e5f-9689-c36443c51323.png)

命令行的live系统不能复制，可以考虑用另一台机器ssh到要安装的机器上，比较方便。
[//]: # (TODO: ssh安装)


[//]: # (TODO: 键位)

## Raid
可选步骤，可以直接跳到[下一部分](#磁盘分区)。

之前在搜教程的时候看到一个Arch + Raid 0经验贴下面的的[评论](https://forum.level1techs.com/t/arch-linux-install-with-2-nvmes-in-raid-0/147268/2)，笑了一下午。必须放在这 /笑哭

![image](https://user-images.githubusercontent.com/29757093/152065902-cb1b40ca-3005-48c9-9415-0cd39a9e38f4.png)

个人在存储技术方面可以说没有任何经验，下面的背景部分只是记录一些自己在调研过程中的理解和想法。

基础的软件Raid大概有两条路线，一种是基本的ext4文件系统+软件Raid+逻辑卷管理，另一种是直接用zfs，btfs这种带Raid支持的文件系统。就我的在2块SSD的笔记本上加快读写速度的场景下似乎第一种更合适。

网上冲浪的过程中感觉btfs风评极差，在Raid这种追求持续在线和稳定的用例下开发团队似乎并不重视软件质量。评价用btfs不是会不会丢数据的问题，只是什么时候丢数据。zfs功能很多，除了自带Raid以外copy on write带来的灵活创建备份点和快速格式化很大的存储听起来很有用。不过不是做NAS盘比较小ext4还能应付，我也没有备份系统的习惯最多备份文件，所以功能上二者没有决定性的差别。而且因为开源协议的问题zfs不能合进linux内核，自己做iso比较耗时间。综上选了第一条路线。

Raid的实现分为三大类：硬件Raid，软件Raid和主板Raid(fakeraid)。硬Raid用专门的芯片和独立于uefi的固件实现，性能最好，对主板固件和操作系统来说一个硬Raid的阵列就是一块盘。软件Raid和fakeraid都依赖CPU进行运算，理论上性能差别不是很大，一般没有硬件Raid就会采用软件Raid，fakeraid没人推荐。Raid有多种级别，具体可以参考[这篇](https://www.prepressure.com/library/technology/raid)。笔记本大概就是Raid 0 和 Raid 1,主要着眼提速，Raid 0 用一倍的故障率换速度，Raid 1 用一半的空间换容错。我做的是Raid 0不过其他级别流程上区别不大。

做Raid要分区，因为就算一个厂商一个型号的盘大小也会有一些不同。如果阵列有容错，换盘的时候软件Raid要求换进来的盘和之前的盘大小完全相同。如果直接用整块盘做Raid，之前的盘还偏大就很不巧了。

做Arch至少要两个，一般有三个分区，分别是uefi，主分区和可选的swap。软件Raid依赖操作系统，而uefi分区在进操作系统之前就要用，所以这个分区要么不Raid要么Raid 1。linux支持多个swap，如果有两个swap在两个盘上默认就会用类似Raid 0的方式读写，所以swap也没必要放进Raid。这样就只需要给系统分区做Raid。如果继续对系统分区细分，只想对存数据的部分进行Raid安装过程和不配Raid完全相同。系统做完之后配Raid挂载就可以。

废话结束，综上我的两块SSD Raid 0分区布局如下
- SSD 1
  - UEFI分区：512M
  - 系统分区：剩下的空间取个整数
  - swap分区： 1/2 swap大小
- SSD 2
  - 系统分区：和SSD 1上的系统分区一样大
  - swap分区： 1/2 swap大小

格盘的细节参考下一节，记一下mdadm的内容

```shell
# 删除旧记录
mdadm --misc --zero-superblock /dev/drive

# 创建Raid 0
mdadm --create /dev/md0 --level=stripe --raid-devices=2 /dev/drive[1-2]
# 或者盘分开写
mdadm --create /dev/md0 --level=stripe --raid-devices=2 /dev/drive1 /dev/drive2

# 查看mdadm运行状态，raid细节
cat /proc/mdstat
mdadm -E /dev/drive[1-2]
mdadm --detail /dev/md0

```

Raid做完之后如下步骤，替换成自己的设备文件
1. 在分区1上 `mkfs.fat -F32 /dev/[uefi分区]`
2. 在分区3和5上分别 `mkswap /dev/[swap分区]`
3. `swapon /dev/[SSD 1上的swap分区] /dev/[SSD 2上的swap分区]`
4. 在md0里创建一个ext4分区，`mkfs.ext4 /dev/raid 0里的分区`，后面mount的时候也mount这个分区

Raid部分已经跑起来了，在挂载主分区之后，arch-chroot之前和之后还有几行命令需要执行。

## 磁盘分区
[//]: # (TODO: gparted)
首先看看电脑是不是用了uefi，如果开了的话需要多做一个分区。
```shell
ls /sys/firmware/efi/efivars
```
如果没有这个路径那就是没用uefi，下面分区的时候跳过uefi分区的部分。如果出了一堆文件就是用了uefi。

根据电脑的用途可以采用不同的分区方案，但是至少是需要一个root分区，一般会做一个swap。一些特殊用途的linux比如邮件服务器可能一些路径下会存巨多的文件，这样可以给这个路径单开一个分区放到一个比较大的盘上。这里就做一个uefi，root和swap。
```shell
lsblk # 查看机器存储硬件情况
```

![image](https://user-images.githubusercontent.com/29757093/139162981-83ea94a6-ea59-4571-8e4c-5f35f7e7c6f0.png)

可以看出我的机器有两块SSD

```shell
fdisk -l # 同上，信息一般多一些
```

![image](https://user-images.githubusercontent.com/29757093/139162961-f746574c-ee3f-4f98-b6c1-ad51bc9ca8d8.png)

进入fdisk开始创建分区
```shell
fdisk /dev/盘号 # 注意这块就写到盘，不要写到分区p1p2这种的
```

![image](https://user-images.githubusercontent.com/29757093/139163236-0ff6a616-fcab-444a-9c69-5615dc705eaa.png)

```shell
d # 删除一个分区，多次执行直到提示没有分区
```

![image](https://user-images.githubusercontent.com/29757093/139163286-48ca983a-e714-4b17-8103-646a401f7ec4.png)

首先做efi分区，如果上面没有uefi就跳过这段，直接创建下面的主分区
```shell
n # 创建分区
p # 主分区
# 回车，默认分区号
# 默认起始位置
+512M # 大小512M
t # 修改分区类型
L # 查看所有类型，应该有一个uefi
uefi
```

![image](https://user-images.githubusercontent.com/29757093/139163361-e6dfc02c-c2ba-448b-bc9c-23f83e128945.png)

![image](https://user-images.githubusercontent.com/29757093/139163393-61d88194-ae49-4411-be5e-2297b573409c.png)


创建主分区
```shell
n
p
# 回车，默认分区号
# 回车，默认起始位置
-8G # 这个8G就是给swap留下的大小，根据自己的硬件情况调整
```

![image](https://user-images.githubusercontent.com/29757093/139163458-8e811f34-0d6d-428c-a793-169d05080ae0.png)

创建swap
```shell
n
p
# 回车，默认分区号
# 回车，默认起始位置
# 回车，直接占满剩下的容量
t # 修改分区类型
# 回车，选择最后一个分区
L # 列出所有，应该有一个swap
swap
```

![image](https://user-images.githubusercontent.com/29757093/139163567-9b19e3a7-d129-407f-ac4f-cda871d0b87f.png)

分区创建完成，看一下目前磁盘情况，应该有三个分区，类型分别是EFI，Linux和Linux swap。
```shell
p
```

![image](https://user-images.githubusercontent.com/29757093/139163576-cb113d26-fbb4-480e-a1e6-937450e9a766.png)

如果没问题就把修改写入磁盘
```shell
w # 确认无误后写入
lsblk # 再次查看分区情况
```

之后需要在分区上创建文件系统
```shell
# 没有uefi分区跳过这行
mkfs.fat -F32 /dev/[uefi分区]
mkfs.ext4 /dev/[主分区]
mkswap /dev/[swap分区]
swapon /dev/[swap分区]
```

下一步需要联网安装软件，选一个快的镜像可以节省很多时间
```shell
pacman -Syy # 更新pacman数据库
pacman -S reflector
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bk # 备份镜像列表
reflector -c "CN" -l 20 -n 10 --sort rate --save /etc/pacman.d/mirrorlist
```

## 安装Arch
```shell
mount /dev/[主分区] /mnt
pacstrap /mnt base linux linux-firmware vim sudo base-devel # 往主分区里安装，几分钟
genfstab -U /mnt >> /mnt/etc/fstab
```

不做Raid跳过下面这行，把mdadm配置写入文件
```shell
mdadm --detail --scan >> /mnt/etc/mdadm.conf
```

```shell
arch-chroot /mnt
```

依旧是不做Raid跳过，在新系统里装mdadm，和修改一个配置文件。Raid所有配置完成，下面正常安装就行。
```shell
pacman -S mdadm

vim /etc/mkinitcpio.conf
HOOKS=(base udev autodetect keyboard modconf block mdadm_udev filesystems fsck) # 在HOOKS这行添加 mdadm_udev

mkinitcpio -p linux
```

## 语言
```shell
vim /etc/locale.gen
# 按 / 进入查找，输入en_US，回车下一条结果
# 找到 en_US.UTF-8 UTF-8 这一行
# 按i编辑，删除前面的 #

# 按两次esc进入命令模式
# 按 / 进入查找，输入zh_CN，回车下一条结果
# 找到 zh_CN.UTF-8 UTF-8 这一行
# 按i编辑，删除前面的 #

# 两下esc，输入:wq保存退出
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
```
![image](https://user-images.githubusercontent.com/29757093/139164535-05d4f355-2975-4df0-a53d-99a438fee4fa.png)

## 设置网络
```shell
vim /etc/hostname
# 按 i 编辑，输入一个hostname
# 两下esc，输入 :wq 保存并退出
```

![image](https://user-images.githubusercontent.com/29757093/139164588-c485c164-7398-4d73-8719-e9805efd8d6b.png)

```shell
vim /etc/hosts
# 按 i 编辑，输入以下内容
127.0.0.1 localhost
::1 localhost
127.0.0.1 [刚才设置的的hostname]
# 两下esc，输入 :wq 保存并退出
```

![image](https://user-images.githubusercontent.com/29757093/139164725-ee7d9110-04f6-4170-a647-3d6d456eeeb5.png)

## 安装grub
uefi系统
```shell
pacman -S grub efibootmgr
mkdir /boot/efi
mount /dev/[uefi分区] /boot/efi
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi # 注意那个x是小写的
grub-mkconfig -o /boot/grub/grub.cfg
```
非uefi系统
```shell
pacman -S grub
grub-install /dev/[主分区]
grub-mkconfig -o /boot/grub/grub.cfg
```

## 设置密码
```shell
passwd ＃ 设置密码
```

## 网络
前面设置的网络连接只在这次安装过程中生效，还需要给刚装好的系统装联网软件。下面只写基本的连接wifi的部分，DSL，移动网络之类的连接可以参考[这篇详细教程](https://linuxhint.com/arch_linux_network_manager/)
```shell
pacman -S wpa_supplicant wireless_tools networkmanager
pacman -S nm-connection-editor network-manager-applet
systemctl enable NetworkManager.service
systemctl disable dhcpd.service # 如果说dhcpd not found也没关系，目标就是把他关了
systemctl enable wpa_supplicant.service
```
这里就可以重启进入只有命令行的系统了，可以选择现在重启看一下前面的步骤是不是做的有问题，下面的步骤在进入系统后做，或者也可以不重启直接继续装。

## 校准时间
```shell
timedatectl list-timezones # 显示所有时区，按q退出
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp true # 开启联网时间校准
```

## 桌面
xfce4桌面
```shell
pacman -S xorg
pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
systemctl enable lightdm
```

## 添加用户
一般日常使用不会直接用root账户，创建一个用户帐户。
```shell
useradd [用户名]
passwd [用户名] # 设置新用户密码
vim /etc/sudoers
# 添加一行
[用户名] ALL=(ALL) ALL # 允许用户使用sudo
# :wq! 保存并退出
ls -lah /home/[用户名] # 检查是不是创建了用户的home目录，如果没有或者用户没有权限访问这个目录会在登录的时候登录成功，但是仍然返回登录界面
mkdir /home/[用户名]
chown -R [用户名]:[用户名] /home/[用户名]
```

安装完成，重启进入系统
```shell
# 按 ctrl+D 退出 chroot
reboot
```
重启之后应该就能看到一个登陆界面，能登陆进去就是安装成功了！如果安装过程中有任何问题欢迎在下方留言。有关一些常用软件的安装在[下一篇文章](https://linhandev.github.io/posts/Arch-Apps/)中记录。

参考资料：

[官方安装教程](https://wiki.archlinux.org/title/installation_guide)

[Arch安装教程 (不带swap)](https://itsfoss.com/install-arch-linux/)

[安装教程 (带swap)](https://frontpagelinux.com/tutorials/how-to-install-arch-linux-installation-guide/)

[iwctl连接wifi](https://wiki.archlinux.org/title/Iwd)

[mdadm+arch](https://www.serveradminz.com/blog/installation-of-arch-linux-using-software-raid/)

[Arch Raid](https://wiki.archlinux.org/title/RAID)