# 2D 绘制 阴影

## Paint

public void setShadowLayer(float radius, float dx, float dy, int shadowColor) // 阴影层，radius 模糊度，dx，dy 位移，shadowColor，阴影颜色

文字阴影可以硬件加速，图形阴影需要关闭硬件加速 `setLayerType(LAYER_TYPE_SOFTAWRE, null);`

图片图形的阴影，近模糊边界部分

public void clearShadowLayer() 清除阴影

## XML 设置

```xml
<TextView
        android:shadowRadius="3"
        android:shadowDx="5"
        android:shadowDy="5"
        android:shadowColor="@android:color/darker_gray"/>
```

- shadowColor：阴影颜色
- shadowDx：水平偏移量
- shadowDy：垂直偏移量
- shadowRadius：阴影范围，数值越大，阴影越大、模糊、软化

## 特性

- Dx、Dy 可以改变光源方向
- 浮雕效果：小半径的白色阴影和小偏移量，加上金属背景。
- 光芒效果：偏移量为0，较大白半径，颜色和文字相匹配。
- 外发光效果：文字颜色和背景色一样，阴影颜色形成对比效果，偏移量为0。改变阴影颜色就能改变发光效果。