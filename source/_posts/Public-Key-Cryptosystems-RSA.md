---
title: 公钥密码学-RSA
date: 2021-06-13 13:43:56
tags:
- cryptography
- math
- RSA
---

多年以后，面对ssl中奔流不息的数据时，我可能仍会想起在11教第一次听董荣胜说起非对称加密的那个飘着桂花味的下午。那时五教和B区之间的荒地才被烧过没多久，围墙还没拆，综合楼和新足球场也没建起来。我还是一个会认真到在图书馆找资料手算反码和补码的大一新生，下课后带着一个手抓饼回宿舍做了几道以3 7 9 11之类的小素数计算验证RSA加解密过程的课后练习，一切都很新鲜，一切似乎才刚刚开始。

<!--more-->

### 开始新的旅途

公钥密码学的复习其实酝酿了很久，抛开椭圆曲线先不论，基于数论的RSA设计原理让一个中学生去学习应该也问题不大。但是近来感觉思维没有那么灵活，做欧拉函数和中国剩余定理证明时总觉得脑子转的不够快，而且中途也老被七七八八的杂事打断，导致有些过程不停地卡关，到现在磕磕绊绊大体完成。最初计划公钥密码学是按照课本的目录顺序，先从数论基础和抽象代数开始切入，但是想了想当初自己就是这么学的，效果似乎没那么好，而且学习方法因人而异，没有什么万能的套路。总之我还是决定直接把内容拆成RSA和ECC两个部分，顺着算法设计的视角看待涉及到的数学本质问题。

公钥密码学的发展可以被称为密码学出现以来最伟大的一次革命，甚至是唯一一次革命。在这之前，几乎所有的密码体制都是基于代替与置换来设计的，即使DES，AES这类密码系统也都是在初等方法之上尽可能地消除明文的分布特征。而公钥密码体制则是找出了一个单向函数（注意并不是单向hash函数，是one-way trap door function），在没有一些特定的附加信息情况下做逆运算是时间上不可行的事情，而拥有了这些附加信息就可以在多项式时间内计算出函数的逆。其实这里已经足够说明公钥密码与传统密码的本质区别：公钥算法是基于数学函数而非代替和置换。在Diffie和Hellman提出的设计概念中，拥有两个独立密钥的公钥密码体制能够解决对称密码秘密钥分配问题与数字签名问题。在这里也顺便澄清几个概念：
+ 公钥密码不一定比传统密码安全。可以说任何加密方法的安全性都需要依赖密钥长度和破译密文的时间成本，在算力暴增的如今，RSA能做的事情也只是声明512 bits的密钥已经不够安全，应全面停止使用。甚至可以想象当量子计算商用化那天来到时，现有的密码体系都要面临大灾难，想要达到与传统计算相同的安全性，所需的密钥长度就要以量子比特存储器单元的个数呈指数膨胀。
+ 公钥密码不是用来取代传统密码。公钥密码更多的是与传统密码相结合，进而完善加密体系。“公钥密码仅限于用在密钥管理和签名这类应用中，这几乎是已被广泛接收的事实。”
+ 公钥密码不是独自完成密钥分配。并不是说公钥密码学诞生后就立刻轻松地解决了对称密码的分配的问题，它也需要通过一系列政策、服务平台、软件策略等来管理证书与密钥对，也就是CA中心及所部署的PKI系统。但公钥密码体系确实使密钥管理更加容易。
+ 还有诸如“公钥只能加密不能解密，私钥只能解密不能加密”，“公钥不能算出私钥但私钥可以算出公钥”等等这些无聊的争论我实在是不想解释，希望大家不要钻牛角尖，多学习，多实践，少bb。

文章将以[RIVE78](https://people.csail.mit.edu/rivest/Rsapaper.pdf)为理论基础，介绍公钥密码学中最早提出且成功的RSA算法设计概论，以及算法中涉及到的数学运算原理。随后结合密钥及证书文件来看PKCS制定的编码标准，如何通过著名的openssl工具解析RSA密钥和证书，如何在c/c++中使用openssl提供的接口进行编程。

### 公钥密码体系与一点题外话

单纯说对明文进行加密，二战后发明的DES、AES这类现代对称密码完全够用，但是在使用对称密码时有一个根本问题：发送方和接收方必须预先共享一个密钥（secret key 秘密钥）。就算费点劲，整点魔法，双方通过托梦、心灵感应、灵魂交换等等手段分享好了密钥，也不可能一直只和某个特定的对象会话，那么涉及到多方的密钥管理会带来很夸张的工作量。例如在一个拥有100人的网络中，总共需要存在4950把不同的密钥（99+98+97+...+2+1=5050-100）才能保证两两之间通信可靠。

公钥密码体系在设计之初可能更多是为了解决这类通信密钥的管理问题，在1980年代美国海军舰队需要使用叉车来装卸船只之间的通信密钥记录文件，因为eve这个拥有执着这一美好品质的反派角色会锲而不舍地无所不用其极获取alice与bob之间会话密钥，所以用于保护明文消息的对称密钥需要频繁地更换。更不用说到互联网时代更为庞大的通信需求，这种形式的密钥管理那完全没法用，非常阻碍社会发展。

在Diffie和Hellman于1976年发表的*New directions in cryptography*一文中其实已经给出了指导思想，设计的四个特性如下：
对于加密过程E，解密过程D，有消息M
+ a. 解密一个被加密的消息，可以表达为： D(E(M)) = M 。
+ b. 过程E和过程D都很容易计算。
+ c. 被公开的过程E不能够公开过程D的快速运算方法。只有加密者本人才能够解密被加密的信息，或快速运算过程D。
+ d. 如果消息M先解密再加密，仍然会得到M，表达为： E(D(M)) = M 。

对于满足a b c三个特性的方法被称为单向陷门函数（trap-door on-way function）；如果同时满足特性d，则被称为单向陷门置换（trap-door on-way permutation）。一般来说只有在设计数字签名算法的时候才需要额外满足特性d。如果找到了这样一种全新的密码体系，那么参与通信的一方只需要将自己的过程E发布，一旦有会话请求，请求方会寻找这个公开的过程E处理信息发给对应的接收方。接收方使用只有自己知道的过程D即可还原得到请求方发送的原文信息。同样在一个100人的网络中，只需要100个过程E（当然每个过程E都有一个对应的过程D，在RSA中是modular multiplicative inverse 模乘法逆元）就可满足两两之间的通信保障要求。当然当然，可爱又迷人的反派角色eve仍然有办法做一些攻击，比如中间人、共模、低指数CRT分解等。所以实际应用中还有PKI，PKCS这类设施及标准来完善整个密码应用体系。

除了最基本的传递秘密消息之外，更进一步，单向陷门置换有个更加令人激动的应用场景就是数字签名。正如签名的含义，通过个人独特的笔迹来判断信息是否来源于对应的发送方。如果说加密保证了数据的保密性，那么数字签名就提供了不可篡改性与不可抵赖性。即一方将消息使用仅自己知道的过程D处理后公布，其余所有人都能用公开的过程E还原出原始消息，并且这个消息一定是由过程E所属的发送方发出。实际使用中一般是公布方对消息的摘要值做签名，随着消息明文一并公布；接收方经过程E处理得到摘要值，并使用同样的摘要算法对消息明文计算，对比两边的结果判断这份信息是否可靠。

最后在具体聊RSA之前我想插一个关于密码的思考，例如古典密码中的代替技术最典型代表凯撒密码。当然这并不一定值得推广，只是提一下我自己脑海中的一个概念。我一直倾向将这类密码近似地理解为一次函数，即$f(x)=kx+b$。对于这类单调的加密过程，很容易通过已知明文还原$x$：令$m=f(x)$, 有$x=\frac{m-b}{k}$。更进一步说，古典密码追求的极致是无限接近$f(x)=b$且有办法还原$x$，消除代替中遗留的语义统计特征，当然不同的语言系统中使用的文字符号及其组合方式会直接影响密码体系中信息熵的变化，这又是另一个话题。但还是要强调这只是我自己的一个直觉性思考方式，将具象的加密操作抽象成为符号语言，或者转化成几何的角度来看待问题，称不上准确。

### RSA与数论基础
之前和朋友们瞎比聊过这帮数学怪们到底怎么发明出这些诡异的算法，大致结论是最终呈现在大家眼前的已经是精华中的精华，他们需要深刻地掌握和运用足够多的基础理论知识来形成对应问题的体系框架，通过经验也好，灵感也罢，甚至远在星辰之外的一丝运气，才在无尽的数学花园中采到了一些看起来合适的原料，不断加工组合验证推导，花费数年、数十年，甚至终其一生扑在一个问题中。没有写下Q.E.D之前四周都是迷雾。我们学习这些理论时，所有正确的原料早已明确，重复一个过程谈不上困难，尽量在经历这个过程中领悟一些东西吧。

#### 算法设计
RSA的设计遵循了单向陷门置换，算法本质是大素数分解的计算困难性。并巧妙地借助了模运算来还原明文，通过幂运算隐藏参与通信双方的秘密。虽然不知道Diffile和Hellman是怎么想到了构造幂模运算(RSA paper中的 "The aforementioned method should not be confused with the “exponentiation” technique presented by Diffie and Hellman [1] to solve the key distribution problem")，但我猜同余是个很抓眼球的特性，结合费马小定理看底数和模余数在特定条件下有机会相等，很值得用在这种密码体系的解密中。总之顺着算法思路，可以考虑如下设计:
$f(x) \bmod n = x，f(x)=x^a$，令其中a包含两个密钥相关的内容{e, d}，得到计算式 $(M^e \bmod n)^d \bmod n = M$
到这里已经有一个隐约的概念，将消息M做幂运算后取模，就算余数和模数被公开也无法得知原消息M，而参与运算的指数{e, d}经过了精心挑选，能够使用余数的幂做模运算还原消息M。这个过程的正确性就由数论中的几个经典问题支撑。

一般教材会计划提前一章学习一些数论知识，有的学校在给信息安全专业制定培养计划时会专门安排专业数学基础课程。其实最初我的打算是写公式时加入相关定理的证明过程，以此来充实整个理论推导过程，但是写着写着发现太<sup>TM</sup>不流畅了，看公式体验真滴差。最后决定先只对RSA Paper稍微补充少量中间步骤，毕竟原论文也是真滴简洁，便于在一定数论基础上的理解。这篇文章并不是科普数论基础，但我认为也有的地方值得多说几句，就写在本章的证明之后。

<center>draft && paper</center>![math_draft_paper](https://ftp.bmp.ovh/imgs/2021/07/fa4e651545e73287.png)

#### 相关定理及公式
那么来康康完成这个设计自己手上有多少牌可以打。

+ 费马小定理
    $$a^{p-1} \equiv 1 \pmod p \; , \; a是整数，p是素数$$
+ 欧拉定理
    $$a^{\phi(n)} \equiv 1 \pmod n \; , \; a与n互素$$
+ 中国剩余定理
    假设存在两两互素的整数$m_1, m_2 \cdots m_n$，对于任意整数$a_1, a_2, \cdots a_n$，构成一元线性同余方程组$(S)$
    $$(S):
    \begin{cases}
    x \equiv a_1 \pmod {m_1} & \\
    x \equiv a_2 \pmod {m_2} & \\
    \quad \vdots & \\
    x \equiv a_n \pmod {m_n} & \\
    \end{cases}
    $$
    增加如下定义：
    $M = \prod_{i=1}^n m_i$ 表示所有模数的乘积
    $M_i = \frac{M} {m_i}$ 表示除当前外$n-1$个模数的乘积
    $t_i = {M_i}^{-1} \Longleftrightarrow M_it_i \equiv 1 \pmod {m_i}$ 表示$M_i$的模乘法逆元
    则方程组有通解为
    $$x = a_1t_1M_1 + a_2t_2M_2 + \cdots + a_nt_nM_n + kM = \sum_{i=1}^n{a_it_iM_i} + kM, \quad k\in \mathbb{Z}$$
    其中，在模$M$意义下有唯一解
    $$x = \left(\sum_{i=1}^n{a_it_iM_i}\right) \bmod M$$
+ 模运算及幂运算的一些计算性质

#### 正确性证明
算法的安全基础在于素因数分解的计算困难性，那么至少需要存在两个素数，记为p与q，将p与q的乘积记为n。将消息记为M，用于指代M的数字取值范围为(0, n-1]。
$$n=p \cdot q$$
要利用同余，模数必然要公开，结合欧拉定理当M与n互素时，有$M^{\phi(n)} \equiv 1 \pmod n$，欧拉定理在RSA中起到了很重要的作用，不过现在离终点还很远，需要往里边加点料。在这里可以先猜想一个更理想的算式表达形式：$M^{\phi(n)+1} \equiv M \pmod n$。
$\phi(n)$是一个很关键的因素，有必要在附录中给出一些证明过程，这里先直接给结论。由于p和q选择了素数，则有
$$\phi(n) = \phi(p \cdot q) = \phi(p)\cdot\phi(q) = (p-1)(q-1)$$
回过头来看一眼想要的结果
$$(M^e \bmod n)^d \bmod n = M$$
如果用抽象代数的方式来表达会非常简洁，模映射是一种环同态，朴素点说呢，这是个这是个朴素的模运算分配律，幂运算可以表示连续的乘法运算，乘法运算可以表示连续的加法运算，所以there's a modular exponentiation property：
$$a^b \bmod c = (a \bmod c)^b \bmod c$$
于是可以把目标式可以写作
$$(M^e \bmod n)^d \bmod n = M^{ed} \bmod n$$
这个转换带来了$e$和$d$的乘积表达。且算法的终极目标是论证下式成立
$$M^{ed} \equiv M \pmod n$$
现在手握 $M^{\phi(n)} \equiv 1 \pmod n$ 与 $M^{ed} \bmod n$，可以考虑通过1来把 $ed$ 和 $\phi(n)$ 连上线。接下来是一个很关键的设计：随机挑选一个小于$\phi(n)$的大整数$d$，使$d$与$\phi(n)$互素，即
$$gcd(d, \phi(n)) = 1$$
有了这个就可以保证一定存在模乘法逆元，即一定存在$e$，使得
$$ed \equiv 1 \pmod {\phi(n)}$$
结合同余的性质，必定存在$k, k\in \mathbb{Z}$，使得$ed = k \phi(n) + 1$。这个时候不要再往欧拉定理里钻了，再钻就快到死胡同了，换个角度请出费马小定理和中国剩余定理。来看$M^{ed} \bmod p$ 与 $M^{ed} \bmod q$
$$
\begin{align}
&\because ed = k \phi(n) + 1 \\
&\therefore M^{ed} = M^{k \phi(n) + 1} = M^{k(p-1)(q-1) + 1} = M \cdot (M^{(p-1)})^{k(q-1)}
\end{align}
$$
模数是素数，M可为任意正整数。带入$M^{p-1} \equiv 1 \pmod p$，得到了
$$M^{ed} \equiv M^{k \phi(n) + 1} \equiv M \cdot 1^{k(q-1)} \equiv M \pmod p$$
同理对于q而言，相似的因为$M^{q-1} \equiv 1 \pmod q$，则
$$M^{ed} \equiv M^{k \phi(n) + 1} \equiv M \cdot 1^{k(p-1)} \equiv M \pmod q$$
因为n是p和q的乘积，必然有
$$M^{ed} \equiv M^{k \phi(n) + 1} \equiv M \pmod n$$
得到结论：对于任意$M, \; 0 \leq M < n$，过程$E$与过程$D$是逆置换。

根据以上论证，RSA算法密钥对产生过程为
>1. $选择两个大素数p与q，计算出乘积n = p \cdot q，其中n作为模数是公开的，p与q是保密的$
2. $计算欧拉函数\phi(n)=(p-1)(q-1)$
3. $寻找一个大整数d与\phi(n)互素，并计算d模\phi(n)的乘法逆元e，使得ed \equiv 1 \pmod {\phi(n)}$
4. $元素\lbrace e, n \rbrace作为公钥可以公开发布，元素\lbrace d, n\rbrace作为私钥由产生者保管$

传递消息的保密性
>1. $会话请求方需要传递消息M，计算C = M^e \bmod n，将C发送给接收方$
2. $接收方用自己的私钥解密C，计算M = C^d \bmod n$

数字签名与验签
>1. $消息发布方需求公开消息M，计算S = M^d \bmod n，将签名S公开$
2. $获取方用公钥对签名S验证，计算M = S^e \bmod n$


**以下是对一些“易得，显然，不难证明，由上可知”的补充。**
对于模运算的分配率换个有意思的说法：女术士打桩机杰洛特每次需要用3个弱效突变诱发物合成1个突变诱发物，杰洛特有a个弱效突变诱发物，一波操作之后合成几次忘记了，总之还剩下1个弱效突变诱发物；跑图打了一圈孽鬼沼泽巫婆腐食魔之后又得到了b个弱效突变诱发物，又操作一波合成突变诱发物，最后刚好一个弱效突变诱发物都不剩。这和先有a个弱效突变诱发物再打b个弱效突变诱发物攒在一起合成突变诱发物没有区别。公式一下就是$(a \bmod c + b) \bmod c = (a + b) \bmod c$。多次重复这个过程就是乘法的模运算。而用3个突变诱发物合成1个强效突变诱发物，从弱效突变诱发物的角度看就是幂模。

当我们看到形如$a \equiv b \pmod n$，需要理解$a$与$b$的绝对值是$n$的倍数，即a b之间形成了模运算下的相等，该式完整表达为$a \bmod n =  b \bmod n$，千万不要理解成$a \bmod n =  b$。在没有特别指明时，$a \; b \; n$ 之间应该是没有具体的大小关系，当然如果$a$和$b$都是正整数且都小于$n$，那么$a$与$b$数值相等。

RSA中用来指代明文消息的M是一个取值范围在0到n的正整数，这是中国剩余定理中提到的唯一解（特解）。中国剩余定理本质上是在求一元线性同余方程组的通解。通俗地说，给出互素的两个数p和q，p q的乘积n，对于正整数a小于n，a可以由a除以p的余数和a除以q的余数唯一确定。加密的消息要能够正确地被还原出来，就需要加以这个范围的限制，否则就只能得到通解。

结合幂模运算，$ed \equiv 1 \pmod {\phi(n)}$是一个关键设计。如果算法满足单向陷门函数，就需要设计某个方法及其对应的逆方法。而满足单向陷门置换，相当于这个逆方法不能对参数的顺序做要求。RSA中选择以乘法的形式来构造D方法和E方法，而通过乘法逆元又可以将除法这个不可靠的逆运算转化为乘法运算，modular mutiplicative inverse满足了单向陷门置换的方法统一。保证乘法逆元有且唯一，则需满足$gcd(d, \phi(n)) = 1$，这个d就需要生成密钥对时去构造。

对于$M^{ed} \equiv M \pmod p,  M^{ed} \equiv M \pmod q \Longrightarrow M^{ed} \equiv M \pmod n$，不妨这么来看：
令$a = M^{ed} - M$，上式表达的含义则可以转化为存在一个数a能够同时被p和q整除，那么a一定能被p和q的最小公倍数整除。而n是p和q最小公倍数与最大公约数的乘积，所以a一定能被n整除。既然推导了整除成立，在此基础上两边加相同的数显然同余成立。多说一句，单论推论成立p和q为正整数即可，模为素数是费马小定理的要求。但p和q互素是中国剩余定理中有且有唯一解的必要条件。

### 计算优化

#### e与d的选择
较于对称系统，非对称密码有一个尤其突出的特点是通信双方的计算内容不同，更进一步说，如RSA可以为加解密过程设计不同的计算量级。仔细看过RSA paper的话可以发现，当时作者们设计的方法是选择一个足够大的整数d（You then pick the integer d to be a large, random integer which is relatively prime to (p − 1) · (q − 1)），再计算乘法逆元e。而在实际使用中，往往是先选择一个小素数e，再计算d。从两个角度来看这个做法。首先，d是私钥的元素，所以必须保证d是一个很难被猜测的值，而e设计就是用来公开的，所以即使选择一个很小的e（但注意小素数可能会因低加密指数广播攻击带来明文泄露的风险）并不会破坏非对称系统整体的安全性，反倒让攻击者更难确定私钥d；另一点是使用小e可以减小加密（及验签）方的计算量，一般考虑服务器具备更强大的运算性能，所以会算你就多算点。

针对RSA也有很多攻击方法，毕竟保密性、可靠性、完整性、不可篡改性哪一个挂逼都不安全，安全等级也各有区别。这里借着公钥参数e说一说低加密指数的相关攻击。

##### 低加密指数攻击
这种攻击很容易理解，甚至有一刹那间让我产生了“这就是专门为CTF而存在的攻击方法吧”的怀疑(°ー°〃)。简而言之如果生成公钥时选择了如3这样的小素数，由于n可能很大，那么在加密一些小数值的时候，例如5<sup>3</sup>=125，可能这个密文值还是会比n小，那么攻击者直接对加密后的值也就是125开三次根即可还原明文。

##### Hastad广播攻击
也被称为低加密指数广播攻击，结合RSA加密过程与中国剩余定理来理解。首先可以列出RSA加密过程cipher的表达式
$$C = M^e \bmod N$$
那么在一个网络中，如果发送方使用相同的公钥元素e对同一个消息M加密发给n个接收方，设公钥为$(e, N_i)$。有同余方程组
$$
\begin{cases}
M^e \equiv C_1 \pmod {N_1} & \\
M^e \equiv C_2 \pmod {N_2} & \\
\quad \vdots & \\
M^e \equiv C_n \pmod {N_n} & \\
\end{cases}
$$
如果满足$gcd(N_i, N_j)=1$（p q是两个大素数，其实基本可以认定N之间极大概率是两两互素的），根据中国剩余定理，通过$C_i, N_i, C_j, N_j, \cdots$总能够在一个给定范围内得到$M^e$的确定值，设该值为$\mathbb{C}$。根据RSA算法设计，M一定小于N，可得如下条件
$$\mathbb{C} = M^e \tag{4-1-1}\label{4-1-1}$$
$$\mathbb{C} \equiv C_i \pmod {N_i} \tag{4-1-2}\label{4-1-2}$$
$$\mathbb{C} = \left(\sum_{i=1}^n{C_it_i\mathbb{N}_i}\right) \bmod \mathbb{N} \tag{4-1-3}\label{4-1-3}$$
$$M < N_i \quad \forall i | i \in \mathbb{Z},i \leq e \tag{4-1-4}\label{4-1-4}$$
*注意在${\eqref{4-1-3}}$式中使用的$\mathbb{N}$需要结合CRT的语境来理解，并不是指公钥参数$(e, N_i)$中的$N$。这里不赘述，参考**相关定理及公式**章节**中国剩余定理**内容。*
根据上述的四个表达式，如果想要确定唯一的$\mathbb{C}$，由${\eqref{4-1-3}}$可知首先要确定模数$\mathbb{N}$的组成。结合等式${\eqref{4-1-1}}$与不等式${\eqref{4-1-4}}$可知$\mathbb{C}$一定小于任意$e$个模数$N$的乘积。作为攻击方在网络中获取发送方至少$e$条数据后，则很容易根据公开信息计算出明文$M$
$$M = \sqrt[e]{\left(\sum_{i=1}^e{C_it_i\mathbb{N}_i}\right)\bmod\left(\prod_{i=1}^e{N_i}\right)}$$

顺便澄清网上的谣言啊，都是抄来抄去不过脑子，说因为$e$太小容易计算所以不安全，👴🏻笑了，以现代计算机的视角3和65537在计算复杂度上几乎没有区别。根本原因是对于小指数$e$而言，获取这个数量的ciphertext相对更容易，攻击者就更容易确定$M$的特解。其实解决这两种攻击漏洞的办法并不是增大$e$的值，而是将$M$编码成对于整个加密体系而言更安全的数值，例如RSA_PKCS1_PADDING，RSA_PKCS1_OAEP_PADDING等。

#### 中国剩余定理的特殊场景
由于公钥参数$e$选择了一个较小值，相对的解密过程做$d$的幂运算量就特别大，如何把解密搞快点是一个值得讨论的问题。在之前描述中国剩余定理的同余方程组可知，一个整数能够通过以下方式转换为由两两间互素的模数与余数的表达

$$
\begin{cases}
x \equiv 2 \pmod 5 & \\
x \equiv 3 \pmod 7 & \\
\end{cases}
x在模35 意义下有唯一解17 
$$

$$
\begin{cases}
x \equiv 2 \pmod 5 & \\
x \equiv 3 \pmod 7 & \\
x \equiv 3 \pmod 8 & \\
\end{cases}
x在模280意义下有唯一解227
$$

根据这个特点就可以将$M^d$拆成几组更小的数值运算，而恰好解密方是拥有素数$p$和$q$的，对于解密过程可以列出方程组

$$
\begin{cases}
C^d \equiv a_1 \pmod p \\
C^d \equiv a_2 \pmod q
\end{cases}
$$

使用$a_1$与$a_2$代替$C^d$能够将乘法拆成加法运算，将模$n$转化为模素数又可以通过费马小定理降幂做快速计算。

综上得到第一种优化的解密算法

$$
\begin{align}
C^d \bmod n & = (a_1t_1M_1 + a_2t_2M_2) \bmod M \\
& = \left(C^d \bmod p \cdot (q^{-1} \bmod p) \cdot q + C^d \bmod q \cdot (p^{-1} \bmod q) \cdot p \right) \bmod (p \cdot q) \\
& = \left(C^{d \bmod \phi(p)} \bmod p \cdot (p^{-1} \bmod q) \cdot p + C^{d \bmod \phi(q)} \bmod q \cdot (q^{-1} \bmod p) \cdot q \right) \bmod (p \cdot q) \\
\end{align}
$$

其中可以提前计算的部分有

$$
X_p = q \cdot (q^{-1} \bmod p) \quad X_q = p \cdot (p^{-1} \bmod q) \\
dP = d \bmod \phi(p) \quad dQ = d \bmod \phi(q) \\
$$

则原式
$$C^d \bmod n = (C^{dP} \cdot X_p + C^{dQ} \cdot X_q) \bmod (p \cdot q) \tag{4-2-1}$$

这里运用了求解中国剩余定理最为常见的一种方法，也被称为单基数转换法（Single Radix Conversion, SRC）。

除SRC外，优化混合基数转换（Modified Mixed Radix Conversion, MMRC）是目前应用更为广泛的方法，其基本思想是对模余数逐级逼近，引用前章中介绍中国剩余定理的一元线性同余方程组
$$
\begin{align}
x &\equiv a_1 \pmod {m_1} \\
x &\equiv a_2 \pmod {m_2} \\
  &\vdots \\
x &\equiv a_n \pmod {m_n} \\
\end{align}
$$
同样，设$M = \prod_{i=1}^n{m_i}$且$m_i$之间两两互素。
现在需要寻求一种表达方法，让小于$M$的正整数$x$由$x$模$m_i$的余数组成。即通过模余数来直接构造唯一解，表达为下式
$$x = v_1 + v_2 m_1 + v_3 m_1 m_2 + \ldots + v_n m_1 \ldots m_{n-1}$$
具体来说，$v_1$代表了$x$数值中模$m_1$的部分（很显然随后的项中都含有$m_1$，即都能被$m_1$整除），同理$v_2$?负责除$v_1$外余项模$m_2$的内容，根据模运算加法性质以此类推将整个方程组展开。或者可以理解为这个多项式中每项的系数$v_x$都是对应模意义下的向量，$x$就由各个向量构成。具体的分解运算如下

首先定义$r_{ij}$表示$m_i$模$m_j$的逆元
$$r_{ij} = m_i^{-1} \bmod m_j$$
可以列出原式的一项式

$$a_1 = v_1 = x \bmod m_1$$

同理在二项式时，可以计算得到$v_2$
$$a_2 = v_1 + v_2 m_1 = x \bmod m_2$$
$$
\begin{array}{rclr}
a_2 - v_1 & \equiv& v_2 m_1 &\pmod {m_2} \\
(a_2 - v_1)m_1^{-1} & \equiv& v_2 m_1 m_1^{-1} &\pmod {m_2} \\
(a_2 - v_1)r_{12} & \equiv& v_2 &\pmod {m_2} \\
\end{array}
$$
$$v_2 = ((a_2 - v_1)r_{12}) \bmod m_2$$

三项式时则有
$$a_3 = v_1 + v_2 m_1 + v_3 m_1 m_2 = x \bmod m_3$$
$$v_3 = (((a_3 - v_1)r_{13} - v_2)r_{23}) \bmod m_3$$

对于一个由n组同余方程展开的n项式，第n项的值可以由前n-1项确定。

在RSA解密中，MMRC应用即带入二项式运算，有
$$
\begin{align}
C^d \bmod n &= v_1 + v_2 p \\
&= C^d \bmod p + \left(\left(\left(C^d \bmod q - C^d \bmod p\right) \cdot (p^{-1} \bmod q)\right) \bmod q \right) \cdot p \\
&= C^{d \bmod \phi(p)} \bmod p + \left(\left(\left(C^{d \bmod \phi(q)} \bmod q - C^{d \bmod \phi(p)} \bmod p\right) \cdot \left(p^{-1} \bmod q \right)\right) \bmod q \right) \cdot p
\end{align}
$$
其中可以提前计算的部分有
$$
dP = d \bmod \phi(p) \quad dQ = d \bmod \phi(q) \\
pInv = p^{-1} \bmod q
$$

则原式
$$C^d \bmod n = C^{dP} \bmod p + \left(\left(\left(C^{dQ} \bmod q - C^{dP} \bmod p\right) \cdot pInv \right) \bmod q\right) \cdot p \tag{4-2-2}\label{4-2-2}$$

一般更友善的表达为（Garner's formula）
$$
\begin{align}
&m_1 = C^{dP} \bmod p \\
&m_2 = C^{dQ} \bmod q \\
&M = m_1 + (pInv \cdot (m_2 - m_1) \bmod q) \cdot p
\end{align}
$$

不过在实际运用中，如OpenSSL使用的表达式为
$$
\begin{align}
&qInv = q^{-1} \bmod p \\
&M = m_2 + (qInv \cdot (m_1 - m_2) \bmod p) \cdot q \tag{4-2-3}\label{4-2-3}
\end{align}
$$

>注：${\eqref{4-2-3}}$式的来源很简单，将MMRC的二项式写成$C^d \bmod n = v_1 q + v_2$展开即可，但是这么做的目的是什么暂时没有找到比较详细的说明，但是根据我的一些实验，以及对PEM格式的RSA私钥做解析，发现素数$p$都是大于$q$的，一个猜测是构造$m_1 - m_2$有的更大的概率为正整数？
这个问题有待考证，先抛出来吧。
**211223 update**
根据pkcs文档和一些网上的讨论<sup>[16][17]</sup>，因为p被定义大于q，在早期BIGNUM不支持负数表示的时候可以通过构造(m1 + p - m2)的式子来保证进行$\bmod$ p操作数一定是非负整数，因为m2是模q的余数，最大值为q - 1一定小于p。那么形如(m1 - m2)的式子就必然对应了$C^d \bmod n = v_1 q + v_2$二项式表达。


以上就是OpenSSL中抽象RSA数据结构实现rsa_st结构体成员dmp1, dmq1, iqmp的来源
```C
struct rsa_st {
    /*
     * #legacy
     * The first field is used to pickup errors where this is passed
     * instead of an EVP_PKEY.  It is always zero.
     * THIS MUST REMAIN THE FIRST FIELD.
     */
    int dummy_zero;

    OSSL_LIB_CTX *libctx;
    int32_t version;
    const RSA_METHOD *meth;
    /* functional reference if 'meth' is ENGINE-provided */
    ENGINE *engine;
    BIGNUM *n;
    BIGNUM *e;
    BIGNUM *d;
    BIGNUM *p;
    BIGNUM *q;
    //---------------------------------------------------
    BIGNUM *dmp1;	// d mod (p - 1)
    BIGNUM *dmq1;	// d mod (q - 1)
    BIGNUM *iqmp;	// q inverse mod p
    //---------------------------------------------------

    /*
     * If a PSS only key this contains the parameter restrictions.
     * There are two structures for the same thing, used in different cases.
     */
    /* This is used uniquely by OpenSSL provider implementations. */
    RSA_PSS_PARAMS_30 pss_params;

#if defined(FIPS_MODULE) && !defined(OPENSSL_NO_ACVP_TESTS)
    RSA_ACVP_TEST *acvp_test;
#endif

#ifndef FIPS_MODULE
    /* This is used uniquely by rsa_ameth.c and rsa_pmeth.c. */
    RSA_PSS_PARAMS *pss;
    /* for multi-prime RSA, defined in RFC 8017 */
    STACK_OF(RSA_PRIME_INFO) *prime_infos;
    /* Be careful using this if the RSA structure is shared */
    CRYPTO_EX_DATA ex_data;
#endif
    CRYPTO_REF_COUNT references;
    int flags;
    /* Used to cache montgomery values */
    BN_MONT_CTX *_method_mod_n;
    BN_MONT_CTX *_method_mod_p;
    BN_MONT_CTX *_method_mod_q;
    BN_BLINDING *blinding;
    BN_BLINDING *mt_blinding;
    CRYPTO_RWLOCK *lock;

    int dirty_cnt;
};
```

### Back to the real world
终 于 到 了 我 第 二 喜 欢 的 代 码 环 节 来 点 真 实 的
这章主要是根据现在比较通用的密码体系对RSA相关内容做解析，至于自己写代码实现一个RSA计算过程，无非是些素性判定、扩展欧几里得、快速幂等等，大数运算是有些技巧的，解密应用MMRC应该能写个有点意思的递归吧。~~但是要玩游戏所以时间不太够~~不过这些大都已经有很标准的实现，所以就可以但没必要了 ∠(ᐛ」∠)＿

#### OpenSSL命令
赞美伟大的开源工作者，虽然我从大二开始吐槽openssl的文档到现在，但讲道理源码都能拉所以有问题自己看就好了。不过网上对openssl cmd的论调基本是"something like voodoo"还是有点让人想笑。

那么解析一个RSA公钥的命令为
```shell
openssl rsa -in [your/path/to/pem/file] -pubin -noout -text
```

得到的结果为
```C
RSA Public-Key: (2048 bit)
Modulus:
    00:bd:60:7a:d1:3e:95:29:a8:34:5e:f8:ad:e2:63:
    2b:e7:37:b5:36:c9:7f:3e:50:bf:5f:37:67:f1:cf:
    6b:7b:e2:f1:63:29:8d:59:a4:75:90:ff:f3:48:c9:
    99:c1:00:51:14:d6:25:d5:c8:a9:4e:58:fd:24:9e:
    8f:c7:ff:f0:4d:9c:ba:58:8e:b8:d8:df:81:d0:4f:
    f0:70:ea:80:be:89:cd:0d:c4:a0:74:0c:ea:b7:57:
    57:d1:70:24:3a:09:dc:cd:e2:91:80:67:53:58:62:
    82:ae:fa:e2:19:4f:5a:25:59:c4:01:be:c1:63:d9:
    f9:31:d0:9c:a7:2f:24:96:fb:11:21:ef:2d:37:56:
    ce:5e:67:e0:26:a3:8b:45:db:40:16:82:79:a1:3c:
    6d:f5:d4:3c:5f:27:3c:ae:f1:84:f6:70:8f:a9:d5:
    24:8b:94:88:08:9b:44:04:49:7a:54:f9:2a:75:ad:
    22:fc:bf:3f:80:5d:cd:ab:6d:76:b0:0a:be:6c:c4:
    1f:bc:25:95:76:78:88:7e:be:69:23:40:e2:63:06:
    d2:fc:5d:2c:3d:b5:6d:60:a5:dd:5a:3c:d4:1c:8d:
    a5:dd:dd:ef:93:2c:15:be:07:02:8a:b3:f0:a4:c5:
    05:bc:d4:25:f8:1f:70:bc:83:a3:62:b5:2f:58:58:
    4f:0b
Exponent: 65537 (0x10001)
```
**注意这里的Modulus因为编码规则原因，16进制值表达"bd"第一个bit为1，所以添加一个"00"以区别负数。**

如果想带PEM的原式数据内容，则可以去掉-noout参数
```shell
openssl rsa -in [your/path/to/pem/file] -pubin -text
```

```C
RSA Public-Key: (2048 bit)
Modulus:
    00:bd:60:7a:d1:3e:95:29:a8:34:5e:f8:ad:e2:63:
    2b:e7:37:b5:36:c9:7f:3e:50:bf:5f:37:67:f1:cf:
    6b:7b:e2:f1:63:29:8d:59:a4:75:90:ff:f3:48:c9:
    99:c1:00:51:14:d6:25:d5:c8:a9:4e:58:fd:24:9e:
    8f:c7:ff:f0:4d:9c:ba:58:8e:b8:d8:df:81:d0:4f:
    f0:70:ea:80:be:89:cd:0d:c4:a0:74:0c:ea:b7:57:
    57:d1:70:24:3a:09:dc:cd:e2:91:80:67:53:58:62:
    82:ae:fa:e2:19:4f:5a:25:59:c4:01:be:c1:63:d9:
    f9:31:d0:9c:a7:2f:24:96:fb:11:21:ef:2d:37:56:
    ce:5e:67:e0:26:a3:8b:45:db:40:16:82:79:a1:3c:
    6d:f5:d4:3c:5f:27:3c:ae:f1:84:f6:70:8f:a9:d5:
    24:8b:94:88:08:9b:44:04:49:7a:54:f9:2a:75:ad:
    22:fc:bf:3f:80:5d:cd:ab:6d:76:b0:0a:be:6c:c4:
    1f:bc:25:95:76:78:88:7e:be:69:23:40:e2:63:06:
    d2:fc:5d:2c:3d:b5:6d:60:a5:dd:5a:3c:d4:1c:8d:
    a5:dd:dd:ef:93:2c:15:be:07:02:8a:b3:f0:a4:c5:
    05:bc:d4:25:f8:1f:70:bc:83:a3:62:b5:2f:58:58:
    4f:0b
Exponent: 65537 (0x10001)
writing RSA key
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAvWB60T6VKag0Xvit4mMr
5ze1Nsl/PlC/Xzdn8c9re+LxYymNWaR1kP/zSMmZwQBRFNYl1cipTlj9JJ6Px//w
TZy6WI642N+B0E/wcOqAvonNDcSgdAzqt1dX0XAkOgnczeKRgGdTWGKCrvriGU9a
JVnEAb7BY9n5MdCcpy8klvsRIe8tN1bOXmfgJqOLRdtAFoJ5oTxt9dQ8Xyc8rvGE
9nCPqdUki5SICJtEBEl6VPkqda0i/L8/gF3Nq212sAq+bMQfvCWVdniIfr5pI0Di
YwbS/F0sPbVtYKXdWjzUHI2l3d3vkywVvgcCirPwpMUFvNQl+B9wvIOjYrUvWFhP
CwIDAQAB
-----END PUBLIC KEY-----
```

对私钥的解析如下，就是解析公钥去掉-pubin参数

```shell
openssl rsa -in yf_rsaprikey.pem -noout -text
```

```C
RSA Private-Key: (2048 bit, 2 primes)
modulus:
    00:bd:60:7a:d1:3e:95:29:a8:34:5e:f8:ad:e2:63:
    2b:e7:37:b5:36:c9:7f:3e:50:bf:5f:37:67:f1:cf:
    6b:7b:e2:f1:63:29:8d:59:a4:75:90:ff:f3:48:c9:
    99:c1:00:51:14:d6:25:d5:c8:a9:4e:58:fd:24:9e:
    8f:c7:ff:f0:4d:9c:ba:58:8e:b8:d8:df:81:d0:4f:
    f0:70:ea:80:be:89:cd:0d:c4:a0:74:0c:ea:b7:57:
    57:d1:70:24:3a:09:dc:cd:e2:91:80:67:53:58:62:
    82:ae:fa:e2:19:4f:5a:25:59:c4:01:be:c1:63:d9:
    f9:31:d0:9c:a7:2f:24:96:fb:11:21:ef:2d:37:56:
    ce:5e:67:e0:26:a3:8b:45:db:40:16:82:79:a1:3c:
    6d:f5:d4:3c:5f:27:3c:ae:f1:84:f6:70:8f:a9:d5:
    24:8b:94:88:08:9b:44:04:49:7a:54:f9:2a:75:ad:
    22:fc:bf:3f:80:5d:cd:ab:6d:76:b0:0a:be:6c:c4:
    1f:bc:25:95:76:78:88:7e:be:69:23:40:e2:63:06:
    d2:fc:5d:2c:3d:b5:6d:60:a5:dd:5a:3c:d4:1c:8d:
    a5:dd:dd:ef:93:2c:15:be:07:02:8a:b3:f0:a4:c5:
    05:bc:d4:25:f8:1f:70:bc:83:a3:62:b5:2f:58:58:
    4f:0b
publicExponent: 65537 (0x10001)
privateExponent:
    6f:c5:39:b7:b5:d0:23:bd:fa:ea:f2:aa:ee:2a:ca:
    06:b5:82:66:cb:96:26:19:52:59:c8:41:b9:1e:4a:
    b9:db:bf:cc:5f:01:e6:1e:82:a5:09:eb:74:d2:47:
    c4:f9:82:e1:61:63:03:42:63:6a:b2:6a:f5:e9:ff:
    c2:72:f4:49:5a:6f:41:45:3b:24:05:06:81:04:2d:
    4c:f7:9a:f4:da:30:04:28:40:eb:3d:94:6a:91:4a:
    6b:7a:5c:67:44:da:e5:49:0b:c7:55:34:83:bd:e0:
    93:95:cf:4c:50:e1:4b:9a:27:6d:40:40:b3:c6:3a:
    a5:84:12:71:3a:09:c6:71:73:27:0f:e9:e2:f0:e3:
    b8:0a:cb:00:f1:83:cd:76:10:fd:47:ae:1f:52:84:
    3d:9c:c7:dc:92:bd:c5:31:48:e9:5c:bc:a7:93:85:
    40:e9:ba:ba:5c:a2:48:19:f3:6d:90:fc:8f:82:3f:
    40:50:25:92:52:8a:dc:fa:0b:f6:4e:fa:b0:e2:0c:
    dc:f0:db:6b:71:1e:47:8f:18:91:e2:af:7e:db:cb:
    6e:3b:ee:20:5b:99:69:af:eb:bf:4a:c1:73:05:28:
    e5:c6:ec:d0:d7:17:bf:a6:9e:33:f4:7d:ee:2e:c5:
    d5:06:50:33:9e:57:cc:58:2a:18:da:2a:32:1e:4a:
    01
prime1:
    00:f1:c8:e6:c8:79:dc:e5:34:d4:eb:f8:6c:1c:ed:
    4c:45:20:94:7a:37:c3:03:cd:25:fd:ef:ac:64:36:
    7b:ad:ac:62:5f:88:59:f5:82:e1:d8:77:00:43:b0:
    55:65:ae:d3:8a:b8:89:22:63:41:13:93:48:23:77:
    a0:ae:60:4a:6f:2f:a5:df:f5:91:a0:ce:05:7a:1c:
    96:b4:c5:75:05:60:6f:c1:ed:56:53:8e:84:ce:3d:
    08:8c:ba:d7:0d:a4:b6:dd:92:61:0d:14:be:b0:31:
    df:0e:51:bc:d4:6f:c6:8d:e3:4f:17:3c:bf:89:3f:
    c9:51:7d:cb:28:39:47:95:eb
prime2:
    00:c8:82:c9:b2:9b:4e:8e:4c:79:e3:bd:fd:cb:02:
    aa:6a:d0:ea:44:d5:1c:a0:a4:0a:7e:a6:ca:e2:e6:
    cf:04:46:e4:e6:fb:69:33:e0:33:37:b8:d7:f7:3e:
    1c:74:c0:61:2c:f2:26:13:53:72:97:13:34:b5:e8:
    28:a3:c6:8b:9e:d1:c3:3f:69:67:e6:b4:55:21:2c:
    da:c5:76:3c:86:a4:96:41:dc:d6:43:d2:3a:c6:9a:
    50:3d:9e:aa:8c:81:6e:80:d9:7e:59:a0:72:51:1e:
    3f:95:2a:47:f2:bb:4a:43:16:ce:24:a3:a9:f4:71:
    06:d6:c8:fb:96:0c:f4:43:61
exponent1:
    00:82:45:bb:cb:02:95:f9:4d:48:f7:c7:47:01:22:
    fe:38:34:c0:ab:45:46:26:d3:2f:08:2e:4d:d5:44:
    e1:c8:86:9c:0e:5b:1a:15:45:2a:c8:85:fd:b7:7a:
    d7:d8:4c:a5:20:16:23:95:4a:a3:32:97:e5:83:6e:
    9e:3d:b6:16:04:e8:48:58:6e:28:c3:da:9d:6a:d8:
    e2:7e:8d:f1:6a:2f:36:a7:e7:67:de:e7:68:38:f2:
    fb:9b:4f:c4:35:4e:ad:54:9e:dc:f9:be:56:ab:fa:
    82:f3:65:28:f7:d1:2d:cb:1f:51:6a:f4:c9:42:7b:
    02:ce:8c:97:9c:99:98:2f:77
exponent2:
    16:d2:ec:6a:ac:4b:10:df:9b:b0:54:dc:22:d3:b6:
    da:59:d5:90:e8:41:4d:f7:de:49:f4:6a:7b:d1:92:
    17:06:8a:df:d0:16:75:95:3b:bf:48:07:2d:59:a0:
    9b:99:9a:76:27:4a:36:40:f5:76:44:f5:67:0f:7a:
    30:ca:54:f2:4b:26:52:7d:89:1a:35:c4:ca:f5:f4:
    21:2e:08:4d:bb:46:6f:50:d8:02:f8:57:40:6c:28:
    5e:1b:45:86:a0:e5:17:3d:aa:a8:41:1f:42:24:93:
    50:43:73:d5:29:84:96:86:6e:08:b5:a8:8e:ee:9e:
    bc:ac:3c:17:24:7a:59:81
coefficient:
    00:dc:d9:15:81:8e:a5:d4:ed:3a:da:f3:b8:63:9e:
    17:25:dc:a0:39:30:23:64:43:7d:24:a7:80:8f:c4:
    52:9c:3b:4d:b7:08:bc:01:c9:04:1d:66:4b:53:59:
    98:7b:97:fb:b5:c5:19:8b:b3:64:a5:f3:c4:e8:ca:
    dd:72:2a:51:a3:cf:80:09:be:36:e4:7b:6b:ab:62:
    d6:f5:59:68:74:51:d1:3a:53:78:35:90:5c:e3:26:
    59:a4:2b:c9:25:bd:f1:ca:54:28:37:89:4c:e7:04:
    13:4f:4e:a1:08:9c:18:c9:f1:f9:e6:ad:ed:a6:a2:
    4d:c0:25:72:09:ef:f9:3e:e8
```

带PEM内容的输出
```shell
openssl rsa -in yf_rsaprikey.pem text
```

```C
RSA Private-Key: (2048 bit, 2 primes)
modulus:
    00:bd:60:7a:d1:3e:95:29:a8:34:5e:f8:ad:e2:63:
    2b:e7:37:b5:36:c9:7f:3e:50:bf:5f:37:67:f1:cf:
    6b:7b:e2:f1:63:29:8d:59:a4:75:90:ff:f3:48:c9:
    99:c1:00:51:14:d6:25:d5:c8:a9:4e:58:fd:24:9e:
    8f:c7:ff:f0:4d:9c:ba:58:8e:b8:d8:df:81:d0:4f:
    f0:70:ea:80:be:89:cd:0d:c4:a0:74:0c:ea:b7:57:
    57:d1:70:24:3a:09:dc:cd:e2:91:80:67:53:58:62:
    82:ae:fa:e2:19:4f:5a:25:59:c4:01:be:c1:63:d9:
    f9:31:d0:9c:a7:2f:24:96:fb:11:21:ef:2d:37:56:
    ce:5e:67:e0:26:a3:8b:45:db:40:16:82:79:a1:3c:
    6d:f5:d4:3c:5f:27:3c:ae:f1:84:f6:70:8f:a9:d5:
    24:8b:94:88:08:9b:44:04:49:7a:54:f9:2a:75:ad:
    22:fc:bf:3f:80:5d:cd:ab:6d:76:b0:0a:be:6c:c4:
    1f:bc:25:95:76:78:88:7e:be:69:23:40:e2:63:06:
    d2:fc:5d:2c:3d:b5:6d:60:a5:dd:5a:3c:d4:1c:8d:
    a5:dd:dd:ef:93:2c:15:be:07:02:8a:b3:f0:a4:c5:
    05:bc:d4:25:f8:1f:70:bc:83:a3:62:b5:2f:58:58:
    4f:0b
publicExponent: 65537 (0x10001)
privateExponent:
    6f:c5:39:b7:b5:d0:23:bd:fa:ea:f2:aa:ee:2a:ca:
    06:b5:82:66:cb:96:26:19:52:59:c8:41:b9:1e:4a:
    b9:db:bf:cc:5f:01:e6:1e:82:a5:09:eb:74:d2:47:
    c4:f9:82:e1:61:63:03:42:63:6a:b2:6a:f5:e9:ff:
    c2:72:f4:49:5a:6f:41:45:3b:24:05:06:81:04:2d:
    4c:f7:9a:f4:da:30:04:28:40:eb:3d:94:6a:91:4a:
    6b:7a:5c:67:44:da:e5:49:0b:c7:55:34:83:bd:e0:
    93:95:cf:4c:50:e1:4b:9a:27:6d:40:40:b3:c6:3a:
    a5:84:12:71:3a:09:c6:71:73:27:0f:e9:e2:f0:e3:
    b8:0a:cb:00:f1:83:cd:76:10:fd:47:ae:1f:52:84:
    3d:9c:c7:dc:92:bd:c5:31:48:e9:5c:bc:a7:93:85:
    40:e9:ba:ba:5c:a2:48:19:f3:6d:90:fc:8f:82:3f:
    40:50:25:92:52:8a:dc:fa:0b:f6:4e:fa:b0:e2:0c:
    dc:f0:db:6b:71:1e:47:8f:18:91:e2:af:7e:db:cb:
    6e:3b:ee:20:5b:99:69:af:eb:bf:4a:c1:73:05:28:
    e5:c6:ec:d0:d7:17:bf:a6:9e:33:f4:7d:ee:2e:c5:
    d5:06:50:33:9e:57:cc:58:2a:18:da:2a:32:1e:4a:
    01
prime1:
    00:f1:c8:e6:c8:79:dc:e5:34:d4:eb:f8:6c:1c:ed:
    4c:45:20:94:7a:37:c3:03:cd:25:fd:ef:ac:64:36:
    7b:ad:ac:62:5f:88:59:f5:82:e1:d8:77:00:43:b0:
    55:65:ae:d3:8a:b8:89:22:63:41:13:93:48:23:77:
    a0:ae:60:4a:6f:2f:a5:df:f5:91:a0:ce:05:7a:1c:
    96:b4:c5:75:05:60:6f:c1:ed:56:53:8e:84:ce:3d:
    08:8c:ba:d7:0d:a4:b6:dd:92:61:0d:14:be:b0:31:
    df:0e:51:bc:d4:6f:c6:8d:e3:4f:17:3c:bf:89:3f:
    c9:51:7d:cb:28:39:47:95:eb
prime2:
    00:c8:82:c9:b2:9b:4e:8e:4c:79:e3:bd:fd:cb:02:
    aa:6a:d0:ea:44:d5:1c:a0:a4:0a:7e:a6:ca:e2:e6:
    cf:04:46:e4:e6:fb:69:33:e0:33:37:b8:d7:f7:3e:
    1c:74:c0:61:2c:f2:26:13:53:72:97:13:34:b5:e8:
    28:a3:c6:8b:9e:d1:c3:3f:69:67:e6:b4:55:21:2c:
    da:c5:76:3c:86:a4:96:41:dc:d6:43:d2:3a:c6:9a:
    50:3d:9e:aa:8c:81:6e:80:d9:7e:59:a0:72:51:1e:
    3f:95:2a:47:f2:bb:4a:43:16:ce:24:a3:a9:f4:71:
    06:d6:c8:fb:96:0c:f4:43:61
exponent1:
    00:82:45:bb:cb:02:95:f9:4d:48:f7:c7:47:01:22:
    fe:38:34:c0:ab:45:46:26:d3:2f:08:2e:4d:d5:44:
    e1:c8:86:9c:0e:5b:1a:15:45:2a:c8:85:fd:b7:7a:
    d7:d8:4c:a5:20:16:23:95:4a:a3:32:97:e5:83:6e:
    9e:3d:b6:16:04:e8:48:58:6e:28:c3:da:9d:6a:d8:
    e2:7e:8d:f1:6a:2f:36:a7:e7:67:de:e7:68:38:f2:
    fb:9b:4f:c4:35:4e:ad:54:9e:dc:f9:be:56:ab:fa:
    82:f3:65:28:f7:d1:2d:cb:1f:51:6a:f4:c9:42:7b:
    02:ce:8c:97:9c:99:98:2f:77
exponent2:
    16:d2:ec:6a:ac:4b:10:df:9b:b0:54:dc:22:d3:b6:
    da:59:d5:90:e8:41:4d:f7:de:49:f4:6a:7b:d1:92:
    17:06:8a:df:d0:16:75:95:3b:bf:48:07:2d:59:a0:
    9b:99:9a:76:27:4a:36:40:f5:76:44:f5:67:0f:7a:
    30:ca:54:f2:4b:26:52:7d:89:1a:35:c4:ca:f5:f4:
    21:2e:08:4d:bb:46:6f:50:d8:02:f8:57:40:6c:28:
    5e:1b:45:86:a0:e5:17:3d:aa:a8:41:1f:42:24:93:
    50:43:73:d5:29:84:96:86:6e:08:b5:a8:8e:ee:9e:
    bc:ac:3c:17:24:7a:59:81
coefficient:
    00:dc:d9:15:81:8e:a5:d4:ed:3a:da:f3:b8:63:9e:
    17:25:dc:a0:39:30:23:64:43:7d:24:a7:80:8f:c4:
    52:9c:3b:4d:b7:08:bc:01:c9:04:1d:66:4b:53:59:
    98:7b:97:fb:b5:c5:19:8b:b3:64:a5:f3:c4:e8:ca:
    dd:72:2a:51:a3:cf:80:09:be:36:e4:7b:6b:ab:62:
    d6:f5:59:68:74:51:d1:3a:53:78:35:90:5c:e3:26:
    59:a4:2b:c9:25:bd:f1:ca:54:28:37:89:4c:e7:04:
    13:4f:4e:a1:08:9c:18:c9:f1:f9:e6:ad:ed:a6:a2:
    4d:c0:25:72:09:ef:f9:3e:e8
writing RSA key
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAvWB60T6VKag0Xvit4mMr5ze1Nsl/PlC/Xzdn8c9re+LxYymN
WaR1kP/zSMmZwQBRFNYl1cipTlj9JJ6Px//wTZy6WI642N+B0E/wcOqAvonNDcSg
dAzqt1dX0XAkOgnczeKRgGdTWGKCrvriGU9aJVnEAb7BY9n5MdCcpy8klvsRIe8t
N1bOXmfgJqOLRdtAFoJ5oTxt9dQ8Xyc8rvGE9nCPqdUki5SICJtEBEl6VPkqda0i
/L8/gF3Nq212sAq+bMQfvCWVdniIfr5pI0DiYwbS/F0sPbVtYKXdWjzUHI2l3d3v
kywVvgcCirPwpMUFvNQl+B9wvIOjYrUvWFhPCwIDAQABAoIBAG/FObe10CO9+ury
qu4qyga1gmbLliYZUlnIQbkeSrnbv8xfAeYegqUJ63TSR8T5guFhYwNCY2qyavXp
/8Jy9Elab0FFOyQFBoEELUz3mvTaMAQoQOs9lGqRSmt6XGdE2uVJC8dVNIO94JOV
z0xQ4UuaJ21AQLPGOqWEEnE6CcZxcycP6eLw47gKywDxg812EP1Hrh9ShD2cx9yS
vcUxSOlcvKeThUDpurpcokgZ822Q/I+CP0BQJZJSitz6C/ZO+rDiDNzw22txHkeP
GJHir37by2477iBbmWmv679KwXMFKOXG7NDXF7+mnjP0fe4uxdUGUDOeV8xYKhja
KjIeSgECgYEA8cjmyHnc5TTU6/hsHO1MRSCUejfDA80l/e+sZDZ7raxiX4hZ9YLh
2HcAQ7BVZa7TiriJImNBE5NII3egrmBKby+l3/WRoM4FehyWtMV1BWBvwe1WU46E
zj0IjLrXDaS23ZJhDRS+sDHfDlG81G/GjeNPFzy/iT/JUX3LKDlHlesCgYEAyILJ
sptOjkx54739ywKqatDqRNUcoKQKfqbK4ubPBEbk5vtpM+AzN7jX9z4cdMBhLPIm
E1NylxM0tegoo8aLntHDP2ln5rRVISzaxXY8hqSWQdzWQ9I6xppQPZ6qjIFugNl+
WaByUR4/lSpH8rtKQxbOJKOp9HEG1sj7lgz0Q2ECgYEAgkW7ywKV+U1I98dHASL+
ODTAq0VGJtMvCC5N1UThyIacDlsaFUUqyIX9t3rX2EylIBYjlUqjMpflg26ePbYW
BOhIWG4ow9qdatjifo3xai82p+dn3udoOPL7m0/ENU6tVJ7c+b5Wq/qC82Uo99Et
yx9RavTJQnsCzoyXnJmYL3cCgYAW0uxqrEsQ35uwVNwi07baWdWQ6EFN995J9Gp7
0ZIXBorf0BZ1lTu/SActWaCbmZp2J0o2QPV2RPVnD3owylTySyZSfYkaNcTK9fQh
LghNu0ZvUNgC+FdAbCheG0WGoOUXPaqoQR9CJJNQQ3PVKYSWhm4ItaiO7p68rDwX
JHpZgQKBgQDc2RWBjqXU7Tra87hjnhcl3KA5MCNkQ30kp4CPxFKcO023CLwByQQd
ZktTWZh7l/u1xRmLs2Sl88Toyt1yKlGjz4AJvjbke2urYtb1WWh0UdE6U3g1kFzj
JlmkK8klvfHKVCg3iUznBBNPTqEInBjJ8fnmre2mok3AJXIJ7/k+6A==
-----END RSA PRIVATE KEY-----

```

#### OpenSSL接口
没什么太多好讲的，直接上代码。
1. 私钥里包含了modulus和e，缩短篇幅解析公钥就不单独再贴出来了
2. 随缘释放内存随缘异常处理

```C
void parsePrivKey(const char * key)
{
	FILE *fin = NULL;
	EVP_PKEY *pkey = NULL;
	int keylen, pubkey_algo_id = 0;

	fin = fopen(key, "r");
	if (!fin)
		goto parse_priv_key_err;

	if (!PEM_read_PrivateKey(fin, &pkey, NULL, NULL))
	{
		LOGI("Loading private key failed.\n");
		goto parse_priv_key_err;
	}

	keylen = EVP_PKEY_size(pkey);
	pubkey_algo_id = EVP_PKEY_id(pkey);

	switch (pubkey_algo_id) {
	case EVP_PKEY_RSA:
		LOGI("======>Parsing a %d RSA private key\n", keylen * 8);
		RSA *rsa_key = NULL;
		// const BIGNUM **rsa_n, **rsa_e, **rsa_d, **rsa_p, **rsa_q;
		const BIGNUM *rsa_n, *rsa_e, *rsa_d, *rsa_p, *rsa_q;
		const BIGNUM *rsa_dp, *rsa_dq, *rsa_iq;
		char *rsa_e_dec = NULL, *rsa_e_hex = NULL, *rsa_n_hex = NULL, *rsa_d_hex = NULL;
		char *rsa_p_hex = NULL, *rsa_q_hex = NULL; //two primes
		char *rsa_dp_hex = NULL, *rsa_dq_hex = NULL, *rsa_iq_hex = NULL; //quick calc
		rsa_key = EVP_PKEY_get0_RSA(pkey);
		if (!rsa_key)
			goto parse_priv_key_err;

		//get key components
		RSA_get0_key(rsa_key, &rsa_n, &rsa_e, &rsa_d); //phi(n) = phi(p) * phi(q)
		rsa_n_hex = BN_bn2hex(rsa_n);
		rsa_e_dec = BN_bn2dec(rsa_e);
		// rsa_e_hex = BN_bn2hex(*rsa_e);
		rsa_d_hex = BN_bn2hex(rsa_d);

		prettyPrint("Modulus:", rsa_n_hex, keylen * 2);

		int e_num = atoi(rsa_e_dec);
		LOGI("Public Exponent:\n");
		LOGI("%d (0x%x)\n", e_num, e_num);
		keylen = strlen(rsa_d_hex);
		prettyPrint("Private Exponent:", rsa_d_hex, keylen);

		//get primes
		RSA_get0_factors(rsa_key, &rsa_p, &rsa_q);
		rsa_p_hex = BN_bn2hex(rsa_p);
		rsa_q_hex = BN_bn2hex(rsa_q);

		keylen = strlen(rsa_p_hex);
		prettyPrint("Prime1:", rsa_p_hex, keylen);
		keylen = strlen(rsa_q_hex);
		prettyPrint("Prime2:", rsa_q_hex, keylen);

		//get quick calculate components
		RSA_get0_crt_params(rsa_key, &rsa_dp, &rsa_dq, &rsa_iq);
		rsa_dp_hex = BN_bn2hex(rsa_dp);
		rsa_dq_hex = BN_bn2hex(rsa_dq);
		rsa_iq_hex = BN_bn2hex(rsa_iq);

		keylen = strlen(rsa_dp_hex);
		prettyPrint("Private exponent mod p:", rsa_dp_hex, keylen);
		keylen = strlen(rsa_dq_hex);
		prettyPrint("Private exponent mod q:", rsa_dq_hex, keylen);
		keylen = strlen(rsa_iq_hex);
		prettyPrint("inverse q mod p:", rsa_iq_hex, keylen);

		break;

	default:
		break;
	}

	EVP_PKEY_free(pkey);

parse_priv_key_err:
	LOGI("err\n");
}
```

输出
```shell
======>Parsing a 2048 RSA private key
Modulus:
BD 60 7A D1 3E 95 29 A8 34 5E F8 AD E2 63 2B E7 37 B5 36 C9 7F 3E 50 BF 5F 37 67 F1 CF 6B 7B E2 
F1 63 29 8D 59 A4 75 90 FF F3 48 C9 99 C1 00 51 14 D6 25 D5 C8 A9 4E 58 FD 24 9E 8F C7 FF F0 4D 
9C BA 58 8E B8 D8 DF 81 D0 4F F0 70 EA 80 BE 89 CD 0D C4 A0 74 0C EA B7 57 57 D1 70 24 3A 09 DC 
CD E2 91 80 67 53 58 62 82 AE FA E2 19 4F 5A 25 59 C4 01 BE C1 63 D9 F9 31 D0 9C A7 2F 24 96 FB 
11 21 EF 2D 37 56 CE 5E 67 E0 26 A3 8B 45 DB 40 16 82 79 A1 3C 6D F5 D4 3C 5F 27 3C AE F1 84 F6 
70 8F A9 D5 24 8B 94 88 08 9B 44 04 49 7A 54 F9 2A 75 AD 22 FC BF 3F 80 5D CD AB 6D 76 B0 0A BE 
6C C4 1F BC 25 95 76 78 88 7E BE 69 23 40 E2 63 06 D2 FC 5D 2C 3D B5 6D 60 A5 DD 5A 3C D4 1C 8D 
A5 DD DD EF 93 2C 15 BE 07 02 8A B3 F0 A4 C5 05 BC D4 25 F8 1F 70 BC 83 A3 62 B5 2F 58 58 4F 0B 
Public Exponent:
65537 (0x10001)
Private Exponent:
6F C5 39 B7 B5 D0 23 BD FA EA F2 AA EE 2A CA 06 B5 82 66 CB 96 26 19 52 59 C8 41 B9 1E 4A B9 DB 
BF CC 5F 01 E6 1E 82 A5 09 EB 74 D2 47 C4 F9 82 E1 61 63 03 42 63 6A B2 6A F5 E9 FF C2 72 F4 49 
5A 6F 41 45 3B 24 05 06 81 04 2D 4C F7 9A F4 DA 30 04 28 40 EB 3D 94 6A 91 4A 6B 7A 5C 67 44 DA 
E5 49 0B C7 55 34 83 BD E0 93 95 CF 4C 50 E1 4B 9A 27 6D 40 40 B3 C6 3A A5 84 12 71 3A 09 C6 71 
73 27 0F E9 E2 F0 E3 B8 0A CB 00 F1 83 CD 76 10 FD 47 AE 1F 52 84 3D 9C C7 DC 92 BD C5 31 48 E9 
5C BC A7 93 85 40 E9 BA BA 5C A2 48 19 F3 6D 90 FC 8F 82 3F 40 50 25 92 52 8A DC FA 0B F6 4E FA 
B0 E2 0C DC F0 DB 6B 71 1E 47 8F 18 91 E2 AF 7E DB CB 6E 3B EE 20 5B 99 69 AF EB BF 4A C1 73 05 
28 E5 C6 EC D0 D7 17 BF A6 9E 33 F4 7D EE 2E C5 D5 06 50 33 9E 57 CC 58 2A 18 DA 2A 32 1E 4A 01 
Prime1:
F1 C8 E6 C8 79 DC E5 34 D4 EB F8 6C 1C ED 4C 45 20 94 7A 37 C3 03 CD 25 FD EF AC 64 36 7B AD AC 
62 5F 88 59 F5 82 E1 D8 77 00 43 B0 55 65 AE D3 8A B8 89 22 63 41 13 93 48 23 77 A0 AE 60 4A 6F 
2F A5 DF F5 91 A0 CE 05 7A 1C 96 B4 C5 75 05 60 6F C1 ED 56 53 8E 84 CE 3D 08 8C BA D7 0D A4 B6 
DD 92 61 0D 14 BE B0 31 DF 0E 51 BC D4 6F C6 8D E3 4F 17 3C BF 89 3F C9 51 7D CB 28 39 47 95 EB 
Prime2:
C8 82 C9 B2 9B 4E 8E 4C 79 E3 BD FD CB 02 AA 6A D0 EA 44 D5 1C A0 A4 0A 7E A6 CA E2 E6 CF 04 46 
E4 E6 FB 69 33 E0 33 37 B8 D7 F7 3E 1C 74 C0 61 2C F2 26 13 53 72 97 13 34 B5 E8 28 A3 C6 8B 9E 
D1 C3 3F 69 67 E6 B4 55 21 2C DA C5 76 3C 86 A4 96 41 DC D6 43 D2 3A C6 9A 50 3D 9E AA 8C 81 6E 
80 D9 7E 59 A0 72 51 1E 3F 95 2A 47 F2 BB 4A 43 16 CE 24 A3 A9 F4 71 06 D6 C8 FB 96 0C F4 43 61 
Private exponent mod p:
82 45 BB CB 02 95 F9 4D 48 F7 C7 47 01 22 FE 38 34 C0 AB 45 46 26 D3 2F 08 2E 4D D5 44 E1 C8 86 
9C 0E 5B 1A 15 45 2A C8 85 FD B7 7A D7 D8 4C A5 20 16 23 95 4A A3 32 97 E5 83 6E 9E 3D B6 16 04 
E8 48 58 6E 28 C3 DA 9D 6A D8 E2 7E 8D F1 6A 2F 36 A7 E7 67 DE E7 68 38 F2 FB 9B 4F C4 35 4E AD 
54 9E DC F9 BE 56 AB FA 82 F3 65 28 F7 D1 2D CB 1F 51 6A F4 C9 42 7B 02 CE 8C 97 9C 99 98 2F 77 
Private exponent mod q:
16 D2 EC 6A AC 4B 10 DF 9B B0 54 DC 22 D3 B6 DA 59 D5 90 E8 41 4D F7 DE 49 F4 6A 7B D1 92 17 06 
8A DF D0 16 75 95 3B BF 48 07 2D 59 A0 9B 99 9A 76 27 4A 36 40 F5 76 44 F5 67 0F 7A 30 CA 54 F2 
4B 26 52 7D 89 1A 35 C4 CA F5 F4 21 2E 08 4D BB 46 6F 50 D8 02 F8 57 40 6C 28 5E 1B 45 86 A0 E5 
17 3D AA A8 41 1F 42 24 93 50 43 73 D5 29 84 96 86 6E 08 B5 A8 8E EE 9E BC AC 3C 17 24 7A 59 81 
inverse q mod p:
DC D9 15 81 8E A5 D4 ED 3A DA F3 B8 63 9E 17 25 DC A0 39 30 23 64 43 7D 24 A7 80 8F C4 52 9C 3B 
4D B7 08 BC 01 C9 04 1D 66 4B 53 59 98 7B 97 FB B5 C5 19 8B B3 64 A5 F3 C4 E8 CA DD 72 2A 51 A3 
CF 80 09 BE 36 E4 7B 6B AB 62 D6 F5 59 68 74 51 D1 3A 53 78 35 90 5C E3 26 59 A4 2B C9 25 BD F1 
CA 54 28 37 89 4C E7 04 13 4F 4E A1 08 9C 18 C9 F1 F9 E6 AD ED A6 A2 4D C0 25 72 09 EF F9 3E E8
```

### 一些问题
好像除了Garner's formula选取二项式的方式之外暂时没有别的问题。。
*211223已更新

### 老套但必须存在的总结环节
Cryptography Review系列在这两年多以来陆陆续续出了几篇文章，确实是温故知新。按照原本的计划就剩下椭圆曲线密码学的相关内容了。当然Diffle-Hellman和ElGamal密码体制要不要单独开一篇还没确定，以后也可能会继续补充消息认证码之类的内容（又挖新坑）。总之ECC肯定是接下来最主要的方向，开始之前我应该会把包括离散和抽代的知识再过一遍，这玩意一头扎进去就不知道什么时候能出来了，那就拖更一年吧  ─=≡(ง ᐛ )ว

### 参考文献

1. 密码编码学与网络安全（第五版）
2. [A Method for Obtaining Digital Signatures and Public-key Cryptosystems](https://people.csail.mit.edu/rivest/Rsapaper.pdf)
3. [New Directions in Cryptography](https://ee.stanford.edu/~hellman/publications/24.pdf)
4. [The science of encryption: prime numbers and mod n arithmetic](https://math.berkeley.edu/~kpmann/encryption.pdf)
5. [跨越千年的RSA算法](http://www.matrix67.com/blog/archives/5100)
6. [Number Theory - Euclid's Algorithm](https://crypto.stanford.edu/pbc/notes/numbertheory/euclid.html)
7. [Number Theory - Chinese Remainder Theorem](https://crypto.stanford.edu/pbc/notes/numbertheory/crt.html)
8. [Number Theory - Modular Exponentiation](https://crypto.stanford.edu/pbc/notes/numbertheory/exp.html)
9. [Does RSA work for any message m](https://crypto.stackexchange.com/questions/1004/does-rsa-work-for-any-message-m)
10. [Why is important to choose e coprime to $\phi(n)$](https://crypto.stackexchange.com/questions/12255/in-rsa-why-is-it-important-to-choose-e-so-that-it-is-coprime-to-%CF%86n)
11. [中国剩余定理在RSA解密中的应用[J]. 河北省科学院学报,2003,20(3)](http://www.cnki.com.cn/Article/CJFDTotal-HBKX200303003.htm)
12. [Using the CRT with RSA](https://www.di-mgt.com.au/crt_rsa.html)
13. [Garner's Algorithm in CRT](https://cp-algorithms.com/algebra/chinese-remainder-theorem.html)
14. [An Example of A Garner's Algorithm Calculation](https://www.csee.umbc.edu/~lomonaco/s08/441/handouts/Garner-Alg-Example.pdf)
15. [openssl@github](https://github.com/openssl/openssl)
16. [PKCS #1: RSA Cryptography Specifications Version 2.2](https://datatracker.ietf.org/doc/html/rfc8017#section-3.2)
17. [RSA Algorithm](https://www.di-mgt.com.au/rsa_alg.html)

### 附录

#### APPENDIX A 证明过程

#### APPENDIX B 基于小素数的计算
