---
layout: post
title:  "GestureRulerView GestureDetector与View触摸事件"
crawlertitle: "GestureDetector与View触摸事件"
summary: "GestureDetector与View触摸事件"
date:   2017-10-21 23:09:47 +0700
categories: posts
tags: '自定义View'
author: hewking
---
> 本文所写为GestureDetector与View onTouchEvent 的关系以及源码分析，并给出自定义View 具体示例

以下为通过GestureDetectorCompat 实现的基于触摸反馈滑动选中刻度的尺子。设计来自于薄荷运动
app

class GestureRulerView(context : Context,attrs : AttributeSet) : View(context,attrs) {

    private val mScroller : Scroller by lazy {
        Scroller(context)
    }

    init {
        mScroller
    }

    val gestureDetector  = GestureDetectorCompat(getContext(),object : OnGestureListener
            ,GestureDetector.OnDoubleTapListener
            ,OnContextClickListener{
        override fun onContextClick(v: View?): Boolean {
            return true
        }

        override fun onDoubleTap(e: MotionEvent?): Boolean {
            return true
        }

        override fun onDoubleTapEvent(e: MotionEvent?): Boolean {
            return true
        }

        override fun onSingleTapConfirmed(e: MotionEvent?): Boolean {
            return true
        }

        override fun onShowPress(e: MotionEvent?) {

        }

        override fun onSingleTapUp(e: MotionEvent?): Boolean {
            return true
        }

        override fun onDown(e: MotionEvent?): Boolean {
            return true
        }

        override fun onFling(e1: MotionEvent?, e2: MotionEvent?, velocityX: Float, velocityY: Float): Boolean {
            return true
        }

        /**
         * e1 是滑动序列事件中的 down
         * e2 是滑动序列事件中的 move
         *
         * distanceX 是两次move 的差值 不是 e1 e2 的x 差 distnceY 同理
         */
        override fun onScroll(e1: MotionEvent?, e2: MotionEvent?, distanceX: Float, distanceY: Float): Boolean {
//            if (distanceX > 0) {
//                return true
//            }
            if (scrollX < 0 && distanceX < 0) {
                return false
            }
            scrollBy(distanceX.toInt(),0)
            return true
        }

        override fun onLongPress(e: MotionEvent?) {
        }

    })

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when (event.action) {
            MotionEvent.ACTION_UP -> {

            }
        }
        return gestureDetector.onTouchEvent(event)
//        return true
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)

    }

    /**
     * 滑动到目标点
     */
    private fun smoothTo(distanceX : Int, distanceY : Int) {
        val dx = distanceX - scrollX
        val dy = distanceY - scrollY
        mScroller.startScroll(scrollX,scrollY,dx,dy)
        invalidate()
    }

    override fun computeScroll() {
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.currX,mScroller.currY)
            postInvalidate()
        }
    }

    private var mWidth = 0
    private var mHeight = 0

    private var space : Int = dp2px(10f)
    private var scaleW = dp2px(2f)
    private var scaleH = dp2px(15f)
    private var scaleHLarge = dp2px(30f)

    private val mPaint : Paint by lazy {
        Paint().apply {
            color = Color.GRAY
            style = Paint.Style.STROKE
            strokeWidth = 2f
            isAntiAlias = true
            textSize = dp2px(12f).toFloat()
        }
    }

    private val mCenterPaint : Paint by lazy {
        Paint().apply {
            color = Color.GREEN
            style = Paint.Style.FILL
            strokeWidth = 2f
            isAntiAlias = true
            textSize = dp2px(12f).toFloat() * 2
        }
    }

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        mWidth = w
        mHeight = h
        backRect = Rect(0,0,mWidth,mHeight)
    }

    private var backRect : Rect? = null
    private var smallRect : Rect = Rect()
    private var largeRect : Rect = Rect()
    private var centerRect : Rect = Rect()

    override fun onDraw(canvas: Canvas) {
        mPaint.style = Paint.Style.STROKE

        canvas.drawRect(centerRect.apply {
            top = 0
            bottom = scaleHLarge + scaleH
            left = mWidth / 2 - scaleW + scrollX
            right = mWidth / 2 + scaleW + scrollX
        },mCenterPaint)

        backRect?.apply {
            left = 0
            top = 0
            right = mWidth + scrollX
            bottom = mHeight
        }
        canvas.drawRect(backRect,mPaint)
        // 间距 10
        // 每5 间距 一个中等刻度
        // 在中间为 大刻度
        mPaint.style = Paint.Style.FILL
        for (i in 1..(mWidth + scrollX) / space) {
            val text = i.toString()
            val textW = mPaint.measureText(text)
            if (i % 5 == 0) {
                val half = scaleW / 2
                val tleft = i * space - half
                smallRect.apply {
                    this.left = tleft
                    top = 0
                    right = i * space + half
                    bottom = scaleHLarge
                }
                canvas.drawRect(smallRect,mPaint)
                canvas.drawText(text,tleft - textW / 2,scaleHLarge + mPaint.textHeight() + 10f,mPaint)
            } else {
                val half = scaleW / 2
                val left = i * space - half
                largeRect.apply {
                    this.left = left
                    top = 0
                    right = i * space + half
                    bottom = scaleH
                }
                canvas.drawRect(largeRect,mPaint)
//                canvas.drawText(text,left - textW / 2,scaleH + mPaint.textHeight() + 10f,mPaint)
            }

        }

        /**
         * 画分数
         */
        val textScroe = (centerRect.centerX() / space).toFloat().toString()
        val tScoreWidth = mCenterPaint.measureText(textScroe)
        canvas.drawText(textScroe
                ,centerRect.centerX() - tScoreWidth / 2
                ,scaleHLarge * 2 + mPaint.textHeight() + 10f
                ,mCenterPaint)


    }


}



