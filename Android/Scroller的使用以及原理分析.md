# Scroller对象
  弹性滑动对象，用于实现View的弹性滑动。我们知道，当使用View的scrollTo/scrollBy方法来进行滑动时，其过程是瞬间完成的，这种没有过渡效果的滑动用户体验是
  非常差的。而Scroller可以用来实现具有过渡效果的滑动，因为它不是在一瞬间完成的，而是在一定的时间间隔内完成的。

  ## Scroller使用的一般步骤
  * 创建Scroller的实例 
  * 调用startScroll()方法来初始化滚动数据并刷新界面 
  * 重写computeScroll()方法，并在其内部完成平滑滚动的逻辑 
  
  ## Scroller典型的使用方法
  ```java
  private Scroller mScroller;
  
  private void smoothScrollTo(int fx, int fy){
        int scrollX = getScrollX();
        int dx = fx - scrollX;
 
        int scrollY = getScrollY();
        int dy = fy - scrollY;
        //设置mScroller的滚动偏移量
        mScroller.startScroll(scrollX, scrollY, dx, dy, Math.abs((dx)*3));
 
        invalidate();
    }
    
    @Override
    public void computeScroll() {
        //先判断mScroller滚动是否完成
        if(mScroller.computeScrollOffset()){
            //这里调用View的scrollTo()完成实际的滚动
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            //必须调用该方法，否则不一定能看到滚动效果
            postInvalidate();
        }
        super.computeScroll();
    }
  ```
  ## 分析Scroller实现滑动效果的机制
  * 我们先来看一看startScroll()的源码
  ```java
  public void startScroll(int startX, int startY, int dx, int dy, int duration) {
  // mMode 分两种方式 1.滑动:SCROLL_MODE 2. 加速度滑动:FLING_MODE
  mMode = SCROLL_MODE;
  // 是否滑动结束 这里是开始所以设置为false
  mFinished = false;
  // 滑动的时间
  mDuration = duration;
  // 开始的时间
  mStartTime = AnimationUtils.currentAnimationTimeMillis();
  // 开始滑动点的X坐标
  mStartX = startX;
  // 开始滑动点的Y坐标
  mStartY = startY;
  // 最终滑动到位置的X坐标
  mFinalX = startX + dx;
  // 最终滑动到位置的Y坐标
  mFinalY = startY + dy;
  // X方向上滑动的偏移量
  mDeltaX = dx;
  // Y方向上滑动的偏移量
  mDeltaY = dy;
  // 持续时间的倒数 最终用来计算得到插值器返回的值
  mDurationReciprocal = 1.0f / (float) mDuration;
}
```
看到这里，大家会发现，startScroll()方法并没有去实现真正的滑动逻辑，它只是完成了一部分变量的保存。这个时候你应该会很奇怪，Scroller到底是怎么让View
滑动起来的呢？你仔细看看在mScroller调用startScroll()的下面，还有一个invalidate()方法，对，就是它，View的滑动就是从这里开始的。invalidate()方法会
导致View重绘，View的重绘过程会调用draw()方法，而draw又会去调用computeScroll，computeScroll在View中是一个空的实现，所以在使用的时候，我们需要自己
去完成具体的逻辑。上面Scroller的典型使用方法中我们已经实现了computeScroll方法，也正是因为它的存在，View才能够进行有过渡效果的滑动。这个时候你是不是
有点不可思议，没事，接下来，我就来给你分析分析这个过程。invalidate-->draw(View的重绘-)-->computeScroll(从Scroller中获取当前的scrollerX、
scrollerY(父视图中的content所滑动到的坐标)--->通过scrollTo来实现滑动)-->postInvalidate(第二次重绘)-->computeScroller()-->postInvalidate...直
到computeScroller中由mScroller.computeScrollOffset()判断得到的结果为false，则滑动过程结束，结束重绘。
  我们来看看mScroller.computeScrollOffset()中做了什么工作：
```java
public boolean computeScrollOffset() {
        if (mFinished) {
            return false;
        }

        int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);

        if (timePassed < mDuration) {
            switch (mMode) {
                case SCROLL_MODE:
                    ...
                    break;
                case FLING_MODE:
                    // 当前已滑动的时间与总滑动时间的比值
                    final float t = (float) timePassed / mDuration;
                    final int index = (int) (NB_SAMPLES * t);
                    // 距离系数
                    float distanceCoef = 1.f;
                    // 加速度系数
                    float velocityCoef = 0.f;
                    if (index < NB_SAMPLES) {
                        final float t_inf = (float) index / NB_SAMPLES;
                        final float t_sup = (float) (index + 1) / NB_SAMPLES;
                        final float d_inf = SPLINE_POSITION[index];
                        final float d_sup = SPLINE_POSITION[index + 1];
                        velocityCoef = (d_sup - d_inf) / (t_sup - t_inf);
                        distanceCoef = d_inf + (t - t_inf) * velocityCoef;
                    }

                    // 计算出当前的加速度
                    mCurrVelocity = velocityCoef * mDistance / mDuration * 1000.0f;
                    // 计算出当前的mCurrX 与mCurrY
                    mCurrX = mStartX + Math.round(distanceCoef * (mFinalX - mStartX));
                    // Pin to mMinX <= mCurrX <= mMaxX
                    mCurrX = Math.min(mCurrX, mMaxX);
                    mCurrX = Math.max(mCurrX, mMinX);

                    mCurrY = mStartY + Math.round(distanceCoef * (mFinalY - mStartY));
                    // Pin to mMinY <= mCurrY <= mMaxY
                    mCurrY = Math.min(mCurrY, mMaxY);
                    mCurrY = Math.max(mCurrY, mMinY);

                    // 如果到达了终点 则结束
                    if (mCurrX == mFinalX && mCurrY == mFinalY) {
                        mFinished = true;
                    }

                    break;
            }
        }
        else {
            mCurrX = mFinalX;
            mCurrY = mFinalY;
            mFinished = true;
        }
        return true;
}
```
   从中我们发现，这个方法通过时间流逝的多少与完成这个滑动过程的总时间计算得到了一个百分比，然后根据这个百分比来计算滑动过程完成的程度，从而计算出当前
的scrollX和scrollY。它返回true表示这个滑动过程还未结束，返回false表示该过程已经结束了。













