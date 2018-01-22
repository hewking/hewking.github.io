---
layout: post
title:  "ParallaxPageTransfomer视差与旋转"
crawlertitle: "ParallaxPageTransfomer视差与旋转"
summary: "自定义View"
date:   2017-07-21 23:09:47 +0700
categories: posts
tags: '编程基础'
author: hewking
---

> 设计提出一个需求，在导航guide 页面中，能够旋转页面带有角度，同事页面中的元素能够跟随滑动
有视差滚动的效果。

1. 分析
导航滑动目前使用的丝viewpager + fragment ，实现该效果可以通过设置viewpager切换对view 的操作实现。通过 viewpager.setpageTransfomer() 方法实现。需要实现自定义的PagerTransfomer

2. 实现原理
PageTransfomer 有一个方法需要实现
transformPage(page: View, position: Float) 
查看ViewPager 源码可知在onPageScroll()中调用

相关代码片段
```
        if (mPageTransformer != null) {
            final int scrollX = getScrollX();
            final int childCount = getChildCount();
            for (int i = 0; i < childCount; i++) {
                final View child = getChildAt(i);
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();

                if (lp.isDecor) continue;
                final float transformPos = (float) (child.getLeft() - scrollX) / getClientWidth();
                mPageTransformer.transformPage(child, transformPos);
            }
        }
```
通过代码可知 taransformpos 代表的意义。 
具体区间可氛围 (-inflnite ,-1)  [-1,1]  (1，+infinite)
这里仅仅关心[-1,1]区间值 也只对这里值进行操作 这代表着当前相邻两个view 的滑动状态。


3.具体实现
```
class ParallaxPageTransfomer(val resid : Int) : ViewPager.PageTransformer{

    override fun transformPage(page: View, position: Float) {
        if (position < -1) {

        } else if (position > 1) {

        } else {
            // [-1,1]
            val width = page.width // view 宽度
            page.pivotX = width * 0.5f // 设置旋转 x prvotx
            page.pivotY = page.height * 1f // 设置选择 y prvotY
            page.rotation = 30 * position // 旋转度数  根据postion 正负决定 方向

            page.alpha = 1 - Math.abs(position) // 滑动过程中 view alpha 变化
            page.scaleX = 0.7f + 0.3f * (1- Math.abs(position))  // 滑动过程中 沿x轴的缩放比例裱画

            /**
             * 重头戏 以上仅仅针对 view 本身的变换操作。
             * 这里是针对于page 的子view
             * 子view 已经是作为page 的一部分拥有了以上变换。
             * 同时子view 相对page 也有相对平移就实现了视差效果
             */
            page.v<View>(resid).translationX = position * width * 0.5f 
        }

    }

}
```

可以看到代码量不多，这也有kotlin的效果，安利一波kotlin。告别java 人生苦短。
具体代码已经在代码中了这里标识下注释





