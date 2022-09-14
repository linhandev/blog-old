---
title: "网络层"
author: Lin Han
date: "2021-11-28 00:07"
categories:
  - Note
  - IAP
tags:
  - Note
  - IAP
---

![ip-example](/assets/img/post/IAP/ip-example.png)

![iptv-example](/assets/img/post/IAP/iptv-example.png)

## IP
v4 32位，v6 128位

loopback interface：帧沿着协议栈下去再上来，不会出现在interface硬件上
- ip
  - 一般是127.0.0.1，localhost
  - 127.0.0.1～127.0.0.254，都是localhost
  - 127.255.255.255是一个广播地址，但是lo不支持BROADCAST
- 作用
  - 在本机上测试C/S架构的程序
  - localhost ping通说明TCP/IP协议栈没问题


![interface](/assets/img/post/IAP/interface.png)

### Classful IP

![ip-range](/assets/img/post/IAP/ip-range.png)

![classful-ip](/assets/img/post/IAP/classful-ip.png)

- Class A：1位，1个7
- Class B：2位，2个7
- Class C：3位，3个7
- Class D：4位，组播
- Class E：5位，预留

公网不能用的IP主要有：[完整列表](https://en.wikipedia.org/wiki/Reserved_IP_addresses)
![special ip](/assets/img/post/IAP/special-ip.png)
- loopback：127/8 127.0.0.0～127.255.255.255
- link-local：DHCP宕了大家可以ARP探已经占用的IP，给自己一个link-local IP
  - 169.254/16 169.254.0.0～169.254.255.255
- 0/8：没有IP的设备用于表明自己
- 局域网IP：局域网设备可以复用IP;局域网数据不会上公网，更安全
  - 10/8：10.0.0.0～10.255.255.255
  - 172.16/12：172.16.0.0～172.31.255.255
  - 192.168/16：192.168.0.0～192.168.255.255
- 组播IP：224.0.0.0～239.255.255.255
- 实验IP：240.0.0.0～225.225.225.225

### CIDR
Classless Interdomain Routing，用子网掩码区分network id和host id，让网段的长短和host的数量更贴近。可以多次分段
- 节省IP：分给一个组织的网段长度更接近实际需要的数量
- 简化路由：一些子网在远处的路由上可以合并，减少路由记录数量


#### 划分网段
- host部分全0：网段IP
- host部分全1：广播IP
- 第一个IP通常给gateway

给出
- 一段待划分的ip
- 一些子网，每个要能放下一定数量的host

一般按host数量从大到小分
- 确定host数量的要求需要几位host ID来满足
  - host ID全0是网段地址，全1是广播地址，一般第一个IP给网关
  - n个host需要n + 3个地址，找大于等于n+3最小的 2^x
    - 7个host 3位host id是不够的，3位最多8 - 2 = 6个host，还得有一个网关
  - 确定host id h位，network id就是32-h位
  - 计算掩码部分不变，host id全1的广播地址，是这个网段广播ip
  - 下一个网段的ip是上一个网段广播ip+1

e.g：10.189.24.0/24分三个网段，一个100 host，两个10 host
- 100 host
  - 从 10.189.24.0 开始
  - 100 host至少103个地址，host id 7位
  - 网段地址 10.189.24.0
  - 掩码 25 位 10.189.24.0/25
  - 广播地址掩码部分不变，host id全是1 10.189.24.127
- 10 host
  - 从 10.189.24.128 开始
  - 10 host至少需要13个地址，host id 4位
  - 网段地址 10.189.24.128
  - 掩码 28 位 10.189.24.128/4
  - 广播地址 10.189.24.143
- 10 host
  - 从10.189.24.144 开始
  - 10 host至少需要13个地址，host id 4位
  - 网段地址 10.189.24.144
  - 掩码 28 位
  - 广播地址 10.189.24.159

![allocate subnet](/assets/img/post/IAP/allocate-subnet.png)

**网段地址肯定是偶数，广播地址肯定是奇数**

## 路由

确定下一跳，转发数据包

收到数据包
- 检查IP和子网掩吗
  - 和自己一个网段
    - 直接送达
  - 不和自己一个网段
    - 间接送达：查路由表，给下一跳
      - 最长前缀匹配：目标Host IP > 目标网段 > 默认网关
      - 没有match：报告错误，丢掉数据包

### Routing Table

信息来源
- 手动设置
- ICMP更新
- 动态路由协议配置

![routing-table](/assets/img/post/IAP/routing-table.png)

内容
- 目标IP：具体IP或网段
- 下一跳IP
- Flag
  - U：下一跳在线
  - G：Gateway 下一跳是路由器
  - H：Host 下一跳是host
  - D：这条信息是ICMP重定向创建的
  - M：这条信息是ICMP重定向修改的
- 走哪个interface发

路由表需要经常查，越短越好

### ICMP
![icmp-format](/assets/img/post/IAP/icmp-format.png)

- Type：大类。8 Echo Request，0 Echo Reply，11 Time Exceeded，3 Destination Unrechable。[完整列表](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol#Control_messages)
- Code：大类下的小类。比如Destination Unreachable有15种具体情况，3是端口不可达，比如端口没开;4是需要分块但是DF set了。
- Checksum：checksum先置0，用整个帧算checksum写在这
- Rest of Header：不同的类别有不同的格式
- Data：最多576 Bytes。包含产生错误packet的IP V4 header和至少前8 Bytes数据（假定传输层协议的端口号应该在前8 Bytes里）

![icmp-redirect](/assets/img/post/IAP/icmp-redirect.png)

#### 重定向
没必要发给我，直接给下一跳
- 路由器收到一个数据包，发现路由表里下一跳就在收数据包的端口上
  - 转发数据
  - 回ICMP redirect错误
- host更新路由表，后续流量不再绕一下
  - redirect有timeout
  - host不一定听redirect

![icmp-redirect-format](/assets/img/post/IAP/icmp-redirect-format.png)

#### 路由发现
ICMP route discovery

- solicitation：启动过程中host给 224.0.0.2 所有路由器的组播发ICMP router solicitation请网关信息
- advertisement：host收到路由信息选一个默认网关
  - 路由器回ICMP router advertisement
  - 固定时间间隔就发

![icmp-router-advertisement](/assets/img/post/IAP/icmp-router-advertisement.png)

### 静态路由
用于
- 网络规模小
- 没有冗余路径
- 和外界单点连接

路由信息来源
- DHCP
- 本地配置文件
- ICMP 重定向
- ICMP 路由发现

### 动态路由

- IGP：interior gateway protocols，AS内用，不管AS外的网络
  - eg
    - Routing Information Protocol (RIP)
    - Open Shortest Path First protocol (OSPF)
- EGP：exterior gateway protocols，AS间用
  - 每个AS至少有一个路由器负责AS见的路由
  - eg
    - Border Gateway Protocol(BGP)

代价：
- path length
- reliability
- delay
- bandwidth
- traffic
  - 可能导致路由震荡：大家都走现在流量最小的路径，更新之后这个路径就成流量最多的了
  ![oscillations](/assets/img/post/IAP/oscillations.png)
- communication cost

#### 路由算法
- distance vector routing：距离口耳相传
  - 只发自己到不同目标的最小代价，不转发别人的信息
    - split horizon：不告诉你我去哪经过你最近
  - 只知道去哪下一跳该走哪
  - 优点
    - 简单
    - 节省计算和带宽
  - 缺点
    - 收敛慢
    - Count to Infinity -> 互相count形成环路
- link state routing：网络拓扑口耳相传
  - 发自己的连接状态，并且转发别人的信息
  - 每个节点都知道整个网络拓扑
  - 优点
    - 收敛快
  - 缺点
    - 占用资源多
      - 计算：每个路由器重复计算路由表
      - 带宽：每个路由器的连接状态需要过每一根网线

#### RIP
Routing Information Protocol
![rip-v1-format](/assets/img/post/IAP/rip-v1-format.png)

![rip-v2-format](/assets/img/post/IAP/rip-v2-format.png)

- RIP，RIP-2，RIP-ng（IP v6）
- 520/UDP
- 代价只能是跳数，最多15跳，16跳代表unrechable
- 只存最优路径
- 触发路由信息发送
  - 定时发送
  - 路由表更新
    - 和邻居的代价发生变化 / 有新路由器入网
    - 收到邻居的路由信息，计算后发现更短路径
- 定时器
  - route-update：默认30s + 小的随机时间，大家错开
    - 倒计时结束就发自己的路由信息
  - route-invalid：默认180s
    - 过了这么长时间没收到下一跳消息
      - 标记这个路径inaccessible
      - 广播经过自己到目标的这个路径unreachable
        - 会往下传播告诉所有到达这个目标经过这个路径的路由器这个路径不可达了，但是不影响到达这个目标不走这个路径的路由器
      - 但是还会向这个下一跳转数据包：源在听到不可达之前还在发数据，能不能到都试着把这些送过去
  - route-hold-down：默认180s，至少route-update三倍
    - 听到一个路径unrechable后，route-hold-down时间内不更新这个路径
  - route-flush：默认240s，需要比route-invalid和route-hold-down都大
    - 经过route-flush删除这条路径信息

![rip-process](/assets/img/post/IAP/rip-process.png)

Count to Infinity

![count-to-infinity](/assets/img/post/IAP/count-to-infinity.png)

- Router B原来通过Router A到LAN A
- Router A到LAN A的连接宕掉了
- Router A听Router B的路由信息发现Router B有一条到LAN A 1跳的路径
  - Router A标记去LAN A下一跳Router B，距离2跳
- Router B发现下一跳Router A去LAN A的最短路径更新了
  - Router B标记去LAN A下一跳Router A，距离3跳
- ......
- Count to Infinity

避免Count to Infinity
- RIP里数到16路径unrechable
  - 缺点
    - 限制网络规模
    - 数到16还是要时间
- route-hold-down
  - 路径宕掉了通知所有最短路径包含这条路径的路由器
  - 计时期间路径不更新，两个人不许互相数
  - 其他的路径在倒计时期间被添加
- horizontal split
  - 不告诉你我去哪经过你最近

black hole：一个路由器可以说我到所有路由器的距离都很近，这样网络上很多路由都会经过自己

#### ospf
Open Shortest Path First，IP组播，协议号89
- 支持多种代价
- 支持多路径
  - 不同服务走不同路径保证质量
  - 负载均衡
- 收敛快

![OSPF-hierarchy](/assets/img/post/IAP/ospf-hierarchy.png)

- 分级路由
  - 级别
    - AS
    - Area：每个Area一个32 bit ID
      - 骨干Area：只有一个，ID 0.0.0.0
  - 路由器
    - AS边界
    - 区域边界：不完全包含在任何区域里
    - OSPF骨干：所有区域边界和把区域边界连起来的路由器
    - 内部路由器

消息
- Hello：发现邻居，监测邻居状态
- Database description
- LinkState
  - Request
  - Update
  - Acknowledgement

[//]: # (TODO: 这些消息的细节)

- adjacent：link state dataset相同

![OSPF-frame](/assets/img/post/IAP/ospf-frame.png)

[//]: # (TODO:ospf 帧结构)

迪杰斯特拉
- 已知最短路径的点是一个集合
- 每次找出和集合相邻的点中离源点最近的点，加入已知集合
  - 循环进行：O(n^2)
  - 优先队列：O(nlog(n))

![dij-1](/assets/img/post/IAP/dij-1.png)

![dij-2](/assets/img/post/IAP/dij-2.png)

![dij-3](/assets/img/post/IAP/dij-3.png)

![dij-4](/assets/img/post/IAP/dij-4.png)

![dij-5](/assets/img/post/IAP/dij-5.png)

![dij-6](/assets/img/post/IAP/dij-6.png)

![dij-7](/assets/img/post/IAP/dij-7.png)

![dij-8](/assets/img/post/IAP/dij-8.png)

![dij-9](/assets/img/post/IAP/dij-9.png)

#### BGP
Border Gateway Protocol，AS间路由协议。TCP/179

- 跟RIP一样是距离矩阵算法
  - 记录完整路径
- 记录所有的路径，不是只要最好的路径
- 两部分
  - External BGP：学习到其他AS的路径
    - 交换信息
    ![bgp-exchange](/assets/img/post/IAP/bgp-exchange.png)
      - 目标网络
      - 经过的AS
    - 政策
      - 告诉其他路由器什么
      - 如何选择路径
  - Internal BGP：将外部信息告诉自己AS里的路由器

  - 流量类型
  - 内部流量
    - 从AS内产生的
    - 目的地在AS内的
  - 通勤流量：只是从AS过。在AS外产生，目的地也在AS外
- AS类型
  - Stub AS：只连一个AS，死胡同
  - Multihomed AS：连两个或以上AS，但是不接通勤流量
  - Transit AS：接两个或以上AS，而且允许通勤流量


### MPLS
Multi-protocol label switching在二层和三层报头之间指定forward方法，可以在packet switch网络上给数据流固定路径

- 避免动态路由协议开销
- traffic engineering更高效利用带宽
- 绕开拥堵
- 区分对待不同服务，保证一些服务质量
  - 比如VoIP和best effort分开，给VoIP质量更好的线路
- 可以只经过信任路由器，更安全

### Traceroute

![traceroute-proces](/assets/img/post/IAP/traceroute-process.png)

没进行一次路由决策，IP包的TTL就-1。在到达host的时候TTL不减

发UDP数据包，TTL从1开始，每次 +1
- 途中的router回ICMP Time Exceeded，TTL expired in transit
- 目标设备回Destination Unreachable，Destination port unreachable

每个发出去的包目标端口号不同，看ICMP的payload可以知道是哪个包触发的，从而计算RTT


## UDP

将第三层的IP延伸到第四层，加入多播，端口号和一个checksum

特点
- 无连接，不可靠
  - 不保证送达
  - 不保证顺序
- 简单
  - 快

不可靠的协议上也可以做可靠的服务，应用层需要加入保证可靠的功能

### TFTP
Trivial File Transfer Protocol，服务器 69/udp 接控制，之后开一个动态端口传数据;客户端两个动态端口，一个控制一个数据。

![tftp frame format](/assets/img/post/IAP/tftp-frame-format.png)

![tftp process](/assets/img/post/IAP/tftp-process.png)
一块数据一个ack
- 客户端给服务器69/UDP发一个文件请求
- 服务器端发一个block数据
- 客户端一个ack
- 服务器端发下一个block
- 如果超时没有等到ack重发
- 反复直到发完

- 没有监权
- 没有加密


## TCP
Transmission Control Protocol，只支持单播，对上层暴露byte stream。ack的byte是已经成功接收的byte + 1，就是下一次期望接收的byte

帧格式

![tcp format](/assets/img/post/IAP/tcp-format.png)

- Source/Destination port number：发送和接收的端口号
- Sequence Number：这个报文第一个Byte在发送的数据流中是第多少Byte，纯ACK报文不占ID，也不需要被ACK
- Acknowledgement Number：接收端期待收到第几个Byte，收到了x就ACK x+1
- Data Offset/Header Length：TCP报头的长度，单位word（4 Bytes）
- Flags：1 set
  - ACK：说明ACK值是有用的
  - SYN：建立连接
  - FIN：中断连接
  - RST：出问题了终止连接，或者收到SYN的时候直接回RST拒绝连接
  - PSH：发送端告诉接收端应该尽快把数据交给上层，可能传输要结束了
- Window Size：rwnd，接受端在发送这个报文的时候最多还能收多少Byte
- Checksum：计算用到IP伪报头，TCP报头和payload
- Urgent Pointer：如果URG=1，指向Payload里紧急消息


![tcp state](/assets/img/post/IAP/tcp-state.png)

- 面向连接，可靠
  - 错误控制：每个数据包要ack，丢失或出错重发
    - 出错：任何一层checksum没过
      - 继续ack上一次的byte
    - 丢失：Retransmission Timer timeout重发
      - RTO（Retransmisson Timeout）应该比RTT大，但是在一个数量级
      ![rtt rto](/assets/img/post/IAP/rtt-rto.png)
      - RTO不足1s进位，最大可能有cap，一般60s
      - 重发帧的RTT不参与运算，重发帧也timeout了RTO*2
    - SACK：selective ack，不是所有host都支持
      - 还是ACK出错的byte
      - 同时SACK这个出错的包后面成功收到了什么
      - 避免一个包出错就卡住传输
  - 流控制：滑动传口避免接收缓冲区溢出
  - 拥塞控制：slow start，congestion avoidance，fast retransmission
- 全双工：双向可以同时通信


![tcp connection](/assets/img/post/IAP/tcp-connection.png)

建立连接
- 确认
  - maximum segment size：最大一个包多大，一般实际就发这么大
  - receiving buffer size：接受端buffer大小
  - initial sequence number：随即的
- 三次握手
  - SYN
  - SYN+ACK
  - ACK
- 拒绝连接用RST

断开连接
- 两端都可以断开连接
- TCP Half Close：一端还没发完另一端就可以先断
- 最后一个ACK之后要等两个maximum segment life，防止这个连接没到的数据包影响下一个连接
![terminate without timewait](/assets/img/post/IAP/terminate-without-timewait.png)
- 出现问题需要断开用RST flag

定时器
- Connection Establishment Timer：经过这么长时间连接还没成功建立放弃
- Retransmission Timer：这么长时间没收到ACK就重发，需要比RTT大，但是在一个数量级。每次重发*2，最多64s
- Delayed ACK Timer：这个timer timeout前如果要发数据顺便把ACK带回去，如果不发就等到这个timer timeout之后再发ACK
- Persist Timer：快发慢收的时候，如果收的缓冲区满了，每次这个timer到0问一下缓冲区有空没。时间5～60s，指数退避
- Keepalive Timer：KeepAlive连接，每这么长时间问问对面还在不在
- Two Maximum Segment Life Wait Timer：关闭连接之后听两个MSL
  - 可以重发没收到的ACK
  - 避免上一个连接的数据包被下一个连接收到

流控：负反馈控制，减少丢包
- 传输速度限制
  - 接收速度：faster sender slow receiver
  - 网络瓶颈：路由器接收buffer满了只能丢掉数据包
- 反馈
  - 推测：丢包说明路径上瓶颈不行了
  - 收到反馈
    - 接收方RWND（receiver window）：没被ACK的in flight数据不能超过RWND
    ![rwnd](/assets/img/post/IAP/rwnd.png)
    - 路由器反馈丢包，可以建议发送速度
      - ECN（Explicit congestion notification）
      ![ecn](/assets/img/post/IAP/ecn.png)
      _ecn_
- 参数
  - 接收方
    - rwnd（receiver window）：接收buffer还有多少byte可以用，发给发送方
    - MSS：一个数据包最大多大
  - 发送方
    - cwnd（congestion window）：本地的变量，不发送，in flight < min(rwnd, cwnd)
    - ssthresh（slow start threshold）：cwnd小于这个数指数增长，大于这个数线性增长
- 过程
![tcp congestion control](/assets/img/post/IAP/tcp-congestion-control.png)
  - 初始状态
    - ssthresh = 65535
    - cwnd = MSS
  - slow start & congestion avoidance：每次收到ACK
    - 当 cwnd < ssthresh：cwnd = cwnd + MSS
    - 当 cwnd > ssthresh：![cwnd over ssthresh](/assets/img/post/IAP/cwnd-over-ssthresh.png)
  - fast retransmit：收到三个相同的ACK，不等RTO就重发
  - fast recovery
    - 收到三个相同的ACK
      - ssthresh = max[2 * MSS, min(cwnd , awnd )/2]
      - cwnd = ssthresh + 3 * MSS
    - 之后每收到一个重复的ACK
      - cwnd = cwnd + MSS
    - 收到一个新的ACK
      - cwnd = ssthresh + segsize

![congestion control](/assets/img/post/IAP/congestion-control.png)


交互式数据流
- 用于telnet，ssh这种
- 过程
![tcp interactive](/assets/img/post/IAP/tcp-interactive.png)
  - 用户按键
  - 字符发给服务器
  - 服务器将这个字符发回来，同时ACK
  - 客户端在屏幕上展示这个字符，同时给服务器ACK
- 减少发包
  - 延迟ACK：收到数据包之后等待K ms
    - K ms过程中需要和另一方通信，就顺便把ACK带过去
    - K ms内没有顺风车，在K ms时发ACK
  - Nagle algorithm
    - 本地cache一小块用户输入，收到服务器ACK之后将cache的输入一块发过去
    - 增加延迟，减少发包

### FTP
File Transfer Protocol，服务器20/TCP数据，21/TCP控制，客户算两个动态端口。

![tfp process](/assets/img/post/IAP/tfp-process.png)

![FTP COMMANDS](/assets/img/post/IAP/ftp-commands.png)

PORT：PORT的一方想用什么端口，Server和Client都可以PORT
![ftp reply](/assets/img/post/IAP/ftp-reply.png)

模式
- passive
- active

## 多播


![multicast](/assets/img/post/IAP/multicast.png)

类型
- 单播：1对1
- 多播：1对多，多对多。一个内容的数据包在一个连接上只过一次
- 广播：1对所有

用途
- 不知道具体目标地址，发给一类目标 eg：路由
- 发同样的数据给多个host eg：直播

关键
- 地址：所有人都可以给组播地址发，所有IP包会发给组内所有host
  - ip：class d，1110开头，224.0.0.0 ～ 239.255.255.255
  - 预留地址
  ![reseved multicast address](/assets/img/post/IAP/reseved-multicast-address.png)
  - ipv4 -> mac：不需要ARP
  ![multicast mac](/assets/img/post/IAP/multicast-mac.png)
  ![wellknown multicast](/assets/img/post/IAP/wellknown-multicast.png)
    - ipv4组播固定0x01-00-5e开头
      - 01:00:5e:00:00:00 ~ 01:00:5e:7f:ff:ff
    - 25位是0
    - 后23位是组播IP后23位
      - class d host id前5位没写进组播MAC
      - 2^5给组播IP共享一个组播MAC：在三层进行过滤

- 群组管理
  - IGMP group membership table
    - 对于每个端口，记哪个组播地址有成员在这个端口上
  - host membership query
    - 路由器发给所有能组播的host（224.0.0.1），TTL 1，60s一次
      - 地址 0.0.0.0 query所有组的成员，也可以query具体一个组
    - host对每个自己属于的组回复一个 IGMP report 给这个组播地址
      - 随即延迟，大家不一起发
      - 如果收到发给自己组的一个report就不再发了
      - 最开始入组的时候没人问就发一个report
  - 离开组播
    - v1什么都不做，60s后membership query不回复就离开了
    - v2可以通知所有路由器
  - 开始发送：flood and pruning
    - 根据RPF flood组播数据
    - 不需要的路由器把自己prune掉
  - IGMP snooping：部署在二层，一直听query，report和leave消息
    - 知道端口的组播情况
    - 一开始不需要flood数据，减少负载
- 路由
  - 三个级别
  ![multicast routing](/assets/img/post/IAP/multicast-routing.png)
    - 局域网网段：IGMP
    - AS内
      - DVMRP：Distance Vector Multicast Routing Protocol
        - source based, RPF, flood and pruning
      - MOSPF：Multicast Open Shortest Path First
        - source based
      - CBT：Core based tree
        - group shared，不广播第一个组播包，适合密集组播网络
      - PIM：Protocol Independent Multicast
        - 同时支持source based和group shared，在group基础上给流量多的source source based
    - AS间：MBGP
    - MBONE：用tunnel连接island。tunnel间p2p，单播数据;island内用上面的方法
  - 路由树
  ![multicast tree](/assets/img/post/IAP/multicast-tree.png)
    - source based：每个一sender一个树
      - 优缺点
        - 流量分布更均匀
        - 每个source发出来的包都走最短路径，延迟小
        - 每个路由器需要给每个source维护一棵树
      - 方法
        - Dijistra
        - Reverse Path Forwarding（RPF）：从到source的端口将组播包转到所有其他端口
          - 路由表需要稳定
          - 上行和下行需要走同一个线路
            - 否则组播不一定是最短路径
            - 单向连接可能发不过去
    - group shared：一个组共享一个树
      - 优缺点
        - 组播流量集中
        - 对每个sender不一定是最优的树，延迟大
        - 维护一棵树，开销小
        - Rendezvous Point的选择影响性能
  - pruning：收到一个组播包发现没人需要，退出组播树
    - 向上一级发pruning
    - 上一级收到后停止转发，看自己有没有其他需要转发的地方，没有的话继续往上发pruning

应用
- DNS：组播query，不能递归query
- RIPv2：只向RIP路由器发消息
- SNMP
- ICMP：向所有路由器组播路由发现
- IP多媒体：直播，游戏


### IGMP
Internet Group Management Protocol，3版

功能
- 加入组播
- 离开组播（v2）
- 查询成员
- 发送成员报告

## 实时服务
类别
- 直播准备好的内容
  - 类似cdn，直接把片子整个推给用户，也可以调进度
- 直播实时上传内容
- 实时互动

特点
- 对错误不敏感
- 延迟敏感
  - 延迟要小
  - 延迟要稳定
    - 否则延迟就要大
