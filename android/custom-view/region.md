# Region

## Regions

表示 canvas 的一块区域

```
public Region()

public Region(Region region)

public Region(Rect r)

public Region(int left, int top, int right, int bottom)
```

Method

```
public void setEmpty()

public boolean set(Region region)

public boolean set(Rect r)

public boolean set(int left, int top, int right, int ,bottom)

public boolean setPath(Path path, Region clip)
```

setPath 说明

用来构造非矩形的区域，path 参数为用来构造区域的参数， clip 为与前边 path 所构造的区域取交集的 Region，通常要设置的比 path 的区域大

### RegionIterator

非矩形区域，是用多个矩形来表达其形状的

RegionIterator(Region region) // 根据区域构建对应的矩形集
boolean next(Rect r) // 获取下一个矩形

### 区域的操作

```
public final boolean union(Rect r)

public boolean op(Rect r, Op op)

public boolean op(int left, int top, int right, int bottom, Op op)

public boolean op(Region region, Op op)

public boolean op(Rect rect, Region region, Op op)

public enum Op{
  DIFFERENCE(0), // 最终区域为 region1 与 region2 不同的区域
  INTERSECT(1),  // 最终区域为 region1 与 region2 相交的区域
  UNION(2),      // 最终区域为 region1 与 region2 组合一起的区域
  XOR(3),        // 最终区域为 region1 与 region2 相交之外的区域
  REVERSE_DIFFERENCE(4), // 最终区域为 region2 与 region1 不同的区域
  REPLACE(5); // 最终区域为 region2 的区域
}
```

### 获取 Canvas

#### 自定义 View

onDraw() 绘制 View 自身

dispatchDraw() 绘制子 View

调用顺序是 onDraw() -> dispatchDraw()

ViewGroup 中，如果有背景，就会调用 onDraw()，否则会跳过，直接调用 dispatchDraw()，因此 ViewGroup 中通常重写 dispatchDraw()

View 中，两者都会调用，原则上讲，绘制 View 时，应该重写 onDraw()

#### 使用 Bitmap 创建

```
Canvas c = new Canvas(bitmap);

Canvas c = new Canvas();
c.setBitmap(bitmap);

//bitmap 可以从文件 decode，也可以直接创建
```

如果使用 bitmap 构造了一个 canvas，那么 canvas 上绘制的图像也会保存在 bitmap 上而不是 view 上，如果想要绘制到 view 上，需要调用 onDraw() 中的 canvas 去绘制 bitmap

### 变换

```
public void translate(float dx, float dy)

public void rotate(float degress) //z 轴旋转

public void rotate(float degress, float px, float py)

public void scale(float sx, float sy)

public final void scale(float sx, float sy, float px, float py)

public void skew(float sx, float sy)
```

### 裁剪

```
boolean clipPath(Path path)

boolean clipPath(Path path, Region.Op op)

boolean clipRect(Rect rect, Region.Op op)

boolean clipRect(RectF rect, Region.Op op)

boolean clipRect(int left, int top, int right, int bottom)

boolean clipRect(float left, float top, float right, float bottom)

boolean clipRect(RectF rect)

boolean clipRect(float left, float top, float right, float bottom, Region.Op op)

boolean clipRect(Rect rect)

boolean clipRegion(Region region)

boolean clipRegion(Region region, Region.Op op)
```

### 图层与画布

```
public int save()
public int save(int saveFlags)
public int saveLayer(RectF bounds, Paint paint, int saveFlags)
public int saveLayer(float left, float top, float right, float bottom,Paint paint, int saveFlags)
public int saveLayerAlpha(RectF bounds, int alpha, int saveFlags)
public int saveLayerAlpha(float left, float top, float right, float bottom,int alpha, int saveFlags)

restore()
restoreToCount()
```

#### save()

save() 的两个函数，不会创建新画布，会保存当前画布状态，放入特定栈中

#### restore() restoreToCount()

restore() 把栈顶的画布状态取出来，并在这个画布上绘制

restoreToCount() 退栈，直到指定的 count。count 比 index 多1，save() 返回的是 index，因此直接传入 index 表示返回到 save 之前的状态

#### saveLayer()

RectF 要保存的区域

saveFlags：`ALL_SAVE_FLAG、MATRIX_SAVE_FLAG、CLIP_SAVE_FLAG、HAS_ALPHA_LAYER_SAVE_FLAG、FULL_COLOR_LAYER_SAVE_FLAG、CLIP_TO_LAYER_SAVE_FLAG`

调用 saveLayer 后，会生成一个全新的 bitmap，大小是指定的保存区域的大小，像素是全透明的，调用后，所有的绘图操作都是在这个 bitmap 上进行，绘制结束后，会直接盖在上一层的 bitmap 显示

Bitmap 画布：每一个画布都是一个 bitmap，所有的图像都是画到 bitmap 上的。

Layer 图层：每次调用 canvas.drawXXX 时，都会生成一个透明的图层来专门画这个图形，然后盖在画布上

画布有两种，一种是 view 原始画布。另一种是人造画布，通过 saveLayer() newCanvas(bitmap) 创建的。尤其是 saveLayer()，一旦调用它创建一个画布后，之后所有的 drawXXX 都是在这个画布上绘制，只有调用 restore() restoreToCount() 才会回到原始画布上

#### saveLayerAlpha()

指定新建画布透明度，取值范围为 0-255，可以用 0x 十六进制表示

#### 用法

保存画布信息需要保存：位置信息、大小信息

- saveLayer() 后的所有动作只对新建画布有效
- 通过 Rect 指定的矩形大小就是新建画布的大小，如果每次 saveLayer() 新建画布都使用屏幕大小，很可能 OOM

Flag

- `ALL_SAVE_FLAG` 保存所有
- `MATRIX_SAVE_FLAG` 仅保存 canvas 的 matrix 数组
- `CLIP_SAVE_FLAG` 仅保存 canvas 的当前大小

以下3个只用于 saveLayer()，用于指定新建 bitmap 具有那种特性，已经不是保存画布的范畴

- `HAS_ALPHA_LAYER_SAVE_FLAG` 标识新建的 bmp 画布在与上层画布合成时，不会将上层画布清空
- `FULL_COLOR_LAYER_SAVE_FLAG` 标识新建 bmp 颜色完全独立，与上层画布结合时，先清空上层画布，再覆盖上去
- `CLIP_TO_LAYER_SAVE_FLAG` 先把当前 canvas 根据 bounds 裁剪，影响每一个画布，无法恢复。与`CLIP_SAVE_FLAG`共用时，可以恢复原来大小

没有指定`FULL_COLOR_LAYER_SAVE_FLAG`或者`HAS_ALPHA_LAYER_SAVE_FLAG`时，默认使用`FULL_COLOR_LAYER_SAVE_FLAG`
