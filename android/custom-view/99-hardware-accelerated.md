```
<application
    android:hardwareAccelerated="false"
...>
</application>
```

```
<application
    android:hardwareAccelerated="true">
    <activity ... />
    <activity android:hardwareAccelerated="false" />
</application>
```

```
getWindow().setFlags(
   WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
   WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED);
```

```
myView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
```

Note: 你可以关闭 View 级别的硬件加速，但是你不能在 View 级别开启硬件加速，因为它还依赖其他的设置

- 两种获取是否支持硬件加速的方式

```
View.isHardwareAccelerated()   //returns true if the View is attached to a hardware accelerated window.
Canvas.isHardwareAccelerated() //returns true if the Canvas is hardware accelerated
```

如果必须进行这样的验证，建议你在 draw 的代码块中使用：Canvas.isHardwareAccelerated()，因为如果一个 View 被 attach 到一个硬件加速的 Window 上，即使没有硬件加速的 Canvas，它也是可以被绘制的。比如：将一个 View 以 bitmap 的形式进行缓存