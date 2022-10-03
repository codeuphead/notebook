# 2D 绘制 滤镜

## 矩阵相乘

- A 矩阵的列数必须与 B 矩阵的行数相同才能相乘
- 矩阵 AxB 和矩阵 BxA 是不同的结果
- m x p 矩阵 和 p x n 矩阵，相乘的结果矩阵是 m x n

## 色彩矩阵

五阶矩阵

R、G、B、A

### 最后一列为平移变换，可增加特定颜色的饱和度

```java
ColorMatrix colrMatrix = new ColorMatrix(new float[]{
  1, 0, 0, 0, 0,
  0, 1, 0, 0, 50,
  0, 0, 1, 0, 0,
  0, 0, 0, 1, 0,
});
mPaint.setColorFilter(new ColorMatrixColorFilter(colorMatrix));
```

### 补色

```java
ColorMatrix colorMatrix = new ColorMatrix(new float[]{
        -1,  0,  0, 0, 255,
         0, -1,  0, 0, 255,
         0,  0, -1, 0, 255,
         0,  0,  0, 1,   0
});
```

### 色彩的缩放

就是色彩的乘法，针对某一个颜色进行缩放运算。

当对 RGBA 同时进行缩放时，就是调节亮度

```java
ColorMatrix colorMatrix = new ColorMatrix(new float[]{
        1.2f, 0, 0, 0, 0,
        0, 1.2f, 0, 0, 50,
        0, 0, 1.2f, 0, 0,
        0, 0, 0, 1.2f, 0,
});
```

### 通道色

只取某一个颜色

```java
ColorMatrix colorMatrix2 = new ColorMatrix(new float[]{
        0, 0, 0, 0, 0,
        0, 1, 0, 0, 0,
        0, 0, 0, 0, 0,
        0, 0, 0, 1, 0,
});
```

### 旋转 色相调节

绕某一轴旋转，利用三角函数变换另外两个轴的颜色值，叫做色相调节

蓝轴旋转

cosX  sinX  0 0 0
-sinX consX 0 0 0
 0     0    1 0 0
 0     0    0 1 0
 0     0    0 0 1

红轴旋转

1   0    0   0 0
0 cosX  sinX 0 0
0 -sinX cosX 0 0
0   0    0   1 0
0   0    0   0 1

绿轴旋转

cosX 0 -sinX 0 0
 0   1   0   0 0
sinX 0  cosX 0 0
 0   0   0   1 0
 0   0   0   0 1

### 色彩的投射运算

利用其他色彩分量的倍数来更改自己的色彩分量，1-4列中，非对角线的位置修改，可以增加来自于其他颜色的分量

#### 黑白

去色/黑白，将 RGB 三通道的色彩信息设置成一样，就变成了灰色

为了保持亮度不变 同一个通道中 R + G + B = 1 : 0.213 + 0.715 + 0.072 = 1，以此为基础的图像去色，保留了大量的 G 通道信息

```java
ColorMatrix colorMatrix = new ColorMatrix(new float[]{
        0.213f, 0.715f, 0.072f, 0, 0,
        0.213f, 0.715f, 0.072f, 0, 0,
        0.213f, 0.715f, 0.072f, 0, 0,
        0,       0,    0, 1, 0,
});
```

#### 色彩反转，反色

```java
ColorMatrix colorMatrix = new ColorMatrix(new float[]{
        0,1,0,0,0,
        1,0,0,0,0,
        0,0,1,0,0,
        0,0,0,1,0
});
```

将红色变为绿色，绿色变为红色

#### 照片变旧

```java
ColorMatrix colorMatrix = new ColorMatrix(new float[]{
        1/2f,1/2f,1/2f,0,0,
        1/3f,1/3f,1/3f,0,0,
        1/4f,1/4f,1/4f,0,0,
        0,0,0,1,0
});
```

## ColorMatrix

构造

```java
ColorMatrix()

ColorMatrix(float[] src)

ColorMatrix(ColorMatrix src)
```

方法

```java
public void set(ColorMatrix src)

public void set(float[] src)

public void reset()

public float[] getArray()
```

### 饱和度

```java
public void setSatruation（float sat) // 增强饱和度，同时增强 RGB。sat 为色彩饱和度放大倍数，0 为无色彩，灰度图像；1为元色彩不动，超过1为色彩过度饱和
```

```java
private Bitmap  handleColorMatrixBmp(int progress){
    // 创建一个相同尺寸的可变的位图区,用于绘制调色后的图片
    Canvas canvas = new Canvas(mTempBmp); // 得到画笔对象
    Paint paint = new Paint(); // 新建paint
    paint.setAntiAlias(true); // 设置抗锯齿,也即是边缘做平滑处理
    ColorMatrix mSaturationMatrix = new ColorMatrix();
    mSaturationMatrix.setSaturation(progress);

    paint.setColorFilter(new ColorMatrixColorFilter(mSaturationMatrix));// 设置颜色变换效果
    canvas.drawBitmap(mOriginBmp, 0, 0, paint); // 将颜色变化后的图片输出到新创建的位图区
    // 返回新的位图，也即调色处理后的图片
    return mTempBmp;
}
```

### 缩放

```java
public void setScale(float rSclae, float gScale, float bScale, float aScale)
```

```java
canvas.drawBitmap(bitmap, null, new Rect(0, 0, 500, 500 * bitmap.getHeight() / bitmap.getWidth()), mPaint);
canvas.save();
canvas.translate(510, 0);
// 生成色彩矩阵
ColorMatrix colorMatrix = new ColorMatrix();
colorMatrix.setScale(1,1.3f,1,1);
mPaint.setColorFilter(new ColorMatrixColorFilter(colorMatrix));
canvas.drawBitmap(bitmap, null, new Rect(0, 0, 500, 500 * bitmap.getHeight() / bitmap.getWidth()), mPaint);
```

### 旋转

```java
public void setRotate(int axis, float degress) // axis = 0 围绕红轴旋转；1，绿轴；2，蓝轴
```

### 相乘

```java
public void setConcat(ColorMatrix matA, ColorMatrix matB) // matA * matB

public void preConcat(ColorMatrix matrix) // matX * matrix

public void postConcat(ColorMatrix matrix) // matrix * matX
```

## ColorFilter

```java
public ColorFilter setColorFilter(ColorFilter filter)
```

子类

```java
ColorMatrixColorFilter(ColorMatrix matrix)

ColorMatrixColorFilter(float[] array)

LightingColorFilter(int mul, int add) // mul add 取值都是 0xRRGGBB，前者是惩罚，后者是加法


mPaint.setColorFilter(new LightingColorFilter(0x00ff00,0x000000)); // 只显示绿色
mPaint.setColorFilter(new LightingColorFilter(0xffffff,0x0000f0)); // 颜色变得更蓝
```

public PorterDuffColorFilter(int srcColor, PorterDuff.Mode mode)

Mode.SRC //
Mode.SRC_OVER //
Mode.SRC_IN //
Mode.SRC_OUT //
Mode.SRC_ATOP //

Mode.DST //
Mode.DST_OVER //
Mode.DST_IN //
Mode.DST_OUT //
Mode.DST_ATOP //

Mode.CLEAR // 清空图像
Mode.XOR // 清空图像

Mode.DARKEN // 变暗
Mode.LIGHTEN // 变亮
Mode.MULTIPLY // 正片叠底
Mode.SCREEN // 滤色
Mode.OVERLAY // 叠加
Mode.ADD // 饱和度相加

```java
private void drawPorterDuffFilter(Canvas canvas){
  int width  = 500;
  int height = width * mBmp.getHeight()/mBmp.getWidth();
  canvas.drawBitmap(mBmp,null,new Rect(0,0,width,height),mPaint);
  canvas.translate(550,0);
  mPaint.setColorFilter(new PorterDuffColorFilter(Color.RED, PorterDuff.Mode.MULTIPLY));
  canvas.drawBitmap(mBmp,null,new Rect(0,0,width,height),mPaint);
}
```

- PorterDuffColorFilter只能实现与一个特定颜色值的合成
- 通过Mode.ADD(饱和度相加)，Mode.DARKEN（变暗），Mode.LIGHTEN（变亮），Mode.MULTIPLY（正片叠底），Mode.OVERLAY（叠加），Mode.SCREEN（滤色）可以实现与指定颜色的复合
- 通过Mode.SRC、Mode.SRC_IN、Mode.SRC_ATOP能够实现setTint()的功能，可以改变纯色图标的颜色