---
title: Embedded Android -- Chapter1
date: 2018-03-18 16:44:25
tags: 
- Embedded Android Translation
---


### 简介 Introduction
将安卓操作系统移植到嵌入式设备上是一项复杂的任务，这包括了对系统内部复杂设计的理解，巧妙地将修改融入到安卓开源项目，以及对内核运行平台——Linux的掌握。在开始进入到嵌入式安卓细节之前，先让我们了解一些安卓系统背后的本质内容，例如安卓的硬件需求，安卓的法律体制和对嵌入式设备的影响等。首先，让我们一起看一看安卓的诞生与发展。
Putting Android on an embedded device is a complex task involving an intricate understanding of its internals and a clever mix of modifications to the Android Open Source Project (AOSP) and the kernel on which it runs, Linux. Before we get into the details of embedding Android, however, let's start by covering some essential background that embedded developers should factor in when dealing with Android, such as Android's hardware requirements and the legal framework surrounding Android and its implications within an embedded setting. First let's look at where Android comes from and how it's developed.

<!-- more -->

--------------------------------------------------

### 历史 Histroy
故事的开始要追溯到2002年早期，Larry page和Sergey Brin参加了Danger公司关于Sidekick手机研发在斯坦福举办的一次访谈，主讲人是当时Danger公司的CEO Andy Rubin。Sidekick手机是第一批具备多功能并且能够介入互联网的设备。活动结束后，Larry上前查看Sidekick手机，他非常开心地发现这款手机使用了谷歌作为默认搜索引擎。不久之后，Larry和Sergey都成为了Sidekick的用户。
The story goes that back in early 2002 Google's Larry Page and Sergy Brin attended a talk at Standford about the development of the then-new Sidekick phone by Danger Inc. The speaker was Andy Rubin, Danger's CEO at the time, and the Sidekick was one of the first multi-function, Internet-enabled devices. After the talk, Larry went up to look at the device and was happy to see that Google was the default search engine. Soon after, both Larry and Sergey became Sidekick users.

尽管Sidekick十分新颖且迎合用户，但它并没有取得商业上的成功。2003年，Rubin和公司董事会一致认为到了让他离开的时候。在尝试了其他的一些方向后，Rubin决定重返手机操作系统行业。他使用早前注册的域名android.com，打算为手机制造商创造一个开放的操作系统。Rubin为安卓项目投入了大部分积蓄，同时获得了一些启动资金。于是Rubin着手创办安卓公司。不久后，在2005年的8月份，谷歌低调地收购了安卓。
Despite its novelty and enthusiatic users, however, the Sidekick didn't achieve commercial success. By 2003, Rubin and Danger's borad agreed it was time for him to leave. After trying out a few things, Rubin decided he wanted to get back into the phone OS business. Using a domain name he previously-owned. *android.com*, he set out to create an open OS for phone manufacturers. After investing most of his savings on the project and having received some additional seed money, he set out to get the company funded. Soon after, in August 2005, Google acquired Android Inc. with little fanfare.

在安卓从被收购到2007年11月公之于众的日子里，谷歌很少对外放出它的消息。相反，研发团队正在幕后拼命地工作。最初的声明是由开放手机联盟（OHA）发布，联盟由一系列移动设备领域相关的公司组建，安卓正是它的首款产品。一年后，安卓的第一版开源版本，Android 1.0在2008年12月被推出。
Between its acquisition and its announcement to the world in November 2007, Google let little to no information out about Android. Instead, the development team worked furiously on the OS while deals and prototypes were being worked on behind the scenes. The initial announcement was made by the *Open Handset Alliance* (OHA), a group of companies unveiled for mobile device and Android being its first product. A year later, in September 2008, the first open source version of Android, 1.0, was made available.

此后，安卓陆续发布多个版本。虽然还有很多工作仍是在秘密地开展，但以后我们会看到，安卓操作系统的研发进度以及发展状态终将变得更加公开透明。表1-1汇总了不同安卓系统版本在AOSP（Android Open Source Project）中对应最显著的特性。
Several Android versions have been released since then, and the OS's progression and development is obviously more public. As we will see later, though, much of the work on Android continues to be done behind closed doors. Table 1-1 provides a summary of the various Android releases and the most notable features found in the corresponding AOSP.

- *Table 1-1. Android Versions*

Version | Release date | codename | Most notable feature(s) | Open source 
- | - | - | - | -
1.0 | September 2008 | *Unknown* |  | Yes
1.1 | February 2009 | *Unknown* |  | Yes
1.5 | April 2009 | Cupcake | On-screen soft keyboard | Yes
1.6 | September 2009 | Dount | Battery usage screen and VPN support | Yes
2.0, 2.0.1, 2.1 | October 2009 | Eclair | Exchange support | Yes
2.2 | May 2010 | Froyo | Just-In-Time (JIT) compile | Yes
2.3 | December 2010 | Gingerbread | SIP and NFC support | Yes
3.0 | January 2011 | Honeycomb | Tablet form-factor support | No
3.1 | May 2011 | Honeycomb | USB host support And APIs | No
4.0 | December 2011 (projected) | Ice-Cream Sandwich | Merged phone and tablet form-factor support | Yes (projected)


--------------------------------------------------


### 特征与特色 Features and Characteristics
谷歌例举了安卓如下的特征
Google advertizes the following features about Android:

>*应用程序框架*
开发者们使用应用程序框架来开发人们常说的安卓应用。在 http://developer.android.com 网站和O'Reilly的*Learning Android*中都有介绍。
*Application framework*
The application framework used by app developers to create what is commonly-referred to as Android apps. The use of this framework is documented online at http://developer.android.com and in books like O'Reilly's *Learning Android*.

>*Dalvik虚拟机*
安卓使用字节码解释器来代替Sun公司的java虚拟机。与java被解释为.class与.jar文件不同，Dalvik将java代码通过dx作用？？？解释成.dex文件。
*Dalvik Virtual Machine*
The clean-room bytecode interpreter implementation used in Android as a replacement for the Sun java VM. While the latter interprets .class and .jar files, Dalvik interprets .dex files. These files are generated by the *dx* utility using the .class files generate by the Java compiler part of the JDK.

>*浏览器集成*
安卓包含了基于WebKit的浏览器作为其应用标准列表的一部分。应用开发者们可以通过调用**WebView**类来使用WebKit引擎。
*Integrated browser*
Android includes a WebKit-based browser as part of it standard list of applications. App developers can use the **WebView** class to use the WebKit engine within their own apps.

>*优化的图形库*
安卓提供了自己的2D图形库，但是3D表现能力仍然依赖OpenGL为嵌入式设备设计的API。
*Optimized graphics*
Android provides its own 2D graphics library but relies on OpenGL ES for its 3D capabilities.

>*SQLite数据库*
安卓使用了标准的SQLite数据库，应用开发者们可以通过应用程序框架操作数据库引擎。
*SQLite*
This is the standard SQLite database found at http://www.sqlite.org and made available to app developers through the application framework.

>*多媒体支持*
安卓通过StageFright框架提供了非常广泛的媒体格式支持。在2.2版本之前，安卓一度使用了PacketVideo的OpenCore框架。
*Media support*
Android provides support for a wide range of media formats through StageFright, its custom media framework. Prior to 2.2, Android used to rely on PacketVideo's OpenCore framework.

>*GSM电话支持*
通话功能是由硬件设备提供的，设备制造商必需提供一个硬件抽象层的模块让安卓系统接入他们的设备。硬件抽象层将会在下一章节来讨论。
*GSM telephony support*
The telephony support is hardware dependent and device manufacturers must provide a HAL module in order enable Android to interface with their hardware. HAL modules will be discussed in the next chapter.

>*蓝牙，EDGE，3G和无线热点*
安卓几乎支持现有的所有无线连接技术。有一些功能由安卓特有的方式实现，例如EDGE和3G，而像蓝牙和WiFi等功能则与普通Linux相似。
*Bluetooth, EDGE, 3G, and WiFi*
Android includes support for most wireless connection technologies. While some implemented in Android-specific fashion, such as EDGE and 3G, others are provided in the same as in plain Linux, as in the case of Bluetooth and WiFi.

>*相机，GPS，指南针和加速度计*
用户环境接口是安卓系统的关键。API的存在使得从应用程序框架去访问设备成为了现实，而使用这些设备及传感器则需要相应的硬件抽象层模块来支持。
*Camera, GPS, compass, and accelerometer*
Interfacing with the user's environment is key to Android. APIs are made available in the application framework to access there devices, and some HAL modules are required to enable their support.

>*丰富的开发环境*
这也是安卓最宝贵的财富之一，这些丰富的资源降低了程序开发的门槛。带有模拟器的完整SDK，Eclipse插件以及数量众多的分析调试工具都被免费提供给开发者。
*Rich development environment*
This is likely one of Android's greatest assets. The development environment available to developers makes it very easy to get started with Android development. A full SDK is freely available to download along with an emulator, an Eclipse plugin, and a number of debugging and profiling tools.

当然，安卓还有很多特征可以被例举，例如USB支持，多任务，多点触控，网络共享，语音指令等等。但是之前提及的那些特征能够给你一个清晰的认识——你能够在安卓系统中发现什么。并且每个新版本发布都会带来新的特性。查看每个版本在平台上高亮标记的公告，你会发现更多关于系统特性与提升的信息。
There are of course a lot more features that could be listed for Android, such as USB support, multitasking, multi-touch SIP, tethering, voice-activated commands, etc., but the previous list should give you a good idea of what you'll find in Android. Also note that every new Android release brings in its own new set of features. Check the *Platform Highlights* published with every version for more information on features and enhancements.

除了这些基本的特性外，安卓平台也有一些对于嵌入式开发而言颇有意思的特点：
In addition to its basic feature-set, the Android platform has a few characteritics that make it an especially interesting basis for embedded use. Here's a quick summary:

>*广阔的应用程序生态*
到目前为止，安卓市场已有超过二十万个应用，已经很接近苹果应用市场的三十五万个应用。这保证了你在为嵌入式设备打包应用时由足够的选择空间。
*Broad app ecosystem*
At the time of this writing there were 200,000 apps in the Android Market. This compares quite favorably to the Apple App Market's 350,000 apps and ensures that you have a large pool to choose from should you want to prepackaged applications with your embedded device.

>*统一的应用API*
应用程序框架提供的所有API都被设计为向上兼容。因此，为嵌入式设备定制的应用能够继续工作在未来的安卓系统中。相比之下，对安卓源码的修改并不能适配，甚至无法正常地工作在后续的新版本上。
*Consistent app APIs*
All APIs provided in the application framework are meant to be forward-compatible. Hence, custom apps you develop for inclusion in your embedded system should continue working in future Android versions. In contrast, modifications you make to Android's source code are not guaranteed to continue applying or even working in the next Android release.

>*可替换的组件*
因为安卓的开源，并且得益于其设计架构，很多组件可以被直接替换。比如你不喜欢安卓默认的启动器（桌面），完全可以自己写一个新的代替它。当然，安卓也可以接收更加底层的修改，例如在不修改任何应用API的情况下使用GStreamer替换默认的多媒体框架StageFright
*Replaceable components*
Because Android is open source, and as a benefit of its architecture, a lot of its components can be replaced outright. For instance, if you don't like the default app Launcher (home screen), you can write your own. More fundamental changes can also be made to Android. The GStreamer developers, for example, were able to replace StageFright, the default media framework in Android, with GStreamer without modifying the app API.

>*可扩展性*
安卓的开放性和架构特点带来的另一个好处是，开发者添加额外的功能和硬件会相对简单。你只需要模仿平台对同类型其他硬件或特性的操作步骤。例如，你可以仅仅在硬件抽象层添加少量的文件达到支持客制化硬件的需求。
*Extendable*
Another benefit from Android's openness and its architecture is that adding support for additional features and hardware is relatively straightforward. You just need to emulate what the platform is doing for other hardware or features of the same type. For instance, you can add support for custom hardware to the HAL by adding a handful of files.

>*客制化*
*Customizable*
如果你更加倾向使用现成的组件，例如默认的启动器应用，但你仍然可以把它变成你喜欢的样子。无论是把这些组件改成你期望的行为，还是更换它们的界面，都可以自由地修改安卓开源项目。
If you'd rather use existing components, such as the existing Launcher app, you can still customize them to your liking. Whether it be tuning their behavior or changing their look and feel, you are again free to modify the AOSP as needed.

--------------------------------------------------

### 发开模式 Development Model

当你考虑是否使用安卓时，了解它的发展过程对修改源码或面对内部依赖时就显得相当重要了。
When considering whether to use Android, it's crucial that you understand the ramifications its development process may have on any modifications you make to it or any dependencies you may have towards its internals.

#### 不同于“传统的”开源项目 Differences With "Classic" Open Source Projects

开源是安卓最为人知的特点。事实上，正如我们所看见，许多开源的软件工程项目被使用在安卓系统中。
Android's open source nature is one of the most trumpeted features. Indeed, as we've just seen, many of the software engineering benefits that derive from being open source apply to Android.

尽管有许可证，但安卓却不像多数开源项目那样，它的开发基本上处于非公开状态。比如，大多数的开源项目有公共邮件列表及论坛，可以与主要开发人员交流，且公共的仓库能够提供主开发分支的最新提交。安卓并没有这些。
Despite its licensing, however, Android is unlike most open source projects in that its development is mostly done behind closed doors. The vast majority of open source projects, for example, have public mailing lists and forums where the main developers can be found interacting with each other, and public source repositories providing access to the main development branch's tip. No such thing can be found for Android.

最好的总结获取来自Andy Rubin，“开源项目与社区驱动型项目不同。相比于社区驱动，安卓更加重视的是开放源代码。”
This is best summarized by Andy Rubin himself: "Open source is different than a community-driven project. Android is light on community-driven, somewhat heavy on open source."

无论我们喜欢与否，安卓主要还是由谷歌的安卓研发团队开发，公众们既无法参与内部讨论，也无法获取最新的开发分支。不过，谷歌仍然会在新机器适配系统后释放一次代码，这个周期一般是六个月。比如在2010年12月，三星代工的二儿子面市之后的几天，代号姜饼的安卓2.3系统代码已经可以在http://android.git.kernel.org/ 上拉取了。
Whether we like it or not, Android is mainly developed within Google by the Android development team and the public is not privy to either internal discussions or the tip of the development branch. Instead, Google makes code-drops every time a new version of Android ships on a new device, which is usually every 6 months. For instance, a few days after the Samsung Nexus S was released in December 2010, the code for the new version of the Android it was running, 2.3/Gingerbread, was made publicly available at http://android.git.kernel.org/.

很显然，开源社区中弥漫着一种不适感。安卓作为一个极其流行的项目，在继续标榜自己开源的同时，其开发模式却又与标准的开源项目背道而驰。在历史上，采用类似这种开发模式的项目从来没有为社区良好地服务过。
Obviously there is a certain amount of discomfort within the open source community with the continued use of the term "open source" within the context of project whose development model contradicts the standard modus operandi typically associated with open source projects, especially given Android's popularity. The open source community has not historically been well served by projects that had adopted a similar development model.

撇开这些问题不谈，安卓的研发也是有限的。事实上，除非成为谷歌安卓团队的一员，否则你无法为安卓的开发分支做出贡献。另外，除极少数个例外，你也无法与核心团队成员一对一地讨论所做的改进。不过，你仍然可以在http://android.git.kernel.org/ 上把修改和改进提交到AOSP。
Political issues aside, though, Android's development is limited. Indeed, unless you become part of the Android development team at Google, you will not be able to make contributions to the tip of the development branch. Also, save for a handful of exceptions, you will unlikely be able to discuss your enhancements one-on-one with the core development team members. However, you are still free to submit enhancements and fixes to the AOSP code dumps made available at http://android.git.kernel.org/.

谷歌这种做法带来最糟糕的副作用就是——你完全没有办法知道安卓研发团队对平台决策的内部消息。如果新的特性被加进了AOSP，例如核心组件被修改时，你只能通过分析之后释放的代码才能了解这些修改是如何完成的，会对你做的改动产生何种影响。此外，你也不可能知道带来这些新增和改动的需求、标准以及问题究竟是什么。如果这是一个原汁原味的开源项目，那么必然会有一个公共邮件列表归档了项目所有相关的信息和观点，这是非常有用的。
The worst side-effect of Google's approach is that you have absolutely no way to get inside information about the platform decisions being made by the Android development team. If new features are added within the AOSP, for example, or if modifications are made to core components, you will find out how such changes are made and how they impact changes you might have made to a previous version only by analyzing the next code dump. Furthermore, you will have no way to find out the underlying requirement or restriction or issue that justified the modification or inclusion. Had this been a true open source project, a public mailing list archive would exist where all this information, or pointers to it, would be available.

尽管如此，谷歌在开源协议下发布安卓操作系统的贡献是那么重要，虽然以开源社区角度来看，它的开发模式十分尴尬。谷歌在安卓上的付出确确实实帮了众多开发者们的大忙。此外，安卓项目还完成了一项创举：创造了一个大规模成功的Linux发行版。因此，安卓团队的努力确实可以称得上无可挑剔。
That being said, it's important to remember how significant a contribution Google is making by distributing Android under an open source license. Despite its awkward development model from an open source community perspective, it remains that Google's work on Android is a godsend for a large number of developers. Plus, it has accomplished one thing no other open source projects was ever able to: create a massively successful Linux distribution. It would, therefore, be hard to fault Android's development team for their work.

此外，以业务与市场的立场也能够容易地去反驳那些观点。一个社区驱动的过程势必会削弱谷歌刻意释放的产品宣传。因为每一个新功能的增加都是公开的，所以不可能在如新闻发布等方面去制造什么噱头。并且不确定性也正是社区驱动过程的性质，正如我们所见，一大群人仅仅基于记录的形式，花费了数年时间来商讨实现某些特定功能的最佳方式。安卓的成功无疑归功于谷歌迅速地推进能力，以及发布新型牛逼产品的兴趣。
Furthermore, it can easily be argued that from a business and go-to-market perspective a community-driven process would definitely knock the wind out of any product announcements Google would attempt to release, making it impossible to create "buzz" around press announcements and the like since every new feature would be developed in the open. That is to say nothing of the non-deterministic nature of community-driven processes that can see a group of people take years to agree on the best way to implement a given feature set. And, simply based on track record, Andoid's success has definitely benefited from Google's ability to rapidly move it forward and generate press interset based on releases of cool new products.

#### 特征，蓝图和新版本 Feature Inclusion, Roadmaps, and New Releases

简而言之，并不存在公开的安卓功能及特征的产品蓝图。谷歌最多只会提前公布下一个版本的代号和大致的发布日期。一般在每年五月召开的谷歌IO大会上发布新系统，年底再推送一次更新。版本更新的内容大家只能猜测。
In brief, there is no publicly available roadmap for features and capabilities in future Android releases. At best, Google will announce ahead of time the name and approximate release date of the next version. Usually, you can expect a new Android release to be made in time for the Google I/O conference, which is typically held in May, and another release by year end. What will be in that release, though, is anyone's guess.

通常，谷歌会选择一家制造商合作完成新的系统。在此期间，谷歌会与这家制造商的工程师密切合作，为新系统打造一款旗舰机。工作期间，合作制造商的团队将获得开发分支顶端的访问权。一旦设备面市，相应的源码就合并到公开的仓库。接着，谷歌再挑选另一个制造商合作研发下一个版本的系统。
Typically, however, Google will choose a single manufacturer to work with on the next Android release. During the period, Google will work very closely with that single manufacturer's engineers to ready up the next Android version to work on a targeted upcoming flagship device. During the period, the manufacturer's team is reported to have access to the tip of the development branch. Once the device is put on the market, the corresponding source code dump is made to the public repositories. For the next release, they choose another manufacturer and start over.

在这个周而复始的过程中有一个例外：安卓3.x/蜂巢。谷歌并没有为该系统的旗舰产品——摩托罗拉Xoom释放对应的源代码。种种迹象表明，代码可能永远不会被公开。原因似乎是为了抢占平板电脑的市场先机（iPad 2），安卓开发团队直接从原分支fork了一份代码来开发适用于平板的系统，导致这个版本对手机相关的体验与向后兼容性方面考虑的并不周全。谷歌并不希望安卓3.x的代码开源导致其平台碎片化难以维护，所以在本书编写时，将要推出的安卓4.0/冰激凌三明治系统中，计划同时兼容支持手机和平板设备。
There is one notable exception to the cycle: Android 3.x/Honeycomb. In that specific case, Google has not released the source code to the corresponding flagship product, the Motorola Xoom, and all indications suggest that that code may never be publicly available. The rational in this case seems to be that Android development team essentially forked the Android code-base at some point in time in order to start working on getting a tablet-ready version of Android out ASAP based on market timing prerogatives. Hence, in that version, very little regard was made to preserving backward compatibility with the phone form-factor. And given that, Google did not wish to make the code available to avoid fragmentation of its platform. Instead, at the time of this writing, plans are that both the phone and tablet form factor support will be merged into the upcoming Android 4.0/Ice-Cream sandwich release.

--------------------------------------------------

### 软件生态系统 Ecosystem

自2011年六月起：
As of June 2011:
+ 每天有40万部安卓手机被激活，2010年12月和8月时，这个数字分别是30万和20万。
+ 400,000 Android phones are activated each day, up from 300,000 in December 2010 and 200,000 in August of that same year.
+ 安卓应用市场已经拥有大约20万个应用，相比之下，苹果应用市场有35万个应用。
+ The Android Market contains around 200,000 apps. In comparison, the Apple App Store has 350,000 apps.
+ 在美国，卖出的手机中基于安卓系统超过三成。
+ More than a third of all phones sold in the US are based on Android.

安卓系统的发展显然处于上升阶段，事实上，Gartner咨询公司在2011年4月时预测，2015年安卓将占据智能手机市场的半壁江山。就像十年前Linux搅动了嵌入式市场一样，安卓也正准备大干一场。不仅仅是甩开竞争对手、颠覆移动市场，在嵌入式领域也要成为广大面向用户型嵌入式设备提供标准用户接口。
Android is clearly on the upswing. In fact, Gartner predicted in April 2011 that Android would hold about 50% of the smartphone market by 2015. Much as Linux disrupted the embedded market about a decade ago, Android is poised to make its mark. Not only will it flip the mobile market on its head, eliminating or sidelining even some of the strongest players, but in the embedded space it is likely going to become the defacto standard UI for a vast majority of user-centric embedded devices.

因此，一个完整的生态系统被迅速建立起来。例如ARM、TI、高通、飞思卡尔和英伟达等芯片制造商们已经增加了对安卓的支持，摩托罗拉、三星、HTC、索爱、LG、爱可视、戴尔、华硕等手机和平板设备制造商也都参与进来，而亚马逊、Verizon、Sprint和Barnes&Nobles都在创建这自己的应用市场。
An entire ecosystem is therefore rapidly building around Android. Silicon and system-on-chip (SoC) manufacturers such as ARM, TI, Qualcomm, Freescale, Nvidia and TI hace added Android support for their products, and handset and tablet manufacturers such as Motorola, Samsung, HTC, Sony-Ericsson, LG, Archos, DELL, ASUS, etc. ship an ever-increasing number of diverse players, such as Amazon, Verizon, Sprint and Barnes&Nobles, creating their own application markets.

尽管安卓的开发处在封闭环境中，草根社区和项目也开始聚集起来。他们大多跟随手机模组的脚步，本质上是靠破解修改制造商提供的二进制文件来实现他们自己的改动或新版本。其实这些行为反倒是更具有开源色彩，依靠安卓源码实现自己的分支或新功能。例如模组制作者会经常光顾*xda-developers.com*在线论坛，而*cyanogenmod.com*（CM）网站中可以获取增强功能和特性的rom分支。其他的分支包括Replicant(*http://replicant.us*)，旨在创立一个比安卓更加自由的操作系统。MIUI(*http://en.miui.com/*)则提供了很酷的用户界面。
Grass-roots communities and projects are also starting to sprout around Android, even though it is developed behind closed doors. Some of those efforts follow in the footsteps of phone modders, which essentially rely on hacking the binaries provided by the manufacturers to create their own modifications or variants, while others have a more open source tint to them, relying on the Android sources to create their own forks or additions. The *xda-developers.com* online forum, for instance, is traditionally frequented by modders, while the *cyanogenmod.com* site hosts an Android fork which modifies the Android sources to provide additional features and enhancements. Ohter Android forks include Replicant (*http://replicant.us*), which aims at replacing as many of the Android components with Free Software as possible, and MIUI (*http://en.miui.com/*), which provides some cool UI hacks.

#### 关于开放手机联盟 A Word on the Open Handset Alliance

正如我之前所提到的，OHA在安卓最初发布时就已经成立，就像网站上对组织的描述一样“82家移动技术领域公司聚集在一起，加快移动设备开发，为消费者们提供更好更丰富且价格实惠的使用体验。我们在一起开发了Android^TM^，这是第一个完整的、开放的、免费的移动平台。”
As I mentioned earlier, the OHA was the initial vehicle through which Android was first announced. It describes itself on its website as "... a group of 82 technology and mobile companies who have come together to accelerate innovation in mobile and offer consumers a richer, less expensive, and better mobile experience. Together we have developed Android^TM^, the first complete, open, and free mobile platform."

在最初声明前，OHA所扮演的角色并不明确。例如2010年谷歌IO大会中，一位“安卓团队访谈（炉边谈话）”的参与者向提问板提问，作为OHA成员公司员工的他们获得了那些权益。而发言者却回答说他们并不清楚，因为他们不属于OHA。因此，这似乎表明连安卓开发团队本身都不清楚OHA成员有哪些好处。
Beyond the initial announcement, however, it is unclear what role the OHA plays. For example, an attendee at the "Fireside Chat with the Android Team" at Google I/O 2010 asked the panel what privileges were confered to him as a developer for belonging to a company which is part of the OHA. After asking around the panel, the speaker essentially answered that they didn't know because they aren't the OHA. Hence, it would appear that OHA membership benefits are not clear to the Android development team itself.

OHA的角色进一步被模糊，因为它并没有像其他全职组织那样拥有董事会成员以及编制员工。相反，它仅仅是一个“联盟”。并且，在谷歌的安卓公告中也没有提及任何与OHA有关的消息，抑或通过OHA发布任何关于安卓的新公告。总而言之，有人猜测谷歌可能把OHA作为一个营销线路来展示Android对行业的支持程度，但实际上它对Android的发展几乎没有影响。
The role of the OHA is  further blurred by the fact that it does not seem to be a full-time organization with board members and permanent staff. Instead, it's just an "alliance." In addition, there is no mention of the OHA within any of Google's Android announcements, nor do any new Android announcements emanate from the OHA. In sum, one would be tempted to speculate that Google likely put the OHA together mainly as a marketing front to show how much industry support there was for Android, but that in practice it has little to no bearing on Android's development.

--------------------------------------------------

### 得到“安卓” Getting "Android"

想要使安卓在你的嵌入式设备上跑起来需要具备两个要素：兼容安卓的Linux内核及安卓平台。
There are two main pieces required to get Android working on your embedded system: an Android-compatible Linux kernel and the Android Platform.

在编写这本书时，还不能够使用vanilla（香草）内核[kernel.org](https://www.kernel.org/)运行安卓。你需要使用AOSP中列出的某个版本内核或为vanilla内核打上兼容安卓的补丁。有少许针对安卓的修改已尝试合并到主线代码中，但是目前还未起作用。但可以预见的是，这个问题能够在长远的未来被解决，内核主线代码也会很好的支持安卓。而眼下，甚至短期内，我们只能接受兼容安卓的Linux内核仅仅是主线代码分支这一事实。
As for this writing, you cannot use a "vanilla" kernel form [kernel.org](https://www.kernel.org/) to run the Platform. Instead, you need to either use one of the kernels avaliable within the AOSP or patch a "vanilla" kernel for it to be Android-compatible. Unforunately, while a few attempts have been made to merge the Android modifications into the mainline kernel, these efforts haven't yet succeeded. It is expected that, in the long term, the pending issues will be resolved and the mainline kernel will support Android "out of the box." For the time being, however, we must contend with the fact Android-compatible kernels are essentially forks of the mainline and will continue being so for the foreseable future.

安卓平台的本质是一个包含了用户空间包的定制化Linux发行版。在表1-1中列举了实际的发行平台。我们会在下一个章节讨论平台的内容和结构。现在，你只需要在脑海中记住，这些发行平台的角色与Ubuntu或Fedora等标准Linux发行版相似。它是一组软件包的自相关集？？？，一旦被构建，就能够通过特殊的工具、接口以及开发API为用户提供独特的体验。
The Android Platform is essentially a custom Linux distribution containing the userspace packages which make up what is typically called "Android". The releases listed in Table 1-1 are actually Platform releases. We will discuss the content and architecture of the Platform in the next chapter. For the moment being, keep in mind that a Platform release has a similar role to standard Linux distributions such as Ubuntu or Fedora. It's a self-coherent set of software packages that, once built, provides a specific user experience with specific tools, interfaces, and developmer APIs.

--------------------------------------------------

### 法律体系 Legal Framework

与其他软件一样，安卓的使用和发行也受到如许可、知识产权、市场现实的限制。
Like any other piece of software, Android's use and distribution is constricted by a set of licenses, intellectual property restrictions, and market realities. Let's look at a few of these.

#### 代码许可 Code Licenses

正如我们之前讨论过的，“安卓”由兼容安卓的Linux内核与AOSP（释放的）两部分组成。虽然仅仅为了运行AOSP而修改，但Linux内核一直处于相同的GNU GPLv2许可下。因此，请记住，你不允许在GPL外的任何其他许可下提交对内核的修改。如果从[android.git.kernel.org](android.git.kernel.org)获取内核代码、修改并在你自己的系统上运行。只要遵守GPL，就能使用源码，甚至包含你所修改的部分去编译映像，并把内核映像发布到产品中去。
As we discussed earlier, there are two parts to "Android": an Android-compatible Linux kernel and an AOSP release. Even through it is modified to run the AOSP, the Linux kernel continues to be subject to the same GNU GPLv2 license that it has always been under. As such, remember that you are not allowed to distribute any modifications you make to the kernel under any other license that the GPL. Hence, if you take a kernel version from android.git.kernel.org and modify it to make it run on your system, you are allowed to distribute the resulting kernel image in your product only so long as you abide by the GPL and, therefore, make the sources used to create the image, including your modifications, available to recipients under the terms of the GPL.

内核中也包含了一份来自Linus Torvalds的声明，在源码中很清楚的指出了，只有内核才遵循GPL协议，而运行在其上的应用则不再受约束。所以，你可以自由地在Linux内核上创建程序，并在选定的许可下发布它们。
The kernel also includes a notice by Linus Torvalds in its COPYING file in its sources that clearly identifies that only the kernel is subject to the GPL and that applications running on top of it are **not** considered "derived works." hence, you are free to create applications that run on top of the Linux kernel and distribute them under the license of your choice.

这些规则及其适用性受到了开源领域，或是许多支持、使用Linux内核作为产品基础公司的理解与接纳。除内核之外，基于Linux的发行版中大量关键组件也通常得到一或多个GPL许可。例如GNU的C库（glibc）和GNU编译器（GCC）就分别在LGPL与GPL下发布。而uClibc和Busybox这类嵌入式Linux系统中常用的重要软件包也被许可在LGPL与GPL下。
These rules and their applicability are generally well understood and accepted within open source circles and by most companies that opt to support the Linux kernel or use it as the basis for their products. In addition to the kernel, a large number of key components of Linux-based distributions are typically licensed under one from or another of the GPL. The GNU C library (glibc) and the GNU compiler (GCC), for example, are licensed under the LGPL and the GPL respectively. Important packages commonly used in embedded Linux systems suchs as uClibc and Busybox are also licensed under the LGPL and the GPL.

当然并不是所有人都喜欢GPL。事实上，在衍生类产品上强加许可约束对于大型组织来说是一个严峻的挑战。尤其是考虑到地理分布、开发单位间的文化差异以及对外部代理商的依赖。例如，在北美销售的制造商可能需要与几十上百家供应商打交道才能将产品推向市场，其中的任何一家供应商都有可能交付遵循或不遵循GPL协议的代码。无论代码来自哪个供应商，在销售给客户的项目中，制造商必须要给GPL提供源码。此外，也需要推出审核机制，从而保证基于GPL项目中的工程师遵守许可。
Not everyone is comfortable with the GNU GPL, however. Indeed, the restrictions it imposes on the licensing of derived works can pose a serious challenge to large organizations, especially given geographic distribution, cultural differences amongst the various locations of development sub-units, and the reliance on external subcontractors. A manufacturer selling a product in North America, for example, might have to deal with dozens, if not hundreds, of suppliers to get that product to market. Each of these suppliers might deliver a piece which may or not contain GPL'd code. Yet the manufacturer whose name appears on the item sold to the customer will be bound to provide source to GPL components regardless of which supplier originated them. In addition, process must be put in place to ensure engineers that work on GPL-based projects are abiding by the licenses.

当谷歌着手与制造商们开始打造“开放式”手机操作系统时，很快便明白必须尽可能地避免GPL污染。事实上，除了Linux之外也考虑过其他内核，但因为Linux已经具备了ARM制造商在内的强大工业支持，所以最终还是选择了它。并且其与系统其他部分隔离得很好，所以GPL造成的影响很小。
When Google set out to work with manufacturers on their "open" phone OS, therefore, it appears that very rapidly it became clear that the GPL had to be avoided in as much as possible. In fact, other kernels than Linux were apparently considered, but Linux was chosen because it already had strong industry support, particularly from ARM silicon manufacturers, and because it was fairly well isolated from the rest of the system, so that its GPL licensing would have little impcat.

尽管做了许多努力确保大量的用户空间组件许可不会带来GPL污染类似的问题，谷歌为AOSP打造的大多数组件都以Apache 2.0许可发布。而一些核心组件，例如Bionic库（安卓下的C/C++库）和工具箱都是在BSD许可下发布。也有一些经典的开源项目被纳入到AOSP的*external/*目录，假设OEM制造商不对这部分内容进行客制化（不做任何衍生修改），那么编译出的二进制格式文件不会带来任何问题，并且可以在android.git.kernel.org提供这些二进制文件给使用者下载，这也符合GPL协议对衍生修改继承GPL的定义。
It was decided, though, that every effort would be made to make sure that the vast majority of user-space components would be based on licenses that did not pose the same logistical issues at the GPL. That is why  the vast majority of the components created by Google for the AOSP are published under the Apache License 2.0 (a.k.a.ASL) with some key components, such as the Bionic library and the Toolbox utility, licensed under the BSD license. Some classic open source projects are also incorporated, mostly as-is, into the AOSP within the *external/* directory. The assumption is that their distribution in binary form should not pose any problems since they aren't meant to be typically customized by the OEM (i.e., no derived works are expected to be created) and are readily available for all to download at android.git.kernel.org, thereby complying, if applicable, with the GPL's requirement that redistribution of derivative works contibue being made under the GPL.

不同于GPL，ASL并不会要求衍生的作品在一个指定的许可下发布。事实上，你可以选择任何一个合适的协议去发布你的修改。以下是ASL的相关部分（完整的协议在http://www.apache.org/licenses/获取）：
Unlike the GPL, the ASL does not require that derivative works be published under a specific license, In fact, you can choose whatever license best suits your needs for the modifications you make. Here are the relevant portions from the ASL 
(the full license is available at http://www.apache.org/licenses/):
+ “根据许可证的条款和条件，每个贡献者在此授予您永久性、全球的、非排他性的、免费的、免版税的、不可撤销的专利实施许可，以源码或的目标的方式复制、筹备、公开展示、公开履行、授权和分发和这些作品及衍生作品。”
+ "Subject to the terms and conditions of the License, each Contributor hereby grants to You a perpetual, wordwide, non-exclusive, no-charge, royalty-free, irrevoacble copyright license to reproduce, perpare Derivative Works of, publicly display, publicly perform, sublicense, and distribute the Work and such Derivative Works in Source or Object form."
+ “您可以给您的修改添加您自己的版权声明，可以提供使用、复制或分发您修改的不同的许可协议条款，或者对于任何这样的派生作品整体上，只要您的使用、复制及分发作品符合本许可规定的条件。”
+ "... You may add Your own copyright statement to Your modifications and may provide additional or different license terms and conditions for use, reproduction, or distribution of Your modifications, or for any such Derivative Works as a whole, provided Your use, reproduction, and distribution of the Work otherwise complies with the conditions stated in this License."

此外，ASL明确地提供了专利许可授权，这意味着你可以不需要从谷歌获取任何专利许可，就可以使用ASL授权的安卓源码。不过也规定了一些“管理”上的要求，例如需要清晰地标记被修改的文件等。尽管如此，本质上你已经可以按照意愿自由地授权所做的修改。包括Bionic与Toolbox在内，BSD许可还能够允许二进制形式的发布。
Furthermore, the ASL explicitly provides a patent license grant, meaning that you do not require any patent license from Google for using the ASL-licensed Android code. It also imposes a few "administrative" requirements such as the need to clearly mark modified files, etc. Essentially, though, you are free to license your modifications under the terms that fit your purpose. The BSD license that covers Bionic and Toolbox allows similar binary-only distribution.

因此，只要继续遵守ASL，制造商可以使用AOSP并客制化成他们所需要的样子，甚至在有需要时保留修改专利。别的不谈，。。。翻不通啊
Hence, manufacturers can take the AOSP and customize to their needs while keeping those modifications proprietary if they wish, so long as they continue abiding by the rest of the provisiongs of the ASL. if noting else, this idmishes the burden of having to implement a process to track all modifications in order to provide those modifications back to recipients who would be entitled to request them had the  GPL been used instead.

#### 品牌使用 Branding Use

相对于源码方面的慷慨，谷歌对安卓相关的品牌元素把控相对严格。让我们来看看这些元素和它们相关的使用术语。官方列表和官方术语可以在[这里](http://www.android.com/branding.html)查阅
While being very generous with Android's source code, Google controls most Android-related branding elements more strictly. Let's take a look at some of those elements and their associated terms of use. For the official list along with the official terms, have a look at <http://www.android.com/branding.html>.

*安卓机器人*
*Android Robot*
这就是那个随处可见的绿色小机器人。它的角色相当于Linux企鹅，其使用权限也类似。事实上，谷歌声称“它可以免费在市场传播中被使用、复制和修改”。唯一的要求是根据创作共用署名许可条款做出适当的规定。
This is the familiar green robot seen everywhere around all things Android. Its role is similar to the Linux penguin and the permissions for its use are similarly permissive. In fact Google states that it "can be used, reproduced, and modified freely in marketing communications." The only requirement is that proper attibution be made according to the terms of the Creative Commons Attribution license.

*安卓标识*
*Android Logo*
这是一个字母集合，使用客制化字型拼出A-N-D-R-O-I-D并且在设备及模拟器开机阶段展示，或使用在android.com网站。在任何情况下你都不被授权使用这个标识。
This is the set of letters in custom typeface that spell out the letters A-N-D-R-O-I-D and that appear during the device and emulator bootup, and on  the android.com website. You are not authorized to use that logo under any circumstance.

*安卓客制化字型*
*Android Custom Typeface*
这是用来渲染Android标识的自定义字型，它的使用与标识一样受到限制。
This is the custom typeface used to render the Android logo and its use is as restricted as the logo.

*官方名称中的“安卓”*
*"Android" in official Names*
如谷歌的声明，“Android” 一词只能在有适当的商标归属时用于描述安卓。例如你不能在未得到谷歌允许的情况下将你的产品命名为“xxx Android”。在后文中将要列出有关安卓兼容性的问答中写到：“如果制造商希望在他们的产品中使用Android名称，那么他们必须先证明这些设备是兼容的”。因此，将设备称为“安卓”是谷歌想要保护的特权。而本质上，必须先保证你的设备能够兼容安卓，与谷歌接洽并达成某种协议后，才能够通告这是“安卓”设备。
As Google states, "the word 'Android' may be used only as a descriptor, 'for Android'" and so long as proper trademark attribution is made. You cannot, for instance, name your product "Foo Android" without Google's permission. As the FAQ for the Android Compatibility Program(ACP), which we will cover later in this chapter, states: "... if a manufacturer wishes to use the Android name with their product ... they must first demonstrate that the device is compatible." Branding your device as being "Android" is therefore a privilege which Google intends to police. In essence, you will have to make sure your device is compliant and then talk to Google and enter into some kind of agreement with them before you can advertize your device as being "Foo Android".

*官方名称中的"Droid"*
*"Droid" in Official Names*
你不能单独使用“Droid”作为名称，例如“xxx Droid”。但是好像作者也没有搞明白啊所以我也不想翻这段了
You may not use "Droid" alone in a name, such as "Foo Droid" for example. For some reason the author hasn't ye entirely figured out, "Droid" is a trademark of Lucasfilm. Achieve a Jedi rank you likely must before you can use it.

*信息中的“安卓”*
*"Android" in messaging*
只要遵循适当的条款（例如Android^TM^使用 <- 这里我不确定写成 安卓^TM^ 后是否是同一个意思）就能在文本中使用，当然也要标记出合适的商标归属。
It is permitted to use "Android" "... in text as a descriptor, as long as it is followed by a proper generic term (e.g. "Android^TM^ application")." And here too, proper trademark attribution must be made.

#### 谷歌的安卓应用程序 Google's Own Android Apps

通常AOSP中含有能够以ASL获取的核心应用，而安卓手机中通常还会包含一系列附加的谷歌应用，例如安卓应用市场、YouTube、谷歌地图、Gmail等。用户们显然希望谷歌的应用成为安卓的一部分，所以你会想要把这些都安装在设备中。在这种情况下就需要通过兼容性测试并与谷歌达成协议，这点与在产品中使用“Android”颇为相似。这里将会简要地提及一下安卓兼容性程序。
While the AOSP contains a core set of applications which are available under the ASL, "Android"-branded phones usually contain an additional set of "Google" applications that are not part of the AOSP, such as the "Android Market", YouTube, "Maps and Navigation", Gmail, etc. Obviously, users expect to have these apps as part of Android, and you might therefore want to make them available on your device. If that is the case,
you will need to abide by the ACP and enter in agreement with Google, very much in line with what you need to do to be allowed to use "Android" in your product's name. We will cover the ACP shortly.

#### 第三方应用市场 Alternative App Mrakets

虽然主要的应用程序市场由谷歌提供，用户通过预先安装在手机上的安卓应用市场应用去下载安装其他应用，不过也有其他玩家正在扩展安卓开放的接口及许可完成自己修改的应用市场。例如亚马逊和巴诺这样的在线电商，或是威瑞森及斯普林特这类移动网络提供商。甚至还有一个开源项目F-Droid([仓库在这里](http://f-droid.org/repository))，提供了应用程序以及发布在GPL许可下的后端服务。
Though the main app market is the one hosted by Google and made available to users through the "Android Market" app installed on "Android"-branded devices, other players are leveraging Android's open APIs and open source licensing to offer their own alternative app markets. Such it the case of online merchants like Amazon and Barnes&Noble, as well as mobile network operators such as Verizon and Sprint. There is, in fact, noting to the author's knowledge that precludes you from creating your own app store. There is even at least one open source project, FDroid Repository (<http://f-droid.org/repository>), that provides both an app market application and a corresponding server backend under the GPL.

#### 甲骨文与谷歌之争 Oracle v Google

作为Sun微系统的一部分，甲骨文也获得了Sun公司对Java语言的知识产权。根据Java创始人之一James Gosling的说法，
As part of acquiring Sun Microsystem, Oracle also acquired Sun's intellectual property (IP) rights to the Java language and, according to Java creator James Gosling, it was clear during the acquisition process that Oracle intended from the onset to go after Google with Sun's Java IP portfolio. And in August 2010 it did just that, filing suit against Google, claiming that it infringed on serveral patents and commited copyright violations.

撇开案子不谈，安卓确实严重依赖Java，而Sun创造了Java并且拥有这门语言大量相关的知识产权。？？？尽管如此，安卓的开发团队还是想方设法让操作系统尽可能少使用Sun的Java。Java的使用主要包括三个方面：语言本身及其语义，运行由Java编译器生成Java字节码的虚拟机，包含Java程序运行时需使用包的类库。
Without going into the merits of the case, it's obvious that Android does indeed heavily rely on Java. And clearly Sun created Java and owned a lot of intellectual property around the language it created. In what appears to have been an effort ot anticipate any claims Sun may put forward against Android, nonethless, the Android development team went out of its way to make the OS use as little of Sun's Java as possible. Java is in fact comprised mainly of three things: the language and its semantics, the virtual machine that runs the Java bytecode generated by the Java compiler, and the class library that contains the packages used by Java applications at run time.

官方版本的Java组件由甲骨文的Java开发工具包（JDK）及Java运行时环境（JRE）提供。而安卓只依赖JDK中的Java编译器。安卓使用了定制的Dalvik虚拟机，而不是甲骨文的Java虚拟机；使用Apache Harmony而不是官方类库。这样看，谷歌似乎在尽一切合理的努力避免卷入版权或发行问题。
The official versions of the Java components are provided by Oracle as part of the Java Development Kit (JDK) and the Java Runtime Environment (JRE). Android, on the other hand, relies only the Java compiler found in the JDK. Instead of using Oracle's Java VM, Android relies on Dalvik, a VM custom-built for Android, and instead of using the official class library, Android relies on Apache Harmony, a clean-room reimplementation of the class library. Hence, it would seem that Google did every reasonable effort to at least avoid any copyright and/or distribution issues.

然而法律诉讼的结果仍有待观望，这可能需要几年时间才能够被解决（*译注：2018年3月28日，美国联邦巡回上诉法院当地时间周二裁定，谷歌Android系统使用Java接口侵犯了甲骨文公司的版权，谷歌面临88亿美元的赔偿*）。在此期间，主持案件的法官似乎希望认真解决这一问题。在2011年5月，他要求甲骨文公司将索赔由132减为3（亿），而谷歌需要将现有的技术应用由数百将至8个。根据Groklaw（译注：一个报道免费及开源社区相关法律新闻的网站）的说法，法官似乎在向各方询问“他们是否预感审判最终会失败”。
Still, it remains to be seen where these legal proceedings will go. And it likely will take a few years to get resolved. In the interim, however, it appears that the judge presiding over the case wants to get the matter resolved in earnest. In May 2011, he ordered Oracle to cut its claims from 132 down to 3 and ordered Google to cut their prior art references down from several hundreds to 8. According to Groklaw, he even seems to be asking the parties whether "they anticipate that a trial will end up being moot."

另一个间接而又非常相关的是，IBM在2010年10月加入甲骨文的OpenJDK。Apache Harmony一直以来主要由IBM推动，用于安卓的类库，现在IBM的离开几乎宣判了这个项目被抛弃。这件事的发展对安卓的影响在本书编写时还是个未知数。
Another indirectly related, yet very relevant, development is that IBM joined Oracle's OpenJDK efforts in October 2010. IBM had been the main driving force behind the Apache Harmony, which is the class library used in Android, and its departure pretty much ensures the project has become orphaned. How this development impacts Android is unknown at the time of this writing.

顺便提一下，James Gosling在2011年3月入职谷歌。（译注：待了五个月就跑路了，目前在亚马逊AWS团队）
Incidently, James Gosling joined Google in March 2011.

--------------------------------------------------

### 硬件要求 Hardware and Compliance Requirements

原则上，安卓能够运行在任何可以运行Linux的硬件上。现在的安卓已经可以运行在ARM，x86，MIPS，SuperH及PowerPC这些支持Linux的架构上。如果想将安卓移植到你的硬件设备上，首先你需要将Linux移植过去。除了使用某种显示和指针机制来允许用户与接口交互之外，运行AOSP几乎没有额外的硬件需求。当然，如果硬件不支持AOSP期望的外设，你还是需要根据具体情况做适当的修改。例如板子上不存在GPS模块，你可能会想要像安卓模拟器那样为AOSP在硬件抽象层提供一个虚拟的GPS模组。另外，你也需要确保有足够的空间存储安卓镜像，并使用一颗足够强大的处理器为用户提供优良的使用体验。
In principle, Android should run on any hardware that runs Linux. Android has in fact been made to run on ARM, x86, MIPS, SuperH, and PowerPC, all architectures supported by Linux. A corrolary to this is that if you want to port Android to your hardware, you must first port Linux to it. Beyond being able to run Linux, though, there are few other hardware requirements for running the AOSP, apart from the logical requirement of having some kind of display and pointer mechanism to allow users to interact with the interface. Obviously, you might have to modify the AOSP to make it work on your hardware configuration, if you don't support a peripheral it expects. For instance, if you don't have a GPS, you might want to provide a mock GPS HAL module, as the Android emulator does, to the AOSP. You will also need to make sure that you have enough memory to store the Android images and a sufficiently powerful CPU to give the user a decent experience.

总的来说，如果仅仅想将AOSP运行在自己的硬件上几乎没有什么限制。但是如果设备必须要贴上“安卓”商标或必须包含如谷歌地图、谷歌应用商店这些标准的谷歌套件，你就需要通过我之前提到的ACP。ACP有两个独立但互补的部分：兼容性定义文件（CDD）及兼容性测试套件（CTS）。即使不打算过ACP，你也可以看一看CDD和CTS，因为它们给出了安卓设计目标的总体思维模式。
In sum, therefore, there are few restrictions if you just want to get the AOSP up and running on your hardware. If, however, you are working on a device that must carry "Android" branding or must include the standard Google-owned applications found in typical consumer Android devices such as the Maps or Market applications, you need to go through the ACP that I mentioned earlier. There are two separate yet complementary parts to the ACP: the Compliance Definition Document (CDD) and the Compliance Test Suite(CTS). Even if you don't intend to participate in the ACP, you might still want to take a look at the CDD and the CTS, as they give a very good idea about the general mindset that went into the design goals of the Android version you intend to use.

ACP的主要目标，细分到CDD与CTS，是确保为用户及应用开发人员提供统一的生态系统。因此，当你被允许使用“安卓”商标之前，谷歌想要保证不会被不兼容或残缺的产品破坏整个安卓的生态。反过来说，制造商们也能够受益于人们对安卓品牌的认同。关于ACP的更多细节，可以参考http://source.android.com/compatibility/
The overarching goal of the ACP, and therefore the CDD and the CTS, is to ensure a uniform ecosystem for user and application developers. Hence, before you are allowed to ship an "Android"-branded devices, Google wants to make sure that you aren't fragmenting the Android ecosystem by introducing incompatible or crippled products. This, in turn, makes sense for manufacturers since they are beneifting from the compliance of others when they usd the "Android" branding. For more detail about the ACP, have a look at http://source.android.com/compatibility/.

#### 兼容性定义文档 Compliance Definition Document

兼容性定义文档（后文简称CDD）是ACP的策略部分，可以在上面列出的URL中获取。它指出了兼容性设备必须满足的需求。CDD基于RFC2119的语言描述，大量使用“MUST（表示必须要这样做）”，“SHOULD（表示一般情况下应该这样做，特殊情况下可以忽视）”，“MAY（表可选，即可做可不做）”等词来描述不同的属性。文档约有25页长，涵盖了软硬件各个方面的能力。本质上，它覆盖的方面不能仅仅通过CTS去做简单的自动化测试。让我们来看一看CDD的要求。
The CDD is the policy part of the ACP and is available at the ACP URL above. It specifies the requirements that must be met for a device to be considered compatible. The language in the CDD is based on RFC2119 with a heavy use of "MUST," "SHOULD," "MAY," etc. to describe the different attributes. Around 25 pages in length, it covers all aspects of the device's hardware and software capabilities. Essentially, it goes over every aspect that cannot simply be automatically tested using the CTS. Let's go over some of what the CDD requires.

##### 软件 Software

这一节列出了Java和原生API以及web、虚拟机、用户接口兼容性要求。如果你使用的是AOSP，那么应该很容易符合这部分CDD的要求。
This section lists the Java and Native APIs along with the web, virtual machine and user interface compatibility requirements. Essentially, if you are using the AOSP, you should readily conform to this section of the CDD.

##### 应用打包兼容性 Application Packaging Compatibility

本节指出你的设备必须能够安装和运行*.apk*文件。所有的安卓应用都使用Android SDK编译成*.apk*文件。这些文件将通过安卓应用市场发布并安装到用户的设备上。
This section specifies that your device must be able to install and run .*apk* files. All Android apps developed using the Android SDK are compiled into .*apk* files, and these are the files that are distributed through the Android Market and installed on users's devices.

##### 多媒体兼容性 Multimedia Compatibility

CDD在这里描述了设备的媒体编解码器（解码器与编码器）、音频录制及音频延迟（区别于delay，更像是潜伏）的要求。AOSP中包含了StageFright多媒体框架，所以使用AOSP是能够通过CDD的。但是，你仍然需要阅读关于音频录制及延迟的章节，这其中包含的特殊技术信息可能对设备上必须配置的硬件配置有所影响。
Here, the CDD describes the media codecs (decoders and encoders), audio recording, and audio latency requirements for the device. The AOSP includes the StageFright multi-media framework, and you can therefore conform to the CDD by using the AOSP. However, you should read the audio recording and latency sections, as they contain specific technical information that may impact the type of hardware configuration your device must be equipped with.

##### 开发人员工具兼容性 Developer Tool Compatibility

这一节列举了设备中必须提供的特定工具。如*adb*，*ddms*和*monkey*都是一些在开发及测试过程中常用的工具。
This section lists the Android-specific tools that must be supported on your device. Basically, these are the common tools used during app development and testing: *adb, ddms,* and *monkey*. Typically, development environment and use the Android Development Tool (ADT), plugin which takes care of interacting the lower-level tools.

##### 硬件兼容性 Hardware Compatibility

这或许对嵌入式开发人员来说是最重要的一部分，这可能会对目标设备的设计决策带来深远的影响。下面列出了每个章节的总结。
This is probably the most important section for embedded developers as it likely has profound ramifications on the design decisions made for the targeted device. Here's a summary of what each subsection spells out.

*显示及图像*
*Display and Graphics*
+ 设备的屏幕在物理显示上至少要有2.5吋
+ Your device's screen must be at least 2.5'' in physical disgonal size.
+ 分辨率至少为100dpi
+ Its density must at least 100dpi
+ 宽高比必须在4:3与16:9之间
+ Its aspect ratio must be between 4:3 and 16:9
+ 屏幕显示必须支动态持横竖屏切换。如果不支持切换，那必须支持宽屏模式，因为应用可能会强制改变显示方向。
+ It must support dynamic screen orientations from portrait to landscape and viceversa. If orientation can't be changed, it must support letterboxing since apps may force orientation changes.
+ 必须支持OpenGL ES 1.0
+ It must support OpenGL ES 1.0 though it may omit 2.0 support.

*输入设备*
*Input Devices*
+ 设备必须支持输入法框架，并且允许开发人员创建定制的屏上软键盘
+ Your device must support the Input Method Framework, which allows developers to create custom on-screen, soft keyboards.
+ 必须提供至少一个软键盘
+ It must provide at least one soft keyboard.
+ 不允许提供不符合API设计的实体键盘
+ It can't include a hardware keyboard that doesn't conform the API.
+ 必须提供“HOME”，“MENU”及“BACK”键
+ It must provide "HOME", "MENU", and "BACK" buttons.
+ 必须具备触摸屏，无论是电容屏还是电阻屏
+ It must have a touchscreen, whether it capacitive or resistive.
+ 一般情况下需要支持独立跟踪点（多点触摸）
+ It should support independent tracked points (multi-touch) if possible.

*传感器*
*Sensors*
虽然所有的传感器描述都是用“SHOULD”，但这并不意味着它们不是强制性的。设备必须准确地报告传感器存在还是不存在，并且必须返回所支持传感器的准确列表。
While all sensors are qualified using "SHOULD," meaning that they aren't compulsory. your device must accurately report the presence or absence of sensors and must return an accurate list of supported sensors.

*数据连接*
*Data Connectivity*
这里最重要的一项是明确指明安卓可能被用于不支持电话的设备。增加这点是因为存在基于安卓的平板设备。此外，文档还指出设备在硬件上应该支持802.11x（wlan）、蓝牙及NFC。最后，设备必须支持带宽200Kbits/s的网络格式。
The most important item here is an explicit statement that Android may be used on devices that don't have telephony hardware. This was added to allow for Android-based tablet devices. Furthermore, the document says that your device should have hardware spport for 802.11x, Bluetooth, and NFC. Ultimately, your device must support some form of networking that permits a bandwidth of 200Kbits/s.

*相机*
*Cameras*
设备必须有一个后置摄像头，也可以有一个前置摄像头。
Your device should include a rear-facing camera and may a front-facing one as well.

*内存与存储*
*Memory and Storage*
+ 设备必须有至少128M来存储内核及用户空间
+ Your device must have at least 128MB for storing the kernel and user space.
+ 必须有至少150M存储用户数据
+ It must have at least 150MB for storing user data.
+ 必须有至少1G的“共享存储”，本质上指SD卡
+ It must have at least 1GB of "shared storage," essentially meaning the SD card.
+ 必须提供从PC访问的共享存储机制。换言之，设备通过USB连接PC时，SD卡的内容必须能够在PC上访问。
+ It must also provide a mechanism to access shared data from a PC. In other words, when the device is connected through USB, the content of the SD card must be accessible on the PC.

*USB*
*USB*
这个需求可能是安卓品牌设备以用户为中心的重要体现。它假设用户拥有一台设备后，要求能够通过PC连接时完全控制这台设备。在某些情况下这对开发者而言是一个严重的问题，因为你可能并不想让使用者通过PC连接你的嵌入式设备。尽管如此，CDD这么要求：
This requirement is likely the one that most heavily demonstrates how user-centric "Android"-branded devices must be, since it essentially assumes the user owns the device and therefore requires you to allow users to fully control the device when it's connected to a PC. In some cases this might be a show-stopper for you, as you may not actually want or be able to have have users connect your embedded device to a user's PC. Nevertheless, the CDD says that:
+ 设备必须实现一个能够通过USB接口连接的USB客户端
+ Your device must implement a USB client which would be connectable through USB-A.
+ 必须实现在USB中的*adb*命令中实现安卓调试桥（ADB）
+ It must implement the Android Debug Bridge (ADB) protocol as provided in the *adb* command over USB.
+ 必须实现USB大容量存储，从而允许通过主机访问设备SD卡
+ It must implement USB mass storage, thereby allowing the device's SD card to be accessed on the host.

##### 性能兼容性 Performance Compatibility

虽然CDD没有指定CPU的速度要求，但它指定了app相关的时间限制。这与硬件所选择的CPU速度相关。例如：
Although the CDD doesn't specify CPU speed requirements, it does specify app-related time limitations that will impact your choice of CPU speed. For instance:
+ 浏览器应用程序必须在1300毫秒内启动
+ The Browser app must launch in less than 1300ms.
+ MMS/SMS程序必须在700毫秒内启动
+ The MMS/SMS app must launch in less than 700ms.
+ 闹钟程序必须在650毫秒内启动
+ The Alarm Clock app mush launch in less than 650ms.
+ 重新启动一个正在运行的程序所花费的时间必须比首次启动时间要少
+ Relaunching an already running app must take less time than original launch.

##### 安全模式的兼容性 Security Model Compatibility

设备必须遵循由安卓应用框架、Dalvik虚拟机与Linux内核实施的安全环境。具体来说，应用程序必需能够按照SDK文档中所描述的那样，能够访问并获得相应的权限。应用程序必须受到像Linux以不同UID作为单独进程运行这样的沙箱限制。文件系统访问权限也必须符合开发人员文档中的对应描述。最后，如果你不使用Dalvik，那么使用任何其他虚拟机都必须提供与Dalvik一样的安全行为。
Your device must conform to the security environment enforced by the Android application framework, Dalvik, and the Linux kernel. Specifically, apps must have access and be submitted to permission model described as part of the SDK's documentation. Apps must also be constrained by the same sandboxing limitations they have by running as separate processes with distinct UIDs in Linux. The filesystem access rights must also conform to those described in the developer documentation. Finally, if you aren't using Dalvik, whatever VM you use should impose the same security behavior as Dalvik.

##### 软件兼容性测试 Software Compatibility Testing

设备必须通过CTS，包括人工操作的CTS验证部分。另外设备必须能够运行安卓应用市场中指定的参考应用程序。
Your device must pass the CTS, including the human-operated CTS Verifier part. In addition, you device must be able to run specific reference applications from the Android Market.

##### 可更新软件 Updatable Software

必须提供一种更新设备的机制。这可以通过OTA去完成，并以重启的方式离线更新。也可以通过USB连接PC机线刷，或者通过其他移动存储设备离线更新。
There has to be a mechanism for your device to be updated. This may be done  over-the-air (OTA) with an offline update via reboot. It also may be done using a "tethered" update via a USB connection to a PC, or be done "offline" using removable storage.

#### 兼容性测试套件 Compliance Test Suite

CTS作为AOSP的一部分，我们将在第10章探讨。AOSP包含一个特殊的构建目标并生成*cts*命令行工具作为控制测试套件的主要接口。CTS通过*adb*在USB连接目标中推送并运行测试项。这些测试项是建立在JUnit上的java单元测试框架，并执行框架中如API、Dalvik、Intent、Permission等不同的部分。一旦测试完成，它们会生成一个包含了XML文件及对应屏幕截图的*.zip*压缩文件，这些文件需要被提供给cts@android.com
The CTS comes as part of the AOSP, and we will discuss it in Chapter 10. The AOSP includes a special build targets that generates the *cts* command-line tool, the main interface for controling the test suite. The CTS relies on *adb* to push and run tests on the USB-connected target. The tests are build on to the JUnit Java unit testing framework and exercise different parts of the framework, such as the APIs, Dalvik, Intents, Permissions, etc. Once the test are done, they will generate a .*zip* file containing XML files and screen-shots that you need to submit to cts@android.com

--------------------------------------------------

### 开发计划及工具 Development Setup and Tools

安卓开发有两套独立的工具：应用开发工具及平台开发工具。如果想搭建一个应用开发环境，可以看看*学习安卓*或谷歌的在线文档。如果想从事平台开发，像我们现在正在做的，需要的工具则有所不同，在本书后面将会提到。
There are two separate sets of tools for Android development: those used for application development and those used for platform development. If you want to set up an application development environment, have a look at *Learning Android* or Google's online documentation. If you want to do platform development, as we will do here, your tools needs will vary, as we will see later in this book.

不过，在最基础的层面上，你需要有一台基于Linux的工作站来编译AOSP。事实上，在本书编写之际，谷歌唯一支持的编译环境是64位Ubuntu 10.04。但这并不意味着其他Ubuntu版本或是32位系统上无法编译AOSP，这只是表明了谷歌官方的编译配置。在不改变你原有操作系统的情况下，另一个简单方法是使用Ubuntu虚拟机。作者通常使用VirtualBox，因为他发现VirtualBox可以方便地访问主机串口。（都开始玩AOSP了主机当然是Linux嘛啊哈哈哈，虽然我也试过用虚拟机编4.4但是没有成功_ (:△」∠)_）
At the most basic level, though, you need to have a Linux-based workstation to build the AOSP. In fact, at the time of this writing, Google's only supported build environment is 64-bit Ubuntu 10.04. That doesn't mean that another Ubuntu version won't work or that you won't be able to build the AOSP on a 32-bit system, but essentially that configuration reflects Google's own Android compile farms configuration. An easy way to get your hands dirty with AOSP work without changing your workstation OS is to create an Ubuntu virtual machine using your favorite virtualization tool. I typically use VirtualBox, since I've found that it makes it easy to access the host's serial ports in the guest OS.

无论你的选择是什么，请记住，AOSP的源码大约4GB，一旦编译后就会增长到10GB（以前的AOSP真小呀）。如果你计划为了测试去编译几个单独的版本，你将很快意识到需要几十个G的硬盘空间才行。另外还需要注意的是，最近的机器（双核高端笔记本）需要花费近一个小时从头开始构建AOSP，就算是一些小修改也要5分钟才能完全编完（这里应该说的是局部编译？）。因此，在开发嵌入式安卓系统时，必须要确保有一台足够强劲的机器。
No matter what your setup is, keep in mind that the AOSP is about 4GB in size, uncompiled, and that it will grow to about 10GB once compiled. If you factor in that you are likely goning to operate on a few separate versions, for testing purposes if for no other reason, you rapidly realize that you'll need tens of GBs for serious AOSP work. Also note that on what is fairly recent machine at the time of this writing (dual-core high-end laptop), it makes about an hour to build the AOSP from scratch. Even a minor modification may result in a 5 min run to complete th build. You will therefore also likely want to make sure you have a fairly powerful machine when developing Android-based embedded systems.

--------------------------------------------------