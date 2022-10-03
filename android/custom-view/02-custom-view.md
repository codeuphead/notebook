# 自定义 View

通常分类两类

- 自定义 ViewGroup

利用现有组件根据特定的布局方式来组成新的组件，大多数继承自 ViewGroup 或现有的 Layout，通常会含子 View，组合现有的其他 View。

- 自定义 View

不包含子 View 一般继承自 View、SurfaceView 或其他 View。

## 几个重要的函数

### 构造函数

```
public void View(Context context) {}
public void View(Context context, AttributeSet attrs) {}
public void View(Context context, AttributeSet attrs, int defStyleAttr) {}
public void View(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}
```

可用于初始化内容，获取自定义属性

- 通过 new 创建的 View 会调用有一个参数的构造
- 通过 xml 创建的 View 会调用有两个参数的构造

### 测量 onMeasure

View 的大小不仅由自身决定，也会收到父 View 的影响

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthsize = MeasureSpec.getSize(widthMeasureSpec);      //取出宽度的确切数值
    int widthmode = MeasureSpec.getMode(widthMeasureSpec);      //取出宽度的测量模式

    int heightsize = MeasureSpec.getSize(heightMeasureSpec);    //取出高度的确切数值
    int heightmode = MeasureSpec.getMode(heightMeasureSpec);    //取出高度的测量模式
}
```

测量模式有三种，由 View 的一个内部类 View.MeasureSpec 定义：

- UNSPECIFIED 00 默认值，父 View 未指定，子 View 可以设置为任意大小
- EXACTLY 01 父控件指定了子 View 的大小
- AT_MOST 02 子 View 没有尺寸限制，但是有上限，一般情况为父 View 的大小

onMeasure 函数中的两个参数，分别表示了 View 宽高的数值与测量模式，以 int 值表示，前 30 位表示宽高，后两位表示测量模式

如果在 onMeasure 函数中对数值进行了修改，要调用 `setMeasureDimension(widthsize, heightsize)` 这个函数。

### 确定 View 大小 onSizeChange

使用系统提供的回调，获取 View 确定的大小

```
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
}
```

### 确定子 View 布局 onLayout

在 ViewGroup 中用到，调用子 View 的 layout 方法

自定义 ViewGroup 中，一般是循环取出子 View，然后计算得出各子 View 的坐标，然后调用`child.layout(l, t, r, b)` 方法

四个参数分别对应 View.getLeft getTop getRight getBottom 的值，是相对于父 View 的坐标值

### 绘制内容 onDraw

使用 Canvas 绘图

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
}
```

### 对外提供接口