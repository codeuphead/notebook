# Scroller

## 滚动实现方式

- 不断修改 View 的 LayoutParams
- 使用动画
- 调用 View 的 scrollTo() scrollBy()

### scrollTo() scrollBy()

```java
public void scrollTo(int x, int y) {
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
               postInvalidateOnAnimation();
        }
    }
}
```

```java
public void scrollBy(int x, int y) {
    scrollTo(mScrollX + x, mScrollY + y);
}
```

- scrollTo() 将 View 内容相对于初始位置滚动某段距离
- scrollBy() 在现有基础上将 View 内容移动指定偏移量
- 首先将参数赋值，然后调用 onScrollChanged 回调，然后重绘 View
- mScrollX 和 mScrollY 默认为 0，表示 View 的内容的滚动距离
- 对于 ViewGroup 来说，内容就是其子 View
- 调用这两个方法会导致 View 重绘，重绘过程中会用`“原来的位置” - mScrollX/mScrollY`来计算滚动后的坐标，即 scroll 值为正，View 的坐标会变小，为负，View 的坐标会变大

## Scroller OverScroller

- scrollTo()/scrollBy() 执行动作非常生硬，因此优化滑动活成，可以使用 Scroller
- 是对滑动的封装，可以收集数据进行滑动动画，可以使用插值器
- 会跟踪滑动偏移，但不会自动执行滑动，最终是通过 scrollTo() 完成滑动的

### 使用步骤

- 初始化 scroller

public Scroller (Context context)

public Scroller (Context context, Interpolator interpolator)

- 调用 startScroll()，该方法为 scroll 做一些准备工作，设置坐标时间距离等，该方法不是滑动的开始，startScroll() 比较重要的一点是设置了动画开始时间
- 调用 invalidate() / postInvalidate() 使 view 重绘，重绘会调用 draw()，从而调用 view 的 computeScroll() 方法，该方法默认是一个空方法
- 重写 View 的 computeScroll()，利用 scroller.computeScrollOffset() 判断移动过程是否完成，true 表示没有完成，若没有结束，则调用 `scrollTo(scroller.getCurrX(), mScroller.getCurrY())、scrollBy(scroller.getCurrX(), mScroller.getCurrY())`，实现与滑动相关的业务逻辑
- 在 computeScroll() 方法中，再次执行 invalidate()。这样会循环调用 draw() 重绘 viewTree，直到 computeScrollOffset 返回 false
- computeScrollOffset() 方法主要根据当前已经消逝的时间来计算 view 新的坐标点，保存在 mCurrX 和 mCurrY 中，因此可以调用 scrollTo 来执行对应的滑动。除此之外还能判断动画是否已经结束
- 注意边界判断

```java
public void startScroll(int startX, int startY, int dx, int dy, int duration) {
    mMode = SCROLL_MODE;
    mFinished = false;
    mDuration = duration;
    mStartTime = AnimationUtils.currentAnimationTimeMillis();
    mStartX = startX;
    mStartY = startY;
    mFinalX = startX + dx;
    mFinalY = startY + dy;
    mDeltaX = dx;
    mDeltaY = dy;
    mDurationReciprocal = 1.0f / (float) mDuration;
}
```

```java
@Override
public void computeScroll() {
   super.computeScroll();
   if (mScroller.computeScrollOffset()) {
       scrollTo(mScroller.getCurrX(), 0);
       invalidate();
   }
}
```