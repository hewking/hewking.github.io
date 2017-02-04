---
layout: post
title:  "自定义CalenderView"
crawlertitle: "自定义CalenderView"
summary: "自定义CalenderView"
date:   2016-11-28 20:09:47 +0700
categories: posts
tags: 'AndroidSource'
author: hewking
---
> 最近开放过程中碰到需求需要有一个类似日历的控件，作为打卡的组件，显示打卡天数，未打等类容

先附上图

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1394860-ffb08a19a2d843b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

作为自定义VIew 首先需要具有相关属性及设置并初始化

```
 // 日历背景
    private int calendarBg = 0xffffffff;

    // 普通字体颜色
    private int mNormalTextColor = 0xff000000;

    // 选中字体的颜色
    private int mBeSelectedTextColor = 0xffffffff;

    // 不可选字体的颜色
    private int mUnBeSelectedTextColor = 0xff999999;

    // 不可选日期的背景
    private int mUnBeSelectedTextBgColor = 0xff000000;

    // 可选日期的背景
    private int mBeSelectedTextBgColor = 0xffff4000;

    // z正常日期背景
    private int mNormalTextBgColor = 0xffffffff;

    // 日期背景半径大小
    private int mBgRadius = 20;
```

并在构造方法中对属性赋值

```
    private void initAttributes(AttributeSet attrs) {
        if (attrs == null) {
            return;
        }
        TypedArray typedArray = getContext().obtainStyledAttributes(attrs, R.styleable.CalendarView);
        mBeSelectedTextBgColor = typedArray.getColor(R.styleable.CalendarView_selected_color, Color.RED);
        mUnBeSelectedTextBgColor = typedArray.getColor(R.styleable.CalendarView_unselected_color, Color.GRAY);
        mNormalTextColor = typedArray.getColor(R.styleable.CalendarView_normaltext_color,Color.BLACK);
        mBeSelectedTextColor = typedArray.getColor(R.styleable.CalendarView_selectedtext_color,Color.BLACK);
        typedArray.recycle();
    }
```

ps : 属性的定义需要在 res values 中的 attrs中设置 如下
```
   <declare-styleable name="CalendarView">
        <attr name="selected_color" format="color"/>
        <attr name="unselected_color" format="color"/>
        <attr name="normaltext_color" format="color"/>
        <attr name="selectedtext_color" format="color"/>
    </declare-styleable>
```
即可在layou.xml布局中使用相应属性。

作为view 需要有宽高
```
    // view的宽度
    private int mViewWidth;
    // view的高度
    private int mViewHeight;
```
及赋值
```
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewHeight = h;
        mViewWidth = w;
    }
```

以上为基本套路，作为自定以view 重头戏 onMeasure onDraw 开始叙述。
本例中因为为基本view 所以对onMeasure 为默认设置，不作自定义实现。

分析题图，本例中需要三行七列的数字从1-21。并需要在每一个数字都需要判断是否设置原型背景，以及数字下方是否设置原点，及颜色。相对比较容易实现

```
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
//        Log.e(TAG, "mViewWidth=" + mViewWidth);
//        Log.e(TAG, "mViewHeight=" + mViewHeight);
        canvas.drawColor(calendarBg);

        int hang_num = 3;
        int lie_num = 7;

        xInterval = mViewWidth / lie_num;
        yInterval = mViewHeight / hang_num;
        //背景圆的半径
        mBgRadius = (int) (Math.min(xInterval, yInterval) / 3);
        mDayTextsize = Math.min(mViewHeight, mViewWidth) / 15;
        mPaint.reset();
        drawDay(canvas);

    }
```
onDraw 首先设置整个view 背景。再设置行列数，以及间隔 xInterval yInterval 背景圆半径 mBgRadius 字体大小 mDayTextsize 
```
    private void drawDay(Canvas canvas) {
        mPaint.setTextSize(mDayTextsize);
        mPaint.setAntiAlias(true);
        float offset;
        float x, y;
        int day = originFirstDay * 21;
        // 绘制的日期
        String str;
        for (int i = 1; i < 4; i++) {
            for (int j = 1; j < 8; j++) {
                day++;
                str = day + "";
//                Log.e(TAG, "string =" + str);
                offset = mPaint.measureText(str);
                x = j * xInterval - xInterval / 2;
                // 因为绘制在第一行
                y = i * yInterval - yInterval / 2;
                drawDayText(x, y, str, mNormalTextColor, isCurrentDay(day), isSignUped(day), canvas, offset);
            }
        }
    }
```
接下来进行绘制天

```
   private void drawDayText(float x, float y, String text, int textColor, boolean isToday, boolean isSignup, Canvas canvas, float offset) {
        mPaint.reset();
        mPaint.setTextSize(mDayTextsize);
        mPaint.setAntiAlias(true);
        mPaint.reset();
        mPaint.setTextSize(mDayTextsize);
        mPaint.setAntiAlias(true);

        Rect lRect = new Rect();
        // 获取text所在rect 的宽高
        mPaint.getTextBounds(text, 0, text.length(), lRect);

        drawDayBg(x, y, isToday, isSignup, canvas, offset, lRect.height() / 2);

        mPaint.setColor(textColor);
        mPaint.setAntiAlias(true);
        if (isToday) {
            mPaint.setColor(mBeSelectedTextColor);
        }
        canvas.drawText(text, x - offset / 2, y, mPaint);
    }
```

```
  private void drawDayBg(float x, float y, boolean isToday, boolean isSignUp, Canvas canvas, float offset, int yOffset) {
        // TODO Auto-generated method stub
        mPaint.setStyle(Paint.Style.FILL);
        if (isToday) {
            if (mCurrentSignUped){
                mPaint.setColor(mBeSelectedTextBgColor);
            }else {
                mPaint.setColor(mUnBeSelectedTextBgColor);
            }
            canvas.drawCircle(x, y - yOffset, mBgRadius, mPaint);
            return;
        }

        if (isSignUp) {
            mPaint.setColor(mBeSelectedTextBgColor);
        } else {
            mPaint.setColor(mUnBeSelectedTextBgColor);
        }
        canvas.drawCircle(x, y + mBgRadius - yOffset, mBgRadius / 5, mPaint);
    }
```
在dispatchTouchEvent 中进行选中某天判断
```
  public boolean dispatchTouchEvent(MotionEvent event) {
        // 获取事件的位置
        float touchX = event.getX();
        float touchY = event.getY();

        switch (event.getAction()) {

            case MotionEvent.ACTION_DOWN:

                // 以下是对日历的事件处理
                int theX = (int) ((touchX) / xInterval);// 获取第几列
                int theY = (int) ((touchY - yInterval / 4) / yInterval);// 获取第几行
//                Log.e("click ds", "第" + theX + "列");
//                Log.e("click ds", "第" + theY + "行");
                int theDay = theY * 7 + theX + 1;
```

