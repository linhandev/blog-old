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

Arch是一个十分干净简洁的Linux发行版，日常使用不吃硬件十分流畅。采用滚动更新方式，安装之后更新就行，没有类似重装的升级，很适合实验室这种多人共用机器的场景。Arch的社区可能是一众Linux发行版中最好的。Arch Wiki基本能解答所有系统相关的问题，里面的内容甚至一些基于Arch发行版用户(比如Manjaro)都用的上。AUR(Arch User Respository)提供了大量的软件安装脚本，基本上装所有的东西都只需要一行命令。总结起来就是简单且强大。

所有折叠的块都是可选步骤，主要有ssh连接，mdadm raid和btrfs文件系统，不需要的话直接跳过。

<details>
<summary>折叠块长这样</summary>
  折叠起来的都是可选步骤，不需要直接跳过
</details>

# 硬件需求
- CPU：x86架构，绝大多数电脑都是
- RAM：512M以上
- 硬盘：2G以上
- 网：插网线比较简单，wifi也可
- U盘：做虚拟机不需要。8G肯定够用，镜像不到1G。读写速度主要影响做启动盘的时间，安装过程联网下载比较多，从U盘中读写比较少

# iso
[Arch官方镜像列表](https://archlinux.org/download/)里有所有的iso下载地址，国内从[清华源](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/2022.02.01/)下一般比较快。文件名是 `archlinux-年月日-x86_64.iso`，蓝色那行

![image](https://user-images.githubusercontent.com/29757093/155031388-1a5cadcc-135c-4831-bb24-53e2defbbf89.png)

做虚拟机不需要做启动盘，直接用iso。在物理机器上装，[Etcher](https://www.balena.io/etcher/)非常适合做任何Linux Distro的安装盘。使用简单，一般一次成功。选盘的时候注意，别选错了！！！否则可能把正在用的盘格掉。Etcher一般会把不像U盘的设备折叠起来，仔细看一眼肯定不至于选错。即使手上有之前的启动盘也建议做一个新的，Arch更新比较频繁，用旧的启动盘可能会出一些软件兼容的小问题，用最新的镜像可以避免麻烦。

# 开始安装
插入做好的启动盘，重启电脑选择U盘作为启动媒体，一般是按F1，F2, F5，F10，F12中的一个键。启动后选择Arch Linux install medium。

![image](https://user-images.githubusercontent.com/29757093/139161890-baedb7c9-3080-4c44-bd33-9d514bbe4ce3.png)

之后会进到一个命令行，开头的提示符是root@archiso，如下

![image](https://user-images.githubusercontent.com/29757093/139161941-9e1e6abc-4e50-4797-b15d-8216001e2866.png)

# 键位
国内的键盘一般都不需要改键位，如果需要可以参考[官方教程](https://wiki.archlinux.org/title/installation_guide#Set_the_console_keyboard_layout)

## 联网
安装过程中需要联网下软件。可以ping一个网站检查网络连接
```shell
ping baidu.com
# ctrl c 停止
```
如果看到 Name or service not know之类的报错就是目前没网。

![image](https://user-images.githubusercontent.com/29757093/139162328-17fe7acf-ddf2-4c69-a537-e9afaba1e19d.png)

直接插网线最简单，不需要进行额外操作。连wifi Arch推荐用[iwctl](https://wiki.archlinux.org/title/Iwd#Connect_to_a_network)。
```shell
iwctl device list # 列出所有网络接口
ip link set [网络接口] up # 我的叫 wlan0。先打开硬件，关闭硬件是down
iwctl # 进入iwctl
device list # 列出所有网络接口，一般会有一个lo是环回的，不是这个。需要用的设备大概叫wlan0
station [网络接口] scan
station [网络接口] get-networks # 获取所有wifi
station [网络接口] connect [wifi name] # 之后输密码
quit # 退出iwctl
```
连接完成重新ping一下，这时候应该看到能ping通。

![image](https://user-images.githubusercontent.com/29757093/139162642-68f2027f-d4cb-4e5f-9689-c36443c51323.png)

连接8021x校园网[1](https://unix.stackexchange.com/questions/145366/how-to-connect-to-an-802-1x-wireless-network-via-nmcli/334675) [2](https://www.reddit.com/r/archlinux/comments/pb3r0f/cannot_connect_to_college_wifi_using/)

## ssh
<details>
  <summary>可选步骤。用另一台电脑ssh到正在装的电脑上可以复制命令，方便一点</summary>

启动盘的live系统不能复制，一些命令手打比较麻烦。可以考虑用另一台机器ssh到要安装的机器上，方便一点。

```shell
pacman -Syy openssh # 安装ssh
systemctl start sshd # 启动ssh服务
passwd root # 给root设置密码
ip a # 查看机器ip

# 在另一台机器上
ssh root@[上面ip a看到的ip]
```
</details>

## 校准时间
```shell
timedatectl list-timezones # 显示所有时区，按q退出
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp true # 开启联网时间校准
```

## 硬盘分区
硬盘上创建了文件系统才能用。首先检查电脑是不是用了uefi
```shell
ls /sys/firmware/efi/efivars
```
如果说没有这个路径那就是没用uefi，下面分区的时候跳过uefi分区的部分。如果像下图一样出了一堆文件就是用了uefi，需要做一个uefi分区。

![image](https://user-images.githubusercontent.com/29757093/155045718-cd58db21-29a2-404e-8d9c-6da3589d608a.png)

除了uefi分区至少还需要一个root分区，一般还会做一个swap。一些特殊用途的系统比如服务器可能/srv或者/var下会存巨多的文件，这样可以给这个路径单开一个分区放到一个比较大的盘上，这个过程和做root分区是相同的。这个教程就做三个分区：uefi，root和swap。

```shell
lsblk # 查看机器硬盘情况
```

![image](https://user-images.githubusercontent.com/29757093/153557189-0d2d5652-8e60-46ba-b131-30a88af07957.png)

这里sda是U盘启动盘，nvme0n1和nvme1n1是ssd。

进入gdisk开始创建分区
```shell
gdisk /dev/[盘号] # 注意这块就写到盘，不要写到分区p1p2这种的
```

![image](https://user-images.githubusercontent.com/29757093/153557435-8e429654-fb00-438a-a823-20fa5acae7cb.png)

```shell
d # 之后打一个数字分区号删除它，多次执行直到提示没有分区
```

![image](https://user-images.githubusercontent.com/29757093/153557471-6bd00602-e2d6-411b-8080-bf5704277e4b.png)

首先做efi分区，如果上面没有uefi就跳过这段，直接创建下面的主分区
```shell
n # 创建分区
# 回车，默认分区号
# 回车，默认起始位置
+512M # 大小512M
ef00 # 修改分区类型为efi分区
```

![image](https://user-images.githubusercontent.com/29757093/153557577-b1c4017e-2ee4-4de0-bbaf-d70669d8b47f.png)


创建主分区
```shell
n # 创建新分区
# 回车，默认分区号
# 回车，默认起始位置
-8G # 这个8G就是给swap留下的大小，一般swap跟内存大小相同。如果有两块盘可以都做一个swap分区，分别内存一半大小
# 回车，分区类型默认就是Linux文件系统
```

![image](https://user-images.githubusercontent.com/29757093/153557905-270aafcc-1db9-4c6e-9831-3f16b2303940.png)

创建swap
```shell
n
# 回车，默认分区号
# 回车，默认起始位置
# 回车，直接占满剩下的容量
8200 # Linux swap
```
![image](https://user-images.githubusercontent.com/29757093/153557958-ae7e3ef2-4779-41fb-9842-fc2018b15c25.png)

分区创建完成，看一下目前磁盘情况，应该有三个分区，类型分别是EFI，Linux和Linux swap。
```shell
p
```

![image](https://user-images.githubusercontent.com/29757093/153558007-fab52bfa-32a7-40b5-b5a1-850bbf366157.png)

如果没问题就把修改写入磁盘
```shell
w # 确认无误后写入
Y # 确认
lsblk # 再次查看分区情况
```

![image](https://user-images.githubusercontent.com/29757093/153558144-8da25103-5029-4eac-973d-b8143aa35641.png)

之后需要在分区上创建文件系统
```shell
mkfs.fat -F32 /dev/[uefi分区] # 没有uefi分区跳过这行
mkswap /dev/[swap分区]
swapon /dev/[swap分区] 
# swapon 也可以写多个swap分区，比如 swapon /dev/nvme0n1p2 /dev/nvme1n1p3，这样多个swap分区应该会像raid 0一样做stripping加快速度
```
主分区文件系统有三种选择，绝大多数情况下最简单的ext4是最合适的，跑下面这一行之后直接到[安装Arch](#安装arch)一节就可以。
```shell
mkfs.ext4 /dev/[主分区]
```
如果想提升一点读写速度可以做[软件raid](#raid)，如果想要snapshot功能可以用[btrfs](#btrfs)文件系统。

## Raid
<details>
  <summary>可选步骤。软件Raid可以提升一些读写速度</summary>

之前在搜教程的时候看到一个Arch + Raid 0经验贴下面的的[评论](https://forum.level1techs.com/t/arch-linux-install-with-2-nvmes-in-raid-0/147268/2)，笑了一下午。必须放在这 /笑哭

![image](https://user-images.githubusercontent.com/29757093/152065902-cb1b40ca-3005-48c9-9415-0cd39a9e38f4.png)

个人在存储技术方面可以说没有任何经验，下面的背景部分只是记录一些自己在调研过程中的理解和想法。

基础的软件Raid大概有两条路线，一种是基本的ext4文件系统+软件Raid+逻辑卷管理，另一种是直接用zfs，btfs这种带Raid支持的文件系统。就我的在2块SSD的笔记本上加快读写速度的场景下似乎第一种更合适。

网上冲浪的过程中感觉btfs风评差很多，在Raid这种追求持续在线和稳定的用例下开发团队似乎并不重视软件质量。评价用btfs不是会不会丢数据的问题，只是什么时候丢数据。zfs功能很多，除了自带Raid以外copy on write带来的灵活创建备份点和快速格式化很大的存储听起来很有用。不过不是做NAS盘比较小ext4还能应付，我也没有备份系统的习惯最多备份文件，所以功能上二者没有决定性的差别。而且因为开源协议的问题zfs不能合进linux内核，自己做iso比较耗时间。综上选了第一条路线。

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
</details>

## btrfs
<details>
  <summary>可选步骤。Btrfs提供raid和快照功能，做起来比较复杂不推荐新手用</summary>

  btrfs和ext4一样是一个文件系统，负责管理存在盘上的文件。btrfs和zfs类似，在文件系统级别融合了传统解决方案ext4+软件Raid+逻辑卷管理的大部分功能。主要的优点是提供快速和不怎么占额外空间的snapshot。和zfs相比btrfs风评差很多，主要是因为bug比较多可能丢数据，而且开发者社区貌似赶不上zfs。但是zfs因为开源协议冲突不能合入linux内核，安装过程比btrfs麻烦一些。

  上一节已经做好了uefi和swap分区，root分区也创建了，从mkfs开始。类似逻辑卷，btrfs的文件系统可以跨盘，详情参考[btrfs wiki](https://btrfs.wiki.kernel.org/index.php/Using_Btrfs_with_Multiple_Devices)，下面是官方给的一些常用例子

  ```shell
  # Create a filesystem across four drives (metadata mirrored, linear data allocation)
  mkfs.btrfs -d single /dev/sdb /dev/sdc /dev/sdd /dev/sde # 简单跨盘，不raid

  # Stripe the data without mirroring, metadata are mirrored
  mkfs.btrfs -d raid0 /dev/sdb /dev/sdc # 相当于raid0

  # Use raid10 for both data and metadata
  mkfs.btrfs -m raid10 -d raid10 /dev/sdb /dev/sdc /dev/sdd /dev/sde

  # Don't duplicate metadata on a single drive (default on single SSDs)
  mkfs.btrfs -m single /dev/sdb
  ```

  比如我做的两块盘raid
  ```shell
  mkfs.btrfs -f -d raid0 /dev/nvme0n1p1 /dev/nvme1n1p2
  ```
  ![image](https://user-images.githubusercontent.com/29757093/153563174-7aba4e07-390e-451e-b607-33e1355a97c9.png)

  挂载btrfs分区，创建子卷

  ```shell
  mount /dev/[btrfs的任意一个分区] /mnt
  btrfs su cr /mnt/@root
  btrfs su cr /mnt/@home
  btrfs su cr /mnt/@var # 一般放可变长度的文件，比如log，临时cache和数据库
  btrfs su cr /mnt/@srv # web服务器和ftp文件
  btrfs su cr /mnt/@opt # 第三方软件
  btrfs su cr /mnt/@tmp # 临时文件和cache
  btrfs su cr /mnt/@swap # swap文件推荐放进单独的子卷
  btrfs su cr /mnt/@.snapshots
  ```

  挂载子卷
  ```shell
  umount /mnt
  part_name=[btrfs的任意一个分区名字]
  mount -o noatime,compress=lzo,space_cache=v2,subvol=@root /dev/${part_name} /mnt
  mkdir /mnt/{home,var,srv,opt,tmp,swap,.snapshots}
  mount -o noatime,compress=lzo,space_cache=v2,subvol=@home /dev/${part_name} /mnt/home
  mount -o noatime,compress=lzo,space_cache=v2,subvol=@srv /dev/${part_name} /mnt/srv
  mount -o noatime,compress=lzo,space_cache=v2,subvol=@tmp /dev/${part_name} /mnt/tmp
  mount -o noatime,compress=lzo,space_cache=v2,subvol=@opt /dev/${part_name} /mnt/opt
  mount -o noatime,compress=lzo,space_cache=v2,subvol=@.snapshots /dev/${part_name} /mnt/.snapshots
  mount -o nodatacow,subvol=@swap /dev/${part_name} /mnt/swap
  mount -o nodatacow,subvol=@var /dev/${part_name} /mnt/var
  ```
  - noatime： 不写accesstime
  - compress： zlib最慢，压缩最率高；lzo最快，压缩率最低；zstd和zlib兼容，压缩率和速度适中，可以调压缩等级
  - space_cache=v2：目前已经是默认开启了，将文件系统中空闲的block地址放在缓存里，创建新文件的时候可以立即开始往里写
  - nodatacow：禁用cow，新数据直接覆盖

  ![image](https://user-images.githubusercontent.com/29757093/153567324-e98fb530-8e95-49ab-9517-575f71ff5032.png)

</details>

## 安装Arch

安装需要联网下载，选一个快的镜像可以节省很多时间
```shell
pacman -Syy # 更新pacman数据库
pacman -S reflector
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bk # 备份镜像列表
reflector -c "CN" -l 20 -n 10 --sort rate --save /etc/pacman.d/mirrorlist
```


```shell
mount /dev/[主分区] /mnt # 挂载主分区
pacstrap /mnt base linux linux-firmware linux-headers vim base-devel opendoas grub efibootmgr
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

<details>
  <summary>Raid</summary>
  
  把mdadm配置写入文件
  ```shell
  mdadm --detail --scan >> /mnt/etc/mdadm.conf
  ```  
  
</details>

<details>
  <summary>btrfs</summary>
  
  ```shell
  pacstrap /mnt btrfs-progs grub-btrfs
  ```
  
</details>

切换到新装好的系统
```shell
arch-chroot /mnt
```
<details>
  <summary>Raid</summary>
  在新系统里装mdadm，和修改一个配置文件。Raid所有配置完成，下面正常安装就行。
 
  ```shell
  pacman -S mdadm

  vim /etc/mkinitcpio.conf
  HOOKS=(base udev autodetect keyboard modconf block mdadm_udev filesystems fsck) # 在HOOKS这行添加 mdadm_udev

  mkinitcpio -p linux
  ```
  
</details>

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
pacman -S wpa_supplicant wireless_tools networkmanager network-manager-applet
pacman -S nm-connection-editor 
systemctl enable NetworkManager.service
systemctl disable dhcpd.service # 如果说dhcpd not found也没关系，目标就是把他关了
systemctl enable wpa_supplicant.service
```
这里就可以重启进入只有命令行的系统了，可以选择现在重启看一下前面的步骤是不是做的有问题，下面的步骤在进入系统后做，或者也可以不重启直接继续装。

## 添加用户
一般日常使用不会直接用root账户，创建一个用户帐户。
```shell
username=[用户名]
useradd -m ${username}
# 添加一行
echo "permit persist ${username} as root" >> /etc/doas.conf # 允许 用户名 作为root执行，persist是输入一次密码之后一段时间不用再输入
mv /usr/bin/sudo /usr/bin/sudo-bk
ln -s /usr/bin/doas /usr/bin/sudo
passwd ${username} # 设置新用户密码
```

<details>
  <summary>Btrfs snapshot</summary>

  ```shell
  exit
  reboot # snapper 默认需要dbus，重启比较方便
  pacman -S snapper
  umount /.snapshots
  rm -rf /.snapshots
  snapper -c root create-config /
  vim /etc/snapper/configs/root
  # ALLOW_USERS='[用户名]'
  # 最后的期限限制
  chmod a+rx /.snapshots

  systemctl start snapper-timeline.timer
  systemctl enable snapper-timeline.timer
  systemctl start snapper-cleanup.timer
  systemctl enable snapper-cleanup.timer
  systemctl start grub-btrfs.path
  systemctl enable grub-btrfs.path

  snapper -c root list
  snapper -c root create --description BeforeGui
  ```
</details>

## 桌面
### xfce4

```shell
pacman -S xorg
pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
systemctl enable lightdm
```

### KDE
```shell
pacman -S xorg plasma plasma-wayland-session kde-applications 
systemctl enable sddm.service
```
![image](https://user-images.githubusercontent.com/29757093/155061233-99de26b9-e02c-41e3-ae7d-a7c2050b0723.png)


有时候新安装桌面可能遇到登录循环的情况，开机后正常输入用户名密码，结果回车登录之后又回到输入密码界面。这种情况可能是因为没有 /home/[用户名] 目录或者用户没有这个目录的权限，创建试一下。
```shell
mkdir /home/[用户名] 
chown -R [用户名]:[用户名] /home/[用户名]
```

安装完成，重启进入系统
```shell
# 按 ctrl+D 退出 chroot
reboot
```

重启之后应该就能看到一个登陆界面，登陆进去看到桌面就是安装成功了！如果安装过程中有任何问题欢迎在下方留言。有关一些常用软件的安装在[下一篇文章](https://linhandev.github.io/posts/Arch-Apps/)中记录。

hwclock --systohc

[//]: # (swap btrfs: truncate -s 0 /swap/swapfile; chattr +C /swap/swapfile; btrfs property set /swap/swapfile compression none; dd if=/dev/zero of=/swap/swapfile bs=1G count=2 status=progress; chmod 600 /swap/swapfile; mkswap /swap/swapfile; swapon /swap/swapfile; vim /etc/fstab； /swap/swapfile none swap defaults 0 0 )

参考资料：

[官方安装教程](https://wiki.archlinux.org/title/installation_guide)

[Arch安装教程 (不带swap)](https://itsfoss.com/install-arch-linux/)

[安装教程 (带swap)](https://frontpagelinux.com/tutorials/how-to-install-arch-linux-installation-guide/)

[iwctl连接wifi](https://wiki.archlinux.org/title/Iwd)

[mdadm+arch](https://www.serveradminz.com/blog/installation-of-arch-linux-using-software-raid/)

[Arch Raid](https://wiki.archlinux.org/title/RAID)

[Arch Linux BTRFS Install](https://www.youtube.com/watch?v=7ituCCKXmMM&t=1143s&ab_channel=EF-LinuxMadeSimple)
