# Gesture

## ViewConfiguration

视图标准类

```java
ViewConfiguration vc = ViewConfigurationlget(context);

// 获取 touchSlop 系统所能识别的最小的滑动距离
vc.getScaledTouchSlop();

// fling 速度的最小值和最大值
vc.getScaledMinimunFlingVelocity();
vc.getScaledMaxmumFlingVelocity();

ViewConfiguration.getDoubleTapTimeout()
ViewConfiguration.getLongPressTimeout()
ViewConfiguration.getKeyRepeatTimeout()
```

## GestureDetector

接口：OnGestureListener，OnDoubleTapListener

内部类：SimpleOnGestureListener

### OnGestureListener 实现类

- onDown()
- onShowPress()
- onSingleTap()
- onScroll()
- onLongPress()
- onFling()

### OnDoubleTapListener 实现类

- onSingleTapConfirmed()
- onDoubleTapConfirmed()
- onDoubleTapEvent() 双击间隔中发生的动作。触发了 onDoubleTap 后，在双击之间发生的其他事件

### SimpleOnGestureListener

已经实现两个接口的方法，空实现

### 使用

- 创建 GestureDetecotr.OnGestureListener 实现类

- 创建 GestureDetector

```java
GestureDetector gestureDetector=new GestureDetector(GestureDetector.OnGestureListener listener);
GestureDetector gestureDetector=new GestureDetector(Context context,GestureDetector.OnGestureListener listener);
GestureDetector gestureDetector=new GestureDetector(Context context,GestureDetector.SimpleOnGestureListener listener);
```

- 将 touch 事件交给 GestureDetector 处理，通常在 onTouchEvent 中

`return mGestureDetector.onTouchEvent(event)`

- 默认情况下 onFling 等事件不会被捕捉到，可以设置 view.setLongClickable(true)

- onScroll 是在滑动的时候触发的，onFling 是在手指放开的时候触发的

## VelocityTraker 速度追踪器

开始追踪

```java
private void startTracker(MotionEvent event) {
    if (mVelocityTracker == null) {
        mVelocityTracker = VelocityTracker.obtain();
    }
    // 在 onTouchEvent 的开始，向 velocityTracker 添加滑动事件
    mVelocityTracker.addMovement(event);
}
```

获取追踪到的速度

```java
public int getVelocity() {
    // 设置VelocityTracker单位.1000表示1秒时间内运动的像素
    mVelocityTracker.computeCurrentVelocity(1000);
    // 获取在1秒内X方向所滑动像素值
    int xVelocity = (int) mVelocityTracker.getXVelocity();
    return Math.abs(xVelocity);
}
```

通常在 onDetachedFromWindow 中，解除速度追踪

```java
private void endTracker() {
    if (mVelocityTracker != null) {
        mVelocityTracker.recycle();
        mVelocityTracker = null;
    }
}
```

## ViewDragHelper
