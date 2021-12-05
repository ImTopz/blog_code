---
title: frp内网穿透
date: 2021-12-4 21:04:02
tags:
---
从公网中访问自己的私有设备向来是一件难事儿。
自己的主力台式机、NAS等等设备，它们可能处于路由器后，或者运营商因为IP地址短缺不给你分配公网IP地址。如果我们想直接访问到这些设备（远程桌面，远程文件，SSH等等），一般来说要通过一些转发或者P2P组网软件的帮助。
我有一台计算机位于一个很复杂的局域网中，我想要实现远程桌面和文件访问，目前来看其所处的网络环境很难通过简单的端口映射将其暴露在公网之中，我试过这么几种方法：

    远程桌面使用TeamViewer。可用，但需要访问端也拥有TeamViewer软件，不是很方便，希望能使用Windows自带的远程桌面。且TeamViewer不易实现远程文件访问。
    使用蒲公英VPN软件进行组网，可用，但免费版本网络速度极慢，体验不佳，几乎无法正常使用。
    使用花生壳软件进行DDNS解析，可用，但同第二点所述，免费版本有带宽限制，无法实际使用。
    搭建frp服务器进行内网穿透，可用且推荐，可以达到不错的速度，且理论上可以开放任何想要的端口，可以实现的功能远不止远程桌面或者文件共享。
    1. frp是什么
       简单地说，frp就是一个反向代理软件，它体积轻量但功能很强大，可以使处于内网或防火墙后的设备对外界提供服务，它支持HTTP、TCP、UDP等众多协议。我们今天仅讨论TCP和UDP相关的内容。
    2.所需要的环境
        搭建一个完整的frp服务链，我们需要：
       （1）VPS一台（也可以是具有公网IP服务器||或者是一台实体机）
       （2）访问目标设备（就是你最终要访问的设备，例如可以是树莓派）
       （3）简单的Linux基础（会用cp等几个简单命令）
3.操作
首先下载对应版本的frp上传到linux服务器
```
wget https://github.com/fatedier/frp/releases/download/v0.22.0/frp_0.22.0_linux_amd64.tar.gz
```
解压
```
tar -zxvf frp_0.22.0_linux_amd64.tar.gz
```
文件夹换一个名字
```
cp -r frp_0.22.0_linux_amd64 frp
```
进入该目录
```
cd frp
查看文件   ls -a
```
我们只需要关注如下几个文件

    frps
    frps.ini
    frpc
    frpc.ini

前两个文件（s结尾代表server）分别是服务端程序和服务端配置文件，后两个文件（c结尾代表client）分别是客户端程序和客户端配置文件。
因为我们正在配置服务端，可以删除客户端的两个文件
```
rm frpc
rm frpc.ini
```
然后修改配置文件
 bind_port”表示用于客户端和服务端连接的端口，这个端口号我们之后在配置客户端的时候要用到。
“dashboard_port”是服务端仪表板的端口，若使用7500端口，在配置完成服务启动后可以通过浏览器访问 x.x.x.x:7500 （其中x.x.x.x为VPS的IP）查看frp服务运行信息。
“token”是用于客户端和服务端连接的口令，请自行设置并记录，稍后会用到。
“dashboard_user”和“dashboard_pwd”表示打开仪表板页面登录的用户名和密码，自行设置即可。
“vhost_http_port”和“vhost_https_port”用于反向代理HTTP主机时使用，本文不涉及HTTP协议，因而照抄或者删除这两条均可。
至此，我们的服务端仅运行在前台，如果Ctrl+C停止或者关闭SSH窗口后，frps均会停止运行，因而我们使用 nohup命令将其运行在后台。
不知道可不可以用screen命令新开一个进程 应该也可以，不过建议百度一下nohup。
    同样地，根据客户端设备的情况选择相应的frp程序进行下载，Windows下下载和解压等步骤不再描述。
假定你下载了“frp_0.22.0_windows_amd64.zip”，将其解压在了C盘根目录下，并且将文件夹重命名为“frp”，可以删除其中的frps和frps.ini文件。
用文本编辑器打开frpc.ini，与服务端类似，内容如下。
[common]
server_addr = x.x.x.x
server_port = 7000
token = won517574356
[rdp]
type = tcp
local_ip = 127.0.0.1           
local_port = 3389
remote_port = 7001  
[smb]
type = tcp
local_ip = 127.0.0.1
local_port = 445
remote_port = 7002
配置完成frpc.ini后，就可以运行frpc了
frpc是不能双击运行的
使用命令提示符或Powershell进入该目录下
cd C:\frp
并执行
./frpc -c frpc.ini
运行frpc程序，窗口中输出如下内容表示运行正常。
```
2021/12/04 16:14:56 [I] [service.go:205] login to server success, get run id [2b65b4e58a5917ac], server udp port [0]
2021/12/04 16:14:56 [I] [proxy_manager.go:136] [2b65b4e58a5917ac] proxy added: [rdp smb]
2021/12/04 16:14:56 [I] [control.go:143] [smb] start proxy success
2021/12/04 16:14:56 [I] [control.go:143] [rdp] start proxy success
```

客户端后台运行及开机自启

frpc运行时始终有一个命令行窗口运行在前台，影响美观，我们可以使用一个批处理文件来将其运行在后台，而且可以双击执行，每次打开frpc不用再自己输命令了。
在任何一个目录下新建一个文本文件并将其重命名为“frpc.bat”，编辑，粘贴如下内容并保存。
```
@echo off
if "%1" == "h" goto begin
mshta vbscript:createobject("wscript.shell").run("""%~nx0"" h",0)(window.close)&&exit
:begin
REM
cd C:\frp
frpc -c frpc.ini
exit
```
直接双击运行这个bat文件就可以运行并且隐藏窗口。非常实用
frp具有非常多的应用 比如说远程办公，公网访问内网设备等等......