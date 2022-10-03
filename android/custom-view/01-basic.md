# 基础知识

## Android 坐标系

- 坐标系原点

左上角为原点 (0,0)，x 轴正方向向右，y 轴正方向向下

- 屏幕坐标系

屏幕的左上角为坐标原点

- View 坐标系

View 坐标系是相对于父 View 而言的，父 View 的左上角为坐标原点

```
//这两个相当于获取到子 View 的左上角的坐标
getTop();    //子 View 左上角距父 View 顶部的距离
getLeft();   //子 View 左上角距父 View 左侧的距离


//这两个相当于获取到子 View 的右下角的坐标
getBottom(); //子 View 右下角距父 View 顶部的距离
getRight();  //子 View 右下角距父 View 左侧的距离
```

- MotionEvent 中的坐标

```
event.getX();     //触摸点相对于其所在组件的坐标系的坐标
event.getY();

event.getRawX();  //触摸点相对于屏幕坐标系的坐标
event.getRawY();
```

## 角度 弧度

- 角度 60 进制，弧度 10 进制
- 角度为角正对的弧长，等于圆周的 360 分之一时，角度为 1 度
- 弧度为角正对的弧长，等于半径时，弧度为 1 度
- 因此，一个圆周对应的角度为 360 度，对应的弧度为 2派
- 换算

```
rad 为弧度，deg 为角度
rad = deg * pi / 180
deg = rad * 180 / pi
```

- 顺时针为角度增大的方向

## 颜色

- ARGB8888 四通道高精度 32位
- ARGB4444 四通道低精度 16位
- RGB555   屏幕默认     16位
- Alpha8   仅透明通道   8位

ARGB888

```
A 透明度 0-透明 255-不透明
R 红色   0-无色 255-红色
G
B
```

### 创建颜色

代码

```
//java 中定义的颜色
Color.GRAY;

Color.argb(127, 255, 0, 0)
color = 0xaaff0000;

#f00        //低精度，没有透明
#af00       //低精度
#ff0000     //高精度，没有透明
#aaff000000 //高精度

Color.parse("#ffffff")
getResource().getColor(R.color.red);
getColor(R.color.red);
```

当一个颜色绘制到 Canvas 上时，用的是混色模式计算方式：

`(RGB通道) 最终颜色 = 绘制的颜色 + (1 - 绘制颜色的透明度) × Canvas上的原有颜色`

- 每个通道的颜色从 0-255 映射至 0.1 的浮点数表示
- 这里等式右边的“绘制的颜色"、“Canvas上的原有颜色”都是经过预乘了自己的 Alpha 通道的值。

如绘制颜色：0x88ffffff，那么参与运算时的每个颜色通道的值不是1.0，而是(1.0 * 0.5333 = 0.5333)。 (其中0.5333 = 0x88/0xff)

使用这种方式的混合，就会造成后绘制的内容以半透明的方式叠在上面的视觉效果。

其实还可以有不同的混合模式供我们选择，用Paint.setXfermode，指定不同的PorterDuff.Mode。

详情搜索 Xfermode 模式的文章