# Matrix

## 概述

Matrix 矩阵 齐次坐标系

```java
MSCALE_X MSKEW_X  MTRANS_X
MSKEW_Y  MSCALE_Y MTRANS_Y
MPERSP_0 MPERSP_1 MPERSP_2
```

- 用数学方式，将变换放在 3x3 矩阵中表示，变换时候进行矩阵乘法，而不用进行矩阵加法
- 平移无法使用 2x2 矩阵的乘法表示
- 作用是坐标映射，开发时用的内容区坐标系，最后在绘制时都要转换为物理坐标系来绘制，即以屏幕左上角为原点，Matrix 作用就是转换这些数值
- Canvas 是对 Matrix 的封装，Matrix 更接近底层，操作比 Canvas 灵活
- Matrix 用的变换是*仿射变换*，仿射变换是线性变换和平移变换的复合
- 基本变换有 4 种，平移，缩放，旋转，错切
  - 旋转控制参数：`MSCALE_X、MSKEW_X、MSKEW_Y、MSCALE_Y`
  - 位移控制参数：`MTRANS_X、MTRANS_Y`
  - 缩放控制参数：`MSCALE_X、MSCALE_Y`
  - 错切控制参数：`MSKEW_X、MSKEW_Y`
  - `MPERSP_0、MPERSP_1、MPERSP_2`在 3D 变换中启动作用，透视，值同通常为(0,0,1)

## Matrix 复合原理

matrix 的多种复合操作都是使用矩阵乘法实现的，后面的操作可能会影响到前面的操作，所以顺序很重要

- Matrix 方法主要有 pre、post、set 三种
- 矩阵乘法不满足交换律，pre 相当于矩阵右乘；post 相当于矩阵左乘；set 是直接赋值，不进行运算
- 所有操作都是默认坐标原点为基准点的
- 之前操作的坐标系状态会保留，并且影响到后续状态
- pre 和 post 是用来调整乘法顺序的
- 初始矩阵是 new 出来的，或者调用 reset() 的矩阵，是一个单位矩阵

### 示例

设原始矩阵为 M，平移为 T，旋转为 R，单位矩阵为 I，最终结果为 M'

矩阵乘法不凡组交换律，即：A\*B ≠ B\*A 矩阵乘法满足结合律，即：(A\*B)\*C = A\*(B\*C) 矩阵与单位矩阵相乘，结果不变，即 A\*I = A

假设 M 是一个 单位矩阵 I pre 右乘，先 translate，后 rotate M' = (M\*T)\*R = I\*T\*R = T\*R

post 左乘，先 rotate，后 translate

M' = T\*(R\*M) = T\*R\*I = T\*R

两者虽然结果等价，但并不能说明 pre 是顺序执行， post 是逆序执行。

这里的 pre post 只是指代了矩阵 M 的相对位置，在目标左边，相当于右乘，是为 pre，反之。

### 如何使用

先构造正常的 Matrix 乘法顺序，之后根据情况使用 pre post 把这个顺序实现

例如：旋转

- 需要将坐标原点移动到指定位置，使用平移 T
- 对坐标系进行旋转 S
- 平移会原来的位置 -T

则公式为 `M' = M*T*R*-T = T*R*-T`

则代码为

```java
matrix.preTranslate(pivotX, pivotY)
matrix.preRotate(angle)
matrix.preTranslate(-pivotX, -pivotY)
```

拓展后，围绕某一点的通用情况为：

```java
matrix.preTranslate(pivotX, pivotY)
.
.
.
matrix.preTranslate(-pivotX, -pivotY)
```

公式为 `M' = M*T* ... *-T = T* ... *-T`

但这样一来，开始和结束的平移函数就被拉开了，因此通常采用：

```java
....各种操作
matrix.postTranslate(x, y)
matrix.preTranslate(-x,-y)
```

公式为 `M' = T*M* ... *-T = T* ... *-T`

其结果是相同的，所以说，pre 和 post 就是用来调整乘法顺序的。正常情况正向进行构建公式，之后根据实际情况调整书写即可。个人建议尽量使用一种乘法，容易确定操作顺序与排查问题

## Matrix 方法

| 方法类别 | 相关 API | 摘要 |
| --- | --- | --- |
| 前乘(pre) | preConcatpreRotatepreScalepreSkewpreTranslate | 前乘变换 |
| 后乘(post) | postConcatpostRotatepostScalepostSkewpostTranslate | 后乘变换 |
| 基本方法 | equalshashCodetoStringtoShortString | 比较、获取哈希值、转换为字符串 |
| 数值操作 | setresetsetValuesgetValues | 设置、重置、设置数值、获取数值 |
| 数值计算 | mapPointsmapRadiusmapRectmapVectors | 计算变换后的数值 |
| 特殊方法 | setPolyToPolysetRectToRectrectStaysRectsetSinCos | 一些特殊 操作 |
| 矩阵相关 | invertisAffineisIdentity | 求逆矩阵、是否为仿射矩阵、是否为单位矩阵... |
| 设置(set) | setConcatsetRotatesetScalesetSkewsetTranslate | 设置变换 |
