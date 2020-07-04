---
title: 在Linux中使用深信服VPN
date: 2020-06-25 17:31:08
tags:
- network
- VPN
- Linux
---

这是一篇旧文章，在远程办公期间写了个操作手册给大家配置虚拟机代理linux主机网络。疯狂加班了半年，最近难得有空，整理到博客里。

<!--more-->

公司用了深信服的easyconnect在公网代理专用网络，win和mac用起来基本无压力，但是对用linux做主力机的用户也太不友好了，尝试了几条路，浏览器跑java applet，这个属实不靠谱，都21世纪了，害跑个锤子java虚拟机，支持java的浏览器都难找（flash下一个死的就是你啦），还要降jdk版本，搞乱了安卓源码编译环境那就得不偿失了。后来又找到了一个deb包，但是装上去提示服务器版本不匹配，并不能用。或许深信服觉得linux客户端是个鸡肋，与其花大力气开发维护，做得不好被diss技术水平，做好了被疯狂逆向挖漏洞，不如让linuxer自己折腾呢，反正最终他们都能搞定。╮(╯-╰)╭确实是这样的嗷。

问了组里的几个同事，大部分图省事直接在win虚拟机里给个共享目录到主机，还有做了端口转发，和我想要的完整内网环境都不太搭。因为网线需要留着telnet调板子，计划从WLAN爬到内网，毕竟七七八八的服务器不知道啥时候就得用。那么网络拓扑（伪）大致是这样的：
![network topographic map](https://ftp.bmp.ovh/imgs/2020/06/e47cabed6a627dae.png)

我的环境是Ubuntu 16.04 Host + Win 7 VM，基本策略为主机与虚拟机走桥接，虚拟机中用来共享网络的网卡设置为Host-only模式，从而将主机与虚拟机中的VPN网络打通。

首先确认windows虚拟机能够通过VPN访问到内网，这个无需多说。可以在网络管理中看到多出了一个网络连接，重命名为SangforVPN。确认完毕后虚拟机关机，接下来在virtualbox中对虚拟机网络进行配置，**网卡1**以**桥接网卡**的方式连接，我是打算用wifi的，所以选择wlp5s0，这里根据自己的需求和ifconfig去设置就可以了。
<center>配置桥接网卡</center>![bridge config](https://ftp.bmp.ovh/imgs/2020/06/fd17fcceec7bb872.png)

接下来在virtualbox应用设置中选择**管理**，点击**全局设定**，选择**网络**栏，在**仅主机(Host-Only)网络**页中添加vboxnet0网络，并做下图的配置：
![vboxnet0 config](https://ftp.bmp.ovh/imgs/2020/06/df594c778a63868b.png)

在**仅主机网络明细配置**窗口的**主机虚拟网络界面页**中将默认的IPv4地址修改为192.168.137.xxx。因为windows系统网络共享自身策略的原因，共享网络后，Host-Only网卡IP会自动变为192.168.137.1，在这里提前设置方便后续操作。剩下的DHCP我随手配了一下，不配置应该也没啥问题。参考配置如下：
![vboxnet0 DHCP config](https://ftp.bmp.ovh/imgs/2020/06/7eea7680f44fc119.png)

配置完成后，可以从linux主机的ifconfig中看到刚才添加的vboxnet0已经起来了。这个vboxnet0应该是跟着virtualbox的，开机默认没有up，配置好后每次打开virtualbox就会起来。

重新回到virtualbox的网络设置，在**网卡2**页中选择**仅主机网络**，经过了之前的配置，可以选择vboxnet0作为虚拟机中Host-Only的interface。具体配置如下图：
![vbox hostonly config](https://ftp.bmp.ovh/imgs/2020/06/55cbd04c95a76023.png)

virtualbox相关的网络配置添加完成，开机进入windows虚拟机。在网络和共享设置中更改适配器设置，可以看到网络连接中新增了两个网络连接（我这里是网络连接1和网络连接2）。对应重命名为bridge和host-only。右键将SangforVPN的网络共享给Host-Osnly网络，注意需要关闭windows防火墙，否则测试时无法被主机ping通。
![win net config](https://ftp.bmp.ovh/imgs/2020/06/4db6922a9e480a2e.png)

之前有提到，共享网络后，Host-Only的IP就自动变为了192.168.137.1。此时linux主机与win虚拟机的bridge与Host-Only可以互ping通，分别对应了主机的无线网络wlp5s0与虚拟机的仅主机虚拟网络vboxnet0。胜利就在眼前。

Linux中输入route -n查看路由表，因为没有接网线，所以只会看到无线路由器网关这类的网络信息，对应内网IP的网关不配置的话，所有去内网的报文就无法被转发到下一跳。所以接下来需要配置路由表。
修改路由表，执行如下命令让192.168.137.1做网关转发去到内网的报文：
> sudo ip r add 192.xxx.xxx.xxx via 192.168.137.1

![route setting](https://ftp.bmp.ovh/imgs/2020/06/498cadcf57a0ff98.png)

我把内网IP和我自己的wifi网关涂掉了，可以看到倒数第二行Host-Only的IP被作为了网关，配置时根据自己设备的实际内容填写即可。因为指定了具体的ip地址，所以子网掩码是8个F。添加网关后，所有数据包都可以准确找到下一跳目标。

该路由表设置不会保存，每次重新开机后打开virtualbox让vboxnet0起来，再执行一次ip r add命令即可。这里只做了一个内网IP的转发，需要访问更多IP地址也可以继续设置。

Linux主机可以顺利ssh
![ssh success](https://ftp.bmp.ovh/imgs/2020/06/67a4d6003d522662.png)
![sshfs success](https://ftp.bmp.ovh/imgs/2020/06/f3ffeb322bb22070.png)

#### Reference
参考了一篇博客，也是在gitpage上，不过忘了保存一时半会儿找不到地址了。这哥们有点意思，win主机在linux虚拟机上写代码，非常抗拒在主机上装深信服软件，于是开了个win虚拟机装vpn把网络共享给linux虚拟机🐂🍺