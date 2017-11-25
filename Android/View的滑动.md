# View的滑动

在Android设备上，滑动几乎是应用的标配，不管是下拉刷新还是SlidingMenu？它们的基础都是滑动，从另外一方面来说，Android 手机由于屏幕比较小，为了给用户呈现更
多的内容，就需要使用滑动来隐藏和显示一些内容。基于上述两点，可以知道，清动在Androd开发中具有很重要的作用，不管一些滑动效果多么绚丽，归根结底，它们都是由不
同的滑动外加一些特效所组成的。因此，掌握滑动的方法是实现绚丽的自定义控件的基础。

通过三种方式可以实现View的滑动：
 * 第一种是通过View本身提供的scrollTo/scrollBy方法来实现滑动；
 * 第二种是通过动画给View施加平移效果来实现滑动；
 * 第三种是通过改变View的LayoutParams使得View重新布局从而实现滑动。
从目前来看，常见的滑动方式就这么三种，下面一一进行分析。

## 使用scrollTo/scrollBy

为了实现View的滑动，View提供了专门的方法来实现这个功能，那就是scrollTo和scrollBy。让我们先看看这两个方法的具体实现：
```

public void scrollTo(int x,int y){
    if(mScrollX!=x || mScrollY!=y)(
        int oldx=mScrollX；
        int oldY=mScrollY；
        mScrollx=x；
        mScrollY=y；
        invalidateParentcaches(）；
        onScrollChanged(mScrol1x,mScrollY,oldx,oldx)；
        if(!awakenScrol1Bars())(
            postInvalidateonAnimation()；
        }
    }
}

public void scrollBy(int x,int y)(
    scro11To(mScrol1x+x,mScrollY+y);
}


```

其中scrollBy的内部是通过调用scrollTo来实现的它实现了基于当前位置的相对滑动，而scrollTo则实现了基于当前位置在所传递参数的绝对滑动，这个不难理解。利用scrollTo和
scrolBy来实现View的滑动，这不是一件困难的事，但是我们要明白滑动过程中View内部的两个属性mScrollX和mScrollY的改变规则，这两个属性可以通过 getScrollX和getScrollY
方法分别得到。这里先简要概况一下：在滑动过程中，mScrollX的值总是等于View左边缘和View内容左边缘在水平方向的距离，而mScrollY的值总是等于View上边缘和View内容上边缘
在竖直方向的距离。View边缘是指View的位置，由四个顶点组成，而View内容边缘是指View中的内容的边缘，**scrollTo和scrollBy只能改变View内容的位置而不能改变View在布局中的
位置**。mScrollX和mScrollY的单位为像素，并且当**View左边缘在View内容左边缘的右边时，mScrollX为正值**，反之为负值；当**View上边缘在View内容上边缘的下边时，
mScrolY为正值**，反之为负值。换句话说，如果从左向右滑动，那么mScrollx为负值，反之为正值；如果从上往下滑动，那么mScrollY为负值，反之为正值。


## 使用动画

使用动画我们能让一个View进行平移，而平移就是一种滑动。使用动画来移动View，主要是操作View的tranglationX和tranglationY属性，既可以采用传统的View动画来移动View,也可以
采用属性动画。

* View动画
```
<?xml version=“1.0”encoding="utf-8"?>
<set xmins:android="http://schemas.android.com/apk/res/android"
    android:fil1After="true"
    android:zAdjustment="normal">
    <translate 
        android:duration=*100
        android：fromxDelta="0"
        android:fromYDelta="0"
        android:interpolatox="@android:anim/linear_interpolator"
        android:toxDelta="100"
        android：toYDelta="100”/>
</set>
```
此动画可以在100ms内将一个View从原始位置向右下角移动100个像素。

* 属性动画
```
ObjectAnimator.ofFloat(targetView,"translationx",0,100).setDuration(100）.start();
```
此动画同样可以在100ms内将一个View从原始位置向右下角移动100个像素。

上面简单介绍了通过动画来移动View的方法，使用动画来做View的滑动需要注意一点，**View动画是对View的影像做操作，它并不能真正改变View的位置参数，包括宽/高**，
并且如果希望动画后的状态得以保留还必须将fillAfter属性设置为true，否则动画完成后其动画结果会消失。比如我们要把View向右移动100像素，如果fillAfter为false，
那么在动画完成的一刹那，View会瞬间恢复到动画前的状态：如果fillAfter为true，在动画完成后，View会停留在距原始位置100像素的右边。使用属性动画并不会存在上述问题。
上面提到View动画并不能真正改变View的位置，这会带来一个很严重的问题。试想一下，比如我们通过View 动画将一不Buton向右移动100px，并且这个View 设置的有单击事件，
然后你会惊奇地发现，单击新位置无法触发onClick事件，而单击原始位置仍然可以触发onClick事件，尽管Buton已经不在原始位置了。这个问题带来的影响是致命的，但是它却又
是可以理解的，因为不管Button怎么做变换，但是它的位置信息（四个顶点和宽/高)并不会随着动画而改变，因此在系统眼里，这个Buton并没有发生任何改变，它的真身仍然在原始
位置。真身仍然在原始位置。在这种情况下，单击新位置当然不会触发onClick事件了，因为Buton的真身并没有发生改变，在新位置上只是View的影像而已。基于这一点，我们不能
简单地给一个View做平移动面并且还希望它在新位置继续触发一些单击事件。从Android3.0开始，使用属性动画可以解决上面的问题，但是大多数应用都需要兼容到Android2.2,在
Android2.2上无法使用属性动画，因此这里还是会有问题。那么这种问题难道就无法解决了吗?也不是的，虽然不能直接解决这个问题，但是还可以间接解决这个问题，这里给出一个
简单的解决方法。针对上面View动画的问题，我们可以在新位置预先创建一个和目标Button一模一样的Buton，它们不但外观一样连onClick事件也一样。当目标Buton完成平移动画后
就把目标Buton隐藏，同时把预先创建的Buton显示出来，通过这种间接的方式我们解决了上面的问题。这仅仅是个参考，面对这种问题时读者可以灵活应对。

## 改变布局参数
这里我们介绍第三种实现View滑动的方法，那就是改变布局参数，即改变LayoutParams。这个比较好理解了，比如我们想把一个Button向右平移100px，我们只需要将这个Button
的LayoutParams里的marginLeft参数的值增加l00px即可，是不是很简单呢?还有一种情形，为了达到移动Buton目的，我们可以在Buton的左边放置一个空的View，这个空View的
默认宽度为0，当我们需要向右移动Button时，只需要重新设置空View的宽度即可，当空View的宽度增大时（假设Button的父容器是水平方向的LinearLayout），Button就自动被
挤向右边，即实现了向右平移的效果。如何重新设置一个View的LayoutParams呢?很简单，如下所示。
```
    MarginLayoutParams params = (MarginLayoutParams)mButton1.getLayoutParams();
    params.width +=100；
    params.leftMargin +=100；
    mButtonl.requestLayout()；
    //或者mButton1,setLayoutParams(params)；`
```
通过改变LayoutParams的方式去实现View的滑动同样是一种很灵活的方法，需要根据不同情况去做不同的处理。

## 各种滑动方式的对比
上面分别介绍了三种不同的滑动方式，它们都能实现View的滑动，那么它们之间的差别是什么呢?

* scrollTo/scrollBy：操作简单，适合对View内容的滑动：
* 动画：操作简单，主要适用于没有交真的View和实现复杂的动画效果：
* 改变布局参数：操作稍微复杂，适用于有交互的View。
