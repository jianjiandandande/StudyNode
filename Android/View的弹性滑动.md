# View的弹性滑动

View的弹性滑动，也就是要实现一种渐进式的滑动效果，给用户更好的体验。

## [使用Scroller](https://github.com/jianjiandandande/StudyNode/blob/master/Android/Scroller%E7%9A%84%E4%BD%BF%E7%94%A8%E4%BB%A5%E5%8F%8A%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.md)


## 使用动画

动面本身就是一种渐近的过程，因此通过它来实现的滑动天然就具有弹性效果，比如以下代码可以让一个View在100ms内向右移动100像素。
```java
    ObjectAnimator.ofFloat (targetView,"translationx",0,100).setDuration(100）.start();
```
不过这里想说的并不是这个问题，我们可以利用动画的特性来实现一些普通的滑动不能实现的效果。还拿scrollTo来说，我们也想模仿 Scroller来实现View的弹性滑动，那么利用动
画的特性，我们可以采用如下方式来实现：
```java
    final int startx-0；
    final int deltax=100；
    ValueAnimator animator = ValueAnimator.oftnt(0,1).setouratlon(1000);
    animator.addUpdateListener(new AninatorUpdatetiatener(){
        @override
        public void onAnimationupdate (ValueAninator aninator){
            float fraction = animator.getAnimatedFraction();
            mButton1.acrol1To(startX+(int)(deltax*fraction),0):
        });
  animator.start(）；
```
在上述代码中，我们的动画本质上没有作用于任何对象上，它只是在1000ms内完成了整个动画过程。利用这个特性，我们就可以在动画的每一帧到来时获取动画完成的比例，
然后再根据这个比例计算出当前View所要滑动的距离。注意，这里的滑动针对的是**View的内容**而非View本身。可以发现，这个方法的思想其实和Scroller比较类似，都是通过改
变一个百分比配合scrollTo方法来完成View的滑动。需要说明一点，采用这种方法除了能够完成弹性滑动以外，还可以实现其他动画效果，我们完全可以在onAnimationUpdate方
法中加上我们想要的其他操作。

## 使用延时策略

延时策略的核心思想是通过发送一系列延时消息从而达到一种渐近式的效果，具体来说可以使用Handler或View的posdlDclayed方法，也可以使用线程的sleep方法。对于postDelayed
方法来说，我们可以通过它来延时发送一个消息，然后在消息中来进行View的滑动，如果接连不断地发送这种延时消息，那么就可以实现弹性滑动的效果。对于sleep方法来说，通过在
while循环中不断地滑动View和sleep，就可以实现弹性滑动的效果。

下面采用Handler来做个示例，其他方法请读者自行去尝试，思想都是类似的。下面的代码在**大约**1000ms内将View的内容向左移动了100像素，代码比较简单，就不再详细介绍了，
之所以说大约1000ms，是因为采用这种方式无法精确地定时，原因是系统的消息调度也是需要时间的，并且所需时间不定。

```java
    prlvate atate final int MESSAGE SCROLL_TO=1;
    private atatic final int FRAME_CoUNT-30;
    private static final int DELAYED_TIME-33;
    private int mCount=0;
    QsuppreaalLint("Handlerteak")
    private Handler mHandler = new Handler(){
        public void handleMesaage(Message msg){

            switch (msg，what){
                case MESSACE SCROLL_TO:{
                    mCount++；
                    if（(nmCount<=ERAMWE_COUNT){
                        float fraction-mCount/(float)PRANMB.coUN7;
                        int scrollX=(int)(fraction*100);
                        mButton1.scrol1To (scr0l1x,0)；
                        mHandler,sendEmptyMessageDelayed(MB88AGE BCROLLTO,DELAYED_TIME);
                    }
                    break;
                }
                default：
                    break;
            }
        }
    }
```
