# Nested Scrolling 嵌套滑动

## Nested Scroll 原理

- support.v4
  - NestedScrollingParent
  - NestedScrollingParentHelper
  - NestedScrollingChild
  - NestedScrollingChildHelper

|NestedScrollingChild|NestedScrollingParent|
|-|-|
|dispatchNestedPreScroll|onNestedPreScroll|
|dispatchNestedScroll|onNestedScroll|
|startNestedScroll|onStartNestedScroll|
|stopNestedScroll|onStopNestedScroll|

- NestedScrollingParent 的内部 View，滑动时，会先将 dx，dy 传入 NestedScrollingParent，其可以决定是否对滑动进行消耗，然后将剩余的部分交给子 View 处理。
- 子 View 回调 NestedScrollingParent 的各种方法是在 onTouchEvent 中进行的。

`ACTION\_DOWN` 调用了 startNestedScroll

`ACTION\_MOVE` 调用了 dispatchNestedPreScroll

`ACTION\_UP` 可能会触发 fling 以调用 resetTouch

## NestedScrollingParent

是一个接口，应该由 ViewGroup 实现。实现这个接口的类，需要创建一个 NestedScrollingParentHelper 实例

需要实现的方法有

- public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes);
  - 反映一个子 view 初始化一个可嵌套滑动的操作，如果需要进行父 view 的分发。
  - 当子 view 执行 ViewCompat#startNestedScroll() 方法时执行。view 的每一个父 view 都有机会来获取这次嵌套滑动事件，通过返回 true。
  - 一定要按照自己的需求返回true，该方法决定了当前控件是否能接收到其内部View(非并非是直接子View)滑动时的参数；假设你只涉及到纵向滑动，这里可以根据nestedScrollAxes这个参数，进行纵向判断。
- public void onNestedScrollAccepted(View child, View target, int nestedScrollAxes);
  - 反映请求到了嵌套滑动操作。
  - 在 onStartNestedScroll 返回 true 之后执行。提供了一个初始化嵌套滑动的时机。
  - 需要执行 super 方法。
- public void onStopNestedScroll(View target);
  - 反映嵌套滑动停止。
  - 执行清理动作。当滑动以 ACTION\_UP 或 ACTION\_CANCEL 结束的时候，会调用。
  - 需要执行 super 方法。
- public void onNestedScroll(View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed);
  - 反映一个正在进行中的嵌套滑动。
  - 调用时机为，父 view 的嵌套滑动子 view 分发了一个嵌套滑动事件。父 view 的 onStartNestedScroll() 方法必须返回 true，才能执行此方法。
  - 消费和未消费的嵌套滑动都会进行报告。
  - target 为控制嵌套滑动的子 view。
- public void onNestedPreScroll(View target, int dx, int dy, int[] consumed);
  - 在 target 消耗滑动之前，反映一个嵌套滑动。
  - 通常嵌套滑动的场景是父 view 希望在子 view 滑动之前可以消耗滑动距离。
  - 调用时机是子 view 调用 View#dispatchNestedPreScroll()，实现方法应该报告滑动了多少像素。
  - consumed 是水平和垂直方向上，父 view 消耗的距离。
- public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed);
  - 在嵌套滑动中请求 fling。
  - 反映一个嵌套滑动的子 view 检测到适合 fling 的条件。如果嵌套滑动的子 view 进行 fling 时候达到了边界，可以使用这个方法将 fling 发送到嵌套滑动的父 view。
  - target 表示初始化嵌套滑动的 view；vX，vY 分别是水平和垂直方向的滑动速度，像素每秒；consumed 表示子 view 是否消费 fling。
  - 返回 true 表示 parent 在 target 之前消费该次 fling。
- public boolean onNestedPreFling(View target, float volecityX, float velocityY);
  - 在目标 view 消耗掉之前，反映一个嵌套滑动。
  - target 表示初始化嵌套滑动的 view；vX，vY 分别是水平和垂直方向的滑动速度，像素每秒。
  - 返回 true 表示 parent 在 target 之前消费该次 fling。
- public int getNestedScrollAxes();
  - 返回当前嵌套滑动的轴。
  - ViewCompat#SCROLL\_AXI\_HORIZONTAL
  - ViewCompat#SCROLL\_AXI\_VERTICAL
  - ViewCompat#SCROLL\_AXI\_NONE
