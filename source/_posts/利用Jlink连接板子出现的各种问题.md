---
title: Keil51高版本与Jlink驱动识别
date: 2021-1-27 17:41:19
tags:
---
最近在工作时用到了Jlink仿真器烧录代码，只能说过程是一言难尽，在自己工作过程中也是学到了一些新的东西，首先就是keil在debug的时候必须选定串口来上传代码，上传的方式必须要区分是JTAG还是SWD，这里的话可以用Jlink command来进行操作，打开Jlink，输入指令如下
```
conenct
```
第一次输入后显示的代码应该是如下
```
Please specify device / core. <Default>: STM51FXXX
```
进行一个默认设备的选择
然后如果没有多个设备可以直接回车来确定，这里可以通过Ulink来查看是否识别到了板子的内核，如果识别到了可以继续操作，没识别到的话就要看一下是不是引脚坏了或者是接的线路不对，如果确认都没问题的话可以考虑是不是flash写入保护的问题。
连接成功后打开Keil5的debug，选择settings,模式选择Jlink/J-Trace，方式根据实际情况选择是Jtag还是SWD，我这里是SWD，选择后观察右侧是否显示出来SW Device的SWD id。
希望以后不要再踩坑