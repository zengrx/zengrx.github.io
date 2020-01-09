---
title: Embedded Android -- Chapter2
date: 2018-11-10 11:06:33
tags: 
- Embedded Android Translation
---


**内部入门 Internals Primer**

正如我们之前所说，安卓源码可以自由地为人们所下载、修改以及安装至选择的机器中。实际上，仅仅是获取、编译代码，再将其运行在安卓模拟器这个过程并不值得一提。能够为你自己的硬件及设备客制化AOSP，做到这件事的前提是需要对安卓的内部结构有一定掌握。所以在这一章，我们先从一个较高的视角去看待安卓内部，在后续章节中再找机会深入理解各个部分。
As we've just seen, Android's source are freely available for you to download, modify, and install for any device you choose. In fact, it is fairly trivial to just grab the code, build it, and run it in the Android emulator. To customize the AOSP to your device and its hardware, however, you'll need to first understand Android's internals to a certain extent. So we'll get high-level view of Android internals in this chapters, and get opportunity in later chapters to dig into parts of internals in greater detail, including trying said internals to the actual AOSP sources.

<!-- more -->

### 应用开发人员的看法 App Developer's View

为安卓研发人员提供的API并不同于其他现有的API，包括Linux世界中能够找到的任何API，这值得花一些时间以应用开发人员的视角去理解“Android”到底是什么，虽然每个人研究AOSP后对安卓的理解都不一样。作为一名嵌入式工程师，你或许不需要像其他一些同事那样直接面对安卓应用的API的特性。如果没有别的，你也可以和他们共享通用的语言。当然，这部分只是一个总结，并且也建议你钻研一些应用开发的知识，从而获得更加深入的理解。
Given that Android's development API is unlike any other existing API, including anything found in the Linux world, it's important to spend some time understanding what "Android" looks like from the app developers' perspective, even though it's very different from what Android looks like for anyone hacking the AOSP. As an embedded developer working on embedding Android on device, you might not have to actually deal directly with the idiosyncracies of Android's app development API, but some of your colleagues might. If nothing else, you might as well share a common linguo with app developers. Of course, this section is merely a summary, and I recommend you read up on Android app development for more in-depth coverage.

#### 安卓概念 Android Concepts

应用开发人员在研发安卓应用时必须要掌握几个关键的概念。这些概念塑造了所有安卓应用的体系，并规范了开发人员什么能做，什么不能做。总的来说，它们把用户的生活变得更好，但有时也会遇到一些挑战。
Application developers must take a few key concepts into account when developing Android apps. These concepts shape the architecture of all Android apps and dictate what developers can and cannot do. Overall, they make the users' life better, but they can sometimes be challenging to deal with.

##### 组件 components

安卓应用程序由一些零散而有联系的组件构成。一个应用程序的组件可以授权使用或使用另一个应用的组件。最重要的是，这里面并没有如**main()**一样单一入口点之类的东西。相反，开发人员使用一个叫做*intent*的预定义事件将组件们联系在一起，从而在发生相应事件时激活它们的组件。一个简单的例子是处理用户联系人数据库的组件，当用户按下拨号器或其他程序的联系人按钮时会被激活。所以一个应用程序可以拥有与它组件相同数量的入口点。
Android applications are made of loosely-tied *components*. Components of one app can invoke/use components of other apps. Most importantly, there is no single entry point to an Android app: no **main()** function or any equivalent. Instead, there are pre-defind events called *intents* that developers can tie their components to, thereby enabling their components to be activated on the occurrence of the corresponding events. A simple example is the component that handles the user's contacts database, which is invoked when the user presses a Contacts button in the Dialer or another app. An app therefore can have as many entry points as it has components.

安卓的四大组件如下：
There are four main types of components:

>*活动* *Activities*
正如“窗体”在基于视窗用户界面的系统中作为主要的视觉交互构建块一样，活动就是安卓应用的构建块。但不同于窗体，活动不能够被“最大化”，“最小化”或者“调整大小”。反之，活动永远占据整个视觉区域，堆叠在其他活动的顶端。并类似浏览器记录网页访问序列那样允许用户“返回”之前的地方。事实上，如之前章节所描述，所有的安卓设备都必须要有一个“BACK”键来指示用户操作行为。与浏览器不同的是，并不存在一个“前进”按钮，活动只允许“返回”。
Just as the "window" is the main building block of all visual interaction in window-based GUI system, acitvites are the building block in an Android app. Unlike a window, however, activities cannot be "maximized," "minimized," or "resized." Instead, activities always take the entirety of the visual area and are made to be stacked up on top of each other in the same way as a browser remembers webpages in the sequence they were accessed, allowing the user to go "back" to where he was previously. In fact, as descirbed in the previous chapter, all Android devices must have a "BACK" button to mark this behavior available to the user. In contrast to web browsing, though, there is no button corresponding to the "forward" browsing action; only "back" is possible.
另一个全局定义是安卓意图允许活动在应用启动器（手机的应用程序列表页）上展示一个图标。因为绝大多数的应用都希望被展示在主程序列表中，它们提供了至少一个活动去响应这个意图。一般而言，使用者将从一个特定 的活动开始，再移动到另一个活动，最后创建与这些活动相关的堆栈。这些活动组成的堆栈就是*任务*。用户可以通过点击HOME按键进入应用启动界面，启动另一个活动堆栈来切换一个新的任务。
One globally defined Android intent allows an acitvity to be displayed as an icon on the app launcher (the main app list on a phone.) Because the vast majority of apps want to appear on the main app list, they provide at least one activity  that is defined as capable of responding to that intent. Typically, the user will start from a particular activity and move through other end up creating a stack of activities all related to the one they originally launched; this stack of activities is a *task*. The user can then switch to another task by clicking the HOME button and starting another activity stack from the app launcher.

>*服务* *Services*
安卓中的服务有些类似于Unix系中的后台进程或守护进程。本质上，服务在其他组件有需求是被激活的，并且一般会根据调用者的需求持续一段时间。最重要的是，服务可以被提供给从程序外的组件，从而为其他应用提供一些核心功能。服务激活时通常没有用户界面。
Android services are akin to background processes or daemons in the Unix world. Essentially, a service is a activated when another component requires its services and typically remains active for the duration required by its caller. Most importantly, though, services can be made available to components outside an app, thereby exposing some of that app's core functionality to other apps. There is usually no visualsign of service being active.

>*广播接收器* *Broadcast Receivers*
广播接收器有点像中断处理程序。当一个关键事件发生时，广播接收器会被触发处理应用的行为。例如一个应用可能希望在低电或飞行模式（为了关闭无线连接）时得到通知。当不处理它们所注册的特殊事件时，广播接收器是不活动的。
Broadcast receivers are akin to interrupt handlers. When a key event occurs, a broadcast receiver is triggered to handle that event on the app's behalf. For instance, an app might want to be notified when the battery level is low when "airplane mode" (to shut down the wireless connections) has been activated. When not handling a specific event for which they are registered, broadcast receivers are otherwise inactive.

>*内容提供者* *Content Providers*
内容提供者本质上就是数据库。一般来说一个应用会在需要为其他应用提供数据访问时包含一个内容提供者。例如开发一个Twitter客户端应用，你可以通过内容提供者给设备上的其他应用程序获取推送信息。大部分的内容提供者依赖安卓中的SQLite功能，但是也可以使用用户文件或者其他类型存储。
Content providers are essentially databases. Usually, an app will include a content provider if it needs to make its data accessible to other apps. If you're building a Twitter client app, for instance, you could give other apps on the device access the tweet feed you're persenting to the user through a content provider. All content providers present the same API to apps, regardless of how they are actually implemented internally. Most content providers rely on the SQLite functionality included in Android, but they can also user files or other types of storage.

##### 意图 Intents

意图也是安卓中最为重要的概念之一。它们是允许组件间相互作用的后期绑定机制。应用开发人员可以通过发送意图的方式，让活动“浏览”一个网页或者PDF，因此即使活动本身不支持这类的能力，使用者也能够以这种形式实现。当然也有更魔幻的意图使用方式，只要开发人员想的话，可以发出一条特殊的意图拨打电话。
Intents are one of the most important concepts in Android. They are the late-binding mechanisms that allow components to interact. An app developer could send an intent for an activity to "view" a web page or "view" a PDF, hence making it possible for the user to view a designated HTML or PDF document even if the requesting app itself doesn't include the capabilites to do so. More fancy use of intents is also possible. An app developer could, for instance, send a specific intent to trigger a phone call.

可以将意图理解为一种不用预定义或需要特殊设计的目标组件和应用的多态Unix信号。意图本身是一种被动的对象，是内容（载荷），激发其的机制会与系统规则一起引导它的行为。例如其中的一个系统规则是意图与其被发送到的组件相关。就像被发送至服务的意图就只能够被服务接收，无法被活动或者广播接收器接收。
Think of intents as polymorphic Unix signals that don't necessarily have to be predefined or require a specific designated target component or app. The intent itself is a passive object. It's contents (payload), the mechanism used to fire it along with the system's rules that will gate its behavior. One of the system's rules, for instance, is that intents are tied to the type of component they are sent to. An intent sent to a service, for example, can only be received by a service, not an activity or broadcast receiver.

组件可以通过*manifast*的过滤被声明为处理特定的意图类型。系统随后将意图与过滤器匹配，并在运行时出发相应的组件。意图也能够发送给显式的过滤器，以绕过接收组件对意图声明的需求。但是，显式调用要求应用提前知道被指定的组件，这通常仅适用于同一个应用程序中的组件发送意图的情况。
Components can be declared as capable of dealing with given intent types using filters in the *manifast* file. The system will thereafter match intents to that filter and trigger the corresponding component at runtime. An intent can also be sent to an explicit filter, bypassing the need to declare that intent within the receiving componnent's filter. The explicit invocation, though, requires the app to know about the designated component ahead of time, which typically applies only when intents are sent within comonents of hte same app.

##### 组件的生命周期 Component Lifecycle

安卓的另一个中心思想是用户不应该管理任务的切换。因此，安卓中并不会存在任务栏之类的功能。相反，用户可以随意启动应用程序，并通过点击HOME键后选择主界面上的其他应用来切换。这些被点击的应用可能是全新的，也可能是之前已经启动过的，即活动栈（也可以说任务）已经存在。
Another central tenant of Android is that the user should never have to manage task switching. Hence, there is no task-bar or any equivalent functionality in Android. Instead, the user is allowed to start as many apps as he wants and "switch" between apps by clicking HOME to go to the home screen and clicking on any other app. The app he clicks may be an entirely new one, or one that he previously started and for which
an activity stack (a.k.a. a "task") already exists.

（corollary？）这种设计会导致应用程序逐渐消耗越来越多的系统资源，而这些资源不能保证操作永远继续下去。有些时候，系统会不得不回收一些最近较少使用或无优先级的组件来获取资源，留给新激活的组件使用。然而，资源回收应该对用户完全透明。换言之，当旧组件被新组件换下后，用户又回到原来的组件时，组件应该从它被换下的地方开始启动，就好像它一直在内存中等待一样。
The corrollary to, or consequence of, this design decision is that apps gradually use up more and more system resources as they are started, which can't go on forever. At some point, the system will have to start reclaiming the resources of the least recently used or non-priority components in order to make way for newly-activated components. Yet still, this resource recycling should be entirely transparent to the user. In other words, when a component is taken down to make way for new one, and then the user returns to the original component, it should start up at the point where it was taken down and act as if it was waiting in memory all along.

为了实现这个行为，安卓定义了一个标准生命周期组件。应用程序开发人员必须通过实现一系列与生命周期相关事件触发的组件回调，从而管理组件的生命周期。例如，当一个活动不再存在于前台（因此比在前台更容易被杀），它的**onPause()**回调则被触发。
To make this behavior possible, Android defines a standard lifecycle components. An app developer must manage her components' lifecycle by implementing a series of callbacks for each component that are triggered by events related to the component lifecycle. For instance, when an activity is no longer in the foreground (and therefore more likely to be destroyed than if it's in the foreground), its **onPause()** callback is triggered.

处理组件回调是应用开发人员最大的挑战之一，因为在处理关键事件时他们必须要小心地保存和恢复组件状态。最理想的结果是，用户不需要在应用间切换任务或担心开启新应用后之前的应用会被杀掉。
Managing component lifecycles is one of the greatest challenges faced by app developers, because they must carefully save and restore component states on key transitional events. The desired end result is that the user never needs to "task switch" between apps or be aware that components from previously-used apps were destroyed to make way for new ones he started.

##### Manifest文件 Manifest File
如果一定要说一个应用中的“主”入口点，那么manifest文件可以算得上。基本上，它告诉了系统应用程序的组件，运行应用的要求，最小API等级以及硬件需求等等。Manifest格式像xml文件一样，以*AndroidManifest.xml*名称被保存在应用程序文件的最顶端目录中。应用的组件通常都是静态地描述在manifest中。事实上，除了广播接收器可以在运行时被动态注册，其余的组件都必须在构建时被声明在manifest文件中。
If there has to be a "main" entry point to an app, the manifest file is likely it. Basically, it informs the system of the app's components, the capabilities required to run the app, the minimum level of the API required, any hardware requirements, etc. The manifest is formatted as an XML file and resides at the top-most directory of the app's sources as *AndroidManifest.xml*. The apps' components are typically all described statically in
the manifest file. In fact, apart from broadcast receivers, which can be registered at runtime, all other components must be declared at build time in the manifest file.

##### 进程与线程 Processses and Threads
无论应用何时被激活，是由系统还是其他应用激活，都会起一个进程来容纳该应用的组件。除非开发人员重载了系统默认方法，否则初始组件起来之后所有的应用组件都会跑在同一个进程中。换句话说，一个应用的所有组件都会运行在单一的Linux进程里。因此，开发人员需要避免在标准组件中进行耗时或阻塞操作，并将这类操作放在线程中实现。
Whenever an app's component is activated, whether it be by the system or another app, a process will be started to house that app's components. And unless the app developer does anything to overide the system defaults, all other components of that app that start after the initial component is activated will run within the same process as that component. In other words, all components of an app are comprised within a single Linux process. Hence, developers should avoid making long or blocking operations in standard components and use threads instead.

而且，用户基本会允许激活任意数量的组件，基本上，这几个为包含用户组件的应用程序服务的Linux进程会一直存在于系统中。当运行太多进程导致无法无法启动新进程时，Linux内核将启动OOM机制。这时，安卓内核中的OOM将被调用，并会杀掉必须结束的进程以释放空间。
And because the user is essentially allowed to activate as many components as he wants, several Linux processes are typically active at any time to serve the many apps containing the user's components. When there are too many processes running to allow for new ones to start, the Linux kernel's Out-Of-Memory (OOM) killing mechanisms will kick in. At that point, Android's in-kernel OOM handler will get called and it will determine which processes must be killed to make space.

简而言之，安卓的整体行为取决于低内存情况。
Put simply, the entirety of Android's behavior is predicated on low-memory conditions. 

如果应用开发人员已经正确实现了组件的生命周期，这时应用的进程被OOM杀掉后，用户不会看到任何不良的行为。实际上，出于实际目的，用户甚至不会注意到容纳应用程序组件的进程消失，并且随后又会“自动”重新创建。
If the developer of the app whose process is killed by Android's OOM handler has implemented his components' lifecycles properly, the user should see no adverse behavior. For all practical purposes, in fact, the user should not even notice that the process housing the app's components went away and got recreated "automagically" later.

##### 远程过程调用（RPC） Remote Proceduce Calls (RPC)

与系统中许多其他组件一样，安卓定义了它自己的RPC/IPC（IPC-进程间通信）机制：Binder。所以跨组件通信不是使用典型的系统IPC或套接字实现，而是通过访问/dev/binder的方式使用内核Binder机制，这部分会在本章稍后部分介绍。
Much like many other components of the system, Android defines its own RPC/IPC mechanism: Binder. So communication across components is not done using the typical System V IPC or sockets. Instead, components use the in-kernel Binder mechanism, accessible through /dev/binder, which will be covered later in this chapter.

应用开发人员并不会直接使用Binder机制，相反，他们必须使用安卓的接口描述语言（IDL）。接口描述通常保存在一个.aidl文件中，由aidl工具处理生成适当的存根，使用binder机制传输数据，以及在传输对象的要求下编组和解编代码。
App developers, however, do not use the Binder mechanism directly. Instead, they must define and interact with interfaces using Android's Interface Definition Language (IDL). Interface definitions are usually stored in an .aidl file and are processed by the aidl tool to generate the proper stubs and marshalling/unmarshalling code required to transfer objects and data back and forth using the Binder mechanism.

#### 框架介绍 Framework Intro

除了之前讨论的概念之外，安卓还定义了自己的开发框架，它允许开发人员在其他开发框架中访问基本功能。我们来看一看这个框架以及它的能力。
In addition to the concepts we just discussed, Android also defines its own development framework, which allows developers to access functionality typically found in other development frameworks. Let's take a brief look at this framework and its capabilities.

*用户接口* *User Interface*

安卓的UI元素包含了如按钮，文本框，对话框，菜单和事件处理这些传统控件。如果开发人员使用过其他UI框架，那么接触这部分接口相对简单。
UI elements in Android include traditional widgets such as buttons, text boxes, dialogs, menus, and event handlers. This part of the API is relatively straight-forward and developers usually find their way around it fairly easily if they've already coded for any other UI framework.

安卓中所有的UI对象都是继承于**View**类，并组织在**ViewGroup**的层次结构中。一个活动的UI可以静态地在XML（也是最常用的方法）中定义，也可以在java中动态声明。如果需要的话，也可以在运行时由java修改。一个活动的UI会在其处于**ViewGroup**顶部时才会显示。
All UI objects in Android are built as descendants of the **View** class and are organized within a hierarchy of **ViewGroup**s. An activity's UI can actually be specified either statically in XML (which is the usual way) or declared dynamically in Java. The UI can also be modified at runtime in Java if need be. An activity's UI is displayed when its content is set as the root of a **ViewGroup** hierarchy.

*数据存储* *Data Storage*

安卓为开发人员提供了多种数据存储选项。对于简单的存储需求，安卓提供了shared preferences，允许将密钥对值保存在所有组件共享的组件或特定的单独文件中。开发者也可以直接操作这些文件，文件由应用程序私下存储，所以其他程序可能无法读写。应用开发人员也可以使用安卓中的SQLite管理自己的私有数据库。其他应用可以通过内容提供者组件来访问这些数据库。
Android presents developers with several storage options. For simple storage needs, Android provides shared preferences, which allows developers to store keypair values either in a data-set shared by all components of the app or within a specific separate file. Developers can also manipulate files directly. These files may be stored privately by the app, and therefore inaccessible to other apps, or made readable and/or writeable by other apps. App developers can also use the SQLite functionality included in Android to manage their own private database. Such a database can then be made available to other apps by hosting it within a content provider component.

*安全和权限* *Security and Permissions*

安卓的安全是在进程级别的。换言之，安卓依赖Linux现有的进程隔离机制来实现自己的策略。为此，每个应用安装后都会获得自己的UID个GID。本质上就像每个应用都是系统中单独的用户一样。在多任务的Unix系统中，如果没有明确的权限授权，“用户”们不能够访问彼此的资源。实际上，每个应用都运行在独立的沙盒中。
Security in Android is enforced at the process level. In other words, Android relies on Linux's existing process isolation mechanisms to implement its own policies. To that end, every app installed gets its own UID and GID. Essentially, it's as if every app is a separate "user" in the system. And as in any multi-user Unix system, these "users" cannot access each others' resources unless permissions are explicitely granted to do so. In effect, each app lives in its own separate sandbox.

要离开沙盒并访问关键的系统功能或资源，应用必须要使用安卓权限机制。这就需要开发人员在manifest文件中静态地声明应用所需的权限。如访问互联网（即使用套接字）、拨打电话或使用摄像头都需要由安卓去预先定义。其他权限可由开发者声明，为其他应用与指定应用之间的组件交互。当应用安装之后，提示用户批准应用程序所需的权限。
To exit the sandbox and access key system functionality or resources, apps must use Android's permission mechanisms, which require developers to statically declare the permissions needed by an app in its manifest file. Some permissions, such as the right to access the Internet (i.e. use sockets), dial the phone, or use the camera, are predefined by Android. Other permissions can be declared by app developers and then be required for other apps to interact with a given app's components. When an app is installed, the user is prompted to approve the permissions required to run an app.

Access enforcement is based on per-process operations and requests to access a specific URI, and the decision to grant access to a specific functionality or resource is based on certificates and user prompts. The certificates are the ones used by app developers to sign the apps they make available on the Android Market. Hence, developers can restrict access to their apps' functionality to other apps they themselves created in the past.

安卓开发框架提供了更多功能，当然不是这小节内容涵盖得了的。这部分内容可以阅读其他安卓应用开发的相关信息，或者访问[developer.android.com](developer.android.com)获取更多关于2D、3D图像，多媒体，定位及地图，蓝牙，NFC等内容。
The Android development framework provides a lot more functionality, of course, than can be covered here. I invite you to read up on Android app development elsewhere or visit [developer.android.com](developer.android.com) for more information on 2D and 3D graphics, multi-media, location and maps, Bluetooth, NFC, etc.

#### 应用开发工具 App Development Tools

这段我不想翻了
The typical way to develop Android applications is to use the freely available Android Software Development Kit (SDK). This SDK, along with Eclipse, its corresponding Android Development Tools (ADT) pulgin, and the QEMU-based emulator in the SDK, allow developers to do the vast majority of development work straight from their workstation. Developers will also usually want to test their app on real devices prior to making it availbale on the Android Market, as there usually are runtime behavior differences between the emulator and actual devices. Some software publishers take this to the extreme and test their apps on several dozens of devices before shipping a new release.

就算你不想为你的嵌入式系统开发任何应用，还是强烈建议你在工作站上搭建应用开发环境。这也方便编写测试应用验证AOSP修改的影响。如果计划扩展AOSP的API并且创建和发布自己的SDK，这也是必不可少的。
Even if you aren't going to plan to develop any apps for your embedded system, I highly suggest you set up the development environmnet used by app developers on you  workstation. If nothing else, this will allow you to validate the effects of modifications you make to the AOSP using basic test applications. It will also be essential if you plan on extending the AOSP's API and therefore create and distribute your own cusion SDK.

搭建应用开发环境可以参考之前网站中谷歌给出的手册，顺便推销一下自己的另一本书。
To set up an app development environment, follow the instructions provided by Google at the developer kit site just mentioned, or have a look at the book *Learning Andriod* (O'Reilly).

#### 原生开发 Native Development

事实上，绝大部分的应用都是在我们刚才讨论的环境中使用java所开发的，有的开发者也需要运行原生的C代码。为此，谷歌提供了原生开发工具包（NDK）。正如宣传的那样，这主要是为了让游戏开发者们挤出设备最后一丝性能。因此，NDK中的API主要提供图形渲染以及传感器方面的功能。例如极为出名的游戏——愤怒的小鸟，就非常依赖原生代码。
While the majority of apps are developed exclusively in Java using the development environment we just discussed, certain developers need to run some C code natively. To this end, Google has made the Native Development Kit (NDK) available to developers. As advertized, this is mostly aimed at game developers needing to squeeze every last possible bit of performance out of the device their game is running on. And as such, the APIs made available within the context of the NDK are mostly geared towards graphics rendering and sensor input retrieval. The infamous Angry Birds game, for example, relies heavily on code running natively.

另一个可能用到NDK的情景是将现有的代码库移植到安卓上。如果你手上是开发了多年的古老C代码（对于开发了其他移动应用的工作室来说十分常见），并不希望把这些东西再用java重写一遍。相反，你可以使用NDK编译之后打包到java代码中，并通过SDK的方式提供给安卓使用。例如安卓上的火狐浏览器，就非常依赖NDK去运行那部分历史代码。
Another possible use of the NDK is obviously to port over an existing codebase to Android. If you've developed a lot of legacy C code over several years (a common situation for development houses that have created applications for other mobile devices), you won't necessarily want to rewrite it in Java. Instead, you can use the NDK to compile it for Android and package it with some Java code to use some of the more Android-specific functionality made available by the SDK. The Firefox browser, for instance, relies heavily on the NDK to run some of its legacy code on Android.

使用NDK的好处是可以通过SDK的形式将java代码与C代码结合使用。也正如刚才暗示的，NDK能够提供的仅仅是安卓API中有限的子集。比如你无法在NDK编译的C代码中发出一个意图，必须由java编写的SDK来做这件事情。需要提醒的是，NDK提供的API主要面向游戏开发人员。
As I just hinted, the nice thing about the NDK is that you can combine it with the SDK and therefore have part of your app in Java and parts of your app in C. That said, it's crucial to understand that the NDK gives you access only to a very limited subset of the Android API. There is, for instance, no way to presently send an intent from within C code compiled with the NDK; the SDK must be used to do it in Java instead. Again, the APIs made available through the NDK are mostly geared towards game development.

有时嵌入式及系统开发人员希望通过NDK在安卓上完成一些平台级的工作。NDK中的“原生”一词或许在这里有一些误导，因为使用NDK时还需涉及到刚才所说对开发人员的限制和要求。所以对嵌入式开发者而言，请记住，NDK为应用开发人员在java中调用C代码提供了极大的帮助。除此之外，NDK对你所承担的工作类型几乎毫无用处。
Sometimes embedded and system developers coming to Android expect to be able to use the NDK to do platform-level work. The word "native" in the NDK can be misleading in that regard, because the use of the NDK still involves all of the limitations and requirements that I've said to apply to Java app developers. So, as an embedded developer, remember that the NDK is useful for app developers to run C code that they can call from their Java code. Apart from that, the NDK will be of little to no use for the type of work you are likely to undertake.

### 总体结构 Overall Architecture
![图 2-1. 安卓系统架构](https://i.bmp.ovh/imgs/2019/07/33c5a511dac04dd2.png)

图2-1可能会是本书中最重要的一张图表，建议你想办法在这里放个书签，因为我们会经常应用这张图片。尽管这是个简化的视图，当然我们有机会去丰富它，但它为安卓各个部分如何搭配在一起，以及总体结构提供了一个很好的概念。
Figure 2-1 is probably one of the most important diagrams presented in this book, and I suggest you find a way to bookmark its location as we will often refer back to it, if not explicitly then implicitely. Although it's a simplified view—and we will get the chance to enrich it as we go—it gives a pretty good idea of Android's architecture and how the various bits and pieces fit together.

如果你熟悉Linux开发，第一件让你震惊的事情应该是，除了Linux内核本身，这一部分几乎没有什么Linux/Unix世界中常见的东西。没有glibc，没有X Window系统，没有GTK也没有BusyBox。很多Linux和嵌入式Linux老鸟确实感到安卓比较陌生。尽管安卓还是从完全干净的状态开始跑用户空间，我们会讨论如何将“遗留”或“经典”Linux应用和效应与安卓堆栈共存。
If you are familiar with some form of Linux development, the first thing that should strike you is that beyond the Linux kernel itself, there is little in that stack that resembles anything typically seen in the Linux or Unix world. There is no glibc, no X Window System, no GTK, no BusyBox, and so on. Many veteran Linux and embedded Linux practitioners have indeed noted that Android feels very alien. Though the Android stack starts from a clean slate with regards to user-space, we will discuss how to get "legacy" or "classic" Linux applications and utilities to coexist side-by-side with the Android stack.

让我们深入到安卓架构的每个部分，从图2-1底部开始往上研究，一旦完成了对各个组件的处理，我们将通过总览系统启动过程来结束这一章节。
Let's take a deeper look into each part of Android's architecture, starting from the bottom of Figure 2-1 and going up. Once we are done covering the various components, we'll end this chapter by going over the system's startup process.

### Linux内核 Linux Kernel

Linux内核是所有传统上标记为Linux发行版的中心部分，包含了如Ubuntu、Fedora、Debian这些主流发行版。虽然linux内核归档在香草版本中提供，但大多数发行版在发布给用户之前都打上自己的补丁去修复漏洞，或是提高性能。安卓也一样，开发人员们也修复了内核来满足他们的需求。
The Linux kernel is the center-piece of all distributions traditionally labeled as "Linux," including mainstream distributions such as Ubuntu, Fedora, and Debian. And while it's available in "vanilla" form from the Linux Kernel Archives, most distributions apply their own patches to it to fix bugs and enhance the performance or customize the behavior of certain aspects of it before distributing it to their users. Android, as such, is no different in that the Android developers patch the "vanilla" kernel to meed their needs.

然而，安卓并不按常规操作，能够在内核中找到一些与“香草”明显不同的自定义功能。事实上，linux发行版中的内核可以很容易地被kernel.org中的内核替换，并对发行版中其他的组件几乎没有影响，而安卓的用户空间组件只有在“安卓化”的内核中才能够运行。在之前的章节中也有提及，总的来说，安卓的内核是主线上的分支。
Android differs from standard practice, however, in relying on several custom functionalities that are significantly different from what is found in the "vanilla" kernel. In fact, whereas the kernel shipped by a Linux distribution can easily be replaced by a kernel from kernel.org with little to no impact to the rest of the distribution's components, Android's user-space components will simply not work unless they're running on an "Androidized" kernel. As I had mentioned in the previous chapter, Android kenels are, in sum, forks from the mainline kernel.

讨论Linux内核已经超出了本书的范围，但是我们还是看一看这些“安卓主义者”向内核中添加了哪些东西。你可以通过Robert Love的linux内核开发（第三版），以及订阅linux周报（LWN）来了解内核的内部信息。LWN包含了几篇关于内核结构的重要文章，并且提供了linux内核开发的最新信息。
Although it's beyond the scope of this book to discuss the Linux kernel's internals, let's go over the main "Androidisms" added to the kernel. You can get information about the kernel's internals by having a look at Robert Love's Linux Kernel Development, 3rd ed. and starting to follow the Linux Weekly News (LWN) site. LWN contains several seminal articles on the kernel's internals and provides the latest news regarding the Linux kernel's development.

请注意下面的小节只包含了最重要的部分，安卓化的内核通常在标准内核基础上打了上百个补丁，用于提供特定设备功能，修复以及增强。可以使用git在http://android.git.kernel.org中对安卓内核与主线内核进行详尽分析。另外，如PEME驱动程序是某些安卓内核的特定功能，不一定会在所有的安卓设备中使用。
Note that the following subsections cover only the most important Androidisms. Android-ified kernels typical contain several hundred patches over the standard kernel, often to provide device-specific functionality, fixes and enhancements. You can use git to do an exhaustive analysis on the commit deltas between one of the kernels at http://android.git.kernel.org and the mainline kernel they were forked from. Also, note that some of the functionality that appears in some Android-ified kernels, such as the PMEM driver for instance, is device-specific and isn't necessarily used in all Android devices.

#### 唤醒锁 Wakelocks

在所有安卓主义中，这或许是最具争议的。将这个功能包含进主线内核讨论了将近2000封邮件，但仍然没有明确的路径去合并唤醒锁功能。
Of all the Androidisms, this is likely the most contentious. The discussion threads covering its inclusion into the mainline kernel generated close to 2,000 emails and yet still, there's no clear path for merging the wakelock functionality.

理解唤醒锁是什么、能做什么，我们必须先讨论linux中的电源管理。最常见的就是笔记本电脑，当运行linux的笔记本电脑盖子合上时，操作系统通常会进入“暂停”或“休眠”模式。在这种模式下，操作系统的运行状态被保存在RAM中，其他的硬件功能都被关闭。因此，电脑可以尽可能少地使用电池。当盖子打开时，笔记本电脑就会“醒来”，用户几乎可以立即开始使用。
To understand what wakelocks are and do, we must first discuss how power management is typically used in Linux. The most common use case of Linux's power management is a laptop computer. When the lid is closed on a laptop running Linux, it will usually go into "suspend" or "sleep" mode. In that mode, the system's state is preserved in RAM but all other parts of the hardware are shut down. Hence, the computer uses as little battery power as possible. When the lid is raised, the laptop "wakes up" and the user can resume using it almost instantaneously.

这种操作方式在笔记本电脑和桌面设备上应用相当出色，但是并不适合如手机这样的移动设备。因此，安卓团队设计了一个机制，稍稍改变了一下规则使得其更加适合移动设备场景。安卓化的内核并不是让系统按照用户命令去休眠，而是尽可能快地休眠。只是在重要进程执行或应用程序等待用户输入时防止系统进入休眠。唤醒锁就是用来保持系统的唤醒状态。
That modus operandi works fine for a laptop and desktop-like devices, but it doesn't fit mobile devices such as handsets as well. Hence, Android's development team devised a mechanism that changes the rules slightly to make them more palatable to such use cases. Instead of letting the system be put to sleep at the user's behest, an Androidized kernel is made to go to sleep as soon and as often as possible. And to keep the system from going to sleep while important processing is being done or while an app is waiting for the user's input, wakelocks are provided to... keep the system awake.

唤醒锁和提前休眠功能构建在linux现有的电源管理机制之上。然而，他们引入了完全不同的开发模式。因为应用程序和驱动开发人员必须在用户进行关进操作或输入时抓住唤醒锁。通常，应用程序开发者不需要直接处理唤醒锁，因为他们抽象出的相关操作会自动处理锁定事件。当然，他们也可以直接向电源管理服务申请唤醒锁。另一方面，驱动开发者可以调添加到内核中的唤醒锁原语去申请和释放唤醒锁。而在驱动中使用唤醒锁的缺点就是不能够将这个驱动推到主线内核中，因为主线内核不支持唤醒锁。
The wakelocks and early suspend functionality are actually built on top of Linux's existing power management functionality. However, they introduce a different development model, since application and driver developers must explicitely grab wakelocks whenever they conduct critical operations or must wait for user input. Usually, app developers don't need to deal with wakelocks directly, because the abstractions they use automatically take care of the required locking. They can, nonetheless, communicate with the Power Manager Service if they require explicit wakelocks. Driver developers, on the other hand, can call on the added in-kernel wakelock primitives to grab and release wakelocks. The downside of using wakelocks in a driver,  however, is that it becomes impossible to push that driver into the mainline kernel, because the mainline doesn't include wakelock support.


#### 低内存杀手 Low Memeory Killer

如之前所提及的，安卓行为很大程度上取决于低内存条件。因此，OOM的行为是至关重要的。所以安卓研发团队在内核OOM杀进程之前添加了一个额外的低内存杀手。安卓的低内存杀手按照应用程序开发文档中描述的策略，淘汰了那些长时间未使用且优先级不高的组件进程。
As I mentioned earlier, Android's behavior is very much predicated on low-memory conditions. Hence, out-of-memory behavior is crucial. For this reason, the Android development team has added an additional low memory killer to the kernel that kicks in before the default kernel OOM killer. Android's low-memory killer applies the policies described in the app development documentation, weeding out processes hosting components that haven't been used in a long time and that are not high-priority.

LMK基于linux中OOM调节机制，可以支持对不同进程采用不同OOM杀手优先级。基本上，OOM调度允许用户空间控制一部分内核OOM杀除策略。OOM调整范围在-17到15之间，数字越高意味着在系统内存不足时，关联进程更容易被杀掉。
This low memory killer is based on the OOM adjustments mechanism availalble in Linux that enables the enforcement of different OOM kill priorities for different processes. Basically, the OOM adjustments allow user space to control part of the kernel's OOM killing policies. The OOM adjustments range from -17 to 15, with a higher number meaning the associated process is a better candidate for being killed if the system is out of memory.

因此安卓将不同的OOM等级做了调整，根据运行的组件归纳到不同类型的进程上，并按照进程类型给自己的LMK配置不同的阈值。这让安卓的策略有效取代了内核的OOM killer，使用达到阈值的方式踢掉进程，而不是等到系统内存用尽时再开始处理。
Android therefore attributes different OOM adjustment levels to different types of processes according to the components they are running, and configures its own low memory killer to apply different thresholds for each category of process. This effectively allows it to preempt the activation of the kernel's own OOM killer—which only kicks in when the system has no memory left—by kicking in when the given thresholds are reached, not when the system runs out of memory.

用户空间是初始化阶段被init进程启动（见47页init章节），并在运行时由活动管理服务重新适配及局部加强，这也是系统服务的重要部分之一，负责执行先前介绍的组件生命周期以及其他许多工作。
The user-space policies are themselves applied by the init process at startup (see “Init” on page 47), and readjusted and partly enforced at runtime by the Activity Manager Service, which is part of the System Server. The Activity Manager is one of the most important services in the System Server and is responsible, amongst many other things, for carrying out the component lifecycle presented earlier.

#### Binder Binder
Binder是一种类似Windows下RPC/IPC的机制。它的起源最早可以追溯到Be被Palm收购之前的BeOS中。它在Palm继续开发，并作为Open-Biner项目被发布。尽管Open-Biner并没有作为一个独立的项目存活下来，但是像Dianne Hackborn和Arve Hjonnevag这些核心开发者，最终加入了安卓开发团队。
Binder is an RPC/IPC mechanism akin to COM under Windows. Its roots actually date back to work done within BeOS prior to Be's assets being bought by Palm. It continued life within Plam and the fruits of that work were eventually released as the Open-Binder project. Though OpenBinder never survived as a stand-alone project, a few key developers who had worked on it, such as Dianne Hackborn and Arve Hjonnevag, eventually ended up working within the Android development team.

安卓的Binder机制受到了先前工作的启发，但是安卓的实现并不是直接从OpenBinder代码中直接提取，相反，它重写了OpenBinder的功能子集。如果想了解这个机制的基础及其设计理念，那么OpenBiner的文档还是必读的。
Android's Binder mechanism is therefore inspired by that previous work, but Android's implementation does not derive from the OpenBinder code. Instead, it's a clean room rewrite of a subset of the OpenBinder functionality. The OpenBinder Documentation remains a must-read if you want to understand the mechanism's underpinings and its design philosophy.

本质上，Binder试图在经典操作系统之上提供远程对象调用功能。换言之，Binder是去“尝试接受并超越经典操作系统”，而不是重新设计它们。因此，开发者们只需要面对远程服务这个对象，而不是去面对一个全新的操作系统。那么通过添加远程可调用对象扩展系统功能就比实现新的守护进程更加容易，非常符合UNIX哲学。远程对象可以用任何所需的语言实现，并可以与其他远程服务共享相同的进程空间，或是拥有自己的独立进程。使用时只需要定义接口及引用方法即可。
In essence, Binder attempts to provide remote object invocation capabilities on top of a classic OS. In other words, instead of re-engineering traditional OS concepts, Binder "attempts to embrace and transcend them." Hence, developers get the benefits of dealing with remote services as objects without having to deal with a new OS. It therefore becomes very easy to extend a system's functionality by adding remotely-invocable objects instead of implementing new daemons for providing new services, as would usually be the case in the Unix philosophy. The remote object can therefore be implemented in any desired language and may share the same process space as other remote services or have its own separate process. All that is needed to invoke its methods is its interface definition and a reference to it.

正如在图2-1所示，Binder是安卓架构的一个组成部分。它允许应用程序与系统服务及其他服务组件对话。不过，如之前提到的，应用开发人员并不会直接与Binder对话，相反，他们使用aidl工具生成的接口与stubs。就算程序与系统服务对接，安卓API也会将这些服务抽象，开发人员不会直接看到Binder的实际调用。
And as you can see in Figure 2-1, Binder is a conerstone of Android's architecture. It's what allows apps to talk the System Server and it's what apps use to talk to each others' service components, although, as I mentioned earlier, app developers don't actually talk to the Binder directly. Instead, they use the interfaces and stubs generated by the aidl tool. Even when apps interface with the System Server, the android.* APIs abstract its services and the developer never actually sees that Binder is being used.

Binder机制的内核驱动是一个可通过/dev/binder访问的字符驱动程序。它使用ioctl()方法在通讯双方间传递数据包。它也允许一个进程将自己定义为“上下文管理器”。上下文管理器的重要性以及Binder驱动的实际用户空间使用将会在本章稍后部分详细讨论。
The in-kernel driver part of the Binder mechanism is a character driver accessible through /dev/binder. It's used to transmit parcels of data in between the communicating parties using calls to ioctl() . It also allows one process to designate itself as the "Context Manager." The importance of the Context Manager along with the actual userspace use of the Binder driver will be discussed in more detail later in this chapter.

#### 匿名共享内存 Anonymous Share Memory (ashmem)

另一个在大多数操作系统中使用到的IPC机制就是共享内存。在Linux中，这一般由System V IPC机制中的POSIX SHM功能提供。如果你看过AOSP中的*ndk/docs/system/libc/SYSV-IPC.html*文件，你会发现安卓开发团队似乎并不喜欢SysV IPC。实际上，文件中的一个论点是，在Linux中使用SysV IPC机制可以导致内核资源泄漏，继而让恶意软件或行为不当的软件跑挂系统。
Another IPC mechanism available in most OSes is shared memory. In Linux, this is usually provided by the POSIX SHM functionality, part the System V IPC mechanisms. If you look at the *ndk/docs/system/libc/SYSV-IPC.html* file included in the AOSP, however, you'll discover that the Android development team seems to have a dislike for SysV IPC. Indeed, the argument is made in that file that the use of SysV IPC mechanisms in Linux can lead to resource leakage within the kernel. opening the door in turn for malicious or misbehaving software to cripple the system.

虽然并没有任何安卓开发人员或ashmem代码或者文档中这样声明，ashmem很可能将其存在的愿意归功于安卓团队看到了SysV IPC的缺点。所以ashmem被描述为与POSIX SHM相似“但有着不同的行为。”例如当所有进程退出时，它会引入内存销毁区域计数，并在系统需要内存时收缩映射区域。它还启用了内存压力，“去掉”一个区域允许它被收缩，而“固定”则不允许被收缩。
Though it isn't stated as such by Android developers or any of the documentation within the ashmem code or surrounding its use, ashmem very likely owes part of its existence to SysV IPC's shortcomings as seen by the Android development team. Ashmem is therefore described as being similar to POSIX SHM "but with different behavior." For instance, it does reference counting to destory memory regions when all processes referring to them have exited and will shrink mapped regions if the system is in need of memory. It will also enable memory pressure. "Unpinning" a region allows it to be shrunk, wheras "pinning" a region disallows the shrinking.

通常，第一个进程通过ashmem创建一个共享内存区域，并使用Binder与其他愿意共享区域的进程分享相应的文件描述符。例如Dalvik的JIT代码缓存通过ashmem为Dalvik提供实例。有许多系统服务组件，如Surface Flinger和Audio Finger也都依赖ashmem，尽管通过IMemory接口而不直接使用ashmem。
Typically, a first process creates a shared memory region using ashmem and uses Binder to share the corresponding file descriptor with other processes with which it wishes to share the region. Dalvik's JIT code cache, for instance, is provided to Dalvik instances through ashmem. A lot of System Server components, such as the Surface Flinger and the Audio Flinger, rely on ashmem, though not directly but through the IMemory§ interface.


~§IMemory是一个仅在AOSP中可用的内部接口，不提供给应用发开人员。给app使用的最接近的类是MemoryFile。~

#### 闹钟 Alarm

将闹钟驱动加入内核则是另一种情况，默认的内核功能并不完全满足安卓的需求。安卓的闹钟驱动实际上在内核已有的实时时钟（RTC）与高精度计时器（HRT）的上层。内核的RTC功能为驱动开发人员提供了一个框架，用于实现定制的板级RTC功能，而内核通过主RTC驱动提供了一个独立与硬件的接口。另一方面，也允许调用者在特定的时间点被唤醒。
The alarm driver added to the kernel is another case where the default kernel functionality wasn't sufficient for Android's requirements. Android's alarm driver is actually layered on top of the kernel's existing Real-Time Clock (RTC) and High-Resolution Timers (HRT) functionalities. The kernel's RTC functionality provides a framework for driver developers to create board-specific RTC functions, while the kernel exposes a single hardware-independent interface through the main RTC driver. The kenel HRT functionality, on the other hand, allows callers to get woken up at very specific points in time.

在香草内核中，应用开发人员通常会调用**setitimer()**系统方法来获取一个超时信号。系统调用允许几类计时器，其中一种是**ITIMER_REAL**，使用了 内核高精度计时器（HRT），但是不能在系统被挂起时使用。换言之，如果应用使用**setitimer()**在给定的时间后请求被唤醒，在此期间设备被挂起，应用只会在设备被唤醒时才能收到信号。
In "vanilla" Linux, application developers typically call the **setitimer()** system call to get a signal when a given time value expires. The system call allows for a handful of types of timers, one of which, **ITIMER_REAL**, uses the kernel's High-Resolution Timer (HRT). This functionality, however, doesn't work the system is suspended. In other words, if an application uses **setitimer()** to request being woken up at a given time and then, in the interim, the device is suspended, that application will get its signal only when the device is woken up again.

除了**setitimer()**系统调用，内核RTC驱动可以通过*/dev/rtc*访问，并允许用户使用**ioctl()**方法，设置一个闹钟将被系统的RTC硬件设备激活。无论系统是否被挂起，这个闹钟都会触发。因为它基于行为或RTC设备，即使系统其他部分被挂起，它依旧可以保持活动状态。
Separately from the **setitimer()** system call, the kernel's RTC driver is accessiable through */dev/rtc* and enables its users to use an **ioctl()**, among other things, to set an alarm that will be activated by the RTC hardware device in the system. That alarm will fire off whether the system is suspended or not, since it's predicated on the behavior or the RTC device, which remains active even the rest of the system is suspended.

安卓的闹钟驱动聪明地结合了最好的部分。驱动默认使用内核的高精度计时器为用户提供闹钟，就像构建在内核的计时器功能。如果系统被挂起，就会调用RTC让系统在指定时间被唤醒。因此，用户空间的应用无论在何时需要一个特定的闹钟，只需要让安卓的闹钟驱动在指定时间唤醒，无需关注系统是否是挂起的。
Android's alarm driver cleverly combines the best of both worlds. By default, the driver uses the kernel's High-Reslution Timer (HRT) functionality to provide alarms to its users, much like the kernel's own built-in timer functionality. However, if the system is about to suspend itself, it programs the RTC so that the system gets woken up at the appropriate time, Hence, whenever an application from user space needs a specific alarm, it just needs to use Android's alarm driver to be woken up at the appropriate time, regardless of whether the system is suspended in the interim.

在用户空间，闹钟驱动以*/dev/alarm*的字符设备体现，允许使用者通过**ioctl()**设置闹钟并调整系统时间。有几个关键的AOSP组件依赖*/dev/alarm*。如Toolbox和**SystemClock**类，能够通过应用接口获取或设置系统时间。更为重要的是，作为系统服务的一部分，应用开发人员调用的**AlarmManager**类，就是闹钟管理服务通过这种方法为应用提供了闹钟服务。
From user-space, the alarm driver appears as the */dev/alarm* character device and allows its users to set up alarms and adjust the system's time (wall time) through **ioctl()** calls. There are a few key AOSP components that rely on */dev/alarm*. For instance, Toolbox and the **SystemClock** class, available through the app development API, rely on it to set/get the system's time. Most importantly, though, the Alarm Manager service part of the System Server uses it to provide alarm services to apps that are exposed to app developers through the **AlarmManager** class.

无论驱动还是闹钟管理都使用唤醒锁机制来保持闹钟和其他安卓唤醒锁相关行为之间的一致性。因此，当闹钟触发时，这个消费者应用程序有机会在系统下一次挂起前做任何操作，如果有必要的话。
Both the driver and Alarm Manager use the wakelock mechanism wherever appropriate to maintain consistency between alarms and the rest of Android's wakelock-related behavior. Hence, when an alarm is fired, its consuming app gets the chance to do whatever operation is required before the system is allowed to suspend itself again, if nedd be.

#### 日志系统 Logger

日志是目前包括嵌入式系统中，任何Linux系统上的必备组件。通过实时或出现问题后分析系统日志，可以在警告与错误中定位到致命问题所在，尤其是瞬间发生的错误。大部分Linux发行版都自带两套日志系统：通过*dmesg*命令访问的内核日志与以文件形式存储在*/var/log*路径下的系统日志。内核日志通常由各处**printk()**、核心内核代码或设备驱动调用输出。相对而言，系统日志包含的信息来自于运行在系统中各种守护进程与应用实时消息。实际上，你可以使用*logger*命令将自己的消息发送至系统日志中。
Logging is yet another essential component of any Linux system, embedded ones included. Being able to analyze a system's logs for errors or warnings either in post-mortem or in real-time can be vital to isolate fatal errors, especially transient ones. By default, most Linux distributions include two logging systems: the kernel's own log, typically accessed through the *dmesg* command, and the system logs, typically stored in files in the */var/log* directory. The kernel's log usually contains the messages printed out by the various **printk()** calls made within the kernel, either by core kernel code or by device drivers. For their part, the system logs contain messges coming from various daemons and utilities running in the system. In fact, you can use the *logger* command to send your own messages to the system log.

就安卓而言，日志的功能就是原样使用。然而，安卓的日志系统软件包无法在大多数Linux发行版中找到。相反，安卓定义了自己的日志机制，在内核中添加了安卓日志驱动。syslog依赖于通过套接字发送消息，并以此引出了任务开关。它还使用文件存储信息，这就涉及到存储设备的写操作。反之，安卓日志功能管理了一系列独立的内核缓冲区，用于记录来自用户空间的数据。因此，事件的记录无需任务切换或文件写入。相反，驱动维护的循环缓冲区能够记录记录每个事件并立即返回给调用方。
With regard to Android, the kernel's logging functionality is used as-is. However, none of the usual system logging software packages typeically found in most Linux distributions is found in Android. Instead, Android defines its own logging mechanisms based on the Android logger driver added to the kernel. syslog relies on sending messages through sockets, and therefore generates a  task switch. It also uses files to store its information, therefore generating writes to a storage device. In contrast, Android's logging functionality manages a handful of separate kernel-hosted buffers for logging data coming from user-space. Hence, no task-switches or file writes are required for each event being logged. Instead, the driver maintains circular buffers where it logs every incoming event and returns immediately back to the caller.

正因为其轻量而高效的设计，安卓日志才能真正在运行时以用户空间组件规律地记录事件。实际上，对应用开发者开放**Log**类，或多或少直接授权了日志驱动写入主事件缓存。显然，所有的好东西都会被滥用，最好能够保持日志框架轻量。但是综合考虑应用程序API公开**Log**，以及AOSP本身对日志的使用，如果基于syslog则很难维持这样的使用级别。
Because of its light-weight and efficient design, Android's logger can actually be used by user-space components at run-time to regularly log events. In fact, the **Log** class available to app developers more or less directly invokes the logger driver to write to the main event buffer. Obviously, all good things can be abused and it's preferable to keep the logging light, but still the level of use made possible by exposing **Log** through the app API along with the level of use of logging within the AOSP itself would have likely been very difficult to sustain had Android's logging been based on syslog.

图2-2描述了安卓日志框架的细节。如图所示，日志驱动是所有日志相关功能得以运行的基础核心模块。在*/dev/log/*目录下各个缓冲区作为单独的条目。然而，没有用户空间组件直接与驱动交互。相反，他们都依赖liblog提供一系列不同的日志功能。根据功能使用与传递的参数，事件会记录到不同的缓存中。**Log**和**Slog**类使用liblog库，例如首先检测事件是否来自音频相关模块，如果是，则事件被分发至音频缓存区。否则，**Log**类会将事件传给“主”缓冲区，此时**Slog**类再将事件给“系统”缓冲区。主缓冲区事件在没有任何参数的情况下会在*logcat*命令中显示。
Figure 2-2 describes Android's logging framework in more detail. As you can see, the logger driver is the core building block on which all other logging-related functionality relies. Each buffer it manages is exposed as a separate entry within */dev/log/*. However, no user-space component directly interacts with that driver. Instead, they all rely on liblog which provides a number of different logging functions. Depending on the functions being used and the parameters being passed, events will get logged to different buffers. The liblog functions used by the **Log** and **Slog** classes, for instance, will test whether the event being dispatched comes from a radio-related module. If so, the event is sent to the "radio" buffer. If not, the **Log** class will send the event to the "main" buffer where the **Slog** class will send it to the "system" buffer. The "main" buffer is one whose events are shown by the *logcat* command when it's issued without any parameters.

![Figure 2-2. 安卓的日志系统框架](https://i.bmp.ovh/imgs/2019/07/9ce02cb7562a574c.png)

**Log**和**EventLog**类都可以通过应用的API调用，但**Slog**只能由AOSP内部使用。尽管**EventLog**开放给了应用开发人员，但文档中明确说明了其主要开放给系统集成商而非应用开发者。事实上，绝大部分开发代码样例中都使用了**Log**类。通常，**EventLog**是系统组件使用，尤其是系统服务器主机？类服务将结合**Log**，**Slog**和**EventLog**来记录各类事件。与应用开发人员相关的日志可能使用**Log**，而与平台集成相关的日志就可能使用**Slog**或**EventLog**。
Both the **Log** and **EventLog** classes are exposed through the app development API, while **Slog** is for internal AOSP use only. Despite being available to app developers, though, **EventLog** is clearly identified in the documentation as mainly or system integrators, not app developers. In fact, the vast majority of code samples and examples provided as part of the developer documentation use the **Log** class. Typically, **EventLog** is used by system components, especially System Server-hosted services, will use a combination of **Log**, **Slog**, and **EventLog** to log different events. An event that might be relevant to  app developers, for instance, might be logged using **Log**, while an event relevant to platform developers or system integrators might be logged using either **Slog** or **EventLog**.

要注意的是，**logcat**这样的实用工具也依赖于liblog。除了添加访问日志驱动的接口，liblog还提供了美化打印与过滤器格式化事件功能。另一个特性是，liblog要求每一个事件需要标明优先级、标签与数据。优先级可以选择**verbose**，**debug**，**info**，**warn**或**error**。标签是一个唯一的字符串用于区分日志属于不同的模块或组件。数据就是需要被记录的具体信息。
Note that the **logcat** utility, which is commonly used by app developers to dump the Android logs, also relies on liblog. In addition to providing access functions to the logger driver, liblog also provides functionality for formatting events for pretty printing and filtering. Another feature of liblog is that it requires every envent being logged to have a priority, a tag, and data. The priority is one of **verbose**, **debug**, **info**, **warn**, or **error**. The tag is a unique string that identifies the component or module writing to the log, and the data is the actual information that needs to be logged. This description should in fact sound fairly familiar to anyone exposed to the app development API, as this is exactly what's spelled out by the developer documentation for the **Log** class.

现在最后一块拼图就是*adb*命令了，在后文中会讨论到，AOSP包含了安卓调试通道(ADB)的守护进程，可以通过*adb*命令行工具在主机端访问安卓设备。当输入*adb logcat*时，守护进程实际上在本地发送*logcat*命令，获取主缓冲区输出并推到主机的终端中显示。
The final piece of the puzzle here is the *adb* command. As we'll discuss later, the AOSP includes an Android Debug Bridge (ADB) daemon that runs on the Android device and that is accessed from the host using the *adb* command-line tool. When you type *adb logcat* on the host, the daemon actually launchers the *logcat* command locally on the target to dump its "main" buffer and then transfers that back to the host to be shown on the terminal.

#### 其他值得一提的安卓主义 Other Notable Androidisms

除了上文所描述的部分，还有一些其他“安卓派”的设计值得一提。
A few other Androidisms, in addition to those already covered, are worth mentioning, even if we don't cover them in as much detail.

*Paranoid网络*
*Paranoid Networking*

在Linux中，所有进程都被允许创建进程并连接网络。在安卓的安全模式下，网络访问是受到控制的。因此，内核添加了一个选项，通过判断当前进程属组或能力来筛选套接字创建及网络接口管理的访问。这套机制在IPv4，IPv6和蓝牙上都适用。
Usually in Linux, all processes are allowed to create sockets and interact with the network. Per Android's security model, however, access to network capabilities has to be controlled. Hence, an option is added to the kernel to gate access to socket creation and network interface administration based on whether the current process belongs to a certain group of processes or possesses certain capabilities. This applies to IPv4, IPv6, and Bluetooth.

*RAM控制台*
*RAM Console*

正如之前提到的，内核管理自己的日志，可以使用*dmesg*命令访问这些日志。日志内容非常有价值，其中常常包含驱动或内核子系统的关键信息。在崩溃或内核挂逼时，日志可以帮助问题的事后分析。由于这些信息常常在重启后丢失，安卓添加了一个驱动注册基于RAM的控制台，控制台在重启后仍然存在，并可以通过*/proc/last_kmsg*访问日志内容。
As I mentioned earlier, the kernel manages its own log, which you can access using the *dmesg* command. The content of this log is very useful, as it often contains critical messages from drviers and kernel subsystems. On a crash or a kernel panic, its content can be instrumental for post-mortem analysis. Since this information is typically lost on reboot, Android adds a driver that registers a RAM-based console that survives reboots and makes its content accessible through */proc/last_kmsg*.

*物理内存（pmem）*
*Physical Memory (pmem)*

正如匿名共享内存，物理内存也驱动允许进程间共享内存。但不同的地方在于，pmem允许共享物理上相连的大片内存，而非虚拟内存。此外，这些内存区域也可能在进程与驱动间共享。例如HTC G1手机上就使用pmem进行2D硬件加速。但需要注意，pmem并不适用于所有设备，实际上，来自安卓内核团队的Brain Swetland说，它是专门针对MSM7201A（G1的处理器）编写。
Like ashmem, the pmem driver allows for sharing memory between processes. However, unlike ashmem, it allows the sharing of large chunks of physically-contiguous memory regions, not virtual memory. In addition, these memory regions may be shared between processes and drivers. For the G1 handset, for instance, pmem heaps are used for 2D hardware acceleration. Note, though, that pmem isn't used in all devices. In fact, according to Brian Swetland, one of the Android kernel development team members, it was written to specifically target the MSM7201A's# limitations.

--------------------------------------------------

### 硬件支持 Hardware Support

安卓的硬件支持方式与典型的Linux内核和基于Linux的发行版大不相同。具体来看，硬件支持的实现方式、基于该硬件支持构建的抽象以及围绕最终代码许可和分发理念都是不同的。
Android's hardware support approach is significantly different from the classic approach typically found in the Linux kernel and in Linux-based distributions. Specifically, the way hardware support is implemented, the abstractions built on that hardware support, and the mindset surrounding the licensing and distribution of the resulting code are all different.

#### Linux方式 The Linux Approach

一般为Linux提供新硬件支持的方式包含内核构建分或在运行时动态装载模块。相应的硬件随后通常可以通过进入*/dev*目录在用户空间访问。Linux驱动模型定义了三种基本的设备：字符设备，体现为字节流；块设备（基本为硬盘）以及网络设备。多年以来新增了很多额外设备与子系统，如USB或MTD设备。然而，*/dev*给到相应设备入口的API与接口方法仍然相当标准与稳定。
The usual way to provide support for new hardware in Linux is to create device drivers that are either built as part of the kernel or loaded dynamically at runtime through modules. The corresponding hardware is thereafter generally accessible in user-space through entries in */dev*. Linux's driver model defines three basic types of devices: character devices, devices that appear as stream of bytes, block devices (essentially hard disks), and networking devices. Over the years, quite a few additional device and subsystem types have been added, such as for USB or MTD devices. Nevertheless, the APIs and methods for interfaceing with */dev* entry corresponding to a given type of device have remained fairly standardized and stable.

因此，这就允许了*/dev*节点上构建各样的软件栈，得以直接与硬件交互或应用程序通过调用公开的API访问硬件。实际上，绝大多数的Linux发行版都有相似的内核库与子系统，如ALSA音频库和X Window系统。都能够通过*/dev*访问硬件设备。
This, therefore, has allowed various software stacks to be built on top of */dev* nodes to either interact with the hardware directly or expose generic APIs that are used by user applications to provide access to the hardware. The vast majority of Linux distributions in fact ship with a similar set of core libraries and subsystems, such as the ALSA audio libraries and the X Window System, to interface with hardware devices exposed through */dev*.

对于许可与发布方面，一般的“Linux”方式是始终将驱动作为主线内核的一部分进行合并与发布，并在GPL条款下发布。所以一般不建议将设备驱动独立开发与维护，甚至在其他协议下发布。实际上，在许可方面，非GPL的驱动一直是一个有争议的问题。因此，一般做法是用户和经销商最好从http://kernel.org中获取最新的主线内核驱动。这些内核在早先就已经在GPL下发布，尽管内核有添加，但仍然受GPL约束。
With regard to licensing and distribution, the general "Linux" approach has always been that drivers should be merged and maintained as part of the mainline kernel and distributed with it under the terms of the GPL. So, while some device drivers are developed and maintained independently and some are even distributed under other licenses, the consensus has been that that isn't the preferred approach. In fact, with regard to licensing, non-GPL drivers have always been a contentious issue. Hence, the conventional wisdom is that users' and distributors' best bet to get the latest drivers is usually to get the latest mainline kernel from http://kernel.org. This has been true since the kernel's early days and remains true despite some additions having been made to the kernel to allow the creation of user-space drivers.

#### 安卓通用方式 Android's General Approach

尽管安卓构建在内核硬件抽象与功能上，方式却大不相同。从纯技术层面而言，最显著的区别在于其子系统与库不需要依赖标准的*/dev*来工作。反之，安卓更加依赖由制造商提供的共享库与硬件交互。实际上，安卓依赖于一个被称为硬件抽象层(HAL层)的结构，不过，正如我们所见的，不同类型硬件间组件的接口、行为与功能都差异巨大。
Although Android builds on the kernel's hardware abstractions and capabilities, its approach is very different. On a purely technical level, the most glaring difference is that its subsystems and libraries don't rely on standard */dev* entries to function properly. Instead, the Android stack typically relies on shared libraries provided by manufacturers to interact with hardware. In effect, Android relies on what can be considered a 
Hardware Abstraction Layer (HAL), although, as we will see, the interface, behavior and function of abstracted hardware components differ greatly from type to type.

此外，大部分在Linux发行版上常见的软件栈并没有出现在安卓中。比如安卓中没有X Window系统。虽然有时也会用到ALSA驱动，但硬件制造商会决定提供何种共享库实现对HAL层音频的支持。所以功能访问这点上就与标准的Linux发行版不同。
In addition, most software stacks typically found in Linux distributions to interact with hardware are not found in Android. There is no X Window System, for instance, and while ALSA drivers are sometimes used—a decision left up to the hardware manufacturer who provides the shared library implementing audio support for the HAL—access to their functionality is different from that on standard Linux distributions.

图2-3呈现了安卓中典型的硬件抽象与支持方式，及相应的分发与许可。正如你所见的，安卓最终还是需要靠内核访问硬件。然而，这里的功能早就由设备制造商或AOSP中实现了。
Figure 2-3 presents the typical way in which hardware is abstracted and supported in Android, and the corresponding distribution and licensing. As you can see, Android still ultimately relies on the kernel to access the hardware. However, this is done through shared libraries that are either implemented by the device manufacturer or provided as part of the AOSP.

这种方法的一大特点就是共享库的许可由硬件制造商决定。因此，设备制造商可以创建一个简单的设备驱动实现给定硬件的基础功能，并将驱动发布在GPL下。硬件不会透露过多的内容，因为驱动不会玩什么花活。接着驱动会通过**mmap()**或**ioctl()**接口将硬件公开给用户空间，那么各种复杂的操作将在用户空间专用的共享库中实现，并以此驱动硬件设备。
One of the main features of this approach is that the license under which the shared library is distributed is up to the hardware manufacturer. Hence, a device manufacturer can create a simplistic device driver that implements the most basic primitives to access a given piece of hardware and make that driver available under the GPL. Not much would be revealed about the hardware, since the driver wouldn't do anything fancy. That driver would then expose the hardware to user-space through **mmap()** or **ioctl()** and the bulk of the intelligence would be implemented within a proprietary shared library in user-space that uses those functions to drive the hardware.

安卓并没有规定共享库、驱动或内核子系统应该如何交互。只有为上层提供API的共享库才由安卓指定。因此，只要能实现适用的API，你可以任意决定使用认为最适合的硬件驱动。不过，我们将在下章介绍使用与安卓的典型硬件方法。
Android does not in fact specify how the shared library and the driver or kernel subsystem should interact. Only the API provided by the shared library to the upper layers is specified by Android. Hence, it's up to you to determine the specific driver interface that best fits your hardware, so long as the shared library you provide implements the appropriate API. Nevertheless, we will cover the typical methods used by Android to interface to hardware in the next section.

而安卓相对不一致的地方是上层加载硬件支持共享库的方式。现在请注意，对于大多数的硬件类型，必须由AOSP或开发者提供*.so*类型文件，否则安卓就不能正常工作。
Where Android is relatively inconsistent is the way the hardware-supporting shared libraries are loaded by the upper layers. Remember for now that for most hardware types, there has to be a *.so* file that is either provided by the AOSP or that you must provide for Android to function properly.

无论是哪种方式，都是为了装载硬件支持的共享库，响应硬件的系统服务主要负责加载和连接共享库。这类系统服务负责与其他系统服务协调，使硬件和系统其余部分以及供开发人员使用的API保持一致。如果你需要为一个给定的硬件添加支持，你需要尽可能详细地了解这部分相关的系统服务内部结构。通常系统服务分为两部分，java实现大部分安卓相关的内容，另一部分则由C完成，主要负责支持共享库及其他底层功能的硬件交互。
No matter which mechanism is used to load a hardware-supporting shared library, a system service corresponding to the type of hardware is typically responsible for loading and interfacing with the shared libary. That system service will be responsible for interacting and coordinating with the other system services to make the hardware behave coherently with the rest of the system and the APIs exposed to app developers. If you're adding support for a given type of hardware, it's therefore crucial that you try to understand in as much detail as possible the internals of the system service corresponding to your hardware. Usually, the system service will be split in two parts, one part in Java that implements most of the Android-specific intelligence and another part in C whose main job is to interact with the hardware-supporting shared library and other low-level functions.

![Android's "Hardware Abstraction Layer"](https://i.bmp.ovh/imgs/2019/07/8075b50ec1af0b9c.png)

#### 装载与接口方法 Loading and Interfacing Methods

正如前文所提及，大体上系统服务与安卓有非常多的方法与硬件设备的共享库进行交互，从而实现对硬件的支持。很难完全理解为什么会存在如此多的方法，但是作者怀疑有一些是有组织地形成的。幸运的是，这似乎正在朝着更加统一的方式发展。鉴于安卓在以相当迅速的方式演化，这是在未来一个需要密切关注的领域，因为它或许一眨眼就进化了。
As I mentioned earlier, there are various ways in which system services and Android in general interact with the shared libraries implementing hardware support and hardware devices in general. It's difficult to fully understand why there is such a variety of methods, but I suspect that some of them evolved organically. Luckily, there seems to be a movement towards a more uniform way of doing things. Given that Android moves at a fairly rapid pace, this is one area that will require keeping an eye on for the forseeable future, as it's likely to evolve.

注意，这些方法的描述并不是互斥的。安卓技术栈中经常在加载或交互共享库及一些软件的前后时刻组合使用这些方法。
Note that the methods described here are not necessarily mutually exclusive. Often a combination of these is used within the Android stack to load and interface with a shared library or some software layer before or after it. I'll cover specific hardware in the next section.

**dlopen()** - *通过硬件抽象层加载*
**dlopen()** - *loading through HAL*
+ *相关：GPS，灯，传感器与显示*
一些硬件支持的共享库有libhardware库所装载。这个库属于安卓的硬件抽象层，并提供**hw_get_module()**接口，某些系统服务及子系统通过该接口显示加载特定硬件支持共享库（在**HAL**属于中成为“模块”）。**hw_get_module()**反过来依靠典型的**dlopen()**将加载到调用者的地址空间中。
+ *Applies to; GPS, Lights, Sensors, and Display*
Some hardware-supporting shared libraries are loaded by the libhardware library. This library is part of Android's HAL and exposes **hw_get_module()**, which is used by some system services and subsystems to explicitly load a given specific hardware-supporting shared library (a.k.a. a "module" in **HAL** terminology). **hw_get_module()** in turn relies on the classic **dlopen()** to load libraries into the caller's address space.

连接器装载*.so*文件
*Linker-loaded .so files*
+ *相关：音频，摄像头，wifi，马达及电源管理*
一般情况下，系统服务只是在构建时与指定的.so文件链接。因此，当二进制文件运行起来时，动态链接器会自动将共享库装载至进程的地址空间。
+ *Applies to: Audio, Camera, Wifi, Vibrator, and Power Management*
In some cases, system services are simply linked against a given .so file at build time. Hence, when the corresponding binary is run, the dynamic linker automatically loads the shared library into the process's address space.

*硬编码**dlopen()***
*Hardcoded **dlopen()**s*
+ *相关：StageFright与无线电接口层*
在一些场景下，内核赋予**dlopen()**直接获取硬件使能的共享库，无需再通过**libhardware**实现。目前还不清楚这种方法的原理。
+ *Applies to: StageFright and Radio Interface Layer(RIL)*
In a few cases, the code invokes **dlopen()** directly instead of going through **libhardware** to fetch a hardware-enabling shared library. The rationale for using this method instead of the HAL is unclear.

*套接字*
*Sockets*
+ *相关：蓝牙，网络管理，硬盘挂载和无线接口层*
系统服务或框架组件有时会使用套接字与硬件交互的远程守护进程及服务对话。
+ *Applies to: Bluetooth, Network Management, Disk Mounting, and Radio Interface Layer(RIL)*
Sockets are sometimes used by system services or framework components to talk to a remote daemon or service that actually interacts with the hardware.

*文件系统条目（项）*
*Sysfs entries*
+ *相关：马达与电源管理*
文件系统（/sys）中的一些条目被用来控制硬件或内核子系统行为。在某些情况下，相比于用/dev中的条目，安卓会使用这个方法来控制硬件。
+ *Applies to: Vibrator and Power Management*
Some entries in sysfs (/sys) can be used to control the behavior of hardware and/or kernel subsystems. In some cases, Android uses this method instead of /dev entries to control the hardware.

*/dev节点*
*/dev nodes*
+ *相关：几乎所有类型的硬件*
可以肯定的是，任何硬件抽象都必须在某个时刻与*/dev*中的一个条目通信，这就是驱动向用户空间公开的方式。有些通信通过共享库的方式隐藏在安卓中。有时AOSP也会直接访问设备节点，比如输入管理器使用输入库的场景。
+ *Applies to: Almost every type of hardware*
Aguably, any hardware abstraction must at some point communicate with an entry in */dev*, because that's how drivers are exposed to user-sapce. Some of this communication is likely hidden to Android itself because it interacts with a shared library instead, but in some other cases AOSP components directly acess device nodes. Such is the case of input libraries used by the Input Manager.

*D-Bus*
*D-Bus*
+ *相关：蓝牙*
D-Bus作为一个经典的信息系统，可以在几乎所有的Linux发行版找到，用于促进不同桌面组件间的通信。它之所以出现在安卓中，就是因为它是非GPL组件与GPL许可的BlueZ栈——Linux默认蓝牙堆栈对话的指定方式，如此一来在安卓中使用蓝牙就无需受到GPL再分发的要求。D-Bus本身有教育免费许可（AFL）与GPL的双重认可。关于D-Bus的更多信息，可以访问http://dbus.freedesktop.org。
+ *Applies to: Bluetooth*
D-Bus is a classic messaging system found in most Linux distributions for facilitating communication between various desktop components. It's included in Android because it's the prescribed way for a non-GPL component to talk to the GPL licensed BlueZ stack—Linux's default Bluetooth stack and the one used in Android—without being subject to the GPL's redistribution requirements; D-Bus itself being dual-licensed under the Academic Free License (AFL) and the GPL. Have a look at http://dbus.freedesktop.org for more information about D-Bus.

#### 设备支持细节 Device Support Details

表2-1汇总了安卓中支持每种类型硬件的方式。如你所注意到的，这里面包含了很多机制与接口组合。如果你打算实现特定硬件的支持，那么最好的方法就是从现有的示例中开始。AOSP专门为一些设备增加了硬件支持，主要是谷歌使用的最新测试机与一些旗舰机型。有时硬件支持来源十分广泛，例如三星Nexus S（代号俗称Crespo）。
Table 2-1 summarizes the way in which each type of hardware is supported in Android. As you'll notice, there is a wide variety of combinations of mechanisms and interfaces. If you plan on implementing support for a specific type of hardware, the best way forward is to start from an existing sample implementation. The AOSP typically includes hardware support code for a few handsets, generally those which were used by Google to develop new Android releases and therefore served as flagship devices. Sometimes the sources for the hardware support are quite extensive, as was the case for the Samsung Nexus S (a.k.a. "Crespo", its code-name).

唯一一个不太容易公开获取的硬件类型就是RIL。处于种种原因，最好不要让所有人都玩起无线电波。因此，制造商不提供此类实现。相反，如果你希望实现一个RIL，谷歌提供了其实现参考。
The only type of hardware for which you are unlikely to find publicly-available implementations on which to base your own is the RIL. For various reasons, it's best not to let everyone be able to play with the airwaves. Hence, manufacturers don't make such implementations available. Instead, Google provides a reference RIL implementation in the AOSP should you want to implement a RIL.

+ *表2-1. 安卓硬件支持方法与接口*
+ *Table 2-1. Android's hardware support methods and interfaces*

Hardware|System Service|Interface to user-space HW support|Interface to HW
-|-|-|-
Audio|Audio Flinger|Linker-loaded *libaudio.so*|Up to HW manufacturer, though ALSA is typical
Bluetooth|Bluetooth Service|Socket/D-Bus to BlueZ|BlueZ stack
Camera|Camera Service|Linker-loaded *libcamera.so*|Up to HW manufacturer, sometimes Video4Linux
Display|Suface Flinger|HAL-loaded *gralloc* module|*/dev/fb0 or /dev/graphics/fb0*
GPS|Location Manager|HAL-loaded *gps* module|Up to HW manufacturer
Input|Input Manager|Native library|Entries in */dev/input*
Lights|Lights Service|HAL-loaded *lights* module|Up to HW manufacturer
Media|N/A, StageFright framework within Media Service|dlopen on *libstagefrightw.so*|Up to HW manufacturer
Network interfaces|Network Management Service|Socket to netd|ioctl() on interfaces
Power Management|Power Manager Service|Linker-loaded libhardware_legacy.so|Entries in /sys/android_power/ or /sys/power 
Radio (phone)|N/A, entry point is telephony Java code|Socket to *rild*, which itself does a dlopen() on manufacturer-provided.so|Up to HW manufacturer
Storage|Mount Service|Socket to vold|System calls
Sensors|Sensor Service|HAL-loaded sensors module|Up to HW manufacturer
Vibrator|Vibrator Service|Linker-loaded libhardware_legacy.so|Up to HW manufacturer
Wifi|Wifi Service|Linker-loaded libhardware_legacy.so|Classic *wpa_supplicant*

-----------------------------------------

### 原生用户空间 Native User-Space

至此，我们已经覆盖了安卓的底层内容，接下来开始进入上一层。首先，我们需要了解安卓操作系统中的原生用户空间环境。关于“原生用户空间”，这里指的是所有用户空间组件都是运行在Dalvik虚拟机之外的。这之中包含了很多编译运行在目标CPU架构上的二进制文件。通常是自动或根据init进程的配置文件启动，或集成在命令行中由开发人员的脚本调用。这些二进制文件可以直接访问根文件系统与系统中包含的原生库。它们的功能被赋予其文件系统的权限限制，因为其运行在应用程序框架之外，所以并不会受到如安装框架对安卓应用的限制。
Now that we've covered the low-level layers on which Android is built, let's start going up the stack. First off, we'll cover the native user-space environment in which Android operates. By "native user-space" I mean all the user-space components that run outside the Dalvik virtual machine. This includes quite a few binaries that are compiled to run natively on the target's CPU architecture. These are generally started either automatically or as needed by the init process according to its configuration files, or are available to to be invoked on the command line once a developer shells into the device. Such binaries usually have direct access the root filesystem and the native libraries included in the system. Their capabilities would be gated by the filesystem rights granted to them and wouldn't be subject to any of the restrictions imposed on a typical Android app by the Android framework because they are running outside of it.

注意，安卓的用户空间基本是从一张白纸开始设计，与标准Linux发行版存在很大的不同。因此，下面将尽可能多的解释安卓用户空间与基于Linux系统的异同之处。
Note that Android's user-space was designed pretty much from a blank slate and differs greatly from what you'd find in a standard Linux distribution. Hence, I will try in as much as possible in the following to explain where Android's user-space is different or similar to what you'd usually find in a Linux-based system.

#### 文件系统布局 Filesystem layout

像其他基于Linux的发行版一样，安卓也使用一个根文件系统来存储应用，库和数据。但与这些发行版不同的是，安卓的根文件系统并不符合文件系统层次标准（FHS）。内核本身并不强制要求FHS，但大部分为Linux构建的软件包会假设它们运行的根文件系统符合FHS。因此，如果你打算将一个标准Linux应用移植到安卓中，你可能得费点力气以确保它依赖的文件路径依然有效。
Like any other Linux-based distribution, Android uses a root filesystem to store applications, libraries, and data. Unlike the vast majority of Linux-based distributions, however, the layout of Android's root filesystem does not adhere to the Filesystem Hierarchy Standard (FHS). † The kernel itself doesn't enforce the FHS, but most software packages built for Linux assume that the root filesystem they are running on conforms to the FHS. Hence, if you intend to port a standard Linux application to Android, you'll likely need to do some legwork to ensure that the filepaths it relies on are still valid.

考虑到运行在安卓用户空间的包基本都是为安卓专门编写的，上述的不一致性几乎没有影响。实际上，我们很快可以发现这也有一些好处。总之对学习安卓根文件系统还是很重要的。如果要在硬件上跑安卓系统，或者客制化硬件设备，最好还是在这上面花些时间。
Given that most of the packages running in Android's user space were written from scratch specifically for Android, this lack of conformity is of little to no consequence to Android itself. In fact, it has some benefits, as we'll see shortly. Still, it's important to learn how to navigate Android's root filesystem. If nothing else, you'll likely have to spend quite some time inside of it as you bring Android up on your hardware or customize it for that hardware.

安卓系统中的两个主要目录便是*/system*与*/data*。这两个目录并没有出现在FHS中。实际上，应该没有任何一个主流的Linux发行版会使用这两个目录。相反，这反应了安卓开发团队自己的设计。这或许是一个早起的迹象，暗示了安卓在同一套根文件系统上将自身与Linux发行版并至。我们同样安排在后文对这一点详细讨论。
The two main directories in which Android operates are */system* and */data*. These directories do not emanate from the FHS. In fact, I can't think of any mainstream Linux distribution that uses either of these directories. Rather, they reflect the Android development team's own design. This is one of the first signs hinting to the fact that it might be possible to host Android side-by-side with a common Linux distribution on the same root filesystem. As I said earlier, we'll actually examine this possibility in more detail later in the book.

*/system*目录用于存储由AOSP构建生成的不发生变化的组件，其中包括原生二进制文件，原生库，框架层包基于系统应用。它通常挂载在根文件系统独立出的镜像中，由其自身通过RAM盘镜像挂载。另一方面，*/data*分区用来存储则用来存储随时间会发生变化的文件，如数据及应用。其中包括由用户安装应用及系统组件运行时生成的内容。该分区也是在自己独立的镜像中被挂载。
*/system* is the main Android directory for storing immutable components generated by the build of the AOSP. This includes native binaries, native libraries, framework packages, and stock apps. It's usually mounted from a separate image from the root filesystem, which is itself mounted from a RAM disk image. */data*, on the other hand, is Android's main directory for storing data and apps that change over time. This includes the data generated and stored by apps installed by the user alongside data generated by Android system components at runtime. It too is usually mounted from its own separate image.

安卓中也包含很多在任何Linux系统中常见的目录， 如 */dev*，*/proc*， */sys*， */sbin*， */root*， */mnt*和 */etc*。这些目录的作用与其他Linux系统中基本是相似的，尽管它们经常被删减，如*/sbin*与*/etc*，而像*/root*，有时是空的。
Android also includes many directories commonly found in any Linux system, such as: */dev*, */proc*, */sys*, */sbin*, */root*, */mnt*, and */etc*. These directories often serve similar if not identical purposes to the the ones they serve on any Linux system, although they are very often trimmed down, as is the case of */sbin* and */etc*, and in some cases are empty, such as /root.

有趣的是，安卓没有包括*/bin*与*/lib*目录。这些目录在Linux中是至关重要的，它们中包含了基本的二进制文件与库文件。这也为安卓与标准Linux组件共存打开了一扇大门。
Interestingly, Android doesn't include any */bin* or */lib* directories. These directories are typically crucial in a Linux system, containing, respectively, essential binaries and essential libraries. This is yet another artefact that opens the door for making Android coexist with standard Linux components.

当然，对于安卓的根文件系统还有很多地方可聊。例如刚才的内容仅仅提到了目录与它们的层次结构。安卓的根文件系统还包含了这里未提及的其他目录。我们将在第五章中更加详细地回顾安卓根文件系统及其组成。
There is of course more to be said about Android's root filesystem. The directories just mentioned, for instance, contain their own hierarchies. Also, Android's root filesystem contains other directories that I haven't covered here. We will revist the Android root filesystem and its make-up in more detail in Chapter 5.

#### 库 Libraries

安卓依赖着上百个动态装载库，这些库都被存储在*/system/lib*路径下。有一定数量的库来源于外部项目，这些项目最后也被合并入安卓的源码中，以使其能够可以运行在安卓上，但大部分的*/system/lib*库还是由AOSP本身生成的。
Android relies on about a hundred dynamically-loaded libraries, all stored in the */system/lib* directory. A certain number of these come from external projects that were merged into Android's codebase to make their functionality available within the Android stack, but a large portion of the libraries in */system/lib* are actually generated from within the AOSP itself. Table 2-2 lists the libraries included in the AOSP that come 
from external projects, whereas Table 2-3 summarizes the Android-specific libraries generated from within the AOSP.

+ *表 2-2. 由AOSP收纳外部项目生成的库*
+ *Table 2-2. Libraries generated from external projects imported into the AOSP*

Library(ies)|Extemal Project|Original Location|License
-|-|-|-
*libcrypto.so and libssl.so*|OpenSSL|http://www.openssl.org|Custom, BSD-like
*libdbus.so*|D-Bus|http://dbus.freedesktop.org|AFL and GPL
*libexif.so*|Exif Jpeg header manipulation tool|http://www.sentex.net/~mwandel/jhead/|Public Domain
*libexpat.so*|Expat XML Parser|http://expat.sourceforge.net|MIT
*libFFTEm.so*|neven face recognition library|N/A|ASL
*libicui18n.so and libicuuc.so*|International Components for Unicode|http://icu-project.org|MIT
*libiprouteutil.so and libnetlink.so*|iproute2 TCP/IP networking and traffic control|http://www.linuxfoundation.org/collaborate/workgroups/networking/iproute2|GPL
*libjpeg.so*|libjpeg|http://www.ijg.org|Custom, BSD-like
*libnfc_ndef.so*|NXP Semiconductor's NFC library|N/A|ASL
*libskia.so and libskiagl.so*|skia 2D graphics library|http://code.google.com/p/skia/|ASL
*libsonivox*|Sonic Network's Audio Synthesis library|N/A|ASL
*libsqlite.so*|SQLite database|http://www.sqlite.org|Public Domain
*libSR_AudioIn.so and libsrec_jni.so*|Nuance Communications' Speech Recognition engine|N/A|ASL
*libstlport.so*|Implementation of the C++ Standard Template Library|http://stlport.sourceforge.net|Custom, BSD-like
*libttspico.so*|SVOX's Text-To-Speech speech synthesizer engine|N/A|ASL
*libvorbisidec.so*|Tremolo ARM-optimized Ogg Vorbis decompression library|http://wss.co.uk/pinknoise/tremolo/|Custom, BSD-like
*libwebcore.so*|WebKit Open Source Project|http://www.webkit.org|LGPL and BSD
*libwpa_client*|Library based on wpa_supplicant|http://hostap.epitest.fi/wpa_supplicant/|GPL and BSD
*libz.so*|zlib compression library|http://zlib.net|Custom, BSD-like

+ *表 2-3. 由AOSP生成的安卓定制库*
+ *Table 2-3. Android-specific libraries generated from within the AOSP*

Category|Library(ies)|Description
-|-|-
Bionic|*libc.so; libm.so; libdl.so; libstdc++.so; libthread_db.so;*|C library; Math library; Dynamic linking library; Standard C++ library; Threads library;
Core|*libbinder.so; libutils.so, libcutils.so, libnetutils.so, and libsysutils.so;  libsystem_server.so, libandroid_servers.so, libaudioflinger.so, libsurfaceflinger.so, linsensorservice.so, and libcameraservice.so; libcamera_client.so* and *libsufaceflinger_client.so; libpixelflinger.so; libui.so; liblog.so;*|The Binder library; Various utility libraries; System-service-related libraries; Client libraries for certain system services; The PixelFlinger library; Low-level user-interface-related functionalities, such as user input events handling and dispatching and graphics buffer allocation and manipulation; Sensors-related functions library; The logging library; The Android runtime library;
Dalvik|*lidvm.so; libnativehepler.so*|The Dalvik VM library; JNI-related helper functions;
Hardware|*libhardware.so; libhardware_legacy.so; Various hardware-supporting shared libraries.*|The HAL library that provides hw_get_module() uses dlopen() to load hardware support modules (i.e. shared libraries that provide hardware support to the HAL) on demand; Library providing hardware support for wifi, powermanagement and vibrator; Libraries that provide support for various hardware components, some of which are loaded using through the HAL, while others are loaded automatically by the linker;
Media|*libmediaplayerservice.so; libmedia.so; libstagefright.so; libeffects.so* and the libraries in the *soundfx/* directory; *libdrm1.so* and *libdrm1_jni.so*|The Media Player service library; The low-level media functions used by the Media Player service; The many libraries that make-up the StageFright media framework; The sound effects libraries; The DRM b framework libraries;
OpenGL|*libEGL.so, libETC1.so, libGLESv2.so* and *egl/libGLES_android.so*|Android's OpenGL implementation

#### 初始化 Init

当内核完成启动，它仅仅会调起一个进程，这个进程就是init进程。随后，init负责生成系统中所有其他进程和服务，并执行一些如重启之类的高危操作。传统的Linux发行版使用SystemV init提供init进程，尽管近年来许多发行版都使用了自己的修改版本。如Ubuntu就使用了upstart。在嵌入式Linux系统中，提供init的就是经典的BusyBox包。
When the kernel finishes booting, it starts just one process, the init process. This process is then responsible for spawning all other processes and services in the system and for conducting critical operations such as reboots. The package traditionally provided by Linux distributions for the init process uses SystemV init, although in recent years many distributions have created their own variants. Ubuntu, for instance, uses Upstart. In embedded Linux systems, the classic package that provides init is BusyBox.

安卓引入了自定义的init，也带来了一些新鲜玩意。
Android introduces its own custom init, which brings with it a few novelties.

##### 配置语言 Configuration language

不同于传统的init，传统init根据当前运行等级配置或请求的脚本使用来决定的，而安卓init定义了自己的配置语义，并根据全局属性值来触发特定的指令。
Unlike traditional inits, which are predicated on the use of scripts that run per the current run-levels' configuration or on request, Android's init defines its own configuration semantics and relies mostly on changes to global properties to trigger the execution of specific instructions.

init的主要配置文件存储在*/init.rc*中，同时也会有一个特定设备的配置文件*/init.**某某设备**.rc*。特定的设备脚本则为*/etc/init.**某某设备**.sh*，这里的**某某设备**指的是设备名称。可以通过修改这些文件达到对系统启动及其行为进行高度控制。例如，你可以禁止Zygote的自启动，并将其改为通过adb命令手动启动。
The main configuration file for init is usually stored as */init.rc*, but there's also usually a device-specific configuration file stored as */init.**device_name**.rc* and device-specific script stored as */etc/init.**device_name**.sh*, where **device_name** is the name of the device. You can get a high degree of control over the system's startup and its behavior by modifying those files. For instance, you can disable the Zygote from starting up automatically and then starting it manually yourself after having used adb to shell into the device.

##### 全局属性 Global properties

安卓init中非常有趣的一点是，它如何去管理一组全局属性，并且这些属性可以被系统中很多有适当权限的部分访问。有些属性在编译时确定，有些在init配置文件中，也有些在运行时才被设置。有些属性会被保存到存储中被永久使用。属性由init管理，因此它可以检测到任何修改，并根据其配置触发某组命令的执行。
A very interesting aspect of Android's init is how it manages a global set of properties that can be accessed and set from many parts of the system, with the appropriate rights. Some of these properties are set at build time, while others are set in init's configuration files and still others are set at runtime. Some properties are also persisted to storage for permanent use. Since init manages the properties, it can detect any changes and therefore trigger the execution of a set of commands based on its configuration.

例如之前提及的OOM调整，也是设置为从*init.rc*文件中启动。网络属性也是如此，编译时将该属性写入*/system/build.prop*文件中，并加上编译日期及编译系统详情。当运行时，系统中包含着从IP和GSM配置参数到电池电量上百个属性，使用*getprop*命令可以列出当前的属性及其值。
The OOM adjustments mentioned earlier, for instance, are set on startup by the *init.rc* file. So are network properties. Properties set at build time are stored in the */system/build.prop* file and include the build date and build system details. At runtime, the system will have over a hundred different properties, ranging from IP and GSM configuration parameters to the battery's level. Use the *getprop* command to get the current list of properties and their values.

##### 内核设备管理器事件 udev events

正如之前所解释的，可以通过Linux的*/dev*目录访问设备。曾经Linux发行版在该目录中包含了上千目录，以适应可能遇到的设备配置。最终还是有些人提议将这些目录创建为动态形式。一段时间后，系统就使用了*udev*，它的工作依赖于每次向系统中添加或删除硬件时，内核生成的运行时事件。
As I explained earlier, access to devices in Linux is done through nodes within the */dev* directory. In the older days, Linux distributions would ship with thousands of entries in that directory to accomodate all possible device configurations. Eventually, though, a few schemes were proposed to make the creation of such nodes dynamic. For some time now, the system in use has been *udev*, which relies on runtime events generated by the kernel every time hardware is added or removed from the system.

在大部分Linux发行版中，udev热插拔事件由udevd守护进程处理。而安卓中，这些事件由作为安卓init部分构建的*ueventd*守护进程处理，通过符号链接将*/sbin/ueventd*链至*/init*来访问。想要知道*/dev*中创建了那些条目，*ueventd*依赖于*/ueventd.rc*和*/ueventd.**device_name**.rc*文件      
In most Linux distributions, the handling of udev hotplug events is done by the udevd daemon. In Android, these events are handled by the *ueventd* daemon built as part of Android's init and accessed through a symbolic link from */sbin/ueventd* to */init*. To know which entries to create in */dev*, *ueventd* relies on the */ueventd.rc* and */ueventd.**device_name**.rc* files. 

#### 工具箱 Toolbox

如根文件系统目录层级一般，大部分的Linux发行版由FHS在/bin与/sbin路径下都存在着许多二进制文件。这些目录下的二进制文件由网上很多不同站点、不同的包编译而成。在一个嵌入式系统中，处理如此多不同的包或需要如此多分裂的二进制文件都不甚合理。
Much like the root filesystem's directory hierarchy, there are essential binaries on most Linux system, listed by the FHS for the /bin and /sbin directories. In most Linux distributions, the binaries in those directories are built from separate packages coming from different sites on the net. In an embedded system, it doesn't make sense to have to deal with so many packages, nor necessarily have that many separate binaries.

经典的BusyBox包采取的方法是编译一个独立的二进制文件，这个文件实际上是个巨大的switch-case结构，通过检查命令行中第一个参数，决定执行哪个响应功能。这些指令再软链到busybox命令中。例如当输入ls时，实际上还是在执行BusyBox，但BusyBox根据ls这个参数决定它的行为，就如同在标准命令行中运行这个命令一样。
The approach taken by the classic BusyBox package is to build a single binary that essentially has what amounts to a huge switch-case , which checks for the first parameter on the command line and executes the corresponding functionality. All commands are then made to be symbolic links the busybox command. So when you type ls, for example, you're actually invoking BusyBox. But since BusyBox's behavior is predicated on the first parameter on the command line and that parameter is ls, it will behave as if you had run that command from a standard Linux shell.

安卓没有使用BusyBox，但引入了自己的工具——Toolbox。基本功能也是软链至Toolbox的命令中。不过Toolbox不如BusyBox强大，如果你使用过BusyBox，那基本会对Toolbox很失望。重新创造一个类似的工具似乎和许可有关，毕竟BusyBox是GPL许可。另外，一些安卓开发者们的目标是提供一个基于shell的简洁调试工具，并非与BusyBox一样完全取代shell。而使用BSD许可的Toolbox可以让制造商们在不需要向客户提供源码的前提下进行修改和分发。
Android doesn't use BusyBox, but includes its own tool, Toolbox, that basically functions in the very same way using symbolic links to the toolbox command. Unfortunately, Toolbox is nowhere as feature-full as BusyBox. In fact, if you've ever used BusyBox, you're likely going to be very disappointed when using Toolbox. The rationale for creating a tool from scratch in this case seems to make most sense when viewed from the licensing angle, BusyBox being GPL licensed. In addition, some Android developers have stated that their goal was to create a minimal tool for shell-based debugging and not to provide a full replacement for shell tools as BusyBox does. At any rate, Toolbox is BSD licensed and manufacturers can therefore modify it and distribute it without having to track the modifications made by their developers or making any sources available to their customers.

你或许还是希望使用BusyBox替换Toolbox。如果由于许可原因，不希望将BusyBox作为定版软件的一部分，也可以暂时将BusyBox打包进系统用来调试，再在释放版本时移除。
You might still want to include BusyBox alongside Toolbox to benefit from its capabilities. If you don't want to ship it as part of your final product because of its licensing, you could include it temporarily during development and strip it in the final production release. We'll cover this in more detail later.

#### 守护进程 Daemons

作为系统启动的一部分，安卓的init会启动一些守护进程，并跑完整个系统的生命周期。而如*adbd*，则需要取决于全局属性的设置了。
As part of the system startup, Android's init starts a few key daemons that continue to run throughout the lifetime of the system. Some daemon, such as *adbd*, are started on damand, depending on changes to global properties.

*表 2-4. 安卓原生守护进程*
*Table 2-4. Native Android daemons*
Daemon|Description
-|-
servicemanager|The Binder Context Manager. Acts as an index of all Binder services running in the system.
vold|The volume manager. Handles the mounting and formatting of mounted volumes and images.
netd|The network manager. Handles tethering, NAT, PPP, PAN, and USB RNDIS.
debugerd|The debugger daemon. Invoked by Bionic's linker when a process crashes to do a postmortem analysis. Allows *gdb* to connect from the host.
Zygote|The Zygote process. It's responsible for warming up the system's cache and starting the System Server. We'll discuss it in more detail later in this chapter.
mediaserver|The Media server. Hosts most media-related services. We'll discuss it in more detail later in this chapter.
dbus-daemon|The D-Bus message daemon. Acts as an inter mediary between D-Bus users. Have a look at its man page for more information.
bluetooth|The Bluetooth daemon. Manages Bluetooth devices. Provides services through D-Bus.
installd|The *.apk* installtion daemon. Takes care of installing and uninstalling *.apk* files and managing the related filesystem entries.
keystore|The KeyStore daemon. Manages an encrypted key-pair value store for cryptographic keys, SSL certs for instance.
system_server|Android's System Server. This daemon hosts the vast majority of system services that run in Android.
adbd|The ADB daemon. Manages all aspects of the connection between the target and the host's *adb* command.

#### 命令行组件 Command-Line Utilities

安卓根文件系统中分布着超过150个命令行组件。*/system/bin*中就包含了一大部分，也有一些“额外”的存在*/system/xbin*与*/sbin*中。*/system/bin*中有大约50个组件是*/system/bin/toolbox*的软链。余下的主要来自安卓系统框架，AOSP引入的外部项目或AOSP的其他地方。在第五章可以花时间看看这些二进制文件。
More than 150 command-line utilities are scattered over Android's root filesystem. */system/bin* contains the majority of them, but some "extras" are in */system/xbin* and a handful are in */sbin*. Around 50 of those in */system/bin* are actually symbolic links to */system/bin/toolbox*. The majority of the rest come from the Android base framewrok, from external projects merged into the AOSP, or from various other parts of the AOSP. We'll get the chance to cover the various binaries found in the AOSP in more detail in Chapter 5.

### Dalvik虚拟机与安卓下的Java Dalvik and Android's Java

简而言之，Dalvik就是安卓的java虚拟机。它允许安卓运行由java编写的应用和系统组件生成的字节码，并向系统提供所需的钩子和环境接口，包含原生链接库与原生用户空间。关于Dalvik和安卓品牌可以扯很远。但为了深入研究，首先需要掌握一些java基础。
In a untshell, Dalvik is Android's Java virtual machine. It allows Android to run the byte-code generated from Java-based apps and Android's own system components, and proveides both with the requierd hooks and environment to interface with the rest of the system, including native libraries and the rest of the native user-space. There's more to be said about Dalvik and Android's brand of Java, though. But before we can delveinto that explanation, we must first cover some Java basics.

为了不让你们再上一节关于java语言与其起源的历史课，就直接说java是90年代初期由James Gosling在Sun公司创造的。java一经面试就迅速流行起来，在安卓出现就已经足够完善。对于开发者而言，有两点需要牢牢记住：java与C/C++这些传统的编程语言不同，我们常说的“Java”由组件构成。
Without boring you with yet another histroy lesson on the Java language and its origins, suffice it to say that Java was created by James Gosling at Sun in the early '90s, that it rapidly became very popular, and that it was, in sum, more than well established before Android came around. From a developer perspective, the are two aspects that are important to keep in mind with regard to Java: its differences with a traditional language such as C and C++, and the components that make up what we commonly refer to as "Java."

从设计上看，Java是一种解释性的语言。不同与C/C++，当代码被编译器编译后，变为二进制汇编指令被解释器执行。这些与CPU架构独立的字节码解释器能够在运行时执行字节码，这就是我们常说的“虚拟机”。如此的操作方式与Java语义，使Java中拥有了很多传统语言中不存在的特性，如反射和匿名类。另外，Java也不需要像C/C++那样一直跟踪分配的对象。实际上，它不需要关心任何未使用的对象，因为它自身的垃圾回收机制可以保证一旦没有活动的代码引用后就会将对象销毁。
By design, Java is an interpreted langeuage. Unlike C and C++, where the code you write gets compiled by a compiler into binary assembly instructions to be executed by a CPU matching the architecture-independent byte-code that is executed at a run-time by a byte-code interpreter, also commonly referred to as a "virtual machine." This modus operandi, along with Java's semantics, enable the language to include quite a few features not traditionally found in previous languages, such as  reflection and anonymous classes. Also, unlike C and C++, Java doesn't require you to keep track of objects you allocate. In fact, it requires you to lose track of all unused objects, since it's got an integrated garbage-collector that will ensure all such objects are destroyed when no active code holds a reference to them any longer.

从使用角度看，Java实际由这些东西组成：Java编译器，Java解释器——通常被成为Java虚拟机（JVM），以及Java链接库，通常被称为Java开发套件（JDK）由甲骨文提供。安卓实际上在编译时使用了JDK，但并没有用到JVM或其链接库。它实际用了Apache的Harmony项目。
At a practical level, Java is actually made up of a few distinct things: the Java compiler, the Java byte-code interpreter -- more commonly konwn as the Java Virtual Machine (JVM) -- and the Java libraries commonly used by Java Development Kit (JDK) provided free of charge by Oracle. Android actually relies on the JDK for the Java compiler, but it doesn't use the JVM or the libraries, it relies on the Apache Harmony project, a clean-room implementation of the Java libraries hosted under the umbrella of the Apache project.

据开发者Dan Bornstein说，Dalvik是一个为嵌入式系统定制的JVM。它的目标是那些CPU比较慢，内存空间少，跑操作系统不需要交换空间，且使用电池供电的设备。
According to its developer, Dan Bornstein, Dalvik distinguishes itself form the JVM by being specifically designed for embedded systems. Namely, it targets systems that have slow CPUs and relatively litte RAM, run OSes that don't use swap space, and are battery powered.

相比JVM处理的是*.class*文件，Dalvk更加喜欢*.dex*。*.dex*实际由安卓通过*dx*组件通过Java编译器对*.class*类处理生成。一个未经压缩的*.dex*文件要比原生的*.jar*文件小50%。另一个有趣的事情是，Dalvik基于寄存器，而JVM基于堆栈。除非你是虚拟化或体系结构的学生，亦或者对此非常感兴趣，否则以上内容没有太大意义。
While the JVM muches on *.class* files, Dalvik prefers the *.dex* delicatessen. *.dex* files are actually generated by postprocessing the *.class* files generated by the Java compiler through Android's *dx* utility. Among other things, an uncompressed *.dex* file is 50% smaller than its originating *.jar* file. Another interesting factoid is that Dalvik is register-based where as the JVM is stack-based, though that is likely to have little to no meaning to you unless you're an avid student of VM theory, architecture, and internals.

Davlik有一个特点非常值得重点关注，从2010年开始，它为ARM加入了即时编译器。这意味着Dalvik将应用的字节码转化为二进制汇编指令并可以运行在原生的目标CPU上，再不是在虚拟机中临时解释每条指令。如此变换可以为今后的特性使用。因此，应用程序会在第一次载入时多花些时间，但是一旦已经被载入过，之后就会更加迅速。唯一需要注意的是，JIT不能被ARM外的架构所使用。所以总而言之，目前运行安卓系统最快的架构就是ARM。
A feature of Davlik very much worth highlighting, though, is that since 2010 it has included a Just-In-Time (JIT) compiler for ARM. This means that Dalvik converts apps' byte-codes to binary assembly instructions that run natively on the target's CPU instead of being interpreted one instruction at a time by the VM. The result of this conversion is then stored for future use. Hence, apps take longer to load the first time, but once they've been JIT'ed, they load and run much faster. The only caveat here is that JIT isn't available for any other architecture then ARM. So, in sum, the fastest architecture to run Android on is, for now, ARM.

作为一名嵌入式开发人员，让Dalvik跑起来不需要额外做什么特别的事情。Dalvik是以架构独立的方式实现的。早前有报道Dalvik中存在一些字节序的问题，然而早就被解决了。
 As an embedded developer, you're unlikely to need to do anything specific to get Dalvik to work on your system. Dalvik was written to be architecture-independent. It has been reported that some of the early ports of Dalvik suffered from some endian issues. However, these issues seem to have subsided since.

#### Java原生接口（JNI）Java Native Interface (JNI)

尽管Java强大并且易用，但它总不能自己单打独斗。并且在码代码时还需要调用一些其他语言的接口。尤其在安卓这种嵌入式系统中，总要经常和底层功能打打交道。为了实现这个需求，Java原生接口机制出现了。类似于.NET/C#中的*pinvoke*，它实际上是C/C++这类外部语言的调用入口。
Despite its power and benefits, Java can't always operate in a vacuum, and code written in Java sometimes needs to interface to code coming from other languages. This is especially true in an embedded environment such as Android, where low-level functionality is never too far away. To that end, the Java Native Interface (JNI) mechanism is provided. It's essentially a call gate to other languages such as C and C++. It's an equivalent to *pinvoke* in the .NET/C# world.

应用程序开发人员有时使用JNI调用一些他们编译的NDK原生代码，就像使用SDK编译Java。其实在AOSP内部就大量使用JNI来让Java编码的服务和组件与C/C++实现的底层功能对接。例如Java编写的系统服务，就使用JNI与对应的原生代码接口通信，从而将服务传达至相应的硬件设备。
App developers sometimes use JNI to call the native code they compile with the NDK from their regular Java code built using the SDK. Internally, though, the AOSP massively relies on JNI to enable Java-coded services and components to interface with Android's low-level functionality, which is mostly written in C and C++. Java-written system services, for instance, very often use JNI to communicate with matching native code that interfaces with a given service's corresponding hardware.

允许Java通过JNI的方式与其他语言交互，很大部分的功能是由Dalvik提供的。回到上个章节的表2-3中，你会发现*libnativehelper.so*这个库就是由Dalvik提供，便于使用者调用JNI的。
A large part of the heavy lifting to allow Java to communicate with other languages through JNI is actually done by Dalvik. If you go back to Tbale 2-3 in the previous section, for instacne, you'll notice the *libnativehelper.so* library, which is provided as part of Dalvik for facilitating JNI calls.

在本书稍后部分，我们有机会使用JNI连接Java和C代码。目前阶段，请记住，JNI是安卓系统的核心部分，也是一个使用起来相对复杂的机制。特别要确保调用时正确地给出了语义与参数。
In later parts of the book, we'll actually get the chance to use JNI to interface Java and C code. For the moment being, keep in mind that JNI is central to platform work in Android and that it can be a relatively complex mechanism to use, especially to make sure you use the appropriate call semantics and function parameters.

![image](https://upload-images.jianshu.io/upload_images/2424151-0d1edd6891b48dac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图 2-4. 系统服务

### 系统服务 System Services

系统服务是安卓系统中的幕后人物。就算没有在谷歌开发者文档中特别提及，任何和安卓相关的东西基本都会使用到其中某一个系统服务。这些服务在一起提供了Linux上层的面向对象功能，也就是Binder机制，所有系统服务构建的目的所在。刚才讨论过的原生用户空间就像是安卓系统服务的一个支持环境。所以了解系统服务的存在，知道它们之间及与系统其他部分如何工作是非常重要的。我们已经包含了一些与安卓硬件支持有关的讨论。
System services are Android's man behind the curtain. Even if they aren't explicily mentioned in Google's app development documentation, anything remotely interesting in Android goes through one of about 50 system services. There services cooperate together to collectively provide what essentially amounts to an object-oriented OS built on top of Linux, which is exactly what Binder -- the mechanism on which all system services are built -- was intended for. The native user-space we just covered is actually designed very much as a support environment for Android's system services. It's therefore crucial to understand what system services exist, and how they interact with each other and with the rest of the system. We've already covered some of this as part of discussing Android's hardware support.

图2-4相较于图2-1第一次详细地展示了系统服务。可以看到这其中包含了很多主要的进程。大部分是系统服务，都跑在*system_server*这个进程下。基本是由一个Java字节码编写的服务与两个C/C++服务组成。系统服务中也包含一些原生代码，经过JNI允许一些基于Java的服务访问系统底层。其他的系统服务位于Media Service中，以*media-server*运行。这些服务都是由C/C++编写，与StageFright及audio effects等多媒体相关的组件一起打包。
Figure 2-4 illustrates in grater detail the system services first introduced in Figure 2-1. As you can see, there are in fact a couple of major processes involved. Most prominent is the System Server, whose components all run under the sames process, *system_server*, and which is mostly made up of Java-coded services with two services written in C/C++. The System Server also includes some native code access through JNI to allow some of the Java-based services to interface to Android's lower layers. The rest of the system services are housed within the Media Service which runs as *media-server.* These services are all coded in C/C++ and are packaged alongside media-related components such as the StageFright and audio effects.

注意，尽管只有两个进程撑起了整个安卓系统服务，它们都是以独立的方式通过Binder对服务连接。以下是在安卓模拟器中输出的对*service*组件的结果。
Note that despite there being only two processes to house the entirety of the Android's system services, they all appear to operate independently to anyone connecting to their services through Binder. Here's the output of the *service* utiltiy on the Android emulator:

> **# service list**
Found 50 servics:
0    phone: [com.android.internal.telephony.ITelephony]
1    iphonesubinfo: [com.android.internal.telephony.IPhoneSubInfo]
2    simphonebook: [com.android.internal.telephony.IIccPhoneBook]
3    isms: [com.android.internal.telephony.ISms]
4    diskstats: []
5    appwidget: [com.android.internal.appwidget.IAppWidgetService]
6    backup: [android.app.backup.IBackupManager]
7    uimode: [android.app.IUiModeManager]
8    usb: [android.hardware.usb.IUsbManager]
9    audio: [android.media.IAudioService]
10    wallpaper: [android.app.IWallpaperManager]
11    dropbox: [com.android.internal.os.IDropBoxManagerService]
12    search: [android.app.ISearchManager]
13    location: [android.location.ILocationManager]
14    devicestoragemonitor: []
15    notification: [android.app.INotificationManager]
16    mount: [IMountService]
17    accessibility: [android.view.accessibility.IAccessibilityManager]
18    throttle: [android.net.IThrottleManager]
19    connectivity: [android.net.IConnectivityManager]
20    wifi: [android.net.wifi.IWifiManager]
21    network_management: [android.os.INetworkManagementService]
22    netstat: [android.os.INetStatService]
23    input_method: [com.android.internal.view.IInputMethodManager]
24    clipboard: [android.text.IClipboard]
25    statusbar: [com.android.internal.statusbar.IStatusBarService]
26    device_policy: [android.app.admin.IDevicePolicyManager]
27    window: [android.view.IWindowManager]
28    alarm: [android.app.IAlarmManager]
29    vibrator: [android.os.IVibratorService]
30    hardware: [android.os.IHardwareService]
31    battery: []
32    content: [android.content.IContentService]
33    account: [android.accounts.IAccountManager]
34    permission: [android.os.IPermissionController]
35    cpuinfo: []
36    meminfo: []
37    activity: [android.app.IActivityManager]
38    package: [android.content.pm.IPackageManager]
39    telephony.registry: [com.android.internal.telephony.ITelephonyRegistry]
40    usagestats: [com.android.internal.app.IUsageStats]
41    batteryinfo: [com.android.internal.app.IBatteryStats]
42    power: [android.os.IPowerManager]
43    entropy: []
44    sensorservice: [android.gui.SensorServer]
45    SurfaceFlinger: [android.ui.ISurfaceComposer]
46    media.audio_policy: [android.media.IAudioPolicyService]
47    media.camera: [android.hardware.ICameraService]
48    media.player: [android.media.IMediaPlayerService]
49    media.audio_flinger: [android.media.IAudioFlinger]

目前没有太多文档描述各个服务的操作。感兴趣的话可以看看它们的源码，理解和分析它们是如何相互工作的。
There is unfortunately not much documentation on how each of these services operates. You'll have to look at each service's source code to get a precise idea of how it works and how it interacts with other services.


#### 服务管理器与Binder交互 Service Manager and Binder Interaction

如之前所解释的，Binder机制用于系统服务底层面向对象的远程方法调用。一个系统进程通过Binder调用系统服务，首先需要拿到这个服务的句柄。例如，Binder会让应用开发人员在电源管理中通过**WakeLock**类的**acquire()**方法请求一个休眠唤醒锁，在这个调用完成前，开发人员首先需要获取电源管理服务的句柄。在下一章节可以看到，应用实际上将获取句柄的细节隐藏起来，抽象出了一个接口给开发者，如图2-5所示，所有系统服务句柄查找都是通过服务管理器完成的。
As I explained earlier, the Binder mechanism used as system services' underlying fabricenables object-oriented remote method invocation. For a process in the system to invoke a system service through Binder, though, it must first have a handle to it. For instance, Binder will enable an app developer to request a wakelock from the Power Manager by invoking the **acquire()** method of its **WakeLock** nested class. Before that call can be made, though, the developer must first get a handle to the Power Manager service. As we'll see in the next section, the app development API actually hides the details of how it gets this handle in an abstraction to the developer, but under the hood all system service handle lookups are done through the Service Manager, as illustrated in Figure 2-5.

![Service Manager and Binder interaction](https://upload-images.jianshu.io/upload_images/2424151-1643d2e062f3ad33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*Figure 2-5. Service Manager and Binder interaction*


将服务管理器想象成系统中可以服务的黄页，如果一个系统服务没有经过服务管理器注册，那它就无法作用于剩下的系统。为了提供这个检索功能，服务管理器会在所有服务起来前被*init*调起。然后服务管理器再打开*/dev/binder*并使用特殊的**ioctl()**使自己成为Binder的*上下文管理器*（见图2-5中的A1）。此后，系统中任何试图与Binder ID 0（即代码各部分中的“magic”Binder或“magic object”）通信的进程，实际上都是通过Binder在与服务管理器通信。
Think of the Service Manager as a YellowPages book of all services available in the system. If a system service isn't registered with the Service Manager, it's effectively invisible to the rest of the system. To provide this indexing capability, the Service Manager is started by *init* before any other service. It then opens */dev/binder* and uses a special **ioctl()** call to set itself as the Binder's *Context Manager* ("A1" in Figure 2-5.) Thereafter, any process in the system that attempts to communicate with Binder ID 0 (a.k.a. the "magic" Binder or "magic object" in various parts of the code), is actually communicating through Binder to the Service Manager.

当系统服务起来后，它会通过服务管理器（A2）注册它实例化的每一个服务。之后，当有应用想要访问系统服务，例如电源管理服务，管理器就会问服务（B1）要一个句柄并调用服务的方法（B2）。总之，一个服务组件的调用是应用直接通过Binder（C1）进行的，并不会由服务管理器经手。
When the System Server starts, for instance, it registers every single service it instantiates with the Service Manager ("A2".) Later, when an app tries to talk to a system service, such as the Power Manager service, it first asks the Service Manager for a handle to the service ("B1") and then invokes that service's methods ("B2"). In contrast, a call to a service component running within an app goes directly through Binder ("C1"), and is noy looked up through the Service Manager.

服务管理器还会由“dumpsys”单元这一特殊方式使用，它能让你把单个或全部系统服务的状态转出来。为了得到所有服务的列表，它会循环获取系统服务（D1），在迭代器中请求n<sup>th</sup>+1直到不存在。它会通过服务管理器定位到有特殊句柄的那个服务（D2），并调用服务的**dump()**函数在终端中打印出状态信息（D3）
The Service Manager is also used in a special way by the *dumpsys* utility, which allows you to dump the status of a single or all system services. To get the list of all services, it loops around to get every system service ("D1"), requesting the n<sup>th</sup> plus one at every iteration until there aren't anymore. To get each service, it just asks the Service Manager to locate that specific one ("D2".) With a service handle in hand, it invokes that service's **dump()** function to dump its status ("D3") and displays that on the terminal.

#### 调用服务 Calling on Services

上文中解释的这些，基本都是对用户不可见的。下面这一小段示例代码为我们展示应用程序一般是如何申请休眠唤醒锁接口的。
All of what I just explained is, as I said earlier, almost invisible to the user. Here's a snippet, for instance, that allows us to grab a wakelock within an app using the regular application development API:

> PowerManager pm = (PowerManager) getSystemService(POWER_SERVICE);
PowerManager.WakeLock wakeLock = pm.newWakeLock(PowerManager.FULL_WAKE_LOCK, "myPerciousWakeLock");
wakeLock.acquire(100);

可以看到这里没有任何服务管理器的痕迹，取而代之的是通过**getSystemService()**传递了**POWER_SERVICE**参数。当然在内部，**getSystemService()**还是使用服务管理器搜索到电源管理服务，并申请创建了一个休眠唤醒锁。
Notice that we don't see any hint of the Service Manager here. Instead, we're using **getSystemService()** and passing it the **POWER_SERVICE** parameter. Internally, though, the code that implements **getSystemService()** does actually use the Service Manager to locate the Power Manager service so that we create a wakelock and acquire it.

#### 服务示例：活动管理器 A Service Example: the Activity Manager

虽说介绍每一个系统服务不在本书的范围内，但还是快速过一下活动管理器这个关键的系统服务。活动管理器的资源实际上包含了超过20个文件和两万行代码，非常接近安卓的核心了。它负责启动活动或服务这样的新组件，以及获取内容提供者与上下文广播。如果你之前就见过ANR（应用无响应）对话框。其实活动管理器还参与了内核低内存处理规划，权限以及任务管理等。
Although covering each and every system service is outside the scope of this book, let's have a quick look at the Activity Manager, one of the key system services. The Activity Manager's sources actually span over 30 files and 20k lines of code. If there's a core to Android's internals, this service is very much near it. It takes care of the starting of new components, such as Activities and Services, along with the fetching of Content Providers and intent broadcasting. If you ever got the dreaded ANR (Application Not Responding) dialog box, know that the Activity Manager was behind it. It's also involved in the maintenance of OOM adjustments used by the in-kernel low-memory handler, permissions, task management, etc.

例如用户在主屏幕上点击应用图标，启动了一个应用程序，首先启动器的**onClick()**回调被调用。启动器会通过Binder告知活动管理服务通过**startActivity()**方法来处理这个事件。服务继而调用**startViaZygote()**方法，通过套接字通知Zygote启动一个活动。在阅读完本章的最后一节后就可以清晰地理解这些操作。
For instance, when the user clicks on a icon to start an app from his home screen, the first that happens is that the Launcher's * **onClick()** callback is called. To deal with the event, the Launcher will then call, through Binder, the **startActivity()** method of the Activity Manager service. The service will then call the **startViaZygote()** method, which will open a socket to the Zygote and ask it to start the Activity. All this may make more sense after you read the final section of this chapter.

如果你熟悉Linux的内部结构，将安卓文件中的*kernel/*目录理解为Linux的内核源码，这是一个理解活动管理器的好方法。
If you're familiar with Linux's internals, a good way to think of the Activity Manager is that it's to Android what the content of the *kernel/* directory in the kernel's sources is to Linux. It's that important.

### AOSP应用包 Stock AOSP Packages

在大部分安卓设备中，AOSP都会捆绑几个默认的应用包，在之前章节中也有提到过。如地图、油管、Gmail就不能算在AOSP中。表2-5中列举了AOSP中默认的这些包，表2-6中AOSP中的内容提供者，表2-7中则展示了默认输入法。
The AOSP ships with a certain number of default packages that are found in most Android devices. As I mentioned in the previous chapter, though, some apps such as Maps, YouTube, and Gmail aren't part of the AOSP. Let's take a look at some of those packages included by default. Table 2-5 lists the stock apps include in the AOSP, Table 2-6 lists the stock content providers included in the AOSP, and Table 2-7 lists the stock IMEs (Input Method Editors) included in the AOSP.

*Table 2-5. Stock AOSP Apps*

App in AOSP|Name displayed in Launcher|Description
-|-|-
AccountsAndSettings|N/A|Accounts management app
Bluetooth|N/A|Bluetooth manager
Browser|Browser|Default Android browser, includes bookmark widget
Calculator|Calculator|Calculator app
Camera|Camera|Camera app
Certinstaller|N/A|UI for installing certificates
Contacts|Contacts|Contacts manager app
DeskClock|Clock|Clock and alarm app, including the clock widget
Email|Email|Default Android email app
Development|Dev Tools|Miscellaneous dev tools
Gallery|Gallery|Default gallery app for viewing pictures
Gallery3D|Gallery|Fancy gallery with "sexier" UI
HTMLViewer|N/A|App for viewing HTML files
Launcher2|N/A|Default home screen
Mms|Messaging|SMS/MMS app
Music|Music|Music player
PackageInstaller|N/A|App install/uninstall UI
Phone|Phone|Default phone dialer/UI
Protips|N/A|Home screen tips
Provision|N/A|App for setting a flag indicating whether a device was provisioned
QuickSearchBox|Search|Search app and widget
Settings|Settings|Settings app, also accessible through home screen menu
SoundRecorder|N/A|Sound recording app
SpeechRecorder|Speech Recorder|Speech recording app
SystemUI|N/A|Status bar

*Table 2-6. Stock AOSP Providers*

Provider|Description
-|-
ApplicationProvider|Provider for search installed apps
CalendarProvider|Main Android calendar storage and provider
ContactsProvider|Main Android contacts storage and provider
DownloadProvider|Download management, storage and  access
DmProvider|Management and access of DRM-protected storage
MediaProvider|Media storage and provider
TelephonyProvider|Carrier and SMS/MMS storage and provider
UserDictionnaryProvider|Storage and provider for user-defined words dictionary


*Table 2-7. Stock AOSP Input Methods*

Input Methods|Description
-|-
LatinIME|Latin keyboard
OpenWnn|Japanese keyboard
PinyinIME|Chinese keyboard


### 系统启动 System Startup

通过分析安卓启动过程能够很好地把概念融会贯通。如图2-6所示，首先是CPU开始动作。CPU会从某个硬编码地址获取第一条指令，这个地址一般指向芯片的bootloader编程区域。接着bootloader初始化RAM，将基础硬件置为静止状态，加载内核与RAM区域并进入内核。近来的CPU及单芯片外设类的带片上系统设备，已经可以由格式化SD卡等设备上启动了。如pandaBoard和BeagleBorad的新版本就可以直接由SD卡启动，无需依赖板上的存储芯片。
The best way to bring together all that we discussed is to look at Android's startup. As you can see in Figure 2-6, the first cog to turn is the CPU. It typically has a hard-coded address from which it fetches its first instructions. That address usually points to a chip that has the bootloader programmed on it. The bootloader then initializes the RAM, puts basic hardware in a quiescent state, loads the kernel and RAM disk, and jumps into the kernel. More recent System-on-Chip (SoC) devices, which include a CPU and a slew of peripherials in a single chip, can actually boot straight from a properly formatted SD card or SD-card-like chip. The PandaBoard and recent editions of the BeagleBoard, for instance, don't have any on-board flash chips because they boot straight from an SD card.

内核初始化与硬件密切相关，但它的目的就是尽早地让CPU开始执行C代码。一旦满足条件，内核就能够执行与架构独立的**start_kernel()**方法，初始化一些子系统，并为内核驱动启动"init"方法。内核启动时一大片日志信息就是在这几步中打印的。随后内核开始挂载根文件系统并启动初始化进程。
The initial kernel startup is very hardware dependent, but its purpose is to set things up so that the CPU can start executing C code as early as possible. Once that's done, the kernel jumps to the architecture-independent **start_kernel()** function, initializes its various subsystems, and proceed to call the "init" functions of all built-in drivers. The majority of messages printed out by the kernel at startup come from these steps. The kernel then mounts its root filesystem and starts the init process.

![Android's boot sequence](https://upload-images.jianshu.io/upload_images/2424151-d9c02705a63ece60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Android's boot sequence](https://ftp.bmp.ovh/imgs/2019/11/6e5e88880f4f4492.png)
*Figure 2-6. Android's boot sequence*
*安卓启动流程*

初始化成功后，安卓开始执行*/init.rc*中的指令并构建起如系统路径的一系列环境变量，创建挂载点，挂载文件系统，设置OOM，启动原生守护进程。我们已经介绍过一些安卓中的原生守护进程，但也值得再花些精力讨论Zygote。Zygote的特殊之处在于它是负责启动应用的守护进程，它会整合应用程序共用的组件以缩短应用打开的时间。初始化并没有直接启动Zygote，而是通过*app_process*命令在ART时启动，ART启动系统第一个Dalvik虚拟机并告诉它调用Zygote的**main()**方法。
That's when Android's init kicks in and executes the instructions stored in its */init.rc* file to set up environment variables such as the system path, create mount points, mount filesystems, set OOM adjustments, and start native daemons. We've already covered the various native daemons active in Android, but it's worth focusing a little on the Zygote. The Zygote is a special daemon whose job is to launch apps. Its functionality is centralized here in order to unify the components shared by all apps and to shorten
their start-up time. init doesn't actually start the Zygote directly; instead it uses the *app_process* command to get Zygote started by the Android runtime. The runtime then starts the first Dalvik VM of the system and tells it to invoke the Zygote's **main()**.

Zygote只会在启动新应用时激活。为了实现应用的更快速启动，Zygote会在运行时预加载应用程序所需的Java类和资源。这使得系统RAM载入更加高效。接着，Zygote会在自己的套接字（*/dev/socket/zygote*）上监听启动新应用的连接请求。当它收到启动应用程序的请求时，就会fork一份自己并启动新程序。这个做法的美妙之处在于，应用从Zygote中fork而来就立即拥有了可能需要的类与资源。换言之，一个新应用程序启动不需要等待所需资源的加载与执行。
Zygote is active only when a new app needs to be launched. To achieve a speedier app launch, the Zygote starts by preloading all Java classes and resources that an app may potentially need at runtime. This effectively loads those into the system's RAM. The Zygote then listens for connections on its socket (*/dev/socket/zygote*) for requests to start new apps. When it gets a request to start an app, it forks itself and launches the new app. The beauty of having all apps fork from the Zygote is that it's a "virgin" VM that has all the system classes and resources an app may need preloaded and ready to be used. In other words, new apps don't have to wait until those are loaded to start executing.

这些努力能够实现，是因为Linux内核为fork提供了~~大奶牛~~写时拷贝策略。在Unix中fork创建的新进程与父进程完全相同。而通过写时拷贝，Linux实际上不需要拷贝任何东西，相反，它将新进程页映射到父进程页，只有当新进程写入这页时才会执行拷贝。其中类与资源加载是不会被写入的，因为它们都是默认的，并且在系统的生命周期中几乎不发生变化。所有直接由Zygote fork进程的本质都是使用自身映射的拷贝。因此，无论系统中跑了多少应用程序，RAM中只会加载一份系统类与资源的拷贝。
All of this works because the Linux kernel implements a Copy-On-Write (COW) policy for forks. As you may know, forking in Unix involves creating a new process that is an exact same copy of the parent process. With COW, Linux doesn't actually copy anything. Instead, it maps the pages of the new process over to those of the parent process and makes copies only when the new process writes to a page. But in fact the classes and resources loaded are never written to, because they're the default ones and are pretty much immutable within the lifetime of the system. So all processes directly forking from the Zygote are essentially using its own mapped copies. And therefore, regardless of the number of apps running on the system, only one copy of the system classes and the resources is ever loaded in RAM.

尽管Zygote被设计用于监听fork新应用请求的连接，有一个“应用”却是被Zygote显式启动：系统服务器。系统服务器是Zygote启动的第一个应用， 并独立于父进程一直存在。然后系统服务器开始初始化每个系统服务，并注册到之前启动的服务管理器中。如活动管理器启动，就会通过发送**Intent.CATEGORY_HOME**类型意图结束初始化，这将打开启动器程序，继而将用户们熟悉的主界面展示出来。
Although the Zygote is designed to listen to connections for requests for forking new apps, there is one "app" that the Zygote actually starts explicitly: the System Server. This is the first app started by the Zygote and it continues to live on as an entirely separate process from its parent. The System Server then starts initializing each system service it houses and registering it with the previously-started Service Manager. One of the services it starts, the Activity Manager, will end its initialization by sending an intent of type **Intent.CATEGORY_HOME**. This starts the Launcher app, which then displays the home screen familiar to all Android users.

当用户点击主界面上的图标，启动器告诉活动管理器开启进程，请求被转发给Zygote，由Zygote fork并启动一个新程序，最终能够展示出来。
When the user clicks on an icon on the home screen, the process I described in “System Services” on page 53 takes place. The Launcher asks the Activity Manager to start the process, which in turn "forwards" that request on to the Zygote, which itself forks and starts the new app, which is then displayed to the user.

一旦系统完成启动，进程列表看起来如下：
Once the system has finished starting up, the process list will look something like this:

省略了很多，可以自己在终端ps一下
> **# ps**
USER PID PPID VSIZE RSS WCHAN PC NAME
(for example)
root 1 0 268 180 c009b74c 0000875c S /init
root 2 0 0 0 0 c004e72c 00000000 S kthreadd
...
...

这个输出来自于安卓模拟器，所以包含了如*qemud*这样模拟器特有的进程。注意尽管是由Zygote fork出来，它们是以完整包名展示。有一个小技巧，通过系统的**prctl()**方法带上**PR_SET_NAME**参数可以让内核改变调用进程的名称。如果有兴趣可以看一下**prctl()**的命令手册。另一个需要注意的地方是init启动的第一个进程是*ueventd*。在此之前，所有进程都是由子系统或驱动程序从内核中启动的。
This output actually comes from the Android emulator, so it contains some emulator-specific artefacts such as the *qemud* daemon. Notice that the apps running all bare their fully-qualified package names despite being forked from the Zygote. This is a neat trick that can be pulled in Linux by using the **prctl()** system call with **PR_SET_NAME** to tell the kernel to change the calling process' name. Have a look at **prctl()**'s man page if you're interested in it. Note also that the first process started by init is actually *ueventd*. All processes prior to that are actually started from within the kernel by subsystems or drivers.


---------------------------
完结撒花❀❀❀