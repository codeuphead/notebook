# CoordinatorLayout

`public class CoordinatorLayout extends ViewGroup implements NestedScrollingParent {}`

CoordinatorLayout 是用来协调其子 View 们之间动作的一个父 View，而 Behavior 就是用来给 CoordinatorLayout 的子 View 们实现交互的。

## AppBarlayout

AppBarLayout 是 LinearLayout 的子类，垂直方向，有默认的 Behavior，该 Behaivor 可以响应外部嵌套滑动事件，根据规则去伸缩和滑动内部的子 View。

可以定制在某个可滚动的 View 滚动手势发生变化时，内部子 View 相应的动作。

在它的子 View 添加 app:layout_scrollFlags 属性或调用 setScrollFlags()。

AppBarLayout 子 View 滚动标识：

- scroll：会跟着 ScrollView 一起滚动，其他 flag 必须与这个进行关联
- enterAlways：不消费滚动向下滚动的距离，跟随 ScrollView 一起向下滑动
- enterAlwaysCollapsed：需要配合 entryAlways，不消费向下滚动的距离，达到 minHeight 时候，不再滚动，直到 ScrollView 滚动完成，再向下滚动
- exitUntilCollapsed：向上滚动时，View 滚动到 minHeight 为止，固定在顶部
- snap：自动回滚效果，比如滚动到20%松手，就会恢复滚动。

方法：

- addOnOffsetChangedListener 添加偏移量监听，就是监听 AppBarLayout 的可见高度变化
- removeOnOffsetChangedListener 移除
- setExpanded 设置展开或者收缩
- generateLayoutParams 生成 LayoutParams。一般用不到
- setOrientation 不用关心的方法，方向只能是 vertical
- getTotalScrollRange 获取最大滚动偏移量
- setTargetElevation 设置 Z 轴高度

## CollapsingToolbarLayout

可以将 toolbar 的效果组合起来的布局，增强 Toolbar，可实现 toolbar 的折叠效果（需要 CoordinatorLayout AppBarLayout），其为 Toolbar 带来几个特性：

- Collapsing Title
- Content Scrim
- Status bar scrim
- Parallax scrolling children
- Pinned position children

### 属性

contentScrim：当CollapsingToolbarLayout完全折叠后的背景颜色。
expandedTitleMarginStart：布局张开的时候title与左边的距离。

子布局通过设置 layout_collapseMode，来实现不同的效果

- off 默认，无折叠行为
- pin CollapsingToobarLayout 折叠后，此布局固定在顶部
- parallax CollapsingToobarLayout 折叠后，此布局也会有视差折叠效果，还可设置 `layout_collapsePrarallaxMultipier` 设置视差滚动因子 0-1
