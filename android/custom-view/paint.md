# Paint

## 颜色

### 基本颜色

Canvas 使用 drawXXX 方法，将 Paint 对象当作参数传入。

- 通过 `paint.setColor(int color)` 来设置颜色
- 通过 `paint.setARGB(int a, int r, int g, int b)` 来设置颜色
- 通过 `Shader` 指定着色方案 `paint.setShader(Shader shader)`

#### Shader 着色器

- 着色器设置的是一个颜色方案，或者说是一套着色规则。当设置了 Shader 后，Paint 在绘制图形和文字的时候就不使用 setColor/ARGB 设置的颜色了，而是使用 Shader 的方案中的颜色。
- Android 中直接使用的是 Shader 的几个子类：`LinearGradient` `RadialGradient` `SweepGradient` `BitmapShader` `ComposeShader`

#### LinearGradient 线性渐变

构造方法：

LinearGradient(float x0, float y0, float x1, float y1, int color0, int color1, Shader.TileMode tile)

参数：

- x0 y0 x1 y1：渐变的两个端点的位置
- color0 color1 是端点的颜色
- tile：端点范围之外的着色规则，类型是 TileMode。TileMode 一共有 3 个值可选： CLAMP, MIRROR 和  REPEAT。CLAMP 会在端点之外延续端点处的颜色；MIRROR 是镜像模式；REPEAT 是重复模式。

```java
Shader shader = new LinearGradient(100, 100, 500, 500, Color.parseColor("#E91E63"), Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);
padint.setShader(shader);
...
canvas.drawCircle(300, 300, 200, paint);
```

#### RadialGradient 辐射渐变

```java
Shader shader = new RadialGradient(300, 300, 200, Color.parseColor("#E91E63"),  Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 200, paint);  
```

public void setAntiAlias(boolean aa) // 抗锯齿功能

public void setColor(int color) // 设置画笔颜色

public void setStyle(Style style) // 设置画笔式样 `Style.FILL Style.STROKE Style.FILL_AND_STROKE`

public void setStrokeWidth(float width) // 边框宽度

### 发光效果

public MaskFilter setMaskFilter(MaskFilter filter)

BlurMaskFilter EmbossMaskFilter 一个是模糊，一个是浮雕，不支持硬件加速

public BlurMaskFilter(float radius, Blur style)

radius 是高斯模糊半径

style Blur.INNER Blur.SOLID Blur.NORMAL Blur.OUTER

### Paint 其他方法

setStrokeCap（Paint.Cap cap) // 线冒样式 Cap.ROUND Cap.SQUARE Cap.BUTT

setStrokeJoin(Paint.Join join) // 转弯处样式 Join.MITER Join.ROUND Join.BEVEL

setPathEffect(PathEffect effect)

  CornerPathEffect(float radius) // 利用圆形将尖角代替

  DashPathEffect(float intervals[], float phase) // 虚线，intervals 表示各个线段长度，整个虚线就是由这些基本线段循环组成，长度必须大于等于2，为偶数。phase 开始绘制的偏移值

  DiscretePathEffect(float segmentLength, float deviation) // 离散样式，segmentLength 表示将原来的路径切成多长的线段，值越小，切的线段越多。deviation，表示每个小线段的可偏移距离，值越大，越凌乱

  PathDashPathEffect(Path shape, float advance, float phase, Style style) // 印章效果，shape 表示印章的路径，advance 表示两个印章路径之间的距离，phase 表示整体路径偏移量，style 表示转教的式样：Style.MORPH 印章变形，Style.TRANSLATE 印章位移，Style.ROTATE 印章旋转

  //合并两个特效
  ComposePathEffect(PathEffect outerpe, PathEffect innerpe) // 先将第一个参数的 pe 加上，再在此基础上加上第二个 pe
  SumPathEffect(PathEffect outerpe, PathEffect innerpe) // 分别对原始路径做第一个和第二个 pe， 然后合并
