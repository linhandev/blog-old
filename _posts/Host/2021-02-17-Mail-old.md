---
title: Centos Postfix 邮件服务器
date: "2021-02-17 01:04"
tags:
  - Centos7
  - Mail
  - SelfHost
categories:
  - Tool
published: false
---

邮件作为一个关键的服务非常需要稳定，因此用一个大厂邮箱是个很好的选择。但是可能出于隐私，免费/付费邮箱服务的各种限制等原因你会希望自己部署一个邮件服务器(Gmail 还是很难抢到一个自己喜欢的用户名的)。因为是要搭一个自己用的服务器，我选择的方案类似一个最小化安装，Postfix MTA + Dovecot IMap + RoundCube 网页浏览(可以直接用本地客户端，不装这个)，目标是尽最大可能把系统资源用在刀刃上，不搞那些花里糊少的功能。

## 系统要求

- Centos 7
- 1 CPU
- 1G 内存
- 一个域名：可以 freenom+cloudflare
- 服务器
  - SMTP 协议发邮件走 25 端口，阿里云，腾讯云和 Linode 都是禁 25 端口的，需要额外步骤不推荐
  - Vultr 的服务器默认禁 25 端口，但是提工单可以解封

## 原理

用大厂的邮箱服务发邮件很简单但是背后还是涉及不少步骤，稍作了解很有必要。[这篇文章](https://www.pepipost.com/blog/email-works-behind-scenes-developers-perspective/)介绍的很通俗易懂。

![mail-process](/assets/img/post/Tool/mail/mail-process.png)

## 邮件服务器

这部分的内容主要根据[这篇教程](https://www.krizna.com/centos/setup-mail-server-centos-7/)整理，Lionode 上还有一篇基本相同但是更详细的[教程](https://www.linode.com/docs/guides/email-with-postfix-dovecot-and-mariadb-on-centos-7/)。主要根据前面这篇是因为他用 Linux 的帐号系统，第二篇写进 SQL 更灵活但是也更复杂。加密的部分采用第二篇。

首先需要一个自己的域名，并解析到服务器上。

```shell
linhan.ml A 10 12.34.56.78
linhan.ml MX 10 linhan.ml
mail.linhan.ml MX 10 linhan.ml
```

给域名加一个 A 记录指向服务器公网 IP。SMTP 看 MX 记录，MX 的内容要写服务器域名(MX 不能写 IP)。

之后配置服务器 host

```shell
hostnamectl set-hostname mail.linhan.ml

vi /etc/hosts
# 添加一条记录
12.34.56.78 linhan.ml
```

安装软件，申请 ssl 证书

```shell
yum install -y postfix dovecot epel-release
sudo yum install -y epel-release
sudo yum install -y snapd
sudo systemctl start snapd
sudo ln -s /var/lib/snapd/snap /snap # enable classic snap support
sudo snap install core; sudo snap refresh core # 更新snap
sudo yum remove -y certbot # 确保没有之前安装的cerbot残留
sudo snap install --classic certbot # 安装cerbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot # 链接到执行路径
certbot certonly
# 证书都在 /etc/letsencrypt/live/$domain  里
```

之后的步骤是修改各种配置

```shell
vi /etc/postfix/main.cf

# 删掉这两行的 #
# inet_interfaces = localhost #---> line no 116
# mydestination = $myhostname, localhost.$mydomain, localhost #--> line no 164

# 之后在这个文件最后添加下面的部分，并按注释修改

# 改成前面设置的hostname
myhostname = mail.linhan.ml
# 改成服务器域名
mydomain = linhan.ml
myorigin = $mydomain
# 邮件文件存储的路径，按需要修改
home_mailbox = mail/
mynetworks = 127.0.0.0/8
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_local_domain =
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_loglevel = 1
smtpd_tls_key_file = /etc/letsencrypt/live/$mydomain/privkey.pem
smtpd_tls_cert_file = /etc/letsencrypt/live/$mydomain/fullchain.pem
smtpd_tls_received_header = yes
smtpd_tls_session_cache_timeout = 3600s
tls_random_source = dev:/dev/urandom
```

```shell
vi /etc/postfix/master.cf
# 在 smtp inet n – n – – smtpd 行下一行添加
submission     inet  n       -       n       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
smtps     inet  n       -       n       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING

# postfix的设置完成，检查是否有格式错误
postfix check
```

下面配置 dovecot

```shell
vi /etc/dovecot/conf.d/10-master.conf

# Postfix smtp-auth <-- Line no 95
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
```

```shell
vi /etc/dovecot/conf.d/10-auth.conf

auth_mechanisms = plain login
```

```shell
vi /etc/dovecot/conf.d/10-mail.conf

# 注意这个后面的 ~/mail 要和前面postfix设置的一样
mail_location = maildir:~/mail
```

```shell
vi /etc/dovecot/conf.d/20-pop3.conf

pop3_uidl_format = %08Xu%08Xv
```

配置都结束了，重启服务并添加开机自启，添加防火墙规则

```shell
systemctl restart postfix
systemctl enable postfix
systemctl restart dovecot
systemctl enable dovecot

systemctl start firewalld
# postfix
firewall-cmd --permanent --add-service=smtp
firewall-cmd --permanent --add-port=587/tcp
firewall-cmd --permanent --add-port=465/tcp
# dovecot
firewall-cmd --permanent --add-port=110/tcp
firewall-cmd --permanent --add-service=pop3s
firewall-cmd --permanent --add-port=143/tcp
firewall-cmd --permanent --add-service=imaps

firewall-cmd --reload
```

使用 telnet 检测端口是否开启

```shell
telnet linhan.ml 25
ehlo linhan.ml
```

这个方案配置十分简单是因为用了 linux 的用户而不是写在数据库里，创建一个 linux 用户就创建对应的邮箱用户

```shell
useradd -m me -s /sbin/nologin
passwd me
```

到这里邮件服务器部分就已经起来了，配置客户端过程中 IMAP 应该是 143 端口，SMTP 587。

## 客户端

邮件客户端很多，Thunderbird，MailTime 都是不错的选择，谷歌一下根据需要选就行。本地客户端不占用服务器资源但是有个网页客户端做 backup 总是好的。查了一圈[mailpile](https://www.mailpile.is/)，[rainloop](http://www.rainloop.net/)和[roundcube](https://roundcube.net/)都是不错的选择。mailpile 是 star 最多的，但是貌似团队不是很健全开发很慢。

[rainloop centos7](https://www.vultr.com/docs/how-to-install-rainloop-webmail-on-centos-7)

[//]: # "TODO:研究ireadmail等一键脚本都用什么webmail"
[//]: # "TODO:记录一个webmail安装方法"

## 备份

自己搭建的服务可能会出现一些错误，时常进行备份十分重要。

[//]: # "TODO:如何进行备份"

## spam

[//]: # "TODO:如何过滤垃圾邮件"
