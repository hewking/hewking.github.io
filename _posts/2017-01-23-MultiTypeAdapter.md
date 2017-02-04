---
layout: post
title:  "MultiTypeAdater解析"
crawlertitle: "MultiTypeAdater解析"
summary: "MultiTypeAdater解析"
date:   2016-06-28 23:09:47 +0700
categories: posts
tags: 'AndroidSource'
author: hewking
---
> 如作者所说，MultiType 就是一个多类型列表视图的中间分发框架，它能帮助你快速并且清晰地开发一些复杂的列表页面。它本是为聊天页面开发的，聊天页面的消息类型也是有大量不同种类，并且新增频繁，而 MultiType 能够轻松胜任，代码模块化，随时可拓展新的类型进入列表当中。它内建了 类型 - View 的复用池系统，支持 RecyclerView，使用简单灵活，令代码清晰、拥抱变化。

先上 类图

![类图.png](http://upload-images.jianshu.io/upload_images/1394860-53563be176d5a5b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MultiTypeAdapter作为维护多item，作为管理item 得角色，需要实现两个接口：
- TypePool
-  FlatTypeAdapter

并实现两个接口中方法。本类中TypePool 的实现使用了代理模式，构造方法中new MultiTypePool。如下图

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1394860-ebbb7bfef49e7349.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MultiTypePool 用来管理所用到的 view 类型及数据类型。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1394860-43cda4bf69e2e02f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

contents 保存所用到的数据类型，为.class 类型。providers 保存ItemViewProvider的实现类。通过TypePool 的register的实现填充数据。
      在MultiTypeAdapter 中实现TypePool方法indexOf 判断是否存在指定数据类型。通过

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1394860-b691d5195084e65a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

getItemViewType 中结合indexOf ，OnFlattenClass 获取类型，即register多少数据类型也会有对应多少种class 类型。从而保证在onCreateViewHolder有足够的数据类型可用。
在onCreateViewHolder方法中调用了getProviderByIndex 获取ItemViewProvider,再调用onCreateViewHolder。通过数据类型也就是index。再在onBindViewHolder 中通过getProviderByClass 获取ItemViewProvider，再调用相应的ItemViewProvider 的onBindVIewHolder。通过分析得知，MultiTypePool 通过 MultiTypeAdapter  MultiTypePool ItemViewProvider 实现了view 数据类型 以及两者得管理 MultiTypeAdapter 。再MultiTypeAdapter 通过代理模式 得到MultTypePool ,管理不同得itemviewtype 以及 相应得 viewholder。从这里可以体现本库时非常灵活得。主要得 onCreateViewHolder 以及 onBindViewHolder，需要在ItemViewProvider中实现。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1394860-0765b1212fe4331e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从此可以清晰的认知到，通过ItemViewProvider 实现类，需要实现onCreateViewHolder,OnBindViewHolder。类型的关系绑定，已经MultiTypeAdatper，MultiTypePool 实现。我们仅关注，item的实现既可。

 ##总结
    本库是一个非常灵活简洁的RecyclerView 多类型封装库。能够有效拓展不通过修改Adapter,非常方便的能够增删item类型，代码优雅简洁。拓展性强，非常具有实用性