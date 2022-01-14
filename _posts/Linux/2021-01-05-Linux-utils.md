---
title: 通用Linux功能
author: Lin Han
date: 2021-01-05 22:02 +8
categories:
  - Linux
tags:
  - Shell
  - Git
  - Archive
  - TODO
---

## 查看系统资源占用
linux有很多manylinux的工具可以查看系统中资源的使用情况
### 网络
https://www.binarytides.com/linux-commands-monitor-network/
nload

### CPU
top

### GPU
nvidia-smi

### 内存

### 磁盘
du
df

## Git
git以行为单位追踪更改，比如在一行文本里改了一个字，那么git看作删掉这一行之后添加上修改过的那一行

相关完整教程 [Github Git Handbook](https://guides.github.com/introduction/git-handbook/)

## 重要文件
- .git：git的历史记录都放在这里，这个文件夹删掉了，这个项目copy中的历史就没了，这个路径也就不在是一个repo
- .gitignore：让git不跟踪一些文件，比如py代码的package都会生成一个目录，但是没必要推到服务器上，所以就可以忽略这些文件
- README.md：所有其他开发者看到这个项目一般都会先读readme
- LICENSE：项目采用的协议，如果不加可能一些注意版权的人会不敢用你的项目

- repo：一个包含.git的目录
- commit：对所有代码在不同时间的snapshot历史
- branch：历史中的一根链

## 基础用法

```shell
git config --global user.name "Lin Han"
git config --global user.email "linhandev@qq.com" # 这个只是git在历史文件里记的东西，邮箱写什么Github都推的上去，但是最好和自己的Github账户保持一致。之前遇到过在百度pr需要签协议，因为一个commit邮箱写的不一样需要签两份贡献协议

git init # 初始化一个repo
git add * # 跟踪当前路径下所有文件
git commit -m "Initial Commit" # 创建一个commit记录

git clone # 从服务器上拉一个项目下来

git status # 查看当前修改，commit的状态
git rm --cached file # 加cache是从最后一条commit中删除file的记录
git rm file # 从commit记录和文件系统中删除一个文件

git branch # 创建一个branch
git checkout # 切换到一个branch，在当前branch的修改如果commit了就不会带到另一个branch，否则会跟着过去
# 在branch a中添加文件，branch b中也会多出这个文件；在branch a中删除文件，brnach b中不会跟着删除
git branch -m <new name> # 对一个branch改名
git branch -d <local branch name>
git push origin --delete <remove branch name>

git merge
#TODO: 研究在merge的时候压缩所有commit

git remote add origin <服务器项目网址> # 添加服务器上项目的repo
git push --set-upstream origin <remote-branch> # 推到origin的一个branch里

git config credential.helper store
git config --global http.proxy 'socks5://127.0.0.1:2333' # 设置proxy，加速下载
```

## 删除记录
https://itextpdf.com/en/blog/technical-notes/how-completely-remove-file-git-repository
[删除所有commit](https://gist.github.com/stephenhardy/5470814)

## 修改branch名

https://linuxize.com/post/how-to-rename-local-and-remote-git-branch/

## 压缩
### tar
tar是linux用来创建备份文件的命令，好像zip至少也要1级压缩，可能不能只打包，对于需要重复解包的数据比较快。应该是能保存linux的权限，这对一些web应用来说很方便。 详细的命令菜鸟上有介绍。
```shell
# 创建一个包
# c 创建
# z 压缩
# v verbos,显示过程
# f 备份文件
tar -czvf name.tar.gz file

# 追加文件
# r 追加文件,不写x应该也会跟着包的压缩方式一起压缩
tar -rvf name.tar file

# 取出文件
# x 解压
# 如果写文件名是就取一些文件, 不写文件名就都拿出来
# -C 指定解压目标路径
tar -xvf name.tar file
```

### gzip
感觉gzip是用着最方便的，而且像平时做医学影响，nii和nii.gz都可以直接读。
```shell
# 分别压缩目录下没一个文件成一个 gz
gzip *
# 解压一个文件或者一个目录下所有文件
gunzip filename
```

### zip
zip感觉一般如果想让windows也能用才用到这个。zip用起来，没有gzip方便，如果一个目录下每个文件想单独打一个包需要写for。
```shell
zip archive.zip filename
zip -r archive.zip folder

# -o 如果目标文件存在覆盖
# -n 如果目标文件存在跳过
# -d 设置解压到的目录
unzip archive.zip

# 文件太大了会很不方便
todo 卷怎么合并
zip -r a.zip a --split-size 5g # 分卷，每卷最多5g
zipsplit -n ...(这块是b为单位的) a.zip # 已经压缩好的包拆卷
#  10737418240 是10g，拆包之前会告诉一共拆几个包
```

### 7zip
7zip压缩率高，win/linux上都有，也是个很受欢迎的格式。一般linux不预装。7z和gzip或zip相比一个优点是压缩的时候会先扫所有文件，在压缩过程中给一个进度条，而且应该是会根据压缩算法估计最终的文件大小，对盘不够大的情况可以避免压了很久最后放不下。
```shell
yum install -y p7zip p7zip-plugins
7z a name.7z folder/file # a 是添加
7z a -t7z -m0=lzma -mx=9 -mfb=64 -md=32m -ms=on archive.7z dir1 # 网上的最大压缩比参数
7z a -t7z -mx=9 -mfb=273 -ms -md=31 -myx=9 -mtm=- -mmt -mmtf -md=1536m -mmf=bt3 -mmc=10000 -mpb=0 -mlc=0 archive.7z inputfileordir # 文档中的最优设置
7z a -ttar name.tar folder # 创建tar包
```

### 中文解压
windows上压缩的zip经常使用GBK编码，到Linux上解压就是乱码。zip标准还没有给文件名格式留位置所以这个问题基本没有好的通用解决方法。但是如果你直到这个文件大概率是gbk的可以试试这个[脚本](https://gist.github.com/wangjiezhe/7841a350983a147b6d7e)。


# crontab
crontab的功能是按照你定义的时间规则，定时执行任务。时间规则的编辑大概是这样
```shell
crontab -e # 添加一行
```
分 时 日 月 周几
*：无论是什么值都执行： * * * * * 这样就是每分钟都执行
数字：只要到这个值就执行：5 * * * * 任何一个小时的 5分 执行
*/数字：每这么多执行: */5 * * * * 每五分钟执行

crontab如果做一个监控进程，发现错误可以结合下面的发邮件告警。监控crontab本身是否正常执行可以用[cronhub](https://cronhub.io/)，免费用户可以添加两个监控。

# 换源
## pip
安装
```shell
python -m ensurepip --upgrade
```
或
```shell
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```
换源。
```shell
pip config set global.index-url https://mirror.baidu.com/pypi/simple
# pip config unset global.index-url
```
源地址存在 ~/.config/pip/pip.conf 下。比如使用百度源时，pip.conf内容如下：
```shell
[global]
index-url = https://mirror.baidu.com/pypi/simple
```

## npm

# 添加磁盘
```shell
fdisk -l # 显示所有磁盘
mkdir /data # 给新的磁盘创建mount路径
vi /etc/fstab
# 照着上面的格式添加一行
# 磁盘名（按照fdisk里的写，比如 /dev/nvme1n1p1）中间的按照上面
# 最后一个数是是否启动检查，第一次建议设成0，如果盘做的有问题会启动失败。重启和访问都成功之后再设成非0
```
[/etc/fstab 介绍](https://geek-university.com/linux/etc-fstab-file/#:~:text=The%20%2Fetc%2Ffstab%20file%20is,or%20more%20spaces%20or%20tabs)

[格式化和mount盘介绍](https://www.hostingswift.com/how-to-format-and-mount-a-second-hard-drive-with-linux)

# 网络
```shell
ifconfig eth0 192.168.1.5 netmask 255.255.255.0 up
route add default gw 192.168.1.1
echo "nameserver 1.1.1.1" > /etc/resolv.conf
ping baidu.com
ip addr show
ip link set eth10 up
ip route show
```


# 防火墙
```shell
systemctl start firewalld
sudo firewall-cmd --zone=public --permanent --add-port=59966/tcp
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --reload
```
