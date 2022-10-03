
# Path

关闭硬件加速，避免引起不必要的问题

|作用|相关方法|备注|
|---|---|---|
|移动起点|moveTo|移动下一次操作的起点位置|
|设置终点|setLastPoint|重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同|
|连接直线|lineTo|添加上一个点到当前点之间的直线到Path|
|闭合路径|close|连接第一个点连接到最后一个点，形成一个闭合区域|
|添加内容|addRect, addRoundRect, addOval, addCircle, addPath, addArc, arcTo|添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别)|
|是否为空|isEmpty|判断Path是否为空|
|是否为矩形|isRect|判断path是否是一个矩形|
|替换路径|set|用新的路径替换到当前路径所有内容|
|偏移路径|offset|对当前路径之前的操作进行偏移(不会影响之后的操作)|
|贝塞尔曲线|quadTo, cubicTo|分别为二次和三次贝塞尔曲线的方法|
|rXxx方法|rMoveTo, rLineTo, rQuadTo, rCubicTo|不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)|
|填充模式|setFillType, getFillType, isInverseFillType, toggleInverseFillType|设置,获取,判断和切换填充模式|
|提示方法|incReserve|提示Path还有多少个点等待加入**(这个方法貌似会让Path优化存储结构)**|
|布尔操作(API19)|op|对两个Path进行布尔运算(即取交集、并集等操作)|
|计算边界|computeBounds|计算Path的边界|
|重置路径|reset, rewind|清除Path中的内容，reset不保留内部数据结构，但会保留FillType rewind会保留内部的数据构，但不保留FillType|
|矩阵操作|transform|矩阵变换|

Path 封装了由曲线、直线构成的集合路径。其可以用 Canvas 中的 drawPath 将路径绘制出来，也可以用于裁剪画布、绘制文字。

## 直线

- moveTo
- setLastPoint：设置之前操作的最有一个点的位置，会影响之前的操作
- lineTo
- close：如果链接最后一个点和第一个点，仍然无法闭合图形，那么 close 什么也不会做

## addXXX 与 arcTo

- 第一类 基本形状

```java
// 第一类(基本形状)
// 圆形
public void addCircle (float x, float y, float radius, Path.Direction dir)
// 椭圆
public void addOval (RectF oval, Path.Direction dir)
// 矩形
public void addRect (float left, float top, float right, float bottom, Path.Direction dir)
public void addRect (RectF rect, Path.Direction dir)
// 圆角矩形
public void addRoundRect (RectF rect, float[] radii, Path.Direction dir)
public void addRoundRect (RectF rect, float rx, float ry, Path.Direction dir)
```

在 Path 中直接添加基本图形，其参数 Path.Direction 代表了方向：CW 顺时针；CCW 逆时针，其对各个点的闭合顺序以及图形渲染结果有影响

- 第二类 path

```java
// 第二类(Path)
// path
public void addPath (Path src)
public void addPath (Path src, float dx, float dy)
public void addPath (Path src, Matrix matrix)
```

将两个 Path 合并成一个，其中第二个是将 src 进行位移后添加；第三个是将当前 src 先进行 Matrix 转换，在进行添加。

- 第三类 addArc arcTo

```java
// 第三类(addArc与arcTo)
// addArc
public void addArc (RectF oval, float startAngle, float sweepAngle)
// arcTo
public void arcTo (RectF oval, float startAngle, float sweepAngle)
public void arcTo (RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo)
```

addArc：直接添加一个圆弧到 path 中

arcTo：添加一个圆弧到 path，如果圆弧的起点和上一个点不同，连接两点

arcTo 的 forceMoveTo，true：将左后一个点移动到圆弧起点，即不链接最后一个点与圆弧起点，等价于 addArc；false：链接这两个点，等价于 arcTo 第一个重载方法。

## isEmpty、isRect、isConvex、set、offset

- isEmpty：是否包含内容
- isRect：是否是一个矩形
- set：直接赋值
- offset：对 path 进行一段平移

## 贝塞尔曲线

- 一阶曲线

只有两个数据点，最终效果是一个线段

- 二阶曲线

两个数据点和一个控制点，控制点用来描述曲线状态

- 三阶曲线

两个数据点和两个控制点

---
>[贝塞尔曲线练习](http://bezier.method.ac/)

### 相关函数

//二阶
// x1 y1 是控制点，x2 y2 是结束点
public void quadTo(float x1, float y1, float x2, float y2)

//三阶
public void cubicTo(float x1, float y1, float x2, float y2, float x3, float y3)

### 核心难点

如何得到数据点和控制点

## rXXX 方法

rXXX 方法，基于当前点的位置，进行操作，没有 r 的方法使用的是 view 的坐标系

## 填充模式

|方法|作用|
|---|---|
|EVEN_ODD|奇偶规则|
|INVERSE_EVEN_ODD|反奇偶规则|
|WINDING|非零环绕数规则|
|INVERSE_WINDING|反非零环绕数规则|

## 布尔操作

用一些简单的图形，通过一些规则合成一些相对比较复杂，或者难以直接得到的图形

比如太极图的阴阳鱼，如果用贝塞尔曲线的话，可能需要6断，但可以通过4个 path 通过运算获得

布尔逻辑

||||
|---|---|---|
|DIFFERENCE|差集|Path1 中减去 Path2 后剩下的部分|
|REVERSE_DIFFERENCE|差集|Path1、Path2 互换|
|INTERSECT|交集|Path1 与 Paht2 相交的部分|
|UNION|并集|包含全部的 Path1 和 Path2|
|XOR|异或|包含 Path1 与 Path2，但不包含两者相较的部分|

```java
boolean op(Path path, Path.Op op)
boolean op(Path path1, Path path2, Path.Op op)
```

```java
// 对 path1 和 path2 执行布尔运算，运算方式由第二个参数指定，运算结果存入到path1中。
path1.op(path2, Path.Op.DIFFERENCE);

// 对 path1 和 path2 执行布尔运算，运算方式由第三个参数指定，运算结果存入到path3中。
path3.op(path1, path2, Path.Op.DIFFERENCE)
```

## 计算边界

主要计算 Path 所占用的空间以及所在的位置

```java
void computeBounds(RectF bounds, boolean exact)
```

bounds：测量结果放入这个矩形
exact：是否精确测量，一般写true，已废弃

## 重置路径

reset：保留 FillType
rewind：保留数据结构
