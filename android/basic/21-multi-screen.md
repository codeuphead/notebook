# 多窗口

切换多窗口莫时候，两个 Activity 都会ingli重建的过程。而没有获焦点的 Activity 在执行 onResume 后会执行 onPause，而获得焦点的 Activity 会执行 onResume 后获得活动状态。

切换窗口焦点后，onPause 和 onResume 会进行交换。

configChanges 设置 orientation|keyboardHidden|screenSize|screenLayout 可以禁止 Activity 重建

禁用多窗口模式，需要在 `<application>` 标签下设置 `android:resizeableActivity="true"|"false"`