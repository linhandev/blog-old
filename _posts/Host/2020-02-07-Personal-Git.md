---
title: 部署个人Git
author: Lin Han
date: '2021-02-07 19:02 +8'
categories:
  - Host
  - Git
tags:
  - Git
  - Gitea
  - SelfHost
---

Github是全球最大的开源社区，一般把自己的开源项目放到Github上可以让更多开发者看到，也不用担心项目丢失。但是一些场景下可能也会希望自己部署Git，比如在国内访问Github一般很慢，如果希望频繁地push和pull那么在国内租服务器架上Git速度可以快上几百倍；看往网上的评价如果希望使用一些CI/CD功能，可能Github不支持或者收费，这时候自己部署更灵活或者可以降低成本。这个Post记录自己安装[Gitea](https://gitea.io/)，[Gogs](https://gogs.io/)和[GitLab](https://about.gitlab.com/)的过程，(目前主要发现这三个比较方便的方案，后期发现其他会继续添加)以及一些体验感受。

## Gitea
Gitea由Golang编写。

[//]: # (TODO:评价)
安装步骤基本按照[这篇](https://www.linuxcloudvps.com/blog/how-to-install-gitea-on-centos-7/)教程。
```shell
# 升级系统并安装必要的包
yum update
yum install -y wget git epel-release tree

# 给gitea服务添加用户和存项目的目录
useradd git
mkdir -p /etc/gitea /var/lib/gitea/{custom,data,indexers,public,log}
chown git:git /var/lib/gitea/{data,indexers,log}
chmod 750 /var/lib/gitea/{data,indexers,log}
chown root:git /etc/gitea
chmod 770 /etc/gitea

# 查看刚创建的目录
tree /var/lib/gitea/
/* /var/lib/gitea/
├── custom
├── data
├── indexers
├── log
└── public */

# 安装数据库
yum install -y mariadb-server
systemctl enable mariadb
systemctl start mariadb
mysql_secure_installation # 给root设密码，之后一通 y 就行

# 登录数据库建表
mysql -u root -p
create database gitea;
grant all on gitea.* to gitea@localhost identified by 'Gitea Password';
flush privileges;
quit
```
下面就是安装Gitea，可以先到[Release](https://github.com/go-gitea/gitea/releases)页面看一下最新的版本，把版本号写到下面的变量里。这步不是必需的，会不定期更新下面这个版本号
```
# 下载gitea
export GITEAVER=1.13.2
wget https://github.com/go-gitea/gitea/releases/download/v${GITEAVER}/gitea-${GITEAVER}-linux-amd64 -O /usr/local/bin/gitea
chmod +x /usr/local/bin/gitea
gitea -v

# 创建 gitea 服务
echo """
[Unit]
Description=Gitea
After=syslog.target
After=network.target
After=mariadb.service

[Service]
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web -c /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
""" >> /etc/systemd/system/gitea.service

systemctl daemon-reload
systemctl start gitea
systemctl status gitea
```
到这里安装基本结束了，访问 http://服务器地址:3000/install 应该就能看到初始化配置的界面。如果是按照上面的步骤安装的应该只需要修改数据库密码之后就可以使用了。

## Gogs

## Gitlab
