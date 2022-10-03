# 2D 绘制 其他

## Bitmap 提取 alpha

public Bitmap extractAlpha();

## 绘制自定义阴影

```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        int width = 200;
        int height = width * mAlphaBmp.getHeight()/mAlphaBmp.getWidth();

        //绘制阴影
        mPaint.setColor(Color.RED);
        mPaint.setMaskFilter(new BlurMaskFilter(10, BlurMaskFilter.Blur.NORMAL));
        canvas.drawBitmap(mAlphaBmp,null,new Rect(10,10,width,height),mPaint);
        mPaint.setColor(Color.GREEN);
        canvas.drawBitmap(mAlphaBmp,null,new Rect(10,height+10,width,2*height),mPaint);

        //绘制原图像
        mPaint.setMaskFilter(null);
        canvas.drawBitmap(mBitmap,null,new Rect(0,0,width,height),mPaint);
        canvas.drawBitmap(mBitmap,null,new Rect(0,height,width,2*height),mPaint);
    }
```

## Shader

子类：BitmapShader ComposeShader LinearGradint RadialGradient SweepGradient

getLocalMatrix()
setLocalMatrix()

都是从控件左上角填充整个空间，利用 canvas.drawXXX 只是指定绘制哪一块

### BitmapShader

public BitmapShader(Bitmap bitmap, TileMode tileX, TileMode tileY)

- TileMode.CLAMP 用边缘色彩填充多余空间
- TileMode.REPEAT 重复原图像来填充多余空间
- TileMode.MIRROR 重复使用镜像模式的图像来填充多余空间

先填充 y 轴，再填充 x 轴

无论用 drawXXX 绘制多大的区域，都与 Shader 无关，Shader 总是在控件左上角开始，绘制的区域是显示的区域

#### 绘制望远镜

```
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (mBitmapBG == null){
            mBitmapBG = Bitmap.createBitmap(getWidth(),getHeight(), Bitmap.Config.ARGB_8888);
            Canvas canvasbg = new Canvas(mBitmapBG);
            canvasbg.drawBitmap(mBitmap,null,new Rect(0,0,getWidth(),getHeight()),mPaint);
        }

        if (mDx != -1 && mDy != -1) {
            mPaint.setShader(new BitmapShader(mBitmapBG, Shader.TileMode.REPEAT, Shader.TileMode.REPEAT));
            canvas.drawCircle(mDx, mDy, 150, mPaint);
        }
    }
```

#### 生产不规则头像

```
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Matrix matrix = new Matrix();
    float scale = (float) getWidth()/mBitmap.getWidth();
    matrix.setScale(scale,scale);
    mBitmapShader.setLocalMatrix(matrix);
    mPaint.setShader(mBitmapShader);

    float half = getWidth()/2;

    if (mEnumFormat == 0){
        canvas.drawCircle(half,half,getWidth()/2,mPaint);
    }else  if(mEnumFormat == 1){
        canvas.drawRoundRect(new RectF(0,0,getWidth(),getHeight()),mRadius,mRadius,mPaint);
    }
}
```

### LinearGradient

颜色值 ARGB

public LinearGradient(float x0, float y0, float x1, float y1,int color0, int color1, TileMode tile)

public LinearGradient(float x0, float y0, float x1, float y1,int colors[], float positions[], TileMode tile) // colors 使用的颜色，16进制；positions 0-1，float，每个颜色在整条渐变线中的百分比位置，通常第一个取0，最后一个取1

```java
    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);

        ValueAnimator animator = ValueAnimator.ofInt(0,2*getMeasuredWidth());
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                mDx = (Integer) animation.getAnimatedValue();
                postInvalidate();
            }
        });
        animator.setRepeatMode(ValueAnimator.RESTART);
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.setDuration(2000);
        animator.start();

        mLinearGradient = new LinearGradient(- getMeasuredWidth(),0,0,0,new int[]{
                getCurrentTextColor(),0xff00ff00,getCurrentTextColor()
        },
                new float[]{
                        0,
                        0.5f,
                        1
                },
                Shader.TileMode.CLAMP
        );
    }

    @Override
    protected void onDraw(Canvas canvas) {

        Matrix matrix = new Matrix();
        matrix.setTranslate(mDx,0);
        mLinearGradient.setLocalMatrix(matrix);
        mPaint.setShader(mLinearGradient);

        super.onDraw(canvas);
    }
```

### RadialGradient

//两色渐变
RadialGradient(float centerX, float centerY, float radius, int centerColor, int edgeColor, Shader.TileMode tileMode)
//多色渐变
RadialGradient(float centerX, float centerY, float radius, int[] colors, float[] stops, Shader.TileMode tileMode)

实现按钮水波效果
