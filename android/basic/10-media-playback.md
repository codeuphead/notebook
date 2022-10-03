# Media Playback

## 播放文件

- 通常使用 MediaPlayer 类来实现。
  - setDataSource()
  - prepare()
  - start()
  - pause()
  - reset()   将 mediaPlayer 对象重置到刚刚创建的状态
  - seekTo()
  - stop()    停止播放音频，调用该方法后，mediaPlayer 对象无法再次播放音频
  - release() 释放资源
  - isPlaying()   是否在播放中
  - getDuration()
