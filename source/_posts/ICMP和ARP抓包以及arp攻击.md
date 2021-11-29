---
title: 内网下wireshark进行抓包 抓取ARP包以及ICMP包并对其进行分析
date: 2021-11-27 16:19:21
tags:
---

ARP包：ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP协议。主机发送信息时将包含目标IP地址的ARP请求广播到局域网络上的所有主机，并接收返回消息，以此确定目标的物理地址；收到返回消息后将该IP地址和物理地址存入本机ARP缓存中并保留一定时间，下次请求时直接查询ARP缓存以节约资源。地址解析协议是建立在网络中各个主机互相信任的基础上的，局域网络上的主机可以自主发送ARP应答消息，其他主机收到应答报文时不会检测该报文的真实性就会将其记入本机ARP缓存；由此攻击者就可以向某一主机发送伪ARP应答报文，使其发送的信息无法到达预期的主机或到达错误的主机，这就构成了一个ARP欺骗

ICMP包：icmp数据包是一种网络通信数据包。icmp是“Internet Control Message Protocol”（Internet控制消息协议）的缩写。它是TCP/IP的一个子协议，用于在IP主机、路由器之间传递控制消息。icmp控制包是指用于探查网络通不通、主机是否可达、路由是否可用等网络问题的消息。
1.实验环境
系统：win11预览版
网络：GWifi网络
工具: wireshark win版本
下载地址：https://www.wireshark.org/

2.实验操作
（1）首先打开wireshark工具，一开始会进入网络分析器选择，选择其中一个 网络流量波动最大的 （因为本人实验环境是wlan，所以选择wlan）
（2）打开cmd,
输入
```
ipconfig -all
```
来查看本机的Ip情况 ，查看到网关是10.19.1.1,于是对网关进行ping命令：
```
ping 10.19.1.1
```
观察wireshark中包，在筛选栏中输入arp || icmp 指令，即可筛选出来抓到的包文，因为win下的ping命令默认执行四次，所以可以抓到八个ICMP的包，每两个对应一次是reply（回应）,一次是request(请求)。
其中IPV4（0x0800) 点开包后可以查看一系列的信息 比如frame帧是多少等等。
ARP包如果抓取不到，就用超级管理员权限打开cmd 输入以下指令
```
arp -a//查询当前arp缓存表
arp -d//清除arp缓存表
```
这时候就可以看到对应的arp包
对应点开arp包 如果src里面的地址是全f代表是广播（broadcast） 如果不是全f，代表是单播。可以查看arp包的各种信息，放在图片里面了。



既然说到了arp，那么就用Ubuntu来进行一次arp欺骗（让舍友断网的小技巧） tips:因为校园网有隔离，所以我在家实验的
实验环境:ubuntu 18
一个无线网卡
所用到的工具：disniff arpspoof


实验步骤：
（1）打开terminal，输入ifconfig来查看本机ip。查看到本机ip是192.168.1.104。
ok那么我需要判断谁在上网。我肯定是不能去直接问，所以用fping工具来扫描
安装指令如下
```
sudo apt install fping
```
安装后sudo -i
进入root下,输入
```
fping -asg 192.168.1.1/24
```
进行扫描，扫描完成后，对局域网中存活主机的ip
在root下使用arpspoof，具体的命令格式为 ‘arpspoof -i 网卡 -t 目标 网关’
比如说我的就是
arpspoof -i ens33 -t 192.168.1.101 192.168.1.1
就可以让目标机连不上网络了
如果有兴趣可以继续百度以下如何用ettercap劫持http协议下网站的信息。
本次实验到此结束//