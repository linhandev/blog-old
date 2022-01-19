---
title: "协议解析和订阅制作"
author: "Lin Han"
date: "2021-02-23 11:58 +8"
published: false
categories:
  -
tags:
  -
---

# ss
格式
ss://method:password@server:port#comment
其中 method:password@server:port 部分是urlsafe base64编码，# 后面是urlencode

对一个链接进行解析
```python
from urllib import parse
import base64
link = "ss://YWVzLTI1Ni1nY206Q1VuZFNabllzUEtjdTZLajhUSFZNQkhEQDgyLjEwMi4xNi45OTozOTc3Mg==#%E8%8A%82%E7%82%B9%E5%90%8D%E7%A7%B0"
protocol, content = link.split("//")
content_split = content.split("#")
if len(content_split) != 1:
  comment = content_split[1]
else:
  comment = ""
comment = parse.unquote(comment)
content = base64.urlsafe_b64decode(content)
print(protocol, content, comment)
```

# ssr
格式

ssr://server:port:protocol:method:obfs:password_base64/?params_base64

ssr://c3NyLTAxLnNzcnN1Yi5vbmU6ODg5MTpvcmlnaW46YWVzLTI1Ni1jZmI6cGxhaW46YUhSMGNITTZMeTl6ZFc4dWVYUXZjM055YzNWaS8_b2Jmc3BhcmFtPTVMdVk2TFM1VTFOUzVyT281WWFNT21oMGRIQTZMeTlrYkdvdWRHWXZjM055YzNWaSZwcm90b3BhcmFtPWRDNXRaUzlUVTFKVFZVSSZyZW1hcmtzPVFGTlRVbE5WUWkzbGlxRG1pN19scEtkemMzSXdNUzNrdTVqb3RMbm1qcWpvalpBNmMzVnZMbmwwTDNOemNuTjFZZyZncm91cD1kQzV0WlM5VFUxSlRWVUk
参考
https://www.bilibili.com/read/cv1526493/


## 测速

https://merlinblog.xyz/wiki/speedtest.html
