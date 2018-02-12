---
title: 仿今日头条客户端思路整理（一）
---

# {{ page.title }}

## 目录

+ [序言](#序言)
+ [架构](#架构)
+ [结束](#结束)
----------------------------------

## 序言

我写这个新闻客户端的想法是17年11月份吧，原因一是自己没有写过资讯类的项目，二是每天我打开最多，使用时间最长的app就是今日头条，这是现在为止，新闻阅读体验最好的一款app，于是使我对其产生了模仿的想法。这片文章是为了梳理思路而写。
    
----------------------------------

## 架构

起初的架构，并没有完全的组件化，仅仅是通常的按功能分包+mvp模式 进行，并将自定义控件及三方库做为主module的依赖库进行开发。但是后续就牵扯到了一个严重的问题就是module之间的相互通信及调用问题，这使得我必须使其进行双向依赖，这样的缺点非常明显就是耦合性非常高。不利于复用、扩展及维护。之后就在考虑如何解决这个问题，于是就决心对项目进行组件化架构。
    
组件化是什么？其实，就我的理解而言，它就是面向接口编程的优秀实现，app内的模块之间完全隔离，使代码架构更加清晰，同时模块化的编译可以有效减少编译时间，单独的模块可以进行快速的独立编译及调试，极大的提升了效率，节约了时间。
    
最开始研究是看了简书发布的一篇名为[Android组件化开发实践](https://www.jianshu.com/p/186fa07fc48a)的文章，作者是wutongke，通过对其demo的实践熟悉了组件化开发的原理及思路，架构的基本方式。
    
![](https://upload-images.jianshu.io/upload_images/1407686-2a3990b4b781784e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
    
其中App是主application，ModuleA和ModuleB是两个业务模块，Library是基础模块，包含所有模块需要的依赖库，以及一些工具类：如网络访问、时间工具等。代码结构如图：
    
[](https://upload-images.jianshu.io/upload_images/1407686-0635652ee1b4040b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
    
除此之外，还掌握了
    
1.Module 作为application 及Library之间的切换配置

2.Module 作为application 及Library时不同的资源集配置

3.资源名冲突的解决方法

4.Activity跳转问题
    
Activity在组件化之后，最大的问题便是模块之间的通信及调用。第一种情况是页面跳转，第二种情况是模块间的数据传递与通信。

第一种：我觉得除了阿里的Arouter 这个也比较不错。名字叫[ActivityRouter](https://github.com/mzule/ActivityRouter) 其使用Scheme的方式进行跳转。支持给Activity定义 URL，这样可以通过 URL 跳转到Activity，支持在浏览器以及 app 中跳入。
    
Scheme的方式是建立映射表，集中处理Activity，这种方式可以传递一定的数据。Activity传递大量数据时可以通过EventBus来进行传递（其实即使通过intent显示启动，也不要把大量数据放置在intent中，intent对数据大小有限制）。

ActivityRouter的配置及使用的具体方法如下
    
集成：

AndroidManifest.xml配置

    <activity
    android:name="com.github.mzule.activityrouter.router.RouterActivity"
    android:theme="@android:style/Theme.NoDisplay">
    <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="mzule" /><!--改成自己的scheme-->
    </intent-filter>
    </activity>
    
在需要配置的Activity上添加注解

    @Router("main")
    public class MainActivity extends Activity {
    ...
    }
    这样就可以通过mzule://main来打开MainActivity了。
    
    
支持配置多个地址

    @Router({"main", "root"})
    mzule://main和mzule://root都可以访问到同一个Activity
    
支持获取 url 中?传递的参数

    @Router("main")
    上面的配置，可以通过mzule://main?color=0xff878798&name=you+are+best来传递参数，在MainActivity#onCreate中通过getIntent().getStringExtra("name")的方式来获取参数，所有的参数默认为String类型，但是可以通过配置指定参数类型，后面会介绍。
    
支持在 path 中定义参数

    @Router("main/:color")
    通过:color的方式定义参数，参数名为color，访问mzule://main/0xff878798，可以在MainActivity#onCreate通过getIntent().getStringExtra("color")获取到 color 的值0xff878798
    
    
支持多级 path 参数

    @Router("user/:userId/:topicId/:commentId")
    @Router("user/:userId/topic/:topicId/comment/:commentId")
    上面两种方式都是被支持的，分别定义了三个参数，userId,topicId,commentId
    
支持指定参数类型

    @Router(value = "main/:color", intParams = "color")
    这样指定了参数color的类型为int，在MainActivity#onCreate获取 color 可以通过getIntent().getIntExtra("color", 0)来获取。支持的参数类型有int,long,short,byte,char,float,double,boolean，默认不指定则为String类型。
    
支持优先适配

    @Router("user/:userId")
    public class UserActivity extends Activity {
    ...
    }
    
    @Router("user/statistics")
    public class UserStatisticsActivity extends Activity {
    ...
    }
    
假设有上面两个配置，
    不支持优先适配的情况下，mzule://user/statistics可能会适配到@Router("user/:userId")，并且userId=statistics
    支持优先适配，意味着，mzule://user/statistics会直接适配到@Router("user/statistics")，不会适配前一个@Router("user/:userId")
    
支持 Callback
    
    public class App extends Application implements RouterCallbackProvider {
    @Override
    public RouterCallback provideRouterCallback() {
    return new SimpleRouterCallback() {
    @Override
    public boolean beforeOpen(Context context, Uri uri) {
    context.startActivity(new Intent(context, LaunchActivity.class));
    // 是否拦截，true 拦截，false 不拦截
    return false;
    }
    
    @Override
    public void afterOpen(Context context, Uri uri) {
    }
    
    @Override
    public void notFound(Context context, Uri uri) {
    context.startActivity(new Intent(context, NotFoundActivity.class));
    }
    
    @Override
    public void error(Context context, Uri uri, Throwable e) {
    context.startActivity(ErrorStackActivity.makeIntent(context, uri, e));
    }
    };
    }
    }
    
在Application中实现RouterCallbackProvider接口，通过provideRouterCallback()方法提供RouterCallback，具体 API 如上。
    
支持 Http(s) 协议

    @Router({"http://mzule.com/main", "main"})
    
    AndroidManifest.xml
    <activity
    android:name="com.github.mzule.activityrouter.router.RouterActivity"
    android:theme="@android:style/Theme.NoDisplay">
    ...
    <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="http" android:host="mzule.com" />
    </intent-filter>
    </activity>
    这样，http://mzule.com/main和mzule://main都可以映射到同一个 Activity，值得注意的是，在@Router中声明http协议地址时，需要写全称。
    
支持参数 transfer
    
    @Router(value = "item", longParams = "id", transfer = "id=>itemId")
    
这里通过transfer = "id=>itemId"的方式，设定了 url 中名称为id的参数会被改名成itemId放到参数Bundle中，类型为long. 值得注意的是，这里，通过longParams = "id"或者longParams = "itemId"都可以设置参数类型为long.
    
    
支持应用内调用

    Routers.open(context, "mzule://main/0xff878798")
    Routers.open(context, Uri.parse("mzule://main/0xff878798"))
    Routers.openForResult(activity, "mzule://main/0xff878798", REQUEST_CODE);
    Routers.openForResult(activity, Uri.parse("mzule://main/0xff878798"), REQUEST_CODE);
    // 获取 Intent
    Intent intent = Routers.resolve(context, "mzule://main/0xff878798")
    通过Routers.open(Context, String)或者Routers.open(Context, Uri)可以直接在应用内打开对应的 Activity，不去要经过 RouterActivity 跳转，效率更高。
    
支持获取原始 url 信息

    getIntent().getStringExtra(Routers.KEY_RAW_URL);
    支持通过 url 调用方法
    @Router("logout")
    public static void logout(Context context, Bundle bundle) {
    }
    
在任意参数为 Context 和 Bundle 的静态公共方法上, 通过 @Router 标记即可定义方法的 url. @Router 使用方式与上述一致。
    
支持多模块
    •    每个包含 activity 的 module 都要添加 apt 依赖
    
   •     每个 module(包含主项目) 都要添加一个 @Module(name) 的注解在任意类上面，name 是项目的名称
    
   •    主项目要添加一个 @Modules({name0, name1, name2}) 的注解，指定所有的 module 名称集合

----------------------------------

## 结束

这个库真的非常好用，可是，我还需要第二种需求，那就是模块间的频繁通信及相互调用，ActivityRouter显然侧重在跳转上，不能满足我的需求，而后我在里一片文章里讲到一种组件化框架《ModularizationArchitecture》,此部分内容将在我的后续章节涉及。


    {{ page.date|date_to_string }}



    




