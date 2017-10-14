# Android UI 绘制流程

从源码的角度进行分析(继承Activity)

  ## 布局的加载
  
activity是如何加载我们的布局的--setContentView();

WindowManager接口中定义了Window的类型
    
```java
@see #TYPE_BASE_APPLICATION
@see #TYPE_APPLICATION
@see #TYPE_APPLICATION_STARTING
@see #TYPE_DRAWN_APPLICATION
@see #TYPE_APPLICATION_PANEL
@see #TYPE_APPLICATION_MEDIA
@see #TYPE_APPLICATION_SUB_PANEL
@see #TYPE_APPLICATION_ABOVE_SUB_PANEL
@see #TYPE_APPLICATION_ATTACHED_DIALOG
@see #TYPE_STATUS_BAR
@see #TYPE_SEARCH_BAR
@see #TYPE_PHONE
@see #TYPE_SYSTEM_ALERT
@see #TYPE_TOAST
@see #TYPE_SYSTEM_OVERLAY
@see #TYPE_PRIORITY_PHONE
@see #TYPE_STATUS_BAR_PANEL
@see #TYPE_SYSTEM_DIALOG
@see #TYPE_KEYGUARD_DIALOG
@see #TYPE_SYSTEM_ERROR
@see #TYPE_INPUT_METHOD
@see #TYPE_INPUT_METHOD_DIALOG
```
    
* 然后我们看一看setContentView的实现：
    
```java
@Override
public void setContentView(int layoutResID) {

    installDecor();//初始化DecorView,将系统布局添加到DecorView中，并获取系统布局中的帧布局

    mLayoutInflater.inflate(layoutResID, mContentParent);//将我们的布局放到帧布局当中

}
```

* 其实activity类中的setContentView做了这些事：
setContentView() -->window类的setContentView() -->PhoneWindow的setContentView();

* installDecor()方法的实现
```
        private void installDecor() {
            if (mDecor == null) {
                mDecor = generateDecor(-1);//new 一个DecorView,这个DecorView中包含了一个系统布局，我们开发时所写的布局最终都被添加到这个布局中的帧布局之中
                mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
                mDecor.setIsRootNamespace(true);
                if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                    mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
                }
          } else {
              mDecor.setWindow(this);
          }
          if (mContentParent == null) {
              mContentParent = generateLayout(mDecor);//该方法返回的是一个FrameLayout
          }
       }
```

## 绘制的流程 measure layout draw

* 在PhoneWindow的setContentView()中，调用了ViewGroup的addView()方法,
    
```
    public void addView(View child, int index, LayoutParams params) {

        ...

        requestLayout();
        invalidate(true);
        addViewInner(child, index, params, false);
    }
```
    
可以看到，addView中调用了invalidate()，让我们来看看invalidate方法的实现：

```
    public void invalidate(Rect dirty) {
        final int scrollX = mScrollX;
        final int scrollY = mScrollY;
        invalidateInternal(dirty.left - scrollX, dirty.top - scrollY,
                    dirty.right - scrollX, dirty.bottom - scrollY, true, false);
    }

```
    
可以看到，invalidate中调用了invalidateInternal()，让我们来看看invalidateInternal方法的实现：
    
```
    void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,
               boolean fullInvalidate) {

          ...

          p.invalidateChild(this, damage);//Android中所有的容器都是ViewGroup

          ...

    }
```
    
可以看到，invalidateInternal中调用了invalidateChild()，让我们来看看invalidateChild方法的实现：
    
ViewGroup中的invalidateChild方法：
    
```
       public final void invalidateChild(View child, final Rect dirty) {

           ...

           do {

              通过do-while循环，一次去找父容器，直到父容器为null为止，也就是找到DecorView(最终的父容器)

              (ViewRootImpl类型)parent = parent.invalidateChildInParent(location, dirty);

           } while (parent != null);

       }
```
    
上面的循环中的方法调用大致如下所示：

       invalidateChildInParent()--->invalidate()--->scheduleTraversals()--->mTraversalRunnable(TraversalRunnable类型)--->doTraversal()
       -->performTraversals();

       performTraversals()是ViewRootImpl类中的关键方法

**performTraversals()方法做了三件事：***
* performMeasure() 测量  mView.measure()
* performLayout() 布局   host.layout() host本身就是view
* performDraw() 绘制     mView.draw(canvas);

### View

      MeasureSpec 32位  高两位是mode,低30位是size

* mode的类型
  * View.MeasureSpec.AT_MOST     最大，最多 100dp 最多只能是100dp
  * View.MeasureSpec.EXACTLY     精确的 100dp  就是  100dp
  * View.MeasureSpec.UNSPECIFIED 不确定的


#### draw
* 1. Draw the background
* 2. If necessary, save the canvas' layers to prepare for fading
* 3. Draw view's content
* 4. Draw children
* 5. If necessary, draw the fading edges and restore layers
* 6. Draw decorations (scrollbars for instance)

### ViewGroup
是一个容器  所以需要测量子view 还需要对子view进行摆放
draw --dispatchDraw()--绘制子控件
viewGroup因为有子view,所以复写dispatchDraw()方法  --调用drawChild()-- 调用child.draw()


Android中所有的视图都继承自View或者ViewGroup，如果在自定义控件的时候，我们有可能会直接继承View/ViewGroup,
也有可能继承原生的控件
