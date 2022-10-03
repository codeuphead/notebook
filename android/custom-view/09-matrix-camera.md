# Matrix Camera

graphic 下的 Camera，可以理解为给 View 拍照，将 3D 内容绘制成 2D。

实际上这个类更像是一个操作 Matrix 的工具类，使用 Camera 和 Matrix 可以在不使用 OpenGL 的情况下制作出简单的 3D 效果

|方法类别|相关API|简介|
|---|---|---|
|基本方法|save、restore|保存、 回滚|
|常用方法|getMatrix、applyToCanvas|获取Matrix、应用到画布|
|平移|translate|位移|
|旋转|rotat (API 12)、rotateX、rotateY、rotateZ|各种旋转|
|相机位置|setLocation (API 12)、getLocationX (API 16)、getLocationY (API 16)、getLocationZ (API 16)|设置与获取相机位置|

Android 平台的 3D 坐标系是左手坐标系，Z 轴方向垂直向屏幕内， Y 轴方向向上，X 轴方向向右

Android 上面观察View的摄像机默认位置在屏幕左上角，而且是距屏幕有一段距离的。

>摄像机的位置默认是 (0, 0, -576)。其中 -576＝ -8 x 72，虽然官方文档说距离屏幕的距离是 -8, 但经过测试实际距离是 -576 像素，当距离为 -10 的时候，实际距离为 -720 像素。我使用了3款手机测试，屏幕大小和像素密度均不同，但结果都是一样的。
>这个魔数可以在 Android 底层的图像引擎 Skia 中找到。在 Skia 中，Camera 的位置单位是英寸，英寸和像素的换算单位在 Skia 中被固定为 72 像素，而 Android 中把这个换算单位照搬了过来。

## 基本方法

基本方法就有两个 save 和 restore，主要作用为保存当前状态和恢复到上一次保存的状态，通常成对使用，常用格式如下:

```java
camera.save();      // 保存状态
...                 // 具体操作
camera.retore();    // 回滚状态
```

## 常用方法

```java
void getMatrix(Matrix matrix)
void applyToCanvas(Canvas canvas)
```

## 平移

```java
void translate (float x, float y, float z)
```

x 轴平移，两者一致，y 轴平移，因为 y 轴方向相反，因此结果相反。

z 轴平移，近大远小

## 旋转

伪3D，因为 View 没有厚度

```java
// (API 12) 可以控制View同时绕x，y，z轴旋转，可以由下面几种方法复合而来。
void rotate (float x, float y, float z);

// 控制View绕单个坐标轴旋转
void rotateX (float deg);
void rotateY (float deg);
void rotateZ (float deg);
```

旋转中心默认是坐标原点，对于图片来说就是左上角位置。

我们都知道，在2D中，不论是旋转，错切还是缩放都是能够指定操作中心点位置的，但是在3D中却没有默认的方法，如果我们想要让图片围绕中心点旋转怎么办? 这就要使用到我们在Matrix原理提到过的方法，虽然当时因为有更好的选择方案，并不提倡这样做:

```java
Matrix temp = new Matrix();                 // 临时Matrix变量
this.getMatrix(temp);                       // 获取Matrix
temp.preTranslate(-centerX, -centerY);      // 使用pre将旋转中心移动到和Camera位置相同。
temp.postTranslate(centerX, centerY);       // 使用post将图片(View)移动到原来的位置
```

官方 Rotate3dAnimation

```java
public class Rotate3dAnimation extends Animation {
    private final float mFromDegrees;
    private final float mToDegrees;
    private final float mCenterX;
    private final float mCenterY;
    private final float mDepthZ;
    private final boolean mReverse;
    private Camera mCamera;
    /**
     * 创建一个绕y轴旋转的3D动画效果，旋转过程中具有深度调节，可以指定旋转中心。
     *
     * @param fromDegrees   起始时角度
     * @param toDegrees     结束时角度
     * @param centerX       旋转中心x坐标
     * @param centerY       旋转中心y坐标
     * @param depthZ        最远到达的z轴坐标
     * @param reverse       true 表示由从0到depthZ，false相反
     */
    public Rotate3dAnimation(float fromDegrees, float toDegrees,
            float centerX, float centerY, float depthZ, boolean reverse) {
        mFromDegrees = fromDegrees;
        mToDegrees = toDegrees;
        mCenterX = centerX;
        mCenterY = centerY;
        mDepthZ = depthZ;
        mReverse = reverse;
    }
    @Override
    public void initialize(int width, int height, int parentWidth, int parentHeight) {
        super.initialize(width, height, parentWidth, parentHeight);
        mCamera = new Camera();
    }
    @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
        final float fromDegrees = mFromDegrees;
        float degrees = fromDegrees + ((mToDegrees - fromDegrees) * interpolatedTime);
        final float centerX = mCenterX;
        final float centerY = mCenterY;
        final Camera camera = mCamera;
        final Matrix matrix = t.getMatrix();
        camera.save();

      // 调节深度
        if (mReverse) {
            camera.translate(0.0f, 0.0f, mDepthZ * interpolatedTime);
        } else {
            camera.translate(0.0f, 0.0f, mDepthZ * (1.0f - interpolatedTime));
        }

// 绕y轴旋转
        camera.rotateY(degrees);

        camera.getMatrix(matrix);
        camera.restore();

    // 调节中心点
        matrix.preTranslate(-centerX, -centerY);
        matrix.postTranslate(centerX, centerY);
    }
}
```

修正后的代码

```java
public class Rotate3dAnimation extends Animation {
    private final float mFromDegrees;
    private final float mToDegrees;
    private final float mCenterX;
    private final float mCenterY;
    private final float mDepthZ;
    private final boolean mReverse;
    private Camera mCamera;
    float scale = 1;    // <------- 像素密度

    /**
     * 创建一个绕y轴旋转的3D动画效果，旋转过程中具有深度调节，可以指定旋转中心。
     * @param context     <------- 添加上下文,为获取像素密度准备
     * @param fromDegrees 起始时角度
     * @param toDegrees   结束时角度
     * @param centerX     旋转中心x坐标
     * @param centerY     旋转中心y坐标
     * @param depthZ      最远到达的z轴坐标
     * @param reverse     true 表示由从0到depthZ，false相反
     */
    public Rotate3dAnimation(Context context, float fromDegrees, float toDegrees,
                             float centerX, float centerY, float depthZ, boolean reverse) {
        mFromDegrees = fromDegrees;
        mToDegrees = toDegrees;
        mCenterX = centerX;
        mCenterY = centerY;
        mDepthZ = depthZ;
        mReverse = reverse;

        // 获取手机像素密度 （即dp与px的比例）
        scale = context.getResources().getDisplayMetrics().density;
    }

    @Override
    public void initialize(int width, int height, int parentWidth, int parentHeight) {
        super.initialize(width, height, parentWidth, parentHeight);
        mCamera = new Camera();
    }

    @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
        final float fromDegrees = mFromDegrees;
        float degrees = fromDegrees + ((mToDegrees - fromDegrees) * interpolatedTime);
        final float centerX = mCenterX;
        final float centerY = mCenterY;
        final Camera camera = mCamera;
        final Matrix matrix = t.getMatrix();
        camera.save();

        // 调节深度
        if (mReverse) {
            camera.translate(0.0f, 0.0f, mDepthZ * interpolatedTime);
        } else {
            camera.translate(0.0f, 0.0f, mDepthZ * (1.0f - interpolatedTime));
        }

        // 绕y轴旋转
        camera.rotateY(degrees);

        camera.getMatrix(matrix);
        camera.restore();

        // 修正失真，主要修改 MPERSP_0 和 MPERSP_1
        float[] mValues = new float[9];
        matrix.getValues(mValues);              //获取数值
        mValues[6] = mValues[6]/scale;          //数值修正
        mValues[7] = mValues[7]/scale;          //数值修正
        matrix.setValues(mValues);              //重新赋值

        // 调节中心点
        matrix.preTranslate(-centerX, -centerY);
        matrix.postTranslate(centerX, centerY);
    }
}
```

测试代码

```java
setContentView(R.layout.activity_test_camera_rotate2);
ImageView view = (ImageView) findViewById(R.id.img);
assert view != null;
view.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // 计算中心点（这里是使用view的中心作为旋转的中心点）
        final float centerX = v.getWidth() / 2.0f;
        final float centerY = v.getHeight() / 2.0f;

        //括号内参数分别为（上下文，开始角度，结束角度，x轴中心点，y轴中心点，深度，是否扭曲）
        final Rotate3dAnimation rotation = new Rotate3dAnimation(MainActivity.this, 0, 180, centerX, centerY, 0f, true, 2);

        rotation.setDuration(3000);                         //设置动画时长
        rotation.setFillAfter(true);                        //保持旋转后效果
        rotation.setInterpolator(new LinearInterpolator()); //设置插值器
        v.startAnimation(rotation);
    }
});
```

## 相机位置

```java
void setLocation (float x, float y, float z); // (API 12) 设置相机位置，默认位置是(0, 0, -8)

float getLocationX ();  // (API 16) 获取相机位置的x坐标，下同
float getLocationY ();
float getLocationZ ();
```

不常用，可以用 translate 替代

- 相机和 view 的 z 轴距离不能为0
- 前后均可“拍摄”
- 相机右移，等于 View 左移