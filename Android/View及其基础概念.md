# View及其基础概念

## 什么是View

在介绍Vicw的基础知识之前，我们首先要知道到底什么是View.View是Android中所有控件的基类，不管是简单的Button和TextView还是复杂的RelativeL ayout 和ListView，
它们的共同基类都是View。所以说，View是二种界面层的控件的一种抽象，它代表了一个控件。除了View，还有ViewGroup，从名字来看，它可以被翻译为控件组，言外之意是
ViewGroup内部包含了许多个控件，即一组View.在Android的设计中，ViewGroup 也继承了View，这就意味着Vicw本身就可以是单个控件也可以是由多个控件组成的一组控件。
通过这种关系就形成了View树的结构，这和Web前端中的DOM树的概念是相似的。根据这个概念，我们知道，Buton显然是个View，而LinearLayout不但是一个View而且还是一个
ViewGroup，而ViewGroup内部是可以有子View的，这个子View同样还可以是ViewGroup，依此类推。

## View的位置参数

View的位置主要由它的四个顶点来决定，分别对应于View的四个属性；top、left、right、bottom,其中top是左上角纵坐标，lel是左上角横坐标，right是右下角横坐标，bottom是
右下角纵坐标。需要注意的是，这些坐标都是**相对于View的父容器**来说的，因此它是一种相对坐标。在Android中，x轴和y轴的正方向分别为右和下，这点不难理解，不仅仅是
Android，大部分显示系统都是按照这个标准来定义坐标系的。

### 在视图坐标系中，获取View的top、left、right、bottom

```
    top = getTop();
    left = getLeft();
    right = getRight();
    bottom = getBottom();
```

从Android3.0开始，View增加了额外的几个参数：x、y、translationX和translationY,其中x和y是View左上角的坐标，而translationX和translationY是View左上角相对于父
容器的偏移量。这几个参数也是相对于父容器的坐标，并且translationX和translationY的默认值是0，和View的四个基本的位置参数一样，View也为它们提供了get/sct方法，这
几个参数的换算关系如下所示。

```java
    x = left+tranglationX;
    y = top+tranglationY;
```

要注意的是，View在平移过程中，top和left表示的是原始左上角的位置信息，此值不会发生变化，变化的是x、y、tranglationX、tranglationY这四个参数。

## MotionEvent和TouchSlop

### MotionEvent

在手指接施屏那后所产生的一系列事件中，典型的事件类型有如下几种：

* ACTION_DOWN——手指刚接触屏幕：

* ACTION_MOVE——手指在屏幕上移动；

* ACTION_UP——手指从屏幕上松开的一瞬间.

正常情况下，一次手指触损屏幕的行为会触发一系列点击事件，考虑如下几种情况：

* 点击屏幕后离开松开，事件序列为DOWN>UP；

* 点击屏幕滑动一会再松开，事件序列为DOWN>MOVE>...>MOVE>UP.

上述两种情况是典型的事件序列，同时通过MotionEvent对象我们可以得到点击事件发生的x和y坐标。为此，系统提供了两组方法：getX/getY和getRawX/gelRawY。它们的
区别其实很简单，**getxX/getY**返回的是相对于当前**View左上角**的x和y坐标，而**geiRawX/getRawY**返回的是相对于**手机屏幕左上角**的x和y坐标。

### TouchSlop

TouchSlop是系统所能识别出的被认为是滑动的最小距离，换句话说，当手指在屏幕上潘动时，如果两次滑动之间的距离小于这个常量，那么系统就不认为你是在进行滑动操作。
原因很简单；滑动的距离太短，系统不认为它是滑动。这是一个常量，和设备有关，在不同设备上这个值可能是不同的，通过如下方式即可获取这个常量：ViewConfiguration.
get(getContext()).getScaledTouchSlop()。个常量有什么意义呢？当我们在处理滑动时，可以利用这个常量来做一些过滤，比如当两次滑动事件的滑动距离小于这个值，我们就可以
认为未达到滑动距离的临界值，因此就可以认为它们不是滑动，这样做可以有更好的用户体验。

## VeloctyTracker、GestureDetector和Scroller

### VelocityTracker

速度追踪，用于追踪手指在滑动过程中的速度，包括水平和垂直方向的速度。它的使用过程很简单，省先，在View 的onTouchEvet方法中追踪当前单击事件的速度：
```java
    VelocityTracker velocityTracker = VelocityTracker.obtain();
    velocityTracker.addMovenent(event):
```
接着，当我们想知道当前的滑动速度时，这个时候可以采用如下方式来获得当前的速度:
```java
    velocityrracker.computecurrentVelocity (1000);//计算速度
    int xVelocity = (int)velocityTracker.getXVelocity();//获取速度
    int yVelocity = (int)velocityTracker.getYVelocity();
```
在这一步中有两点需要注意，第一点，**获取速度之前必须先计算速度**，即getXVelocity和getYVelocity这两个方法的前面必须要调用computeCurrentVelocity方法；第二点，这里
的速度是指**一段时间内**手指所滑过的像素数，比如将时间间隔设为1000ms时，在1s内，手指在水平方向从左向右滑过100像素，那么水平速度就是100。注意速度可以为负数，当手指从
右往左滑动时，水平方向速度即为负值，这个需要理解一下。速度的计算可以用如下公式来表示：

速度=（终点位置-起点位置）/时间段

根据上面的公式再加上Android系统的坐标系，可以知道，手指逆着坐标系的正方向滑动，所产生的速度就为负值。另外，computeCurentVelocity这个方法的参数表示的是
个时间单元或者说时间间隔，它的单位是毫秒（ms)，计算速度时得到的速度就是在这时间间隔内手指在水平或竖直方向上所滑动的像素数。针对上面的例子，如果我们通
velocityTracker.computeCurrentVelocity(100)来获取速度，那么得到的速度就是手指在100ms内所滑过的像素数，假设这100ms内滑过的像素值为100，那么水平速度就成了
100像素/每100ms（这里假设滑动过程是匀速的）。
使用完毕之后，记得要调用clear方法来重置并回收内存。

```java
    velocityrracker.clear();//重置
    velocityrracker.recycle();//回收内存
```

### GestureDetector

手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为。要使用GestureDetector也不复杂，参考如下过程。
首先，需要创建一个GestureDetector对象并实现OnGestureListener接口，根据需要我当们还可以实现OnDouble TapListener从而能够监听双击行为：
```
    GestureDetector mGestureDetector-new GestureDetector(this);
    //解决长按屏幕后无法拖动的现象
    nGestureDetector.setIsLongpressEnabled(false);
```
接着，接管目标View的onTouchEvent方法，在待监听View的onTouchEvent方法中
添加如下实现：
```java
    boolean consume=mGestureDetector.onTouchEvent(event);
    return consume;
```
做完了上面两步，我们就可以有选择地实现OnGestureListener 和OnDouble TapListen中的方法了。

### Scroller

弹性滑动对象，用于实观Wew的弹性滑动，我们知道，当使用View的scrollTo/scrollBy方法来进行滑动时，其过程是瞬间完成的，这个没有过渡效果的滑动用户体验不好。这个
时候就可以使用Scroller米实现有过渡效果的滑动，其过程不是瞬间完成的，而是在一定的时间间隔内完成的。Scroller 本身无法让View 弹性滑动，它需要和View 的
computescroll方法配合使用才能共同完成这个功能。那么如何使用Scroller呢?它的典型代码是固定的：
```java
  Scroller scroller = new Scroller(mContext);

  //缓慢滚动到指定位置
  private void smoothscrollTo(int destx,int destY){
      int scrollx = getScrollx()；
      int delta = destx-scrollx;
      //1000ms内滑向destx，效果就是慢慢滑动
      mScroller.startScroll(scrol1x,0,delta,0,1000);
      invalidate();
  }

  @Override
  public void computeScroll(){
      if(mScroller.computeScrollOffset()){
          scrollTo(mScroller.getCurrX(),mScroller.getCurrY());
          postInvalidate();
      }
  }
```
至于它为什么可以实现弹性滑动，请移步到[Sroller原理](https://github.com/jianjiandandande/StudyNode/blob/master/Android/Scroller%E7%9A%84%E4%BD%BF%E7%94%A8%E4%BB%A5%E5%8F%8A%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.md)
