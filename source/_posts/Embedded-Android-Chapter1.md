---
title: Embedded Android -- Chapter1
date: 2018-03-18 16:44:25
tags:
---


### 简介 Introduction
将安卓操作系统移植到嵌入式设备上是一项复杂的任务，这包括了对系统内部复杂设计的理解，巧妙地将修改融入到安卓开源项目，以及对内核运行平台——Linux的掌握。在开始进入到嵌入式安卓细节之前，先让我们了解一些安卓系统背后的本质内容，例如安卓的硬件需求，安卓的法律体制和对嵌入式设备的影响等。首先，让我们一起看一看安卓的诞生与发展。
Putting Android on an embedded device is a complex task involving an intricate understanding of its internals and a clever mix of modifications to the Android Open Source Project (AOSP) and the kernel on which it runs, Linux. Before we get into the details of embedding Android, however, let's start by covering some essential background that embedded developers should factor in when dealing with Android, such as Android's hardware requirements and the legal framework surrounding Android and its implications within an embedded setting. First let's look at where Android comes from and how it's developed.

<!-- more -->

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


### 发开模式 Development Model

当你考虑是否使用安卓时，了解它的发展过程对修改源码或面对内部依赖时就显得相当重要了。
When considering whether to use Android, it's crucial that you understand the ramifications its development process may have on any modifications you make to it or any dependencies you may have towards its internals.

#### 不同于“传统的”开源项目 Differences With "Classic" Open Source Projects

开源是安卓最为人知的特点。事实上，正如我们所看见，许多开源的软件工程项目被使用在安卓系统中。
Android's open source nature is one of the most trumpeted features. Indeed, as we've just seen, many of the software engineering benefits that derive from being open source apply to Android.

尽管有许可证，但安卓却不像多数开源项目那样，它的开发基本上处于非公开状态。比如，大多数的开源项目有公共邮件列表及论坛，可以与主要开发人员交流，且公共的仓库能够提供主开发分支的最新提交。安卓并没有这些。
Despite its licensing, however, Android is unlike most open source projects in that its development is mostly done behind closed doors. The vast majority of open source projects, for example, have public mailing lists and forums where the main developers can be found interacting with each other, and public source repositories providing access to the main development branch's tip. No such thing can be found for Android.

最好的总结获取来自Andy Rubin，“开源项目与社区驱动型项目不同。相比于社区，安卓更加重视开源？？？”
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

Furthermore, it can easily be argued that from a business and go-to-market perspective a community-driven process would definitely knock the wind out of any product announcements Google would attempt to release, making it impossible to create "buzz" around press announcements and the like since every new feature would be developed in the open. That is to say nothing of the non-deterministic nature of community-driven processes that can see a group of people take years to agree on the best way to implement a given feature set. And, simply based on track record, Andoid's success has definitely benefited from Google's ability to rapidly move it forward and generate press interset based on releases of cool new products.

#### Feature Inclusion, Roadmaps, and New Releases

In brief, there is no publicly available roadmap for features and capabilities in future Android releases. At best, Google will announce ahead of time the name and approximate release date of the next version. Usually, you can expect a new Android release to be made in time for the Google I/O conference, which is typically held in May, and another release by year end. What will be in that release, though, is anyone's guess.

Typically, however, Google will choose a single manufacturer to work with on the next Android release. During the period, Google will work very closely with that single manufacturer's engineers to ready up the next Android version to work on a targeted upcoming flagship device. During the period, the manufacturer's team is reported to have access to the tip of the development branch. Once the device is put on the market, the corresponding source code dump is made to the public repositories. For the next release, they choose another manufacturer and start over.

There is one notable exception to the cycle: Android 3.x/Honeycomb. In that specific case, Google has not released the source code to the corresponding flagship product, the Motorola Xoom, and all indications suggest that that code may never be publicly available. The ratinable in this case seems to be that Android development team essentially forked the Android code-base at some point in time in order to start working on getting a tablet-ready version of Android out ASAP based on market timing prerogatives. Hence, in that version, very little regard was made to preserving backward compatibility with the phone form-factor. And given that, Google did not wish to make the code available to avoid fragmentation of its platform. Instead, at the time of this writing, plans are that both the phone and tablet form factor support will be merged into the upcoming Android 4.0/Ice-Cream sandwich release.


### Ecosystem
As of June 2011:
+ 400,000 Android phones are activated each day, up from 300,000 in December 2010 and 200,000 in August of that same year.
+ The Android Market contains around 200,000 apps. In comparison, the Apple App Store has 350,000 apps.
+ More than a third of all phones sold in the US are based on Android.
Android is clearly on the upswing. In fact, Gartner predicted in April 2011 that Android would hold about 50% of the smartphone market by 2015. Much as Linux disrupted the embedded market about a decade ago, Android is poised to make its mark. Not only will it flip the mobile market on its head, eliminating or sidelining even some of the strongest players, but in the embedded space it is likely going to become the defacto standard UI for a vast majority of user-centric embedded devices.

An entire ecosystem is therefore rapidly building around Android. Silicon and system-on-chip (SoC) manufacturers such as ARM, TI, Qualcomm, Freescale, Nvidia and TI hace added Android support for their products, and handset and tablet manufacturers such as Motorola, Samsung, HTC, Sony-Ericsson, LG, Archos, DELL, ASUS, etc. ship an ever-increasing number of diverse players, such as Amazon, Verizon, Sprint and Barnes&Nobles, creating their own application markets.

Grass-roots communities and projects are also starting to sprout around Android, even though it is developed behind closed doors. Some of those efforts follow in the footsteps of phone modders, which essentially rely on hacking the binaries provided by the manufacturers to create their own modifications or variants, while others have a more open source tint to them, relying on the Android sources to create their own forks or additions. THe *xda-developers.com* online forum, for instance, is traditionally frequented by modders, while the *cyanogenmod.com* site hosts an Android fork which modifies the Android sources to provide additional features and enhancements. Ohter Android forks include Replicant (*http://replicant.us*), which aims at replacing as many of the Android components with Free Software as possible, and MIUI (*http://en.miui.com/*), which provides some cool UI hacks.

### A Word on the Open Handset Alliance

to be continued....