# Activity

## 启动 intent filter

```xml
<action android:name="android.intent.action.MAIN"/>
<category android:name="android.intent.category.LAUNCHER"/>
```

## startActivityForResult

目标 Activity 处理完成或失败后，调用 finish()，并在之前调用 setResult()。主 activity 会回调 onActivityResult() 方法。

## 生命周期

onCreate
onStart
onResume
onPause
onStop
onDestroy
onRestart

## 启动模式

`<activity android:launchMode="xxx" />`

- 任务栈：放置 activity 的容器，启动程序，默认创建一个放置 launcher activity 的栈
- 后台任务栈、前台任务栈。手机现实的是前台任务栈中的栈顶元素
- standard 每次启动都会创建一个新的实例
- singleTop 检查栈顶，如果已经存在，不会创建新的实例，调用 onNewIntent()
- singleTask 检查相同 taskAffinity 的栈是否存在，如果栈不存在，那么创建信的栈并创建新的实例；如果栈存在，检查栈中是否存在实例，如果存在，那么直接使用，调用 onNewIntent() 并且把它上边的 activity 都出栈。如果实例不存在，那么创建新的实例
- singleInstance 启用一个新的栈，只存放该 activity
