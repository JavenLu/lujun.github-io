---
title: SmartNews客户端架构思路整理（二）
---

# {{ page.title }}

## 目录

+ [序言](#序言)
+ [架构](#架构)
+ [结束](#结束)

----------------------------------

## 序言

这篇接上篇继续梳理仿今日头条客户端思路，对于模块间频繁的通信与调用，仅仅依靠跳转框架是无法满足需求的，于是我参考一片文章[Android架构思考(模块化、多进程)](http://blog.spinytech.com/2016/12/28/android_modularization/) 作者：Spiny ，建立起我的组件化模块间通信框架。其中使用提及的[ModularizationArchitecture](https://github.com/SpinyTech/ModularizationArchitecture)框架进行搭建。本篇主要阐述本项目的组件化框架的由来及原理。

----------------------------------

## 架构

Spiny展示了各个阶段项目的架构模型，及优缺点。

#### 最初的超小型项目

当我们最开始做Android项目的时候，大多数人都是没考虑项目架构的，我们先上一张图。

![](http://ogsbxb571.bkt.clouddn.com/old_architecture1.jpg)

这个分包结构有没有很熟悉，各种组件都码在一个包里，完全没有层级结构，业务、界面、
逻辑都耦合在一起。这是我12年底刚开始入门Android的时候开发的一个小项目，半年后，来了个小伙伴，然后我们一起开发，然后天天因为谁修改了谁的代码打的不可开交。

#### 架构改进，小型项目

再后来开发App，人员比之前多了，所以不能按照以前那样了，必须得重构。于是我把公用的代码提取出来制作成SDK基础库，把单独的功能封装成Library包，不同业务通过分包结构分到不同module下，组内每人开发自己的module。刚开始都还轻松加愉快，并行开发啥的，一片融洽的场景，如下图。

![](http://ogsbxb571.bkt.clouddn.com/%E6%97%A9%E6%9C%9F%E6%9E%B6%E6%9E%84-1.png)

随着时间推移，我们的App迭代了几个版本，这几个版本也没什么别的，大体来讲就是三件事情：

扩展了一些新业务模块，同时模块间相互调用也增加了。
修改增加了一些新的库文件，来支持新的业务模块。
对Common SDK进行了扩展、修复。
很惭愧，就做了一些微小的工作，但是架构就变成下图这样。

![](http://ogsbxb571.bkt.clouddn.com/%E6%97%A9%E6%9C%9F%E6%9E%B6%E6%9E%84-2.png)

可以看到，随着几个版本业务的增加，各个业务某块之间耦合愈发严重，导致代码很难维护，更新，更别说写测试代码了。虽然后期引入统一广播系统，一定程度改善了模块间相互引用的问题，但是局限性和耦合性还是很高，没办法根治这个问题。这个架构做到最后，扩展性和可维护性都是很差，并且难以测试，所以最终被历史的进程所抛弃。

#### 中小型项目，路由架构

![](http://ogsbxb571.bkt.clouddn.com/%E8%B7%AF%E7%94%B1-1.png)

通过上图可以看到，我们在最基础的Common库中，创建了一个路由Router，中间有n个模块Module，这个Module实际上就是Android Studio中的module，这些Module都是Android Library Module，最上面的Module Main是可运行的Android Application Module。

这几个Module都引用了Common库，同时Main Module还引用了A、B、N这几个Module，经过这样的处理之后，所有的Module之间的相互调用就都消失了，耦合性降低，所有的通信统一都交给Router来处理分发，而注册工作则交由Main Module去进行初始化。这个架构思想其实和Binder的思想很类似，采用C/S模式，模块之间隔离，数据通过共享区域进行传递。模块与模块之间只暴露对外开放的Action，所以也具备面向接口编程思想。

图中的红色矩形代表的是行动Action，Action是具体的执行类，其内部的invoke方法是具体执行的代码逻辑。如果涉及到并发操作的话，可以在invoke方法内加入锁，或者直接在invoke方法上加上synchronized描述。

图中的黄色矩形代表的是供应商Provider，每个Provider中包含1个或多个Action，其内部的数据结构以HashMap来存储Action。首先HashMap查询的时间复杂度是O(1)，符合我们对调用速度上的要求，其次，由于我们是统一进行注册，所以在写入时并不存在并发线程并发问题，在读取时，并发问题则交由Action的invoke去具体处理。在每一个Module内都会有1个或多个供应商Provider（如果不包含Provider，那么这个Module将无法为其他Module提供服务。

途中蓝色矩形代表的是路由Router，每个Router中包含多个Provider，其内部的数据结构也是以HashMap来存储Provider，原理也和Provider是一样的。之所以用了两次HashMap，有两点原因，一个是因为这样做，不容易导致Action的重名，另一个是因为在注册的时候，只注册Provider会减少注册代码，更易读。并且由于HashMap的查询时间复杂度是O(1)，所以两次查找不会浪费太多时间。当查找不到对应Action的时候，Router会生成一个ErrorAction，会告之调用者没有找到对应的Action，由调用者来决定接下来如何处理。

#### 一次请求流程

通过Router调用的具体流程是这样的:

![](http://ogsbxb571.bkt.clouddn.com/Router%E6%97%B6%E5%BA%8F%E5%9B%BE.png)

1.任意代码创建一个RouterRequest，包含Provider和Action信息，向Router进行请求。

2.Router接到请求，通过RouterRequest的Provider信息，在内部的HashMap中查找对应的Provider。

3.Provider接到请求，在内部的HashMap中查找到对应的Action信息。

4.Action调用invoke方法。

5.返回invoke方法生成的ActionResult。

6.将Result封装成RouterResponse，返回给调用者。


我在使用这个框架的时候暂时没有跨进行的需求，所以我使用的是同一个app内的不同模块之间的通信方法。

随着项目的不断扩大，App在运行时的内存消耗也在不断增加，而且有时线上的BUG也会导致整体崩溃。为了保证良好的用户体验，减少对系统资源的消耗，我们开始考虑采取多进程重新架构程序，通过按需加载，及时释放，达到优化的目的。

多进程的优点和使用场景，之前在《Android多进程使用场景》中也做过介绍，大体优点有这么几个：

1.提高各个进程的稳定性，单一进程崩溃后不影响整个程序。

2.对于内存的时候更可控，可以通过手工释放进程，达到内存优化目的。

3.基于独立的JVM，各个模块可以充分解耦。

4.只保留daemon进程的情况下，会使应用存活时间更长，不容易被回收掉。

#### 潜在问题

但是启用多进程，那就意味着Router系统的失效。Router是JVM级别的单例模式，并不支持跨进程访问。也就是说，你的后台进程的所有Provider、Action，是注册给后台Router的。当你在前台进程调用的时候，根本调用不到其他进程的Action。

#### 解决方案

其实解决的方法也并不复杂。原来的路由系统还可以继续使用，我们可以把整套架构想象成互联网，现在多个进程有多个路由，我们只需要把多个路由连接到一起，那么整个路由系统还是可以正常运行的。所以我们把原有的路由Router称之为本地路由LocalRouter，现在，我们需要提供一个IPS、DNS供应商，那就创建一个进程，该进程的作用就是注册路由，链接路由，转发报文，我们称之为广域路由WideRouter。

多进程的情况如图：

![](http://ogsbxb571.bkt.clouddn.com/%E5%A4%9A%E8%BF%9B%E7%A8%8B%E8%B7%AF%E7%94%B1%E8%BF%9E%E6%8E%A5%E5%9B%BE.png)


如图所示，竖直方向上，每一列，代表一个进程，通过虚线隔开，分别有Process WideRouter、Process Main、Process A、···、Process N这些进程。浅黄色的代表WideRouter，深黄色的代表WideRouter的守护Service。浅蓝色的代表每个进程的LocalRouter，深蓝色的代表每个LocalRouter的守护Service。WideRouter通过AIDL与每个进程LocalRouter的守护Service绑定到一起，每个LocalRouter也是通过AIDL与WideRouter的守护Service绑定到一起，这样，就达到了所有路由都是双向互连的目的。

#### 事件分发

之前单一路由的事件分发是通过两层HashMap查找Provider和Action，进行事件下发。那么现在在外面加了一层WideRouter，那么我们再加一层Domain，Domain对应的是Android应用内，各个进程的进程名。通常情况下，如果事件是在同一进程下，那么就类似于局域网内部事件传递，不需要通过WideRouter，直接内部按照之前的路由逻辑进行转发，如果不在相同进程内，就由WideRouter进行进程间通信，达到跨进程调用的效果。


事件请求RouterRequest可以写成两种，一种是URL，一种JSON。（内部处理的时候统一使用JSON），同时也提供了对URL和JSON的解析方法，方便使用。

URL:xxxDomain/xxxProvider/xxxAction?data1=xxx&data2=xxx
这就和Http请求很像了。这样做的好处就是对后续WebView上可以非常便利得直接调用本地Action。

JSON:

    {
      domain: xxx,
      provider: xxx,
      action: xxx,
      data{
      data1: xxx,
      data2: xxx
    }
    
JSON方式简单明了，可作为接口返回值由服务器下发给客户端。

下面仔细讲一下一次跨进程请求，事件是如何传递的：

![](http://ogsbxb571.bkt.clouddn.com/%E5%B9%BF%E5%9F%9F%E4%BA%8B%E4%BB%B6%E4%BC%A0%E9%80%92%E6%97%B6%E5%BA%8F%E5%9B%BE.png)

从图中可以清晰地看出，我们主要是分两大部分去完成事件分发传递的。

第一部分，跨进程判断目标Action是否是异步程序。

第二部分，跨进程执行目标Action调用。

首先我们先通过Domain、Provider、Action去跨进程查找是否是异步程序。如果是异步程序，那么我们直接生成RouterResponse(Step13)，并且，将Step14-Step24统一封装成Future，放在RouterResponse中，直接返回。如果是同步程序，那么就在当前方法内执行Step14-Step24，将返回结果放入RouterResponse内(Step25)，直接返回。这么做的目的是，我们的路由调用方法route(RouterRequest)默认是同步方法，不耗时的，可以直接在主线程里调用而不造成阻塞，不造成ANR。如果调用的目标Action是异步的，那么可以利用Java的FutureTask原理，调用RouterResponse的get()方法，获取结果。这个get()方法有可能是耗时的，是否耗时，取决于RouterResponse.isAsync的值是否是true。

至于本地事件分发，还是与之前的Router模式，从Step17到Step21，都是我们上文中，单进程同步Router分发机制，没有作任何改变。

####多进程Application逻辑分发

在多进程中，每启动一个新的进程，都会重新创建一次Application，所以，我们需要把各个进程的Application逻辑剥离出来，然后根据不同的Process Name，选择不同的Application逻辑进行处理。

实际的Application启动流程如下：

![](http://ogsbxb571.bkt.clouddn.com/%E5%A4%9A%E8%BF%9B%E7%A8%8BApplication%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.png)

首先，我们先把所有ApplicationLogic注册到Application中，然后，Application会根据注册时的进程名信息进行筛选，选择相同进程名的ApplicationLogic，保存到本进程中，然后，对这些本进程的ApplicationLogic进行实例化，最后，调用ApplicationLogic的onCreate方法，实现ApplicationLogic与Application生命周期同步，同时还有onTerminate、onLowMemory、onTrimMemory、onConfigurationChanged等方法，与onCreate一致。

#### 结束进程，释放内存

在我们不使用某些进程的时候，比如听音乐的时候，可以把主界面关掉等等。我们可以调用对应进程的LocalRouter的stopSelf()方法，该方法可以使本进程与WideRouter进行解绑，然后我们在手动关掉进程内的其他组件，最后调用System.exit()，达到释放内存的目的。合理的释放内存，能有效的改善用户体验。

----------------------------------

## 结束

通过本篇文章，详述了本人SmartNews项目组件化框架的由来及原理机制。下篇文章开始正式分析该项目。




    
    







