# LayoutMechanism

## LayoutInflater

获取 LayoutInflater 实例

`LayoutInflater inflater = LayoutInflater.from(context);`

`LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);`

public View inflate(int resource, ViewGroup root)

public View inflate(XmlPullParser parser, ViewGroup root)

public View inflate(int resource, ViewGroup root, boolean attachToRoot)

public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot)

使用 pull 解析 xml，通过反射方式创建出 view 实例

root 为 null，attachToRoot 将不起作用

root 不为 null，attachToRoot 为 true，会给加载的布局文件指定一个父布局，即 root

root 不为 null，attachToRoot 为 false，root 只用来为 xml 中的根节点创建正确的 LayoutParams 子类，当将其添加到父 view 中时，这些属性会生效

不设置 attachToRoot 参数时，如果 root 不为 null，attachToRoot 默认为 true

返回 rootView，如果 root 不为 null 并且 attachToRoot 是 true，那么返回 root，否则是 xml 的根节点 view

## 绘制流程

View 和 ViewGroup 基本相同，只是在 ViewGroup 中不仅需要绘制自己，还要绘制子 View

onMeasure() -> onLayout() -> onDraw()

onMeasure() 测量自己的大小，为正式布局提供建议，至于用不用，依据 onLayout

onlayout() 使用 layout() 函数，对所有子控件布局

onDraw() 根据布局位置绘图

## 测量 Measure

### MeasureSpec

是一个 int 值，高2位代表测量模式，低30位代表测量大小

测量模式：

- `UNSPECIFIED`未指定模式，View 想要多大就多大，没有任何限制，比较少见，用处也比较少
- `EXACTLY`表示父 View 希望子 View 的大小是由 specSize 来决定的（`MATCH_PARENT`使用的是此模式）
- `AT_MOST`表示子 View 最多只能是 specSize 的大小，是设置了`WRAP_CONTENT`

默认`WRAP_CONTENT`和`WRAP_CONTENT`的 specSize 是 父 View 大小

顶层的 DecorView 测量的 MeasureSpec 是由 ViewRootImpl 中的 getRootMeasureSpec 方法确定的，宽高为`MATCH_PARENT`，大小为 windowSize)

### onMeasure()

`protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)`

默认 `AT_MOST` 和 `EXACTLY` 都是直接返回 specSize 进行设置

最终调用 setMeasuredDimension() 来设定测量出的大小

只有 setMeasuredDimension() 调用后，才能使用 getMeasuredWidth() 和 getMeasuredHeight()，否则返回值为0

只要是 ViewGroup 子类，就必须要求 LayoutParams 继承 MarginLayoutParams，否则无法使用`layout_margin`参数

#### ViewGroup 的测量

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int measureWidth = MeasureSpec.getSize(widthMeasureSpec);
    int measureHeight = MeasureSpec.getSize(heightMeasureSpec);
    int measureWidthMode = MeasureSpec.getMode(widthMeasureSpec);
    int measureHeightMode = MeasureSpec.getMode(heightMeasureSpec);
    int height = 0;
    int width = 0;
    int count = getChildCount()
    for (int i=0;i<count;i++) {
        //测量子控件
        View child = getChildAt(i);
        measureChild(child, widthMeasureSpec, heightMeasureSpec);
        //获得子控件的高度和宽度
        MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
        int childHeight = child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin;
        int childWidth = child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin;
        //得到最大宽度，并且累加高度
        height += childHeight;
        width = Math.max(childWidth, width);
    }
    setMeasuredDimension((measureWidthMode == MeasureSpec.EXACTLY) ? measureWidth: width, (measureHeightMode == MeasureSpec.EXACTLY) ? measureHeight: height);
}
```

## 布局 Layout

ViewRoot 默认调用 `layout(0, 0, measuredWidth, measuredHeight)`，会调用 onlayout()

View 的 onLayout() 是空方法，ViewGroup 的 onLayout() 是抽象方法，ViewGroup 的子类必须实现该方法来实现对子 View 的布局

onLayout() 方法调用后，就可以调用 getWidth() 和 getHeight()，相当于 right - left、bottom - top

```java
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int top = 0;
    int count = getChildCount();
    for (int i=0;i<count;i++) {
        View child = getChildAt(i);

        MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
        int childHeight = child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin;
        int childWidth = child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin;
        child.layout(0, top, childWidth, top + childHeight);
        top += childHeight;
    }
}
```

ViewRoot 调用 layout() 时，会设置自己的位置`setFrame(l, t, r, b)`，设置结束后调用 onLayout 方法来设置所有子 View 位置

## LayoutParam

Container 在初始化子控件时，会调用 generatelayoutParams() 来为子控件生成对应的布局属性，默认是调用 generateDefaultLayoutParams()，是没有 margin 的，只有宽高值

重写 generatelayoutParams()，返回 ViewGroup.MarginLayoutParams，这样就可以使用 margin 参数，其原理为，从 xml 读取对应的 margin 值，默认 layoutParam 只读取宽高值

```
@Override
protected LayoutParams generateLayoutParams(LayoutParams p) {
    return new MarginLayoutParams(p);
}

@Override
public LayoutParams generateLayoutParams(AttributeSet attrs) {
    return new MarginLayoutParams(getContext(), attrs);
}

@Override
protected LayoutParams generateDefaultLayoutParams() {
    return new MarginLayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
}
```

### 自定义 LayoutParams

```
public static class XLayoutParams extends ViewGroup.LayoutParams{
  public int left = 0;
  public int top = 0;
  public int right = 0;
  public int bottom = 0;

  public XLayoutParams(Context context, AttributeSet attrs){
    super(context, attrs);
  }

  public XLayoutParams(int arg0, int arg1){
    super(arg0, arg1)
  }

  public XLayoutParams(LayoutParams p){
    super(p);
  }
}
```

在 onMeasure 中计算的值，可以保存在新增的 left top right bottom 中，在 onLayout 中直接使用

还需要重写一个方法

```
@Override
protected boolean checkLayoutParams(ViewGroup.LayoutParams p) {
    return p instanceof WaterfallLayoutParams;
}
```

## View 状态

- enabled 表示是否可用，不可用的 view 无法响应 onTouch 事件
- focused 表示是否可以获得焦点，从而可以通过上下左右键切换，以及调用 requestFocus() 方法。requestFocus()获取焦点的时候，一般只有 focusable 和 focusable in touch mode 同时成立的情况下才能获取焦点，比如 EditText
- window_focus 表示当前视图处于正在交互的窗口中，程序无法改变
- selected 表示是否处于选中状态
- pressed 表示是否处于按下状态，通常由系统设置，也可以程序控制

## View 重绘

调用 setVisibility() setEnabled() setSelected 等方法时，都会导致 View 重绘。调用 invalidate() postInvalidate() 可以手动重绘

调用 invalidate() 实质是层层上传到父级，传递到 ViewRootImpl 触发 scheduleTraversals，整个 view 树开始重绘

setContentView() 调用 addView()，ViewGroup 的 addView 会调用 requestLayout() 和 invalidate(true)

requestLayout() 会调用 measure 和 layout 过程，不会调用 draw 过程，也不会重绘任何 view 包括调用者本身

invalidate 绘制时会判断一些标记，确定需要绘制的 view
