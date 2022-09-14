---
title: "应用层"
author: Lin Han
date: "2021-12-04 19:48"
categories:
  - Note
  - IAP
tags:
  - Note
  - IAP
---

- HTML：Hyper Text Markup Language
- URL：Uniform Resource Locators
![url-format](/assets/img/post/IAP/url-format.png)
  - 协议：http，rtp，rtsp，https...
  - ：//
  - 域名或IP
  - 端口号
  - 文件路径
  - GET参数
- CGI：Common Gateway Interface，可以根据客户端上传数据返回动态网页


## HTTP
HyperText Transfer Protocol
- 没有状态，所有操作一步完成
- 服务器端 TCP 80pessive open等待客户端连接
- 内容是英语不是代码
- HTTPS：三层连接（TCP，TLS，HTTP）
  - TLS加密HTTP通信内容，一个TLS连接可以加密多个HTTP连接
  - 过程
    - 客户端向服务器 443/TCP 握手
    - 后续所有内容用TLS加密，放进HTTP

![http-request-reply](/assets/img/post/IAP/http-request-reply.png)

![http-request](/assets/img/post/IAP/http-request.png)

![http-response](/assets/img/post/IAP/http-response.png)

HTTP message
- start-line
  - request line
    - 请求类型
      - GET：获取文件全文
      - HEAD：获取文件header基本信息 eg：下一个大文件之前先请一下多大
      - PUT：创建一个新文件或者覆盖一个已有的，URL是放到哪
      - POST：和PUT类似，但是URI指定的是谁来处理
      - PATCH：一串对旧文件的修改
      - COPY，MOVE，DELETE...
    - URL
    - HTTP版本
  - status line
    - HTTP版本
    - 状态码
      ![http-response-code](/assets/img/post/IAP/http-response-code.png)
      - 1xx
      - 2xx：成功
        - 200 OK
        - 201 Created：新文件创建
      - 3xx：重定向
        - 301 Moved Permanently：请的东西换地方了
        - 304 Not Modified
      - 4xx：客户端错误
        - 400 Bad Request：request的格式有问题
        - 404 Not Found
      - 5xx：服务器端错误
        - 500 Internal Server Error：服务器内部的问题，比如服务器端代码有bug
        - 505 HTTP Version Not Supported
    - status phrase
- header
- 空行 \r\n
- 请求的文件

代理：截客户端request，自己作为客户端往服务器request
- 结合防火墙可以限制客户端访问范围
- 缓存
  - 更快响应客户端请求
  - 减轻服务器负担

- 非持久HTTP：HTTP 1.0 每次TCP连接最多传一个文件
  ![non-persistent-http](/assets/img/post/IAP/non-persistent-http.png)
  - 接收开始：请求后2RTT
  - 接收结束：2RTT+传文件时间
  - 缺点：
    - 传输前等待时间长
    - 并行传输需要创建很多连接
- 持久HTTP：HTTP 1.1 建立连接后先不关，顺序传多个文件
  - Connection: Keep-Alive
  - 接收开始
    - 第一个文件还是2RTT
    - 后续的文件1RTT就可以开始收
  - 缺点：
    - head of line blocking：first come first served，前面有一个大文件，后续的小文件需要等
    ![hol](/assets/img/post/IAP/hol.png)
    - 重发过程会中断新文件传输
- HTTP/2：一个连接，数据流而不是文件
  - 数据流可以包含文件内容或头文件
  - 内容优先级
  ![https2-priority](/assets/img/post/IAP/https2-priority.png)
  - 缺点
    - 单个TCP连接，重发仍然中断传输
- HTTP/3：使用Quick UDP Internet Connections（QUIC），应用层协议
  - 单管道变多管道
    - 并行传输
    - 处理错误不中断传输
  - 整合传输和安全
  ![handshake](/assets/img/post/IAP/handshake.png)
  - 使用很多类似TCP的技术
    - 创建连接
    - 丢包的处理
    - 拥塞控制
- TCP Fast Open：握手的过程中传数据
  ![tpo-1](/assets/img/post/IAP/tpo-1.png)
  ![tpo-2](/assets/img/post/IAP/tpo-2.png)
  - 第一次握手带上TFO Cookie（如果没有发空的）和要请的数据
  - 服务器的SYN+ACK
    - TFP Cookie有效：直接带数据
    - TFP Cookie无效：带一个TFO Cookie
  - 第一次创建连接还是2RTT
  - 后续创建连接1RTT就开始收数据

## Cookie
![cookie](/assets/img/post/IAP/cookie.png)

## CDN
Content Delivery Network，把大文件放的离用户更近，加快传输速度，降低传输成本

DNS可以做load balance


## DHCP
Dynamic Host Configuration Protocol，Server UDP/67，Client UDP/68

- 手动分配：DHCP查表看每个MAC怎么配置
- 固定分配：记录设备MAC，每次请求给同一组配置
- 动态分配：出租配置，过期不续就分给其他设备

![dhcp-process](/assets/img/post/IAP/dhcp-process.png)
- DHCPDISCOVERY：源IP 0.0.0.0，广播
  - DHCP Relay拓展可以转发Discovery到广播域外，扩大Server管理范围
  - Option 82：支持的设备在Discovery里加地理信息，转到Server，Server可以给附近的设备分接近的IP，简化路由
[//]: # (TODO:relay细节)
- DHCPOFFER：收到请求的DHCP服务器都给这个设备分一个IP发过来
  - 可以不进行Discovery，服务器定时给Offer等着host Request
- DHCPREQUEST：设备广播自己选的IP，Server Identifier标自己选的是哪个服务器的
- DHCPACK：被选中服务器回一个确认消息，包含完整具体配置，比如IP租期
- DHCPRELEASE：设备自己释放IP

IP是租用的，下发的时候带租期，一般租期过半就重新发DHCPREQUEST续租，到期后收回

![dhcp-frame-format](/assets/img/post/IAP/dhcp-frame-format.png)

![dhcp-config-file](/assets/img/post/IAP/dhcp-config-file.png)

## NTP
Network Time Protocol，通常可以将时间校准到10ms误差以内

123 UDP/TCP

层次结构
- stratum 0：可靠时钟源
- stratum 1～15：离时钟源有几层服务器
  - stratum 1 大概300个

时间戳：
- Transmit Timestamp：发送host自己时钟的发出时间
- Origin Timestamp：client发送请求里的Transmit Timestamp
- Receive Timestamp：服务器收到请求的时间
- Reference Timestamp：NTP服务器最近一次校准时间

![ntp-process](/assets/img/post/IAP/ntp-process.png)

RTT：Client收发间隔 - Server收发间隔

知道单程时间和两组时间戳的差，可以校准主机时钟。通常会请多个NTP服务器，统计方法减小误差


## NAT
Network Address Translation，将局域网IP映射到公网IP。一般说NAT都是带端口的，PAT和NAPT的意思。

![nat](/assets/img/post/IAP/gateway-nat-2.svg)

步骤
1. 地址绑定
- 数量
  - one to one：一个局域网IP给一个公网IP
  - many to one：多个局域网IP给一个公网IP
    - 内网IP+端口映射到公网IP+端口
    - 公网端口只能用动态端口
- 固定
  - 静态：映射不顶
  - 动态：一段时间内固定，更好的利用端口资源
2. 地址转换
- 向外
  - 改源IP，源端口
- 向内
  - 改目标IP，目标端口
3. 地址解绑


问题：路由器是三层设备，但是NAT需要改所有4层的IP和端口信息
- 改了IP和端口号需要重新checksum，额外的计算
- 一些协议内容里有端口号，也需要改
  - ICMP报文要带触发数据包IP header和前8 Bytes payload，里面有ip和端口号
  - FTP PORT命令指定IP和端口号，在内容里

## socket
- TCP
- UDP
- RAW：允许程序绕过传输层协议直接用网络曾的IP
