---
title: "管理和安全"
author: Lin Han
date: "2021-12-07 19:35"
categories:
  - Note
  - IAP
tags:
  - Note
  - IAP
---

## SNMP
Simple Network Management Protocol，3个版本。Agent 161/UDP，Manager 162/UDP

网络管理
- 收集设备状态：SNMP
- 监控网络流量
- 解决问题

![snmp architecture](/assets/img/post/IAP/snmp-architecture.png)

Agent收集信息，存在Management Information Base里，通过SNMP发给Manager，网管在Manager上看。

Community：Agent划分成Coomunity

MIB是树状结构，private是企业自己定义的，不是不能公开访问的意思

![mib-structure](/assets/img/post/IAP/mib-structure.png)


## 安全

安全
- Confidentiality：只让两端能看到内容
- Integrity：防篡改
- Avalibility：不被ddos
- Authenticity：保证信息来源是不是假冒的
- Non-repudiation：负责，发了不能说没发过，收到了不能说没收到过

- Authentication：两端，你确实是你，对面确实是对面
- Authorization：你有权
- Accounting：你做了


- 通信安全
![security-parties](/assets/img/post/IAP/security-parties.png)
  - 发送者用一把钥匙加密
  - 接收者用另一把解密
  - 公钥由可信第三方分发
- 访问控制
![access control](/assets/img/post/IAP/access-control.png)
  - 对外：防火墙
  - 对内：

### 加密

![encreption](/assets/img/post/IAP/encreption.png)
目标：难以暴破

方法
- 乱序
- 映射

加密和解密用
- 相同的key：对称加密
![1key](/assets/img/post/IAP/1key.png)
  - key需要下发，路上不安全
    - 可以用非对称下发，之后用对称
  - 每组收发需要一把key，N(N-1)/2
  - 加密解密代价小
- 不同的key：非对称加密
  - 一人两把钥匙，2N
  - 公钥依赖CA分发
  - 私钥加密：是你发的。保证来源，只要公钥能解开来源就是对的，但是信息不保密，大家都可以解
  ![private-encrept](/assets/img/post/IAP/private-encrept.png)
  - 公钥加密：只给你看。保密，只有私钥能解开，但是不保证来源，大家都可以用公钥加密
  ![public-encrept](/assets/img/post/IAP/public-encrept.png)
  - 公私钥加密：先保密，再保证来源
  ![2 keys encrept](/assets/img/post/IAP/2-keys-encrept.png)

电子签名
![digital sign](/assets/img/post/IAP/digital-sign.png)

![encrept digital sign](/assets/img/post/IAP/encrept-digital-sign.png)

![security layers](/assets/img/post/IAP/security-layers.png)

### 应用层

ssh：secure shell

kerberos：

### 传输层


### 网络层
![ipsec](/assets/img/post/IAP/ipsec.png)
