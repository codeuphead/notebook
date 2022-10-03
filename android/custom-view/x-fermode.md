# 2D 绘制 Xfermode

## 禁用硬件加速

```java
应用级
<application android:hardwareAccelerated="false" ...>

Activity 级
<activity android:hardwareAccelerated="false" />

Window 级不支持关闭硬件加速
getWindow().setFlags(WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED, WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED);

View 级
setLayerType(View.LAYER_TYPE_SOFTWARE, null);
android:layerType="software"
```

## Xfermode 图形混合模式

AvoidXfermode PixelXfermode PorterXfermode

### AvoidXfermode

```java
public AvoidXfermode(int opColor, int tolerance, Mode mode)

opColor：使用的颜色值
tolerance：容差
mode：Mode.TARGET 将颜色替换掉；Mode.AVOID 与 Mode.TARGET 所选区域取反
```

```java
protected void onDraw(Canvas canvas) {
  super.onDraw(canvas);
  int width  = 500;
  int height = width * mBmp.getHeight()/mBmp.getWidth();
  mPaint.setColor(Color.RED);
  int layerID = canvas.saveLayer(0, 0, width, height, mPaint, Canvas.ALL_SAVE_FLAG);
  canvas.drawBitmap(mBmp,null,new Rect(0, 0, width, height), mPaint);
  mPaint.setXfermode(new AvoidXfermode(Color.WHITE, 100, AvoidXfermode.Mode.TARGET));
  canvas.drawBitmap(BitmapFactory.decodeResource(getResources(), R.drawable.flower_2), null, new Rect(0, 0, width, height), mPaint);
  canvas.restoreToCount(layerID);
}

protected void onDraw(Canvas canvas) {
  super.onDraw(canvas);
  int width  = 500;
  int height = width * mBmp.getHeight()/mBmp.getWidth();
  mPaint.setColor(Color.RED);
  int layerID = canvas.saveLayer(0,0,width,height,mPaint,Canvas.ALL_SAVE_FLAG);
  canvas.drawBitmap(mBmp,null,new Rect(0,0,width,height),mPaint);
  mPaint.setXfermode(new AvoidXfermode(Color.WHITE,100, AvoidXfermode.Mode.AVOID));
  canvas.drawRect(0,0,width,height,mPaint);
canvas.restoreToCount(layerID);
}
```

canvas的脏区域更新原理：
如果没有设置 Xfermode，那么直接将绘制的图形覆盖 Canvas 对应位置原有的像素
如果设置了Xfermode，那么会按照 Xfermode 具体的规则来更新 Canvas 中对应位置的像素颜色。

所以对于AvoidXfermode而言，这个规则就是先把把目标区域（选区）中的颜色值先清空，然后再把目标颜色给替换上

### PixelXformode

简单的异或运算 op ^ src ^ dst 返回的 alpha 值始终等于255，基本上用不到

### PorterDuffXfermode

public PorterDuffXfermode(PorterDuff.Mode mode)

```java
 Paint paint = new Paint();
 canvas.drawBitmap(destinationImage, 0, 0, paint);

 PorterDuff.Mode mode = // choose a mode
 paint.setXfermode(new PorterDuffXfermode(mode));

 canvas.drawBitmap(sourceImage, 0, 0, paint);
```

示例

```java
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    int layerID = canvas.saveLayer(0,0,width*2,height*2,mPaint,Canvas.ALL_SAVE_FLAG);
    canvas.drawBitmap(dstBmp, 0, 0, mPaint);
    mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
    canvas.drawBitmap(srcBmp, width/2, height/2, mPaint);
    mPaint.setXfermode(null);
    canvas.restoreToCount(layerID);
}
```

## 各种模式

先将目标图像画在底层，然后给 paint 设置上 PorterDuffXfermode，然后将处理过的源图盖在目标图上

### 颜色叠加

`Mode.ADD Mode.DARKEN Mode.LIGHTEN Mode.MULTIPY Mode.OVERLAY Mode.SCREEN`

`Mode.ADD` 饱和度相加，SRC 和 DST 两张图片相交区域饱和度进行相加，没有相交的部分，因为对方的饱和度为0，饱和度不变

`Mode.LIGHTEN` 变亮，加灯光效果

`Mode.DARKEN` 变暗

`Mode.MULTIPLY` 正片叠底，是用源图像的 alpha 值乘以目标图像的 alpha，由于非相交区域，目标图像的 alpha 为0，所以源图像计算后在非相交区域是透明的

`Mode.OVERLAY`

`Mode.SCREEN`

这6中模式都是在 ps 中存在的模式，是通过计算改变相交区域的颜色值，除了 MULTIPLY 会在相交区域外为头民，其他图像的非相交区域都会保持原样

### SRC 相关模式

以源图像显示为主的几个模式，相交区域优先显示 SRC 图像

`Mode.SRC Mode.SRC_IN Mode.SRC_OUT Mode.SRC_OVER Mode.SRC_ATOP`

`Mode.SRC` 公式`[Sa, Sc]` 相交区域以 SRC 图像显示，SRC 图像全部显示

`Mode.SRC_IN` 公式`[Sa * Da, Sc * Da]` 相交时利用 DST 图像的 alpha 来改变 SRC 图像的透明度和饱和度，当 DST 图像透明度为0时，SRC 图像完全不显示，示例：圆角图像、倒影图像

`Mode.SRC_OUT` 公式`[Sa * (1 - Da), Sc * (1 - Da)]` 利用 DST 图像透明度的补值来改变 SRC 图像的透明度和饱和度，当目 DST 有图像时，显示空白像素，DST 图像没有图像时，显示 SRC 图像，示例：橡皮擦，刮刮卡

`Mode.SRC_OVER` 公式`[Sa + (1 - Sa) * Da, Sc + (1 - Sa) * Dc]` 在 DST 图像顶部绘制 SRC 图像，在 SRC 图像基础上增加一部分透明度和色彩值，增加的量是 SRC 图像透明度的补量乘以 DST 图像的值，因此 SRC 图像透明度为 100% 时，就原样显示 SRC 图像

`Mode.SRC_ATOP` 公式`[Da, Sc * Da + (1 - Sa) * Dc]` 透明度直接使用 DST 图像透明度，颜色值在`SRC_IN`的基础上增加了一些，因此当透明度不是 100% 或者 0% 时，`SRC_ATOP` 比 `SRC_IN`的 SRC 图像饱和度会增加

### DST 相关模式

相交区域优先显示目标图像

`Mode.DST Mode.DST_IN Mode.DST_OUT Mode.DST_OVER Mode.DST_ATOP`

`Mode.DST` 公式`[Da, Dc]` 相交区域显示 DST 图像

`Mode.DST_IN` 公式`[Da * Sa, Dc * Sa]` 相交时利用 SRC 图像的 alpha 来改变 DST 图像的透明度和饱和度，当 SRC 图像透明度为0时，DST 图像完全不显示

`Mode.DST_OUT` 公式`[Da * (1 - Sa), Dc * (1 - Sa)]` 利用 SRC 图像透明度的补值来改变 DST 图像的透明度和饱和度

`Mode.DST_OVER` 公式`[Sa + (1 - Sa) * Da, Dc + (1 - Da) * Sc]` 使用 SRC 图像控制输出结果

`Mode.DST_ATOP` 公式`[Sa, Sa * Dc + Sc*(1 - Da)]`

### 其他模式

`Mode.CLEAR` 公式`[0, 0]`

`Mode.XOR` 公式`[Sa + Da - Sa * Da, Sc * (1 - Da) + Dc * (1 - Sa) + min(Sc, Dc)]`
