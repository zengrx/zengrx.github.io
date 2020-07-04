---
title: 我能吞下玻璃而不伤身体
date: 2018-05-05 14:01:00
tags: 
- Arch Linux
- Linux
---

很显然，这是一篇梗文

从16年11月开始写S.M.A.R.T.项目的主框架，直到答辩结束都一直没太敢更新arch。一来是怕万一滚挂了什么奇奇怪怪的依赖，恢复一下还是要花些时间的；二来呢，沉迷叛乱之后很少切linux了，工作日下班回来基本不太开电脑，周末就在 push push push 中度过

所以时隔一年半，这次 sudo pacman -Syu 果然还是出了不少问题：fcitx输入法全挂、gnome-control-center点不开，倒是gnome扩展让人感到很稳定，反正每次更新都挂每次都要重装。

首先是终端字体堆叠，等宽设置成文泉驿 Micro Hei Mono Rregular 基本ok，不过貌似影响了终端窗口大小

大问题是fcitx挂了，以往输入法一直都在，所以碰到什么问题随手一搜，解决到能用就行从不做记录。而这次的现象是fcitx中没有候选输入法可以选，怎么样都调不出输入设置。Wiki是个好东西，所以引一下arch fcitx wiki中所说：
##### 首先诊断问题所在
>当你遇到任何 fcitx 有关的问题，比如 ctrl+space 在有的程序中不能工作，首先应该用 fcitx-diagnose 命令诊断问题的原因。 fcitx-diagnose 会列出所有 fcitx 正常运行所需的前提条件，从输出结果中通常可以找到问题的原因。 在网上（比如在 irc 或者论坛里）询问别人关于 fcitx 配置的问题时，也请首先提供你的 fcitx-diagnose 输出结果（比如贴到 pastebin 服务），这将加速别人帮你找到问题所在

仔细看了下fcitx-diagnose输出，发现之前对GTK和QT的设置居然一直都是错的( ╯-_-)╯┴—┴，果然是以前盲从网上解法不认真看官方wiki，在xim里多写了一对双引号。。。

系统设置打不开，后来用 ps -ef | grep xxx 强杀之后重启ok

记录下常用的几个gnome扩展 Dash to Dock， System Monitor， NetSpeed Master