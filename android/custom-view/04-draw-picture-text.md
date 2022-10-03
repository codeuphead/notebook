# 绘制图片、文字

## 图片

### drawPicture 矢量图

使用 Piture 前，关闭硬件加速

Picture 记录了绘制的调用，因此可以回溯、复用绘制过程

- Picture 相关方法

|相关方法|简介|
|public int getWidth ()|获取宽度|
|public int getHeight ()|获取高度|
|public Canvas beginRecording (int width, int height)|开始录制 (返回一个Canvas，在Canvas中所有的绘制都会存储在Picture中)|
|public void endRecording ()|结束录制|
|public void draw (Canvas canvas)|将Picture中内容绘制到Canvas中|

绘制 Picture 的内容的方法有：

Picture 提供的 draw 方法；Canvas 提供的 drawPicture 方法；将 Picture 包装为 PictureDrawable，使用 PictureDrawable 的 draw 方法绘制

- Picture 的 draw

```java
//将 Picture 内容绘制在 Canvas 上
mPicture.draw(canvas);
```

- Canvas 的 drawPicture 方法，通常使用这个方法

```java
public void drawPicture (Picture picture)
public void drawPicture (Picture picture, Rect dst)
public void drawPicture (Picture picture, RectF dst)
```

- 将 Picture 包装成 drawable

```java
// 包装成为Drawable
PictureDrawable drawable = new PictureDrawable(mPicture);
// 设置绘制区域 -- 注意此处所绘制的实际内容不会缩放
drawable.setBounds(0,0,250,mPicture.getHeight());
// 绘制
drawable.draw(canvas);
```

setBounds 是这是再画布上的绘制区域

### drawBitmap 位图

Bitmap 获取方式：创建、BitmapDrawable、BitmapFactory

```java
// 第一种
public void drawBitmap (Bitmap bitmap, Matrix matrix, Paint paint)

// 第二种
public void drawBitmap (Bitmap bitmap, float left, float top, Paint paint)

// 第三种
public void drawBitmap (Bitmap bitmap, Rect src, Rect dst, Paint paint)
public void drawBitmap (Bitmap bitmap, Rect src, RectF dst, Paint paint)

//src 代表绘制 bitmap 的区域，dst 代表图片在屏幕上绘制的区域，如果 dst 与 src 比例不一致，则会进行缩放
```

第三种方法可以绘制图片的一部分到画布上，可以将大量素材放到一张图片上，可大大减少素材数量

## 文字

```java
// 第一类
public void drawText (String text, float x, float y, Paint paint)
public void drawText (String text, int start, int end, float x, float y, Paint paint)
public void drawText (CharSequence text, int start, int end, float x, float y, Paint paint)
public void drawText (char[] text, int index, int count, float x, float y, Paint paint)

// 第二类
public void drawPosText (String text, float[] pos, Paint paint)
public void drawPosText (char[] text, int index, int count, float[] pos, Paint paint)

// 第三类
public void drawTextOnPath (String text, Path path, float hOffset, float vOffset, Paint paint)
public void drawTextOnPath (char[] text, int index, int count, Path path, float hOffset, float vOffset, Paint paint)
```

- 第一类，指定文字基线进行绘制，x、y 分别为两个方向上的基线
- 第二类，可以指定每个文字的位置
- 第三类，可以指定一个路径进行绘制

### Paint 相关

```java
setColor setARGB setAlpha //设置颜色透明度
setStyle //设置填充方式：FILL STROKE FILL_AND_STROKE

//文字相关
setTextAlign //设置对齐：LEFT、CENTER、RIGHT
setTextSize //设置字体大小
setTypeface //设置或清除字体样式：Typeface.NORMAL BOLD ITALIC BOLD_ITALIC
setTextScaleX //默认 1.0，水平拉伸压缩，高度不变
setTextSkewX //默认 0，普通斜体是 -0.25
setStrikeThruText //删除线
setFakeBoldText //粗体
setUnderlineText //下划线
measureText //测量文本大小
```

### 第一类 drawText

x、y 是 baseline 的位置

x baseline 作用于文字 rect 的相对位置，可以通过 `paint.setAlign(Paint.Align.Right)` 设置。代表的是 x baseline 相对于文字 rect 的位置，默认 Left，即基线在文字 rect 左侧

y baseline 作用于文字 rect 的文字绘制的水平基线

只要 baseline 坐标、文字大小，确定后，文字的位置就是确定的

绘制字符串部分文字，index 是开始索引，count 是个数；start 是开始索引，end 是结束索引，不包含 end

```java
public void drawTextRun(@NonNull char[] text, int index, int count, int contextIndex, int contextCount, float x, float y, boolean isRtl, @NonNull Paint paint)

public void drawTextRun(@NonNull CharSequence text, int start, int end, int contextStart, int contextEnd, float x, float y, boolean isRtl, @NonNull Paint paint)

sdk23 引入
```

### 第二类 drawPosText

没有 x、y 的坐标，而是出现了 float[] pos 的数组，该数组就是指定坐标的，其可以给每个字符指定坐标

- 必须指定所有字符位置，否则直接crash掉，反人类设计
- 性能不佳，在大量使用的时候可能导致卡顿
- 不支持emoji等特殊字符，不支持字形组合与分解

### 第三类 drawTextOnPath

public void drawTextOnPath(char[] text, int index, int count, Path path, float hOffset, float vOffset, Paint paint)

public void drawTextOnPath(String text, Path path, float hOffset, float vOffset, Paint paint)

hOffset，与路径起始点的水平偏移量；vOffset，与路径中心的垂直偏移量

### FontMatrics

设置了 paint 的 textSize 后，通过调用 paint.getFontMetrics() 获取对象

- public float top
- public float ascent
- public float descent
- public float bottom
- public float leading

- top：绘制的最高高度，给定字体和字号
- ascent：系统建议，绘制字符时候的最高高度
- descent：系统建议，绘制字符时候的最低高度
- bottom：绘制的最低高度，给定字体和字号
- leading：系统建议，两行文本之间的距离

ascent/descent top/bottom 相当于安全绘制区域和最大绘制区域，他们的值都是基于 baseline 的高度

5条线，baseline 是通过构造 drawText 时候传入的，其他4个是 FontMatrics 中的成员

### 高度、宽度

字符所占的高度，可以直接用 bottom - top

字符串宽度，`paint.measureText("xxx");`

文字矩形：`getTextBounds();`，获取到的矩形基于（0，0）

### 定点写字 根据提供的数据、FontMatrics 等，推算出 baselineX baselineY 就可以绘制文字了

```java
public class XView extends View {

  private Paint paint;
  private int baseLineX;
  private int baseLineY;
  private int width;
  private int height;
  private Rect minRect;
  private Rect viewRect;

  public XView(Context context, AttributeSet attributeSet) {
    super(context, attributeSet);
    init();
  }

  private void init() {
    baseLineX = 0;
    baseLineY = 200;
    paint = new Paint();
    minRect = new Rect();
    viewRect = new Rect();
    setBackgroundColor(Color.BLACK);
  }

  @Override
  protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    String text = "google now";
    paint.setTextSize(120);
    FontMetrics fontMetrics = paint.getFontMetrics();
    float ascent = baseLineY + fontMetrics.ascent;
    float descent = baseLineY + fontMetrics.descent;
    float top = baseLineY + fontMetrics.top;
    float bottom = baseLineY + fontMetrics.bottom;

    drawTextRect(canvas, text, (int) top, (int) bottom);
    drawText(canvas, text);
    draw5Lines(canvas, ascent, descent, top, bottom);
  }

  private void drawText(Canvas canvas, String text) {
    paint.setColor(Color.WHITE);
    canvas.drawText(text, baseLineX, baseLineY, paint);
  }

  private void drawTextRect(Canvas canvas, String text, int top, int bottom) {
    int textWidth = (int) paint.measureText(text);
    viewRect.set(baseLineX, top, baseLineX + textWidth, bottom);

    paint.setColor(Color.GREEN);
    canvas.drawRect(viewRect, paint);

    paint.getTextBounds(text, 0, text.length(), minRect);
    minRect.top = minRect.top + baseLineY;
    minRect.bottom = minRect.bottom + baseLineY;
    paint.setColor(Color.RED);
    canvas.drawRect(minRect, paint);
  }

  private void draw5Lines(Canvas canvas, float ascent, float descent, float top, float bottom) {
    paint.setColor(Color.RED);
    canvas.drawLine(baseLineX, baseLineY, width, baseLineY, paint);

    paint.setColor(Color.BLUE);
    canvas.drawLine(baseLineX, top, width, top, paint);

    paint.setColor(Color.GREEN);
    canvas.drawLine(baseLineX, ascent, width, ascent, paint);

    paint.setColor(Color.GREEN);
    canvas.drawLine(baseLineX, descent, width, descent, paint);

    paint.setColor(Color.BLUE);
    canvas.drawLine(baseLineX, bottom, width, bottom, paint);
  }

  @Override
  protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    this.width = w;
    this.height = h;
  }
}
```