---
title: Feistel密码结构
date: 2019-05-13 23:45:42
tags:
- cryptography
- C/C++
- Feistel
---

Feistel作为现代密码学中构造分组密码的常用对称结构，我很肯定老师在课上重点分析过。对于这个结构，记忆中一直是模糊的位或、16轮、F函数和纠缠在一起的交叉箭头。想来自己的memory palace也是够奇怪了，居然把feistel和round绑在了一起。直到前段时间无聊搞搞ECU通信加密TEA算法玩，看到图解分析的“1 round”才猛然想起feistel，却又记不起基础理论和实现细节。可能当初手算AES shiftrow mixcolumn留下的心里阴影太大，轰塌了DES大楼，行移位列混淆轮密钥加真<sup>TM</sup>琅琅上口。一般教材进入现代密码学首先会介绍分组密码，尤以feistel密码结构为例，进而讲解DES的设计原理。如DES中所做的代换为了使密文唯一而可逆；扩散（P盒）能够弥补古典密码难以抵抗统计分析的弱点；混淆（S盒）则通过打乱密文与密钥的关系来保护密钥。文章中不打算详细分析分组密码，仅对feistel结构及用到相似设计的算法理解做一点记录。

<!--more-->

Feistel密码的命名来自分组密码加密技术的先驱Horst Feistel，这是一条加密方案设计的通用原则，而并非特定的密码方案。一个典型的加解密示意如下图：
![Feistel struct](https://i.bmp.ovh/imgs/2019/06/33fb9c16eb1d9580.png)

明文P被分为等长的左右两部分，右半部分R<sub>0</sub>与该轮子密钥K经过F函数运算后，再与左半部分L<sub>0</sub>做异或运算，最终结果作为下一轮的右半部分输入R<sub>1</sub>；同时右半部分R<sub>0</sub>直接作为下一轮左半部分输入L<sub>1</sub>，这样的一次运算称为一轮。以2 round 1 feistel来描述这个算法，为描述统一增加最后一次左右调换。
加密运算：        
L<sub>1</sub> = R<sub>0</sub>        
R<sub>1</sub> = L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>)        
L<sub>2</sub> = R<sub>1</sub> = L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>)        
R<sub>2</sub> = L<sub>1</sub> &oplus; F(R<sub>1</sub>， K<sub>1</sub>) = R<sub>0</sub> &oplus; F(L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>)， K<sub>1</sub>)        
L<sub>3</sub> = R<sub>2</sub> = R<sub>0</sub> &oplus; F(L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>)， K<sub>1</sub>)       
R<sub>3</sub> = L<sub>2</sub> = L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>)        
解密时子密钥倒序使用：        
L<sub>3</sub> = R<sub>2</sub> = L<sub>1</sub> &oplus; F(R<sub>1</sub>， K<sub>1</sub>) = R<sub>0</sub> &oplus; F(L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>)， K<sub>1</sub>)         
R<sub>3</sub> = L<sub>2</sub> = R<sub>1</sub> = L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>)        
L<sub>2</sub> = R<sub>3</sub>        
R<sub>2</sub> = L<sub>3</sub> &oplus; F(R<sub>3</sub>, K<sub>1</sub>) = (R<sub>0</sub> ~~&oplus; F(L<sub>0</sub> &oplus; F(R<sub>0</sub>， K<sub>0</sub>), K<sub>1</sub>)) &oplus; F(L<sub>0</sub> &oplus; F(R<sub>0</sub>, K<sub>0</sub>), K<sub>1</sub>)~~ = R<sub>0</sub>        
L<sub>1</sub> = R<sub>2</sub> = R<sub>0</sub>        
R<sub>1</sub> = L<sub>2</sub> &oplus; F(R<sub>2</sub>, K<sub>0</sub>) = (L<sub>0</sub> ~~&oplus; F(R<sub>0</sub>， K<sub>0</sub>)) &oplus; F(R<sub>0</sub>, K<sub>0</sub>)~~ = L<sub>0</sub>        
L<sub>0</sub> = R<sub>1</sub>        
R<sub>0</sub> = L<sub>1</sub>        

<!--
TODO 
2 Round 1 Feistel 示意图
-->

有关feistel的本质理解，[cnblogs](https://www.cnblogs.com/luop/p/4366902.html)里看到过这么一段话：
>但是对于Feistel结构最本质的特点到底是什么，我也一直不太明白，比较浅显的一点认识就是：每一轮只有一半的比特位参与计算，另外一半直接作为下一轮的输入。

根据算法流程，对于每一轮i迭代输入，左右两边上一轮i-1的输出计算方式如下（以K<sub>i</sub>作为第i轮的子密钥）       
L<sub>i</sub> = R<sub>i-1</sub>        
R<sub>i</sub> = L<sub>i-1</sub> &oplus; F(R<sub>i-1</sub>， K<sub>i</sub>)        
所以严格来说，每轮中**左右两半的比特位都**参与了运算（异或与F函数），其中一半同时也会直接作为下一轮的输入，但**每轮中仅有一半的比特位发生了变化**。

Feistel的设计非常精妙，加解密过程使用完全一样的结构，仅初始输入与子密钥调用顺序有区别，所以其加密核心F函数就无需可逆。在软硬件实现上减少了一半的成本。

#### 那么写写代码巩固下理论知识

DES加密算法作为使用feistel结构的典型应用，其F函数做了扩展运算、P盒与S盒，完整解析相对复杂。这篇文章主旨还是对feistel的总结，所以选择一个更加轻量、F函数更简单的TEA算法作为示例来用C实现一遍。但是TEA在加解密中所用的F函数是不同的，见[这篇](https://www.bbsmax.com/A/Vx5M9wrv5N/)
> 关于TEA算法，有种有意思的情况值得引起注意，那就是这个算法不是Feistel密码结构，所以需要分别独立地加密和解密例程。不过，TEA算法在尽可能地接近Feistel密码结构，虽然实际上它并不是--TEA算法使用加法和减法取代了异或运算

```c
#include <>
```



也可以自己构造F函数

要明确，feistel是一类密码结构，并不是某种特定的密码算法。实际上，对于feistel结构而言，不同的F函数就意味着不同的密码算法。
所以可以试着写一个很弱鸡的F函数

Feistel在很多经典的密码算法中都有用到。