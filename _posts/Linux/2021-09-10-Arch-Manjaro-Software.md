---
title: Manjaro/Arch常用软件
author: Lin Han
date: 2021-09-10
published: false
categories:
  - Linux
tags:
  - Arch
  - Manjaro
  - Software
  - Tutorial
---
# 网络
todo

# yay
yay是一个很好用的包管理工具。它一些具体的安装过程会调用pacman，命令行参数也基本一致，主要是多了AUR的支持。AUR里基本能找到所有常用的软件。
```shell
sudo pacman -Syyu # 全系统更新
sudo pacman -S git
git clone https://aur.archlinux.org/yay.git # 下载yay代码
cd yay
makepkg -si # 编译并安装
yay # 更新所有的包，测试安装

yay package_name # 如果是pacman是需要准确的包名，但是直接yay第一步是搜索，所以不需要准确
yay -S pkg_name # 和pacman一样是直接安装这个包
yay -R pkg_name # 删除一个包

sudo vi /etc/makepkg.conf
# 输入 /PKGEXT ，回车搜索
# 把最后面的后缀去掉，只留到.tar，这样本地编译的包不需要压缩直接安装，快很多

# 清除yay缓存
rm -rf ~/.cache/yay
```
如果安装的过程中报一些找不到或者缺少fakeroot，build utils之类的binary，是因为缺少一些依赖。
```shell
sudo pacman -S base-devel
```

# 下拉命令行
xfce4的命令行可以从屏幕最上面下拉，不用的时候收回去，这样terminal不会挤在打开的窗口中，比较方便。可以设置键盘快捷键唤起
```shell
xfce4-keyboard-settings
```
在快捷键设置tab中点击添加，执行命令输入
```shell
xfce4-terminal --drop-down
```
点下一步，录制快捷键，可以设置一个Alt+ESC或者Alt+`这种，一般不会和其他快捷键冲突。

这里可以给搜索并执行程序也设置一个快捷键，添加快捷键，命令为 *xfce4-appfinder*，快捷键可以设个Alt

# 浏览器
```shell
yay -S google-chrome
```

## 插件
https://www.downloadhelper.net/
yay -S vdhcoapp-bin

https://chrome.google.com/webstore/detail/dark-reader/eimadpbcbfnmbkopoojfekhnkhdbieeh?hl=en

# 声音
https://linuxhint.com/pulseaudio_arch_linux/
```shell
sudo pacman -S pulseaudio pavucontrol
```
todo autostart
## 降噪
麦克
https://fedoramagazine.org/real-time-noise-suppression-for-video-conferencing/
https://www.linuxuprising.com/2020/09/how-to-enable-echo-noise-cancellation.html
https://www.reddit.com/r/linux/comments/2yqfqp/just_found_that_pulseaudio_have_noise/
https://askubuntu.com/questions/18958/realtime-noise-removal-with-pulseaudio
https://www.linuxuprising.com/2021/02/noisetorch-is-real-time-microphone.html

Remove Background Noise Using Audacity and Kdenli
 https://www.youtube.com/watch?v=3nMkMn4--5w

# 蓝牙
https://www.jeremymorgan.com/tutorials/linux/how-to-bluetooth-arch-linux/
```shell
sudo pacman -S bluez bluez-utils blueman
sudo systemctl start bluetooth.service
sudo systemctl enable bluetooth.service
yay pulseaudio-bluetooth
```
打开bluetooth manager会自动在面板上添加一个widget。如果蓝牙是关闭状态搜索bluetooth adapter可以打开。

# 中文支持
如果看到中文都显示方块，是缺少字体。完整[中文字体列表](https://wiki.archlinux.org/title/Localization/Chinese#Fonts)
```shell
sudo pacman -S adobe-source-han-sans-cn-fonts
```
安装fcitx中文输入法
```shell
sudo pacman -Syu fcitx fcitx-googlepinyin fcitx-im fcitx-configtool
vim ~/.xinitrc
# 输入
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
搜索启动fcitx configuration，添加google pinyin输入法。之后登出用户并重新登录。


# ssh
```shell
sudo pacman -S openssh
```

# 邮件

# 解压

# 下载

# python

# shell

# 虚拟机
