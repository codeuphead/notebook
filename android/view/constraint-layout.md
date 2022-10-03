# Cosntraint Layout

- `compile 'com.android.support.constraint:constraint-layout:1.1.3'`
- android.support.constraint
  - Barrier
  - ConstraintLayout
  - ConstraintLayout.LayoutParams
  - ConstraintProperties
  - ConstraintsChangedListener
  - ConstraintSet
  - Group
  - Guideline
  - Placeholder

- 每一个约束代表了该 View 和其他 View 的联系，需要定义 View 水平或垂直轴向的约束。
- Android Studio 可以将一个现存的布局文件转换成为 ConstraintLayout：在 ComponentTree 中右键，有 Convert xxx to ConstraintLayout 选项。
- 必须添加水平、垂直两个轴向的约束，否则默认位置为(0,0)。
- 不可有循环依赖。
- View 的宽高有3种模式：WrapContent、Fixed、MatchConstraints。

## 相对位置

![Relative Positioning](http://lovehls.cn/Lychee/uploads/big/671a3098f648708d90af713d5d457cf7.png)

依据另一个 View 的任意边，来约束 View 的一个边。

- 水平方向：Left，Right，Start，End
- 垂直方向：Top，Bottom，Baseline

![Relative Positioning Constraints](http://lovehls.cn/Lychee/uploads/big/198225a6e6c6cb0f9d0e95ded4c1e561.png)

```xml
layout_constraintBottom_toBottomOf|toTopOf
layout_constraintTop_toBottomOf|toTopOf
layout_constraintLeft_toLeftOf|toRightOf
layout_constraintRight_toLeftOf|toRightOf
layout_constraintStart_toEndOf|toStartOf
layout_constraintEnd_toEndOf|toStartOf
layout_constraintBaseline_toBaselineOf
```

## Margin

![Relative Positioning Margin](http://lovehls.cn/Lychee/uploads/big/43db50b342ababb9dc2c3194edae688c.png)

- margin 必须是正数或者0
- 当约束目标的 Visibility 是 View.GONE 时，可以设置不一样的偏移量，通过 `goneMarginXXX`。

```xml
layout_marginStart/End/Top/Bottom/Left/Right
layout_goneMarginStart/End/Top/Bottom/Left/Right
```

## 居中和偏差

除非 ConstraintLayout 和子 View 一样大，否则同一平面的两个约束不会同时满足，如图。

![Centering Positioning](http://lovehls.cn/Lychee/uploads/big/fead83c672450683a2cef8b815864d1c.png)

默认的，相反方向的约束，会把控件在该方向居中。

可使用偏差值来进行设定，偏差值代表剩余空间的百分比

```xml
- layout_constraintHorizontal_bias
- layout_constraintVertical_bias
```

![Centering Positioning Bais](http://lovehls.cn/Lychee/uploads/big/17db53d65509d624b4b890dd2401e6e3.png)

## 环形位置

约束一个控件的中心相对于另一个控件的中心，设置角度和距离。

```xml
layout_constraintCircle:其他控件id
layout_constraintCircleRadius:圆心距离
layout_constraintCircleAngle:角度（0-360）
```

![circle 1](http://lovehls.cn/Lychee/uploads/big/014e4414c5de98ebe255fb877f4a50af.png)

![circle 2](http://lovehls.cn/Lychee/uploads/big/4e732a09dca07c7a10b0b03007576dfb.png)

## Visibility 行为

ConstraintLayout 对于一个 view 设置 View.GONE 时有特殊的处理。

在设置 View.GONE 设置之后 View 不再显示，并且不再属于父容器，其真实尺寸不会改变。

但是在布局计算中 View 依然存在布局中，尺寸会被当作为0，如果有相对其他 view 的约束，约束依然有效，但此时任何方向的 margin 不会处理。

![Visibility Behavior](http://lovehls.cn/Lychee/uploads/big/2e1a0771bd7d361ea7fe263bedd59f90.png)

这种特性，可以使 view 暂时性的消失，但并不会破坏布局。特别对于简单的布局动画来说。

对于 margin 的调整，如果其他 view 对于该 gone view 有 margin，但是在这种情况下并不适用，可以使用 goneMargin 进行设置。

## 尺寸约束

### ConstraintLayout 最小尺寸

ConstraintLayout 自身可以设置 `android:minWith`、`android:minHeight`、`android:maxWidth`、`android:maxHeight`，前提是，其宽高属性为 `wrap_content`。

### 控件尺寸约束

- 可以设置三种模式
  - 指定的尺寸：具体的 dp 值或者尺寸引用。
  - WRAP_CONTENT：以内容大小为准。
  - 0dp(MATCH_CONSTRAINT)：表示按照约束进行设置，对齐至约束的边界(类似 match_parent)。
- 不支持 `MATCH_PARENT`，可以设置成 0dp 后，设置上下左右的约束至 parent 来做。

![Dimension Match Constraint](http://lovehls.cn/Lychee/uploads/big/9253c2f22be38fe51a4f8fec45615dd1.png)

### WRAP_CONTENT 强制约束

使用 WRAP_CONTENT 但是也想用约束限制其尺寸，使用以下两个属性

```xml
layout_constrainedWidth="true|false"
layout_constrainedHeight="true|false"
```

### MATCH_CONSTRAINT 尺寸

当设置了 MATCH_CONSTRAINT 可设置以下属性进行扩展

```mxl
layout_constraintWidth_min/max/percent
layout_constraintHeight_min/max/percent


min/max 设置 dp 值 或者 wrap，wrap 相当于 WRAP_CONTENT
percent 设置其大小相对于父容器的百分比，0-1

```

### 百分比尺寸

- 尺寸设置为 MATCH_CONSTRAINT (0dp)
- 设置上面的百分比值，0-1

## Ratio 比例

设置宽高比例，以其中给一个为基准。

- 至少有一个尺寸的约束为 0dp（宽或高），然后设置属性 `layout_constraintDimentionRatio`
- 值是 float，格式是 width:height。
- 如果两边都设置为 0dp，此时，系统会在能满足所有约束并保持比例的情况下，设置最大的尺寸。这种情况下，如果想要设置一边作为基准，可以在属性前添加 `H|W`，用来指定哪个方向为要约束的方向。

```xml
<Button android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="H,16:9"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintTop_toTopOf="parent"/>
```

例子中，设置的高度为 16：9，宽度为父对齐。

## Chains 约束链

![Chains](http://lovehls.cn/Lychee/uploads/big/27e2763c05488f511259af6afa13a311.png)

- 约束链会将一组 view 在横向或纵向进行排列，另外一个轴向可以独立进行约束。
- 如果多个 view 进行双向链接，如上图，会当作一个约束链。

![Chains Head](http://lovehls.cn/Lychee/uploads/big/156ae3775da05c589450151285c8c719.png)

- 约束链通过链首的 view 来控制约束链的式样。通常是约束链中第一个元素，靠左靠上。
- 约束链也会计算上 view 的 margin
- 如果是 spread 的约束链，分配空间是扣除了 margin 的空间。
- 一个 view 可以同时设置水平和竖直的约束链。
- 一个约束链只有两两 view 相互约束才会正确作用。

### Chains styles 链式样

![Chains Styles](http://lovehls.cn/Lychee/uploads/big/a206593f5b420d0873dc92c5b95f01d1.png)

- 设置链中第一个元素的属性 `layout_constraintHorizontal_chainStyle / layout_constraintVertical_chainStyle`
- `CHAIN_SPREAD` 默认式样，每个 view 平均分布，剩余空间也平均分布。
- `CHAIN_SPREAD_INSIDE`：链两端不会有空间分配。
- `CHAIN_PACKED`：view 会打包在一起，可以通过改变约束链首的bias，来调整整个链的bias。

### Weighted chains

设置成 spread 模式时，如果 view 设置的是 `MATCH_CONSTRAINT`，那么该 view 会占据剩下的空间。

默认是平均分配给所有符合条件的 view，不过可以通过 `layout_constraintHorizontal_weight / layout_constraintVertical_weight` 属性进行设置。

### Margins and chains

view 相互之间的 margin 都会计算在内。

剩余空间不包含边距，计算 view 的位置时候，会将边距包括在内。

## Virtual Helper Class 虚拟辅助类

### Guideline

- 默认宽/高为0，另一边长度为 parent 的尺寸。
- 默认不显示 View.GONE，setVisibility 是空实现。
- 相当于一个 size 为 0 的 view 的封装，具有几个特殊属性，因此使用起来比较方便。

Guideline xml属性

```
layout_constraintGuide_begin (dimension)
layout_constraintGuide_end (dimension)
layout_constraintGuide_percent (0-1)
orientation

ConstraintSet.setGuidelineBegin(int, int)
ConstraintSet.setGuidelineEnd(int, int)
ConstraintSet.setGuidelinePercent(int, int)
```

```xml
<android.support.constraint.Guideline
    android:id="@+id/guideline"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintGuide_begin="212dp"
    android:orientation="horizontal"/>
```

### Barrier 边界

设置 barrierDirection 定义边界方向，constraint_referenced_ids 定义引用的控件 id。

基于指定方向的最极端的控件，创建虚拟的边界。如果控件的尺寸改变，边界也会依据最极端的控件机型改变。

然后其他的控件可以被约束至边界，而不是某个控件。从而实现布局自动适应某些控件的尺寸变化，而导致其他控件变化错误的问题。

```xml
<android.support.constraint.Barrier
    android:id="@+id/barrier"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:barrierDirection="start"
    app:constraint_referenced_ids="button1,button2" />
```

### Group 组

控制引用的空间显隐状态，通过逗号分隔控件 id 进行引用。Gourp 的 visibility 会应用到引用的控件上，可以通过代码，非常方便地控制多控件的显隐。

多个 Group 可以引用同样的控件，后声明的有效

```xml
<android.support.constraint.Group
    android:id="@+id/group"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="visible"
    app:constraint_referenced_ids="button4,button9" />
```

## 优化

设置 `layout_optimizationLevel` 进行优化

- none
- standard：默认，优化直接的和 barrier 约束
- direct：优化直接约束
- barrier：优化 barrier 约束
- chain：优化 chain 约束（实验）
- dimension：优化尺寸测量，减少测量次数（实验）

`app:layout_optimizationLevel="direct|barrier|chain"`
