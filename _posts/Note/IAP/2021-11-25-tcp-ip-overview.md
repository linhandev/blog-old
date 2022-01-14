---
title: "TCP/IP 总览"
author: Lin Han
date: "2021-11-25 23:09"
published: false
categories:
  - Note
  - IAP
tags:
  - Note
  - IAP
---
![internet](/assets/img/post/IAP/internet.png)

Internet: Interconnected networks using standardized protocols

重要节点
  - 1960S: ARPA Net，4 Nodes
  - 1984: NSF Net
  - 1995: Commercial Internet

![organizations](/assets/img/post/IAP/organizations.png)

- Internet Architecture Board: 定义互联网架构，批准协议
- Internet Engineering Task Force: 协议的设计和开发
- Internet Corporation for Assigned Names and Numbers: 截棒IANA，分网上所有独一无二的资源，比如IP，域名，协议代码..管理13个根DNS服务器

所有的协议都是一个RFC，不是所有RFC都是协议

## TCP/IP协议栈

**协议**规定
- 消息格式
- 消息顺序
- 接收消息后的操作

Cerf and Kahn提出

![osi-5](/assets/img/post/IAP/osi-5.png)

OSI五层结构
- 应用层
  - 主机上的应用
  - FTP，HTTP，SMTP，DNS，DHCP，RIP...
- 传输层
  - 只在主机上实现，进程对进程通信
  - TCP，UDP
- 网络层
  - 路由，把数据包从发送者传到目标
  - IP，ICMP，IGMP，OSPF
- 数据链路层
  - 邻居之间怎么通信
  - Ethernet，IEEE 802.3，IEEE 802.11 无线，PPP，Token Ring，ATM，ARP，RARP
- 物理层
  - 定义在媒介上怎么发0和1，不属于TCP/IP协议栈
  - 用周期内上升或下降代表0/1，不同不动的高低电平->时钟和信息一起发送，而且知道什么时候发送结束

设备->两两连接->组网路由数据包->进程间通信->支持应用

![encapsulation](/assets/img/post/IAP/encapsulation.png)

从高层到低层，依次在数据前面和结尾加Protocol Data Unit

multiplex：多路复用
- 一个低层协议可以支持多个高层协议，用低层的Frame Type或者端口号之类的区分 eg：IP支持TCP和UDP
- 一个高层协议可以用多个低层协议 [//]: # (TODO:例子)

源IP，目标IP，源端口，目标端口，协议。这五个值唯一确定一个程序的数据流

![linux-networking](/assets/img/post/IAP/linux-networking.png)


设备
- hub：Level 1
  - 就是网线。一口进，其他所有口出
  - ![hub](/assets/img/post/IAP/hub.png)
- bridge：Level 2
  - 不看高层信息，分割冲突域，可以转协议
  - 如果收发协议相同数据完全不会变
  - ![bridge](/assets/img/post/IAP/bridge.png)
- router：Level 3
  - 路由
  - ![router](/assets/img/post/IAP/router.png)
- switch
- gateway


Linux 实现
- socket layer：接app的数据
- protocol layer：传输和网络层协议
- interface layer：数据链路层协议

/etc/services存well known port

常见程序：
- inetd：
- httpd：web server
- named：DNS



## 命名

### 域名
![domain-name](/assets/img/post/IAP/domain-name.png)

层级结构，每层最多63个字符。
- Root Domain： .
- Top Level Domain
  - arpa：ip->域名
  - generic domain：org，com...
  - 国家代码：cn，us...
- Authoritative Domain： github.com

![generic-domain](/assets/img/post/IAP/generic-domain.png)

DNS分布式部署，一个服务器负责一个类型的一部分，每个服务器多地部署。查询过程
- 循环查询
![iterative-query](/assets/img/post/IAP/iterative-query.png)
- 迭代查询
![recursive-query](/assets/img/post/IAP/recursive-query.png)

缓存：TTL过期就重新查询
- 本地缓存
- ISP缓存：默认DNS服务器。记录一些用户常用域名的IP
  - 1.1.1.1：cloudflare
  - 8.8.8.8：Google

### IP
v4 32位，v6 128位

- host部分全0：标记一个网段
- host部分全1：广播IP
- 第一个IP通常给gateway

loopback interface
- 127.0.0.1，localhost
- 帧沿着协议栈下去再上来，不会出现在网络上

![interface](/assets/img/post/IAP/interface.png)

### Classful IP

![ip-range](/assets/img/post/IAP/ip-range.png)

![classful-ip](/assets/img/post/IAP/classful-ip.png)

- Class A：1位，1个7
- Class B：2位，2个7
- Class C：3位，3个7
- Class D：4位，组播
- Class E：5位，预留

公网不能用的IP：
- 局域网IP
  - 10.0.0.0～10.255.255.255
  - 172.16.0.0～172.31.255.255
  - 192.168.0.0～192.168.255.255
- 组播IP：224.0.0.0～239.255.255.255
- 实验IP：240.0.0.0～225.225.225.225



### 端口号
16位
- 1～1023：well known port，给系统程序
  - 1～255：各种os都用的服务 eg：ssh
  - 255～1023：Unix专用的服务 eg：rlogin
- 1024～49151：registered port，给一般用户程序
- 49151～65525：临时端口，随机分配

<!-- ### MAC
不同的数据链路层协议不同，Ethernet 48位，全球唯一
- 前24位：vendor component，每个生产商一个
- 后24位：group identifier，每个设备不同

IP->MAC：ARP Address Resolution Protocol

MTU：数据链路层中每个帧中最大的Payload长度，不算自己的协议头和尾
- Ethernet，PPP：1500 Bytes
- FDDI：4352 Bytes
- PPP（低延迟）：296 BYtes
- Ethernet Jumbo Frame：9000 Bytes -->

路由分类
![routing-classification](/assets/img/post/IAP/routing-classification.png)
- circuit switch：每组host独占一条电路
  - 通信质量好，带宽固定
  - 带宽可能浪费
- packet switch：用户数据分块，一跳一跳发过去

## 错误校验

checksum：
- 确定一个长度K，比如16
- 将待计算数据分成长度为K的段，最后长度不够的话补0
- 计算所有段的和
- 取反码和数据一起发送
- 对方按同样的方法对收到的数据求和，把求和结果和收到的checksum值相加，结果应该全是1
[//]: # (TODO:checksum和crc怎么算)

CRC（cyclic redundancy check）：
- 确定一个CRC生成数字A
- 将整个帧当作一个数，除A，把余数和数据一起发送
- 对方做同样的除法，得到的余数应该相同
