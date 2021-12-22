---
title: Docker容器的详细配置来刷课....
date: 2021-12-22 20:39:02
tags:
---
Teacher建议我学学用Docker来搭配渗透环境，于是大概的了解了一下Docker

正好在Github上发现了一个项目是可以用Docker来部署的 而且实用性比较高，所以尝试搞一下

项目原地址：
```
https://github.com/TechXueXi/TechXueXi
```
tips:如果学会了的话觉得实用不要忘了给作者点个star

Dokcer 运行方式适用于服务器，云主机，路由器，树莓派等 7x24 运行的设备上。因为我正好有个云服务器没啥事可以干，所以正好拿过来运行一下

附上一张个人觉得很实用的docker命令
```
   docker -v
   systemctl start docker  //开启
   systemctl stop docker //关闭
   systemctl enable docker //开机启动
   service docker restart  //重启
   service docker stop //关闭
```



基础概念的话，简单阐述一下：
镜像：是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
容器：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
仓库（Repository）：仓库（Repository）类似Git的远程仓库，集中存放镜像文件。

OK，基础概念阐述完毕

首先是拉取镜像，拉取的话我个人推荐去Docker Hub上拉取，上面镜像很多
拉取指令是：docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

该项目的容器地址是：https://hub.docker.com/u/techxuexi/

首先

查看 CPU 架构

在控制台中输入 hostnamectl：

如果显示 Architecture: x86-64 ，请使用AMD64 版本的仓库

如果显示 Architecture: arm64 ，请使用ARM64V8 版本的仓库

如果显示 Architecture: arm32 ，请使用ARM32V7 版本的仓库

因为我服务器是X86-64的 所以就用AMD64版本了。
docker pull techxuexi/techxuexi-amd64:latest
···安装的过程中遇到了如下问题
放链接
https://blog.csdn.net/weixin_45496075/article/details/109123709
这篇文章解决了我Docker指令突然失效的问题 万分感谢

修好了Docker后 我成功pull下来了镜像

然后此时选择多种方式

因为之前就想用Server酱但是一直没试过，所以正好配置一下。会员八块钱不贵。开冲！

Server酱的官方地址
```
https://sct.ftqq.com/
```

订阅会员后，继续操作


我的最终命令是
docker run \
  -e "AccessToken=**************" \
  -e "ZhuanXiang=True" \
  -e "Pushmode=3" \
  -e "Scheme=Scheme = https://techxuexi.js.org/jump/techxuexi-20211023.html?" \
  -v /volume1/docker/xuexi/user:/xuexi/user:rw \
  -d --name=techxuexi --shm-size="2g" techxuexi/techxuexi-amd64:latest
然后现在开启了舒服的刷课模式。
本次学会了server酱和Docker的配置。特地分享
中间过程中用到的指令
```
Docker ps  //查看正在运行的指令
Docker restart +id//重启
```
假期会研究如何用Docker搭建一下渗透环境，复现一些CVE里面的漏洞
