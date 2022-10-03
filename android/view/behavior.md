# Behavior

设置给 CoordinatorLayout 的子 View。子 View 设置了 Behavior 后，能对 touch event、window insets、measurement、layout、nested scrolling 等动作进行拦截。Design 库大部分功能都是借助 Behavior 进行实现的。

Behavior 无法独立工作，必须与调用的 CoordinatorLayout 子 View 进行绑定：`app:layout_behavior=""`

系统带的有：

- android.support.design.widget.AppBarLayout$ScrollingViewBehavior
- android.support.design.widget.BottomSheetBehavior
- android.support.design.transformation.FabTransformationScrimBehavior
- android.support.design.transformation.FabTransformationSheetBehavior
- android.support.design.behavior.HideBottomViewOnScrollBehavior

## BottomSheetBehavior 底部弹出

我们在 xml 中给目标 view 添加这个 app:layout_behavior="@string/bottom_sheet_behavior"

我们使用代码获取到这个 BottomSheetBehavior 对象， BottomSheetBehavior sheetBehavior = BottomSheetBehavior.from(shareView)

我们使用 mBottomSheetBehavior.setState(BottomSheetBehavior.STATE_COLLAPSED) 设置状态的方法就能控制展开还是折叠

### 状态

STATE_EXPANDED 展开状态，显示完整布局。

STATE_COLLAPSED 折叠状态，显示peekHeigth 的高度，如果peekHeight为0，则全部隐藏,与STATE_HIDDEN效果一样。

STATE_DRAGGING 拖拽时的状态

STATE_HIDDEN 隐藏时的状态

STATE_SETTLING 释放时的状态

### 注意

- BottomSheetBehavior 默认是显示折叠状态的
- app:behavior_peekHeight="0dp" 属性设置 view 折叠状态的高度
- 设置 behavior_peekHeight=0 后才能做到默认不显示底部卡片，或者用代码设置状态也可以做到默认不显示

### BottomSheetDialog

是对 BottomSheetBehavior 的封装，用一个 dialog 作为模板，绑定 layout 布局。功能更多，如可以支持拖拽到顶部，作为一个 Activity 展示。

```java
if (dialog == null) {
            dialog = new BottomSheetDialog(this);
            dialog.setContentView(R.layout.layout_book_list2);
            dialog.setCanceledOnTouchOutside(true);
            dialog.setCancelable(true);
        }
        dialog.show();
```

## 设置 Behavior

xml 中 app:layout_behavior 设置全限定名，或者当先包名下的类名。

代码中 CoordinatorLayout.LayoutParams.setBehavior();

通过注解为 View 绑定一个对应的 Behavior

```java
@CoordinatorLayout.DefaultBehavior(AppBarLayout.Behavior.class)
public class AppBarLayout extends LinearLayout {}
```

## 自定义 Behavior

一般自定义 behavior 有两个目的，一个是根据依赖 View 的位置进行相应的操作，另一个就是响应 CoordinatorLayout 中一些滑动组件的滑动事件。

### 依赖响应

如果一个 View 依赖另一个 View 需要下面三个 API

```java
public boolean layoutDependsOn(CoordinatorLayout parent, V child, View dependency) { return false; }

public boolean onDependentViewChanged(CoordinatorLayout parent, V child, View dependency) {  return false; }

public void onDependentViewRemoved(CoordinatorLayout parent, V child, View dependency) {}
```

当 layoutDependsOn 返回 true 时候，另外两个方法才会被回调。

onDependentViewChanged() 调用时， dependency View 的改变包括尺寸和位置的变化。如果重写这个方法，改变了 child 的尺寸和位置，需要返回 true，默认返回 false。

onDenpendentRemoved() 调用时，被调用时一般是 dependency 被它的 parent 移除，或者 child 设置了新的 anchor。

### 滑动响应

```java
public void onNestedScrollAccepted(CoordinatorLayout coordinatorLayout, V child,
        View directTargetChild, View target, int nestedScrollAxes) {}

public void onStopNestedScroll(CoordinatorLayout coordinatorLayout, V child, View target) {}

public void onNestedScroll(CoordinatorLayout coordinatorLayout, V child, View target,
        int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {}

public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, V child, View target,
        int dx, int dy, int[] consumed) {}

public boolean onNestedFling(CoordinatorLayout coordinatorLayout, V child, View target,
        float velocityX, float velocityY, boolean consumed) {}

public boolean onNestedPreFling(CoordinatorLayout coordinatorLayout, V child, View target,
        float velocityX, float velocityY) {}
```