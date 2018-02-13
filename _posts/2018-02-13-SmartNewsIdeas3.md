---
title:SmartNews客户端架构思路整理（三）
---

# {{ page.title }}

## 目录

+ [序言](#序言)
+ [架构](#架构)
+ [结束](#结束)

----------------------------------

## 序言

从本篇文章开始，正式进入SmartNews（仿今日头条app）的思路梳理（重点是架构）

----------------------------------

## 架构

该项目采用组件化+MVP+Retrofit+RxJava的总体架构，并结合多种设计模式，包括单例、简单工厂、策略、代理、观察者等。


架构如图：

![](http://ogsbxb571.bkt.clouddn.com/%E8%B7%AF%E7%94%B1-1.png)

app 为空壳，依赖news和通用库，主要功能是为路由提供全局注册。在这个架构体系中，遇到一个问题那就是不同module自成一体，彼此隔离，都有唯一的Application ,但是在同一个apk内，系统只能加载一个Application文件，而忽略其他。这个问题当时把我难住了，经过研究发现通过Application之间的继承可以解决这个问题，于是基类便是CommonLibraryApplication 其继承了路由的Application类，所有的模块都要依赖CommonLibrary，所以如果有多个模块，可以进行链式继承。

CommonLibrary:

作为通用依赖库，我们把公用部分，例如数据库实体类、自定义控件、三方库、mvp、工具类、等资源放进来，这样上层模块就能共用一套资源而互不影响。数据实体bean放在这里的原因是，GreenDao自动生成 DaoMaster、DaoSession、xxxDao类，统一起来后，就仅生成一套，不会多次生成，方便管理。

![](http://a3.qpic.cn/psb?/V11eJGIx1VE7bR/TDqSsH5LN91.w4yY4CGDBSlOlsGhpOlFAsqjx*HOhGc!/m/dGYBAAAAAAAAnull&bo=gAKNAgAAAAADBy8!&rf=photolist&t=5)

news模块：

![](http://a3.qpic.cn/psb?/V11eJGIx1VE7bR/UfMbaUeNU86H70fj8lMCO2K7Ma18lzrkXeJHohbKRVY!/m/dMYAAAAAAAAAnull&bo=gAKIAgAAAAADByo!&rf=photolist&t=5)


#### 聚合与分离：

组件化的其中一个特点就是可以分模块测试、调试、运行、发布，聚合起来后又以library的形式供上层依赖。我是这么做的，CommonLibrary不管是什么模式，都是要被依赖的，所以它一直以Library形式存在。而news模块则需要一个条件在两种模式下进行切换。

在其gradle文件中加入如下代码：

![](http://a2.qpic.cn/psb?/V11eJGIx1VE7bR/.dLtnu4SrUh0oOnJmXa4crjXT2G.gytuSMl7CFguBAo!/m/dGEBAAAAAAAAnull&bo=pgMkAQAAAAADB6I!&rf=photolist&t=5)

![](http://a2.qpic.cn/psb?/V11eJGIx1VE7bR/3*a8GvhxWoeSoViHfjYf4qMlYDpZ0KqsT8WSQfYHv5w!/m/dGEBAAAAAAAAnull&bo=UQWAAgAAAAADB*Q!&rf=photolist&t=5)

不同模式下获取的资源有所不同。

#### 库文件的统一管理：

不同模块独立运行的话，每个模块都会依赖很多三方库，重复依赖问题严重，那我把它们统一放到CommonLibrary不就行了。但是问题又来了，库多起来之后，视觉上的可读性会受影响，版本号查看、修改比较费力。所以我这样做：

在根目录下新建gradle,之后添加如下代码：

![](http://a4.qpic.cn/psb?/V11eJGIx1VE7bR/a3FDDH6cw1McqZRxZ.pNSoLBrB8pwZ4z.GVkrYvRpl0!/m/dJMAAAAAAAAAnull&bo=qAZcAgAAAAADB9I!&rf=photolist&t=5)

![](http://a4.qpic.cn/psb?/V11eJGIx1VE7bR/NY70iHpIK1vvenaT3hwnSQP37DE2w0*riesp7uVFSfM!/m/dFcBAAAAAAAAnull&bo=4gSAAgAAAAADB0Y!&rf=photolist&t=5)

![](http://a1.qpic.cn/psb?/V11eJGIx1VE7bR/bOXqORvXviCCHEbMqrOjBSvoBnQngNTEPiGSwHXJFiU!/m/dMgAAAAAAAAAnull&bo=WweAAgAAAAADB*w!&rf=photolist&t=5)

之后，在CommonLibrary的gradle中添加：

![](http://a4.qpic.cn/psb?/V11eJGIx1VE7bR/c96EuiKK9HBaOxoAQYya0JV9m4obz2f0TBB7atl7I1c!/m/dFcBAAAAAAAAnull&bo=1ASAAgAAAAADB3A!&rf=photolist&t=5)

注意：上图一定要使用compile,不要使用 implementation.因为implementation外部模块是无法访问到内部资源的。

#### 路由的使用：

使用路由框架ModularizationArchitecture,之前的文章已经对其进行过介绍。

1.需要在app的Application中将各个模块的provider进行注册，选择app是因为其是最上层且依赖所有模块，这样可以调用所有模块的类。

    public class CustomApplication extends CommonLibraryApplication {
    @Override
    public void initializeAllProcessRouter() {

    }

    @Override
    protected void initializeLogic() {
    registerApplicationLogic("xxx.xxx.com.smartnews",999, NewsApplicationLogic.class);
    }

    @Override
    public boolean needMultipleProcess() {
    return false;
    }


    }
    
2.NewsApplicationLogic.class，以及其对应的provider、action 类都封装在各自相关的模块内，便于调用相关资源。

    public class NewsApplicationLogic extends BaseApplicationLogic {
    @Override
    public void onCreate() {
    LocalRouter.getInstance(mApplication).registerProvider(Constant.NEWS_PROVIDER, new ProviderNews());
    }
    }
    
    
    public class ProviderNews extends MaProvider {
    @Override
    protected void registerActions() {
    registerAction(Constant.QUERY_EQ_HISTORY_KEY_ACTION, new ActionHistoryEqQuery());
    registerAction(Constant.INSERT_HISTORY_KEY_ACTION, new ActionHistoryKeyInsert());
    registerAction(Constant.SHOW_NEWS_ACTIVITY_BY_HISTORY_KEY_ACTION, new ActionJumpToNewsActivityForShowNewsByHistoryKey());
    registerAction(Constant.QUERY_LIKE_HISTORY_KEY_ACTION, new NewsSearchActivity.ActionHistoryLikeQuery());
    registerAction(Constant.CLEAR_HISTORY_KEY_ACTION, new NewsSearchActivity.ActionClearHistoryKey());
    }
    }
    
    public class ActionHistoryKeyInsert extends MaAction {
    @Override
    public boolean isAsync(Context context, HashMap<String, String> requestData) {
    return false;
    }
    
    @Override
    public MaActionResult invoke(Context context, HashMap<String, String> requestData) {
    
    
    if (requestData != null) {
    
    String historyKey = requestData.get(Constant.HISTORY_KEY_NAME);
   
    if (!TextUtils.isEmpty(historyKey)) {
    SearchHistoryModel.insertHistoryKey(historyKey);
    }
    
    return new MaActionResult.Builder()
    .code(MaActionResult.CODE_SUCCESS)
    .build();
    
    
    }
    
    return null;
    }
    }
    
    
3.通信：

在CommonLibrary中建立RouterManager类，所有模块可以通过这个类与目标模块进行通信。目标模块内的Action接收到后将回调invoke方法执行相关逻辑，并返回一个MaActionResult ,router 会将其封装成RouterResponse返回给调用者。这里还要注意如果执行的是异步操作需要将isAsync 方法返回true

    public static RouterResponse executeNewsHistoryKeyInsert(Context context, String key) {
    RouterResponse response = null;

    try {
    response = LocalRouter.getInstance(MaApplication.getMaApplication())
    .route(context, RouterRequest.obtain(context)
    .provider(Constant.NEWS_PROVIDER)
    .action(Constant.INSERT_HISTORY_KEY_ACTION)
    .data(Constant.HISTORY_KEY_NAME, key));

    } catch (Exception e) {
    e.printStackTrace();
    }

    return response;
    }
    
这样就完成了组件化下模块之间的通信。

----------------------------------

## 结束

本篇重点梳理了SmartNews项目的整体架构及实现方式，这三篇文章记录了自己学习并熟练掌握组件化的原由及实现的思路与资源。从整体上梳理一下思路。
    
    


























