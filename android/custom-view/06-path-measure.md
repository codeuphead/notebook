# PathMeasure

```java
PathMeasure()
PathMeasure(Path path, boolean forceClosed)
```

公共方法

```java
void setPath(Path path, boolean forceClosed) //关联一个Path
boolean isClosed() //是否闭合
float getLength() //获取Path的长度
boolean nextContour() //跳转到下一个轮廓
boolean getSegment(float startD, float stopD, Path dst, boolean startWithMoveTo) //截取片段
boolean getPosTan(float distance, float[] pos, float[] tan) //获取指定长度的位置坐标及该点切线值
boolean getMatrix(float distance, Matrix matrix, int flags) //获取指定长度的位置坐标及该点Matrix
```

## 构造方法

无参的构造创建一个空的 PathMeasure，使用之前需要调用 setPath 来与 Path 进行关联。如果关联后 Path 内容进行了更改，需要重新关联。

有参构造是创建 PathMeasure 并关联一个 Path。第二个参数来确保 Path 闭合。

不会影响原有 Path 的状态

## setPath、isClosed、getLength

## getSegment

用于获取 Path 的一个片段

```java
boolean getSegment (float startD, float stopD, Path dst, boolean startWithMoveTo)
```

返回成功，代表截取成功，存入 dst 中，失败不会改变 dst 内容

startD，开始截取位置距离 Path 起点的长度

stopD，结束截取位置距离 Path 起点的长度

dst，截取的 Path 会添加到 dst 中，不是替换

startWithMoveTo，起始点是否使用 moveTo，用于保证截取的 Path 第一个点位置不变，否则会将起始点移动到 dst 的最后一个点，以保证 dst 的连续性

如果 startD、stopD 取值不在有效范围内 [0, getLength]，或者 startD == stopD，那么返回值未false，不会改变 dst 的内容

## nextContour

跳转到下一条线段，如果成功，返回true，否则 false。

例如创建一个 Path，其中包含了两个 闭合的曲线，可以对两条曲线进行测量。

曲线的顺序与 Path 中添加的顺序有关。

getLength 获取到的是当前一条曲线的 分长度，而不是整个 Path 的长度

getLength 是针对当前的曲线

## getPosTan

得到 Path 上某一长度的位置，以及该位置的正切值

`boolean getPosTan (float distance, float[] pos, float[] tan)`

distance：距离 Path 起点的长度。
pos，该点的坐标
tan，该点的正切值，用来判断这个位置上切线的方向，tan[0]，tan[1]，来描述切下逆方向，因此 tan[0] 相当于临边，tan[1] 相当于对边

## getMatrix

得到路径上某一长度的位置以及该位置的正切值的矩阵

`boolean getMatrix(float distance, Matrix matrix, int flags)`

返回 true 表示成功，数据会存入 matrix，false 失败，matrix 内容不会改变

distance：距离 path 起点长度，0 <= distance <= getLength

matrix：根据 flags 封装好的 matrix

flags：规定那些内容会存入到 matrix 中，可以选择：POSITION_MATRIX_FLAG、TANGENT_AMTRIX_FLAG，flag 可以用 `|`链接

## Path & SVG
