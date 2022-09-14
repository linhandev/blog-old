---
title: Centos环境配置
author: Lin Han
date: 2021-01-05T15:22:16.000Z
categories:
  - Linux
tags:
  - Centos
math: true
---

Centos是基于RedHat Enterprise Linux开源的一个Linux distro，以极高的稳定性闻名。这个Post讲解将Centos作为服务器，安装软件和进行配置的技巧。
# 虚拟化
一些软件（比如mailcow）可能会对服务器的虚拟化技术有要求，用
```shell
virt-what
```
命令可以推断服务器用的是什么虚拟化技术。一般有要求的都会明确提出来，否则大概就是没有。

# 域名
做小程序，做https网站等等一些操作可能需要一个域名。域名可以从[freenom](https://www.freenom.com/en/index.html)免费弄一个，但是如果做邮件服务器注意免费域名可能被当作垃圾邮件或者在一些地方不让用。特别注意如果是服务器在国内，所有ISP都要求域名进行备案，否则域名就算是解析到服务器也只会展示一个漂亮的阻断访问页面提醒你去备案。虽然不能访问网站但是还是可以解析到的，做邮件服务器或者ssh都可以用。

# httpd
从这里开始进入安装，从最好装的开始。如果你希望用服务器托管网页或者你的前端程序需要访问服务器上的一些文件，那么httpd是一个很方便的选择。[Apache httpd](http://httpd.apache.org/) 是一个网页服务器，虽然在现在的标准下看起来可能有点老了，但是小规模场景下效果也不错，安装和使用都很简单。

```shell
yum install -y httpd # httpd安装
systemctl start httpd # 启动
systemctl status httpd
systemctl enable httpd # 开机自动启动
# systemctl stop httpd # 关闭服务
```

Centos下，httpd的配置在 /etc/httpd/conf/ 下，用 httpd -t 可以校验修改后的配置有无语法错误。默认网页文件root在 /var/www/html

现在为了保证安全，很多服务会强制要求https，比如微信小程序不能做http的访问。httpd配置https最简单的方法一定是[LetsEncrept](https://letsencrypt.org/) + Cerbot。

官网上cerbot是通过snap安装的，需要先装snap。
```shell
sudo yum install -y epel-release
sudo yum install -y snapd
sudo systemctl start snapd
sudo ln -s /var/lib/snapd/snap /snap # enable classic snap support
sudo snap install core; sudo snap refresh core # 更新snap
```

安装cerbot，并添加到执行路径
```shell
sudo yum remove -y certbot # 确保没有之前安装的cerbot残留
sudo snap install --classic certbot # 安装cerbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot # 链接到执行路径
```
Let's Encrept并不给ip颁发证书，你需要一个网址指向自己的ip。

```shell
sudo certbot --apache  # 获取证书并顺手改了httpd的配置
# sudo certbot certonly --apache # 只获取证书，不修改配置
```
可能的bug：
- 网络问题
```shell
HTTPSConnectionPool(host='acme-staging-v02.api.letsencrypt.org', port=443): Max retries exceeded with url: /directory (Caused by NewConnectionError('<urllib3.connection.HTTPSConnection object at 0x7f1ac78badc0>: Failed to establish a new connection: [Errno -2] Name or service not known'))
```
这个大概是网络的问题，重新跑一遍应该就好了

- VirtualHost

```shell
Unable to find a virtual host listening on port 80 which is currently needed for Certbot to prove to the CA that you control your domain. Please add a virtual host for port 80.
# 这是要你在80端口创建一个virtual host，可以在httpd设置中添加
<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName 域名
</VirtualHost>
```

- 找不到ssl模块

```shell
Could not find ssl_module; not installing certificate.
# 首先确定自己有没有ssl模块
ls /etc/httpd/modules/ | grep ssl # 如果结果是空的，那需要安装mod_ssl，正常应该显示 mod_ssl.so
yum install -y mod_ssl
# 确定 mod_ssl.so 存在之后在 httpd.conf 中添加
LoadModule ssl_module modules/mod_ssl.so
# 之后重新跑申请证书就可以了
```


目前cerbot安装已经自带更新证书了，测试自动更新
```shell
sudo certbot renew --dry-run
# Congratulations, all simulated renewals succeeded:
```
到这就可以通过https访问网页了，证书自动更新，不用担心过期。

除了网页之外，你可能有一些类似 Flask 的服务不从httpd走流量，而且可能不好配 https。这样可以用httpd配一个端口转发，用httpd监听一个端口的https流量，转发到一个http端口，之后一个类似 Flask 的进程监听这个http端口。响应的数据走相反方向回到用户。这样任何http的服务就可以通过httpd无痛升级到https，而且在服务器内部转发一下也不会有什么安全问题（个人认为）

上面cerboot会把443端口设置成监听https的，可以按照cerboot的写法添加其他端口的监听来代理自己的服务

```shell
# httpd的配置在 /etc/httpd/conf.d
vi /etc/httpd/conf/httpd.conf

# 这么写可以把 5000 端口的 https 流量转到接 8000 端口的 http 服务上，如果同时防火墙挡住 8000/tcp ，不向用户暴露，就可以只让用户访问https的版本了

Listen 5000 https
<VirtualHost *:5000>
  ServerName 服务器域名
  SSLEngine on

  SSLCertificateFile /etc/letsencrypt/live/域名/cert.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/域名/privkey.pem

  ProxyPass / http://localhost:8000/
  ProxyPassReverse / http://localhost:8000/

  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>
</VirtualHost>
```

# shell
[TODO]
shell是我喜欢linux最重要的原因，之前用windows的时候也不知道怎么用命令行，到linux之后发现真是巨方便。shell命令太多了，这里就记一些自己用的多的。菜鸟上有比较全的。
```shell
for zip in `ls *.zip`; do unzip $zip; done
if [$(docker ps | wc -l) -ne 2]; then echo "有情况"; fi


# 查看所有进程
top # 如果风扇莫名开始响可以看是哪个进程在用cpu

# 查看磁盘占用
df # 查看各个挂载点还有多少空间
du # 查看一个目录用了多少空间，默认是会递归进去
du -sh # 如果只是想看一个总数
```

- ctrl-s 冻结命令行
这个时候输入的任何命令会被记录但是不会被执行。
- ctrl-q 解冻命令行
冻结过程中打的命令在这个时候会执行
- ctrl-l 清屏
- ctrl-r

# vim
[TODO]
对新手来说，vi是让人极其崩溃的。这里只记录一些基本的vi命令和技巧。

# vsftpd
这个是ftp服务端程序，ftp客户端有其他的选择，但是这个个人比较熟悉。

```shell
yum install -y vsftpd
systemctl start vsftpd
# systemctl enable vsftpd  # 自动启动三思，因为不一定时刻需要，而且没配好ftp服务端有安全隐患

# 防火墙配置
firewall-cmd --zone=public --permanent --add-port=21/tcp
firewall-cmd --zone=public --permanent --add-service=ftp
firewall-cmd --reload

# 备份配置
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bk

vi /etc/vsftpd/vsftpd.conf
# 之后需要修改一些vsftpd的设置
anonymous_enable=NO             # disable  anonymous login
local_enable=YES		# permit local logins
write_enable=YES		# enable FTP commands which change the filesystem
local_umask=022		        # value of umask for file creation for local users
dirmessage_enable=YES	        # enable showing of messages when users first enter a new directory
xferlog_enable=YES		# a log file will be maintained detailing uploads and downloads
connect_from_port_20=YES        # use port 20 (ftp-data) on the server machine for PORT style connections
xferlog_std_format=YES          # keep standard log file format
listen=NO   			# prevent vsftpd from running in standalone mode
listen_ipv6=YES		        # vsftpd will listen on an IPv6 socket instead of an IPv4 one
pam_service_name=vsftpd         # name of the PAM service vsftpd will use
tcp_wrappers=YES  		# turn on tcp wrappers
# 添加白名单
# vsftpd will load a list of usernames, from the filename given by userlist_file
userlist_enable=YES
# stores usernames
userlist_file=/etc/vsftpd/vsftpd.userlist
# 白名单，只有在上面文件里的用户才能登录
userlist_deny=NO

# 限制只能访问自己的 ~ 路径
# chroot_local_user=YES
# allow_writeable_chroot=YES
```
userlist_enable=YES + userlist_deny=NO 的意思是只允许 /etc/vsftpd/vsftpd.userlist 中的用户登录。所以将想要允许登录的用户添加到这个文件里，一行一个。

[TODO]:Response:	500 OOPS: cannot read user list file:/etc/vsftpd/vsftpd.userlist    # stores usernames.

之后重启vsftpd服务，如果显示 500 OOPS: bad bool value in config file for: 某个配置变量，那么可能是这行后面有空格之类造成的，可以用下面这行处理一下
```shell
sed -i 's,\r,,;s, *$,,' /etc/vsftpd/vsftpd.conf
```

到这ftp基本就配置好了并做了简单的安全措施。本地链接服务器的软件推荐[FileZilla](https://filezilla-project.org/)。如果ftp协议不好用可以试一下sftp，个人经验sftp比ftp容易成功。ftp能做很多事，如果是生产环境安全方面肯定需要额外差资料研究，这些大概是不够的。

# Mysql
Mysql需要通过Oracle维护的单独的一个repo安装，可以下载官网的rpm添加repo之后安装，参考[这个](https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-centos-7/)教程。但是国内一般这种方法都慢到不行，清华源有这个repo的镜像，参考[官方介绍](https://mirrors.tuna.tsinghua.edu.cn/help/mysql/)添加repo，下载速度一般都是很快的。

下载官方repo

```shell
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update -y
```

添加清华源repo

```shell
echo "
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-connectors-community-el7-$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-tools-community-el7-$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-5.6-community]
name=MySQL 5.6 Community Server
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-5.6-community-el7-$basearch/
enabled=0
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-5.7-community]
name=MySQL 5.7 Community Server
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-5.7-community-el7-$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-8.0-community]
name=MySQL 8.0 Community Server
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-8.0-community-el7-$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql
" > /etc/yum.repos.d/mysql-community.repo
```

可能的bug：

Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
GPG key retrieval failed: [Errno 14] curl#37 - "Couldn't open file /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql"
一个校验的问题，最简单的workaround是直接禁用校验。在配置里修改 gpgcheck=0 就能下了，但是这种方法不安全。

添加完repo就进行安装

```shell
sudo yum install -y mysql-server
sudo systemctl start mysqld
mysql_secure_installation # 加强安全，有些版本的初始密码在安装的log文件里，有些默认是空密码
```

如果安到一半出错想要重来，完全卸载mysql用下面两行。如果只是卸了软件，那些忘了的密码之类的还存在数据文件里，装回来还是会需要密码。

```shell
yum remove mysql mysql-server
mv /var/lib/mysql /var/lib/old_backup_mysql # 挪走数据文件，直接删了也行，如果没什么东西
```

## 备份
其实迁移也可以看成是先备份之后原样创建，很多数据库工具都有备份或者迁移功能，但是我使的workbench之前各种出问题。其实mysql自带的mysqldump就能导出创建表格和插入数据的代码，在少量数据的情况下用起来很方便

mysqldump -u root -p <database_name> > out.sql
这样在out.sql中就有了一份创建这个数据库并插入代码的脚本。直接用可能会有问题，有外建约束的时候需要先创建被引的表，dump应该不管这个，之后就是dump的时候会声明字符集，插入到新的数据库如果不支持删掉字符集的声明就行。



# PHP
php是世界上～ 好的我不说了 /滑稽。貌似CentOS上现在yum的repo只到php5，像wp都要求php7了，最简单的php5安装

```shell
yum install -y php
systemctl restart httpd
php -v
echo """
<?php
 phpinfo();
?>""" > /var/www/html/test.php
```
之后访问 hostname/test.php 可以看到php版本之类的信息。如果网页打不开注意看防火墙设置。

php官方推荐通过remi安装，可以自己指定版本，如果有版本要求都推荐这种方法．
```shell
sudo yum install -y epel-release yum-utils
sudo yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum install -y php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysqlnd
php -v
echo """
<?php
 phpinfo();
?>""" > /var/www/html/test.php
```

# Wordpress
有了httpd，mysql和php就可以跑wordpress了。

```shell
# 创建sql表和用户
mysql -u root -p
# 输入密码
CREATE DATABASE wordpress;
CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
exit

# 获取wp
cd ~
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo rsync -avP ~/wordpress/ /var/www/html/ # 保留权限复制
mkdir /var/www/html/wp-content/uploads
sudo chown -R apache:apache /var/www/html/*

# 之后访问网址　
http://example.com/wp-admin/install.php
```

# InfluxDB
上面已经装了一个数据库Mysql，但是如果你的需求是时序数据，比如物联网一些传感器定时产生数据的这种，influxDB是专门针对这个的。比如一定时间窗口下的聚合，mysql实现应该就比较麻烦，influx有默认支持。
8086/tcp
8088/tcp
influx通过yum安装，先添加源
```shell
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```
之后安装和启动
```shell
sudo yum install influxdb # 企业界的东西，装起来就是方便
systemctl start influxdb
```
关于怎么使用，[官方教程](https://docs.influxdata.com/influxdb/v1.8/introduction/get-started/)

# Grafana
Grafana应该是最流行的监控平台，可以对数据库中的数据用多种图表的方式进行展示。老规矩，先添加Grafana源。
```shell
echo "
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
" > /etc/yum.repos.d/grafana.repo
```
之后安装启动，添加防火墙规则
```shell
sudo yum install -y grafana
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
firewall-cmd --zone=public --add-port=3000/tcp --permanent
firewall-cmd --reload
```

# Python3
Centos7 自带的都是2.x的 python，一些包现在已经开始drop对2.x的支持，比如cv2的安装就很费劲，因此上一个3.x的 python 是很有必要的。最初写这个的时候Centos上的repo里最高只到py34，pip都装不上。但是yum已经支持到了py36，而且带pip。

```shell
sudo yum install -y python3
```

如果需要更高版本，py36以上目前我只知道可以通过源码编译安装。下面的脚本在37,38,39中的一个版本中都测试通过。

```shell
# 首先做一波升级
sudo yum update -y

# 因为是编译安装所以需要装一波编译依赖
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
yum install -y libffi-devel zlib1g-dev
yum install zlib* -y

# python.org 下载 .xz 源码压缩包
VERSION=3.7.9 # 3.7.9, 3.8.8, 3.9.2 测试通过
# 从python官网下载可能较慢，华为有一个镜像。不带最后的 /
base_url=https://www.python.org/ftp/python # https://mirrors.huaweicloud.com/python
wget ${base_url}/${VERSION}/Python-${VERSION}.tar.xz
tar -xvf Python-${VERSION}.tar.xz # 解压到当前目录

mkdir /usr/local/python3

cd Python-${VERSION}
./configure --prefix=/usr/local/${VERSION} --enable-optimizations
--with-ssl
# [TODO] with-ssl 不好使
# 第一个指定安装的路径，不指定的话，安装过程中可能软件所需要的文件复制到其他不同目录，删除软件很不方便，复制软件也不方便
# 第二个可以提高python10%-20%代码运行速度

time make && make install # 之后需要编译很长时间，看起来比较吃CPU和IO

# 编译应该不会出什么问题，如果正常结束了就创建到可执行程序的软链接
ln -s /usr/local/${VERSION}/bin/python3 /usr/local/bin/python3
ln -s /usr/local/${VERSION}/bin/pip3 /usr/local/bin/pip3

# 到这就可以在命令行直接运行 python3 了
python3 # 之后就能看到版本了，ctrl+d 退出

# 安装 pip
curl -O https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py
```

## 换pip源
如果服务器在国内，从pypi下载是比较慢的。一些云服务厂商会提供pypi镜像，内网下载会很快。
```shell
mkdir ~/.pip
echo "
[global]
index-url=https://mirror.baidu.com/pypi/simple
" > ~/.pip/pip.config
```

# Paddle-Serving
[TODO]完善内容
甲方有一个深度学习的推荐需求，因此做一个paddleserving。之前用的tfserving是跑在docker容器的，因为当时也菜装的极其绝望，推理相应速度飘忽补丁而且经常自己死掉。但是paddle-serving看起来很简单  paddle官网 paddle-serving github 20年3月paddle-serving 还没有py3的版本，git上说4,5月份会有，pip找不到包的时候注意 之后我这里pip安装有报错，简单来说是这样

  error: command 'gcc' failed with exit status 1
  ----------------------------------------
  ERROR: Failed building wheel for subprocess32
正常pip应该是不会有问题的，这里是因为少依赖

yum install python-devel

# Mail
服务器一般不希望装很多东西，如果就是想发个邮件可以用服务器smtp控制其他邮件服务发，服务器上不需要装。服务器就做了很简单的设置

```shell
vi /etc/mail.rc
set from=邮箱账号 smtp=smtp.qq.com
set smtp-auth-user=邮箱账号  smtp-auth-password=邮箱密码
set smtp-auth=login
echo "邮件主题" | mail -s "邮件内容,这是测试邮件" xx@163.com
```
之后就发出去了。但是邮件服务那边必须开通smtp服务，一般在邮箱的设置里面。需要提一下的是qq邮箱smtp服务开通之后邮箱用的密码不是qq密码，而是开通的时候给的密码，写的时候不要写错了。而且开了smtp要注意发出去的邮件，之前遇到过smtp的密码被破了，自己的邮箱被人用来批量发开发票的广告。。



# 桌面
虽然服务器基本不需要桌面，但是有一些需求可能会需要。Gnome的桌面比较重，个人比较喜欢XFCE。
```shell
yum -y install epel-release

yum -y groupinstall "X Window system" # 大概270个包
yum -y groupinstall "Xfce" # 大概140多个包
systemctl isolate graphical.target # 开启桌面
Gnome的安装也很简单

yum groupinstall -y "GNOME Desktop" # 大概1200个包
# 网上的教程中还有装这个的，少上100个左右的包，效果应该是一样的
yum -y groupinstall "Server with GUI" # 大概1000个
```

# 远程GUI
https://linuxize.com/post/how-to-install-and-configure-vnc-on-centos-7/
[TODO]
服务器本身没有显示器，但是有很多方案把服务器上的图形界面展示到自己电脑的屏幕上。这个方向基本两个解决方法，VNC和X11forwarding，forwarding比较好弄，但是一般比较卡，需要好好配置。vnc可以在本地关掉窗口之后还继续运行，很方便。
```shell
yum -y groupinstall X11  # 大概300个包
sudo yum install -y tigervnc-server

useradd vnc
sudo cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
sudo vi /etc/systemd/system/vncserver@:1.service
# 把里面的用户名改了

sudo systemctl daemon-reload
sudo systemctl enable vncserver@:1.service

sudo systemctl start firewalld
firewall-cmd --add-port=5901/tcp --permanent
# sudo firewall-cmd --permanent --zone=public --add-port=5904-5905/tcp
sudo firewall-cmd --reload

su vnc
vncserver

sudo systemctl daemon-reload
sudo systemctl restart vncserver@:1.service


# 关闭一个桌面
vncserver -kill vultr.guest:1
```

# 清理
服务器用的时间长了可能各种log，各种cache之类的东西会占很大的磁盘，因此可以考虑定期进行清理。需要注意只要是删除东西的操作都是危险的，要知道自己在干什么。
[完整教程](https://www.getpagespeed.com/server-setup/clear-disk-space-centos)

# 用户管理
添加一些低权限的用户给别人使用能防止意外的故障。

```shell
useradd [username]
passwd [username]

gpasswd -a username wheel # 添加sudo权限

userdel [username]
```

# Python web
flask + gunicorn + supervisor

当你通过命令行登录服务器启动了一个进程后，退出服务器会默认终止这个进程。如果你不想进程随着退出ssh被kill掉可以使用nohup命令。如果你有一个flask app 在 flask_app.py 里，那么可以执行
```shell
nohup python flask_app.py &
```

但是nohup有时候很不稳定，而且没有进程出问题后自动重启，这显然不是一个好的方法，flask本身的WSGI不是为了生产设计的，也不是很稳定。推荐使用gunicorn这种生产级别的wsgi。如果要https可以用端口映射的方式，不一定要在gunicorn中配https。

supervisor和systemd类似，是用来保证进程一直运行的，如果进程除了问题supervisor可以重启进行，保证后台的稳定。
```shell
# 安装
yum install supervisor
# 设置一般在 /etc/supervisord.conf ,至少大概在etc下

systemctl daemon-reload
systemctl start supervisord
systemctl status supervisord
systemctl enable supervisord

# 到这supervisord本身应该就起来了，之后编辑supervisord.conf添加program
[program:test]
directory=/path/to/execute
command=python3 main.py
user=root
autostart=true
autorestart=true
redirect_stderr=true
stderr_logfile=/path/to/errlog
stdout_logfile=/path/to/stdlog


supervisorctl reread # 如果改了config文件，这个命令加载新的
supervisorctl stop [appname]
supervisorctl start [appname]
supervisorctl restart [appname]
```


在调试过程中可能有时候没有正确关闭服务，这样就会出现address already in use。这个时候重启肯定可以释放端口，但是有点麻烦，可以通过下面一行代码找到占用端口的进程+杀掉它。慎用，他不管这个端口是什么服务
```shell
fuser -k 4000/tcp
```

# Ruby
主要是为了gollum wiki要装ruby，把ruby和gollum都记一下。ruby貌似不自举，写这个记录的时候基于c的ruby一个依赖有bug，不知道怎么解决，所以用的jruby，[安装教程](https://devops.ionos.com/tutorials/install-ruby-214-with-rvm-on-centos/)。装ruby的时候不写版本号写jruby就行。之后直接
```shell
yum install -y epel-release
yum install -y go
gem install -y gollum
gollum # 开始运行
```

# 软件版本
[TODO]详细研究
```shell
sudo alternatives --config cmake
```
