# Canvas

## 常用操作

|操作|API|说明|
|---|---|---|
|绘制颜色|drawColor drawRGB drawARGB|-|
|绘制形状|drawPoint drawPoints drawLine drawLines drawRect drawRoundRect drawOval drawCircle drawArc|点、线、矩形、圆角矩形、椭圆、圆、圆弧|
|绘制图片|drawBitmap drawPicture|-|
|绘制路径|drawPath|贝塞尔曲线会用到|
|顶点操作|drawVertices drawBitmapMesh|通过对顶点操作可以是图像变形，drawVertices 直接对画布作用，drawBitmapMesh 只对绘制的 Bitmap 作用|
|画布裁剪|clipPath clipRect|设置画布显示区域|-|
|画布快照|save restore saveLayerXxx restoreToCount getSaveCount|保存状态、恢复上次状态、保存图层、回到指定状态、获取保存次数|
|画布变换|translate scale rotate skew|位移、缩放、旋转、裁切|
|矩阵|getMatrix setMatrix concat|画布的变换，实际都是图像矩阵 Matrix，只不过 Matrix 比较难以理解和使用，因此封装了一些常用的方法|

- 绘制纯色

```
public void drawColor(@ColorInt int color)
public void drawColor(@ColorInt int color, @NonNull PorterDuff.Mode mode)
```

- 绘制点

```
public void drawPoint(float x, float y, @NonNull Paint paint)
public void drawPoints(@Size(multiple = 2) @NonNull float[] pts, @NonNull Paint paint)
```

- 绘制直线

```
public void drawLine(float startX, float startY, float stopX, float stopY, @NonNull Paint paint)
public void drawLines(@Size(multiple = 4) @NonNull float[] pts, @NonNull Paint paint)
```

- 绘制矩形

```
public void drawRect(float left, float top, float right, float bottom, @NonNull Paint paint)
public void drawRect(@NonNull Rect r, @NonNull Paint paint)
public void drawRect(@NonNull RectF rect, @NonNull Paint paint)
```

- 绘制圆角矩形

```
public void drawRoundRect(@NonNull RectF rect, float rx, float ry, @NonNull Paint paint)
```

rx, ry 是椭圆的两个半径，当指定的圆角半径大于对应的矩形内切椭圆的半径的时候，绘制出的就是该椭圆。（大于半径强制给半径）

- 绘制椭圆

```
public void drawOval(@NonNull RectF oval, @NonNull Paint paint)
```

- 绘制圆

```
public void drawCircle(float cx, float cy, float radius, @NonNull Paint paint)
```

- 绘制圆弧

```
public void drawArc(@NonNull RectF oval, float startAngle, float sweepAngle, boolean useCenter, @NonNull Paint paint) {
```

其中 X 轴正方向为 0 度

userCenter 代表是否将圆弧两端点与圆心相连，否则是两点直接相连

## Paint 简介

常用 API

```
//设置颜色
public void setColor(@ColorInt int color)
public void setARGB(int a, int r, int g, int b)

//抗锯齿功能
public void setAntiAlias(boolean aa)

//设置画笔式样 Style.FILL Style.STROKE Style.FILL_AND_STROKE
public void setStyle(Style style)

//边框宽度
public void setStrokeWidth(float width)
```

## Canvas 画布操作

所有的画布操作都只影响后续的绘制，对之前已经绘制过的内容没有影响。

### 位移 Translate

`public void translate(float dx, float dy)`

位移是基于当前位置，而不是每次基于屏幕左上角(0,0)位移

```java
canvas.translate(200, 200);
canvas.drawCircle(0, 0, 100, mPaint);

canvas.translate(200, 200);
canvas.drawCircle(0, 0, 100, mPaint);
```

<img src="image/canvas-translate.png" width = "40%" height = "40%" />

先将画布位移，在以 0，0 坐标绘制一个圆，再次位移，再已 0，0 绘制，结果是两个圆没有重合，并且圆心均不在屏幕的 0, 0 上，证明画布已经经过位移，再绘制。

### 缩放 Scale

```
public void scale (float sx, float sy)
public final void scale (float sx, float sy, float px, float py)
```

- 缩放中心默认为原点，缩放中心轴默认为原点的坐标轴

```
// 将坐标系原点移动到画布正中心
canvas.translate(mWidth / 2, mHeight / 2);

RectF rect = new RectF(0,-400,400,0);   // 矩形区域

mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
canvas.drawRect(rect,mPaint);

canvas.scale(0.5f,0.5f);                // 画布缩放

mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
canvas.drawRect(rect,mPaint);


// 将坐标系原点移动到画布正中心
canvas.translate(mWidth / 2, mHeight / 2);

RectF rect = new RectF(0,-400,400,0);   // 矩形区域

mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
canvas.drawRect(rect,mPaint);

canvas.scale(0.5f,0.5f,200,0);          // 画布缩放  <-- 缩放中心向右偏移了200个单位

mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
canvas.drawRect(rect,mPaint);
```

- sx sy 取值范围说明：

|取值范围(n)|说明|
|---|---|
|(-∞, -1)|先根据缩放中心放大n倍，再根据中心轴进行翻转|
|-1|根据缩放中心轴进行翻转|
|(-1, 0)|先根据缩放中心缩小到n，再根据中心轴进行翻转|
|0|不会显示，若sx为0，则宽度为0，不会显示，sy同理|
|(0, 1)|根据缩放中心缩小到n|
|1|没有变化|
|(1, +∞)|根据缩放中心放大n倍|

- 缩放比例为负数的时候，会根据缩放中心轴进行翻转

```
// 将坐标系原点移动到画布正中心
canvas.translate(mWidth / 2, mHeight / 2);

RectF rect = new RectF(0,-400,400,0);   // 矩形区域

mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
canvas.drawRect(rect,mPaint);


canvas.scale(-0.5f,-0.5f);          // 画布缩放

mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
canvas.drawRect(rect,mPaint);
```

- 缩放可以叠加，每次缩放的系数相乘就是最后的缩放结果

```
canvas.scale(0.5f,0.5f);
canvas.scale(0.5f,0.1f);
```

### 旋转 Rotate

```
public void rotate (float degrees)
public final void rotate (float degrees, float px, float py)
```

- px py 与缩放类似，控制中心点的。默认 依旧是中心点

```
// 将坐标系原点移动到画布正中心
canvas.translate(mWidth / 2, mHeight / 2);

RectF rect = new RectF(0,-400,400,0);   // 矩形区域

mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
canvas.drawRect(rect,mPaint);

canvas.rotate(180);                     // 旋转180度 <-- 默认旋转中心为原点

mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
canvas.drawRect(rect,mPaint);
```

- 旋转也可以叠加

### 错切 Skew

```
public void skew (float sx, float sy)
```

参数

sx:将画布在x方向上倾斜相应的角度，sx 倾斜角度的tan值，
sy:将画布在y轴方向上倾斜相应的角度，sy 为倾斜角度的tan值.

变换后

X = x + sx * y
Y = sy * x + y

```
// 将坐标系原点移动到画布正中心
canvas.translate(mWidth / 2, mHeight / 2);

RectF rect = new RectF(0,0,200,200);   // 矩形区域

mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
canvas.drawRect(rect,mPaint);

canvas.skew(1,0);                       // 水平错切 <- 45度

mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
canvas.drawRect(rect,mPaint);
```

- 错切可叠加，但调用顺序不同，绘制结果也不同

### 快照 Save、回滚 Store

相关 API

|API|简介|
|---|---|
|save|把当前的画布的状态进行保存，然后放入特定的栈中|
|saveLayerXxx|新建一个图层，并放入特定的栈中|
|restore|把栈中最顶层的画布状态取出来，并按照这个状态恢复当前的画布|
|restoreToCount|弹出指定位置及其以上所有的状态，并按照指定位置的状态进行恢复|
|getSaveCount|获取栈中内容的数量(即保存次数)|

- save

```
// 保存全部状态
public int save ()

// 根据saveFlags参数保存一部分状态
public int save (int saveFlags)
```
每调用一次save方法，都会在栈顶添加一条状态信息，以上面状态栈图片为例，再调用一次save则会在第5次上面载添加一条状态。

save flag

|数据类型|名称|简介|
|---|---|---|
|int|ALL_SAVE_FLAG|默认，保存全部状态|
|int|CLIP_SAVE_FLAG|保存剪辑区|
|int|CLIP_TO_LAYER_SAVE_FLAG|剪裁区作为图层保存|
|int|FULL_COLOR_LAYER_SAVE_FLAG|保存图层的全部色彩通道|
|int|HAS_ALPHA_LAYER_SAVE_FLAG|保存图层的alpha(不透明度)通道|
|int|MATRIX_SAVE_FLAG|保存 Matrix 信息(translate, rotate, scale, skew)|

- saveLayerXXX

```
// 无图层alpha(不透明度)通道
public int saveLayer (RectF bounds, Paint paint)
public int saveLayer (RectF bounds, Paint paint, int saveFlags)
public int saveLayer (float left, float top, float right, float bottom, Paint paint)
public int saveLayer (float left, float top, float right, float bottom, Paint paint, int saveFlags)

// 有图层alpha(不透明度)通道
public int saveLayerAlpha (RectF bounds, int alpha)
public int saveLayerAlpha (RectF bounds, int alpha, int saveFlags)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha, int saveFlags)
```

使用saveLayerXxx方法，也会将图层状态也放入状态栈中，同样使用restore方法进行恢复。

- restore

状态回滚，就是从栈顶取出一个状态然后根据内容进行恢复。

- restoreToCount

弹出指定位置以及以上所有状态，并根据指定位置状态进行恢复。

- getSaveCount 默认值为1