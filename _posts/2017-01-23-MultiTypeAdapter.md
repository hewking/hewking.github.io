---
layout: post
title:  "MultiTypeAdater解析"
crawlertitle: "MultiTypeAdater解析"
summary: "MultiTypeAdater解析"
date:   2016-06-28 23:09:47 +0700
categories: posts
tags: 'Android Source'
author: hewking
---
> 如作者所说，MultiType 就是一个多类型列表视图的中间分发框架，它能帮助你快速并且清晰地开发一些复杂的列表页面。它本是为聊天页面开发的，聊天页面的消息类型也是有大量不同种类，并且新增频繁，而 MultiType 能够轻松胜任，代码模块化，随时可拓展新的类型进入列表当中。它内建了 类型 - View 的复用池系统，支持 RecyclerView，使用简单灵活，令代码清晰、拥抱变化。

先上 类图

![类图.png](http://upload-images.jianshu.io/upload_images/1394860-53563be176d5a5b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MultiTypeAdapter作为维护多item，作为管理item 得角色，需要实现两个接口：
- TypePool
-  FlatTypeAdapter