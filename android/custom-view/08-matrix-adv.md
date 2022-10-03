# Matrix 详解

| 方法类别 | 相关 API | 摘要 |
| --- | --- | --- |
| 基本方法 | equals hashCode toString toShortString | 比较、 获取哈希值、 转换为字符串 |
| 数值操作 | set reset setValues getValues | 设置、 重置、 设置数值、 获取数值 |
| 数值计算 | mapPoints mapRadius mapRect mapVectors | 计算变换后的数值 |
| 设置(set) | setConcat setRotate setScale setSkew setTranslate | 设置变换 |
| 前乘(pre) | preConcat preRotate preScale preSkew preTranslate | 前乘变换 |
| 后乘(post) | postConcat postRotate postScale postSkew postTranslate | 后乘变换 |
| 特殊方法 | setPolyToPoly setRectToRect rectStaysRect setSinCos | 一些特殊操作 |
| 矩阵相关 | invert isAffine(API21) isIdentity | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 ... |

单位矩阵：

```
1 0 0
0 1 0
0 0 1
```

## 方法详解

- `new Matrix()` 创建出来的是一个单位矩阵
- `Matrix(Matrix src)` 对 src 矩阵进行深拷贝的矩阵

### 数值操作

- `void set(Matrix src)` 将 src 数值复制到当前 Matrix 中，如果参数为空，则重置当前 Matrix，相当于 reset
- `void reset()` 重置为单位矩阵
- `void setValues(float[] values)`
- `void getValues(float[] values)`

### 数值计算

- mapPoints

```java
void mapPoints (float[] pts) //pts数组作为参数传递原始数值，计算结果仍存放在pts中
void mapPoints (float[] dst, float[] src) //src作为参数传递原始数值，计算结果存放在dst中，src不变，如果原始数据需要保留则一般使用这种方法。
void mapPoints (float[] dst, int dstIndex,float[] src, int srcIndex, int pointCount) //可以指定只计算一部分数值。
```

计算一组点基于当前 Matrix 变换后的位置，(由于是计算点，所以参数中的 float 数组长度一般都是偶数的,若为奇数，则最后一个数值不参与计算)。

```java
// 初始数据为三个点 (0, 0) (80, 100) (400, 300)
float[] src = new float[]{0, 0, 80, 100, 400, 300};
float[] dst = new float[6];

// 构造一个matrix，x坐标缩放0.5
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);

// 输出计算之前数据
Log.i(TAG, "before: src="+ Arrays.toString(src));
Log.i(TAG, "before: dst="+ Arrays.toString(dst));

// 调用map方法计算
matrix.mapPoints(dst,src);

// 输出计算之后数据
Log.i(TAG, "after : src="+ Arrays.toString(src));
Log.i(TAG, "after : dst="+ Arrays.toString(dst));
```

- mapRadius

测量半径，测量的是平均半径（变换后的圆可能是椭圆）

```java
float radius = 100;
float result = 0;

// 构造一个matrix，x坐标缩放0.5
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);

Log.i(TAG, "mapRadius: "+radius);

result = matrix.mapRadius(radius);

Log.i(TAG, "mapRadius: "+result);
```

- mapRect

测量矩形变换后的位置

```java
boolean mapRect (RectF rect)

boolean mapRect (RectF dst, RectF src)
```

返回值为变换后是否仍为矩形

```java
RectF rect = new RectF(400, 400, 1000, 800);

// 构造一个matrix
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);
matrix.postSkew(1,0);

Log.i(TAG, "mapRadius: "+rect.toString());

boolean result = matrix.mapRect(rect);

Log.i(TAG, "mapRadius: "+rect.toString());
Log.e(TAG, "isRect: "+ result);
```

- mapVectors

与 mapPoints 基本相同，mapVectors 不受到位移影响，这符合向量的定律

```java
void mapVectors (float[] vecs)

void mapVectors (float[] dst, float[] src)

void mapVectors (float[] dst, int dstIndex, float[] src, int srcIndex, int vectorCount)
```

```java
float[] dst = new float[2];

// 构造一个matrix
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);
matrix.postTranslate(100,100);

// 计算向量, 不受位移影响
matrix.mapVectors(dst, src);
Log.i(TAG, "mapVectors: "+Arrays.toString(dst));

// 计算点
matrix.mapPoints(dst, src);
Log.i(TAG, "mapPoints: "+Arrays.toString(dst));
```

### set pre post

对于 translate、scale、rotata、skew 均有 set、pre、post，他们的基础是 concat，先构造出特殊矩阵然后用原始矩阵 concat 特殊矩阵，达到变换的结果

- set 设置，会覆盖掉之前的数值，导致之前的操作失效。
- pre 前乘，相当于矩阵的右乘， M' = M \* S (S 指为特殊矩阵)
- post 后乘，相当于矩阵的左乘，M' = S \* M （S 指为特殊矩阵）

### Matrix 相关

从 Canvas 中获取到的并不是初始矩阵，而是经过偏移以后的矩阵，偏移举例是距离屏幕左上角的位置。这个可以用于判定 View 在屏幕上的绝对位置，View 可以根据所处位置做出调整。

构造 Matrix 时，使用的是矩阵乘法，前乘与后乘结果差别很大。

受矩阵乘法影响，后面执行的操作可能影响前面的操作，使用时需要注意构造顺序。

### 特殊方法

- setPolyToPoly

```java
boolean setPolyToPoly (
        float[] src,    // 原始数组 src [x,y]，存储内容为一组点
        int srcIndex,   // 原始数组开始位置
        float[] dst,    // 目标数组 dst [x,y]，存储内容为一组点
        int dstIndex,   // 目标数组开始位置
        int pointCount) // 测控点的数量 取值范围是: 0到4
```

| pointCount | 摘要                                        |
| ---------- | ------------------------------------------- |
| 0          | 相当于 reset                                |
| 1          | 相当于 translate                            |
| 2          | 可以进行 缩放、旋转、平移 变换              |
| 3          | 可以进行 缩放、旋转、平移、错切 变换        |
| 4          | 可以进行 缩放、旋转、平移、错切以及任何形变 |

- setRectToRect

```java
boolean setRectToRect (RectF src,           // 源区域
                RectF dst,                  // 目标区域
                Matrix.ScaleToFit stf)      // 缩放适配模式
```

简单来说就是将源矩形的内容填充到目标矩形中，然而在大多数的情况下，源矩形和目标矩形的长宽比是不一致的，到底该如何填充呢，这个填充的模式就由第三个参数 stf 来确定。

ScaleToFit 是一个枚举类型，共包含了四种模式:

| 模式   | 摘要                                               |
| ------ | -------------------------------------------------- |
| CENTER | 居中，对 src 等比例缩放，将其居中放置在 dst 中。   |
| START  | 顶部，对 src 等比例缩放，将其放置在 dst 的左上角。 |
| END    | 底部，对 src 等比例缩放，将其放置在 dst 的右下角。 |
| FILL   | 充满，拉伸 src 的宽和高，使其完全填充满 dst。      |

- rectStaysRect

判断是矩形通过变换后是否仍为矩形，mapRect 方法的返回值就是根据这个方法来判断的

- setSinCos，控制 Matrix 旋转的，因为已经封装了 rotate 方法，因此这个方法不常用

### 矩阵相关

| 方法       | 摘要                                                 |
| ---------- | ---------------------------------------------------- |
| invert     | 求矩阵的逆矩阵                                       |
| isAffine   | 判断当前矩阵是否为仿射矩阵，API21(5.0)才添加的方法。 |
| isIdentity | 判断当前矩阵是否为单位矩阵。                         |
