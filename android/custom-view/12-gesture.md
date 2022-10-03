# 手势检测 GestureDetector

- 创建 GestureDetector 对象
- onTouchEvent 中，调用 GestureDetector 对象的 onTouchEvent 方法

```java
// 1.创建一个监听回调
SimpleOnGestureListener listener = new SimpleOnGestureListener() {
    @Override public boolean onDoubleTap(MotionEvent e) {
        Toast.makeText(MainActivity.this, "双击666", Toast.LENGTH_SHORT).show();
        return super.onDoubleTap(e);
    }
};

// 2.创建一个检测器
final GestureDetector detector = new GestureDetector(this, listener);

// 3.给监听器设置数据源
view.setOnTouchListener(new View.OnTouchListener() {
    @Override public boolean onTouch(View v, MotionEvent event) {
        return detector.onTouchEvent(event);
    }
});
```

通常传入 SimpleOnGestureListener 作为回调。

- OnContextClickListener 主要是用于检测外部设备按钮的，关于它需要注意一点，如果侦听 onContextClick(MotionEvent)，则必须在 View 的 onGenericMotionEvent(MotionEvent)中调用 GestureDetector 的 OnGenericMotionEvent(MotionEvent)。

- 如果你只想监听双击事件，那么只用关注 onDoubleTap 就行了，如果你同时要监听单击事件则需要关注 onSingleTapConfirmed 这个回调函数。

- 关于 onSingleTapConfirmed 原理也非常简单，这一个回调函数在单击事件发生后300ms后触发(注意，不是立即触发的)，只有在确定不会有后续的事件后，既当前事件肯定是单击事件才触发 onSingleTapConfirmed，所以在进行点击操作时，onDoubleTap 和 onSingleTapConfirmed 只会有一个被触发，也就不存在冲突了。

- 关于 onDoubleTapEvent，是在第二次手指按下时，触发 onDoubleTap 以及 onDoubleTapEvent 事件，假如说我们想要在第二次按下抬起后才判定这是一个双击操作，触发后续的内容，则不能使用 onDoubleTap 了，需要使用 onDoubleTapEvent 来进行更细微的控制，如下：

```java
final GestureDetector detector = new GestureDetector(MainActivity.this, new GestureDetector.SimpleOnGestureListener() {
    @Override public boolean onDoubleTap(MotionEvent e) {
        Logger.e("第二次按下时触发");
        return super.onDoubleTap(e);
    }

    @Override public boolean onDoubleTapEvent(MotionEvent e) {
        switch (e.getActionMasked()) {
            case MotionEvent.ACTION_UP:
                Logger.e("第二次抬起时触发");
                break;
        }
        return super.onDoubleTapEvent(e);
    }
});
```

- onDown 保证控件拥有消费事件的能力，接手后续事件

- onFling 快速滑动

```java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float
        velocityY) {
    return super.onFling(e1, e2, velocityX, velocityY);
}
```

- onLongPress
- onScroll
- onShowPress 用户按下时的回调，用于给用户提供一种视觉反馈

单击触发顺序

- onSingleTapUp 单击抬起
- onClick 单击事件
- onSingleTapComfirmed 有延时，单击确认

双击的触发顺序

- onSingleTapUp
- onClick
- onDoubleTap // <- 双击
- onClick

## ScaleGestureDetector

SimpleOnScaleGestureListener

- onScaleBegin 只调用一次，手指 down 时触发 如果返回 false 表示不使用当前缩放手势
- onScale 会触发多次，返回 true，表示当前缩放已被处理，会重新积累缩放因子，返回 false，会累计缩放因子
- onScaleEnd 手势结束

示例：

```java
public class ScaleGestureDemoView extends View {
    private static final String TAG = "ScaleGestureDemoView";

    private ScaleGestureDetector mScaleGestureDetector;

    public ScaleGestureDemoView(Context context) {
        super(context);
    }

    public ScaleGestureDemoView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        initScaleGestureDetector();
    }

    private void initScaleGestureDetector() {
        mScaleGestureDetector = new ScaleGestureDetector(getContext(), new ScaleGestureDetector.SimpleOnScaleGestureListener() {
            @Override
            public boolean onScaleBegin(ScaleGestureDetector detector) {
                return true;
            }

            @Override
            public boolean onScale(ScaleGestureDetector detector) {
                Log.i(TAG, "focusX = " + detector.getFocusX());       // 缩放中心，x坐标
                Log.i(TAG, "focusY = " + detector.getFocusY());       // 缩放中心y坐标
                Log.i(TAG, "scale = " + detector.getScaleFactor());   // 缩放因子
                return true;
            }

            @Override
            public void onScaleEnd(ScaleGestureDetector detector) {
            }
        });
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mScaleGestureDetector.onTouchEvent(event);
        return true;
    }
}
```
