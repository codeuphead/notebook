# ViewPager

通过实现接口 PageTransformer 来实现切换动画，重写 transformPage(View view, float position) 方法，然后通过设置 setPageTransformer 来设置给 ViewPager

position 代表滑动状态

- 0 表示刚好移动到正中间
- 0-1 表示从左向右离开屏幕，移动停止时，等于1
- -1-0 表示从左向右进入屏幕，移动停止时，等于0