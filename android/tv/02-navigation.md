# 导航

- 确保可以导航至所有显示的控制元素
- 确保滚动列表，确认键可以选择列表项。选择后依然可以滚动
- 确保可以切换控制

## 修改方向导航

Android 框架自动为布局提供了方向导航，基于可获取焦点的元素的相对位置。也可以指定特定的导航方向，通常在系统提供的导航无法满足要求或者不正常工作的时候进行。

- nextFocusDown
- nextFocusLeft
- nextFocusRight
- nextFocusUp

显式声明导航属性，最好将导航作为一个循环。

## 清晰的焦点和选择状态

颜色、动画、尺寸来进行焦点的区别。

[参考](https://developer.android.com/guide/topics/resources/drawable-resource.html#StateList)

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
    <item android:state_focused="true"
          android:drawable="@drawable/button_focused" /> <!-- focused -->
    <item android:state_hovered="true"
          android:drawable="@drawable/button_focused" /> <!-- hovered -->
    <item android:drawable="@drawable/button_normal" /> <!-- default -->
</selector>
```