---
title: 部署个人网盘
author: Lin Han
date: "2021-02-07 19:02 +8"
published: false
categories:
  - Host
  - Drive
tags:
  - Drive
  - SelfHost
---

将文件复制到用户 files 文件夹下，让 nextcloud 从盘上扫描文件

sudo -u www-data php console.php files:scan --all

校验 ics
https://icalendar.org/validator.html
要求 crlf 结尾

docker run -t -d -p 127.0.0.1:9980:9980 -e 'domain=cloud\\.linhan\\.ml' -e 'dictionaries=en cn' --restart always --cap-add MKNOD collabora/code

curl -k https://localhost:9980

https://fqdn/index.php/settings/admin/richdocuments

修改日历同步周期

sudo -u www-data php occ config:app:set dav calendarSubscriptionRefreshRate --value "P1H"
