# Alarm

## AlarmManager

`set(int timeType, long delayTime, PendingIntent pi)`

Context

`public AlarmManager getSystemService(Context.ALARM_SERVICE)`

第一个参数代表 AlarmManager 的工作模式

```
ELAPSED_REALTIME // 时间从开机算起，不会唤醒 CPU
ELAPSED_REALTIME_WAKEUP // 时间从开机算起，会唤醒 CPU
RTC // 从1970年1月1日0点开始算起，不会唤醒 CPU
RTC_WAKEUP // 从1970年1月1日0点开始算起，会唤醒 CPU
```

使用`SystemCLock.elapsedRealtime()`方法可以获取到系统开机至今的毫秒数

使用`System.currentTimeMillis()`方法可以获取到1970年1月1日0点至今的毫秒数

系统会检测目前有多少 Alarm 存在，然后将触发时间相近的几个任务放在一起执行

使用`setExact()`方法保证正确执行

## Doze 模式

6.0 如果设备未接入电源，处于静止状态（7.0移除），屏幕关闭了一段时间后，就进入到 Doze 模式。系统会间歇性的退出 Doze 模式，在这段事件中，应用可以完成它们的同步操作，Alarm 任务等。

随着时间拉长，Doze 模式的间隔也会越长。

受限制功能：禁止访问网络、忽略唤醒 CPU 或屏幕的操作、不再执行 WIFI 扫描、不再执行同步任务、Alarm 任务将会在下次退出 Doze 模式时候执行。

Doze 模式下，Alarm 会变得不准时。如果有特殊要求，可以设置 AlarmManager#setAndAllowWhileIdle() 或 setExactAndAllowWhileIdle() 就可以让 Alarm 在 Doze 模式下正常执行了。