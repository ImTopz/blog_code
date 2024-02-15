# Ensp实验学习（华为ICT）

## 1.前言

笔者希望通过ensp来熟悉一下华为设备的操作流程，并顺便搭建一个园区网络。在这里因为笔者的电脑是MacBookPro（M1），对应的芯片架构为Arm，所以需要通过todesk或者Rdp远程到~~学妹~~(学弟)电脑上进行操控，有v6的话会舒服很多，话不多说，实操开始。

## 2.安装

**所需环境与软件：**Windows11 Server，virtualbox(必须是老版本的VB 5.x.x ) ，wireshark , winpcap ,todesk / rdp

安装教程（很详细，记得按照教程顺序安装）

```
https://www.ictstu.com/%E7%BD%91%E7%BB%9C%E6%8A%80%E6%9C%AF/network01.html
```

<img src="/Users/zder/Library/Application Support/typora-user-images/image-20240210184440479.png" alt="image-20240210184440479" style="zoom:50%;" />

安装好后进行一个ensp的启动！

<img src="/Users/zder/Library/Application Support/typora-user-images/image-20240210214853012.png" alt="image-20240210214853012" style="zoom:50%;" />

## 3.搭建一个园区网络

### 任务背景：

我现在是某公司网络工程师，总人数200人左右，现在公司有四个部门，分别是研发部（100人，程序员多），市场部（50人吧）、行政部30人及财务部20人，现在我作为ICT工程师，来给公司设计一个网络，我们通过华子的ensp来实现，任务清单如下：

- [ ] 1.公司各部门之间正常通信，进行资源通信与信息通信，避免出现**广播风暴**，读者可以自行了解一下相关概念，在这里简单讲解一下（见拓展1）
- [ ] 2.
- [ ] 

拓展：交换机是二层设备（即数据链路层），只负责处理MAC帧数据，而我们知道一个mac帧有源MAC帧和目的MAC帧，交换机中维护一张表

<img src="/Users/zder/Library/Application Support/typora-user-images/image-20240211105728010.png" alt="image-20240211105728010" style="zoom:50%;" />

### 3.1 子网划分

<img src="/Users/zder/Library/Application Support/typora-user-images/image-20240211234753848.png" alt="image-20240211234753848" style="zoom:50%;" />

<img src="/Users/zder/Library/Application Support/typora-user-images/image-20240213224532948.png" alt="image-20240213224532948" style="zoom:50%;" />

将pc1和pc2配置好之后ping一下，此时用 **dis mac-address**指令可以看到mac地址表

#### 给路由器设置网关

配置GigabitEthernet口（千兆网口），这里的话**FastEthernet0/0/1是100Mbit的/s** **Ethernet是10Mbit/s的**

这个接口属于二层口，所以不能设置Ip

#### 实验命令

在**路由器**上配置指令

```
dis mac-address
```

```
int GigabitEthernet 0/0/0
```

```
ip address 192.168.1.254 24
```

```
dis cur # 查看全局的状态
```

### 3.2 VLAN划分

先看看概念 什么是vlan，什么是trunk

```
https://info.support.huawei.com/info-finder/encyclopedia/zh/VLAN.html
```

<img src="/Users/zder/Library/Application Support/typora-user-images/image-20240215160502984.png" alt="image-20240215160502984" style="zoom:50%;" />

<img src="/Users/zder/Library/Application Support/typora-user-images/image-20240215160907999.png" alt="image-20240215160907999" style="zoom:50%;" />

简单说一下，首先vlan可以划分不同的网络域，如果黑客拿下VLAN10，没办法影响VLAN20，主要作用是隔离广播域

静态VLAN 动态VLAN（近几年兴起SDN差不多）

