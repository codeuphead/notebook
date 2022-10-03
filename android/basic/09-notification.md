# Notification

## NotificationManager

- Context getSystemService(Context.NOTIFACTION_SERVICE)
- 使用 Builder 创建，通常使用 NotificationCompat.Builder
  - setContentTitle() setContentText() setWhen() setSmallIcon() setLargeIcon()
  - smallIcon 只能使用纯 alpha 图层的图片进行设置；largeIcon 当通知被拉下，就能显示出来
  - setContentIntent()
  - setAutoCancel()
  - setSound() 接收通知的时候发出的声音
  - setVibrate()
  - setLights()
  - setDefaults(NotificationCompat.DEFAULT_ALL)
- 调用 NotifactionManager 的 notify() 方法
- PendingIntent.getActivity()/getService()/getBroadcast()

## 高级用法

- setStyle() 接收一个 NotificationCompat.Style 参数。
- setPriority() PRIORITY\_DEFAULT PRIORITY\_MIN PRIORITY\_LOW PRIORITY\_HIGH PRIORITY\_MAX
