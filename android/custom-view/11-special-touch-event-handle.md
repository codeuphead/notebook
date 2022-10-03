# 特殊形状 View 的点击区域判断

Region 类，与 Path 类似，Path 可以不封闭，但 Region 总是封闭的。可通过 setPath 方法将 Path 转换为 Region

判断的关键是使用 Region 中的 contains 方法，这个方法可以判断一个点是否包含在该区域内。

示例：判断手指是否在圆形区域按下：

```java
public class RegionClickView extends CustomView {
    Region circleRegion;
    Path circlePath;

    public RegionClickView(Context context) {
        super(context);
        mDeafultPaint.setColor(0xFF4E5268);
        circlePath = new Path();
        circleRegion = new Region();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        // ▼在屏幕中间添加一个圆
        circlePath.addCircle(w/2, h/2, 300, Path.Direction.CW);
        // ▼将剪裁边界设置为视图大小
        Region globalRegion = new Region(-w, -h, w, h);
        // ▼将 Path 添加到 Region 中
        circleRegion.setPath(circlePath, globalRegion);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                int x = (int) event.getX();
                int y = (int) event.getY();

                // ▼点击区域判断
                if (circleRegion.contains(x,y)){
                    Toast.makeText(this.getContext(),"圆被点击",Toast.LENGTH_SHORT).show();
                }
                break;
        }
        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // ▼注意此处将全局变量转化为局部变量，方便 GC 回收 canvas
        Path circle = circlePath;
        // 绘制圆
        canvas.drawPath(circle,mDeafultPaint);
    }
}
```

## 画布变换后的坐标转换问题

```java
//其实核心部分就这两点：

// ▼ 注意此处使用 getRawX，而不是 getX
down_x = event.getRawX();
down_y = event.getRawY();

// -------------------------------------

// ▼ 获得当前矩阵的逆矩阵
Matrix invertMatrix = new Matrix();
canvas.getMatrix().invert(invertMatrix);

// ▼ 使用 mapPoints 将触摸位置转换为画布坐标
invertMatrix.mapPoints(pts);
```

- 使用全局坐标系
- 使用逆矩阵的 mapPoints

原理嘛，其实非常简单，我们在画布上正常的绘制，需要将画布坐标系转换为全局坐标系后才能真正的绘制内容。所以我们反着来，将获得到的全局坐标系坐标使用当前画布的逆矩阵转化一下，就转化为当前画布的坐标系坐标了，如果对 Matrix原理 和 Matrix详解 理解了，即便我不说你们也肯定会想到这个方案的。
