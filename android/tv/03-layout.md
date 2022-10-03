# 布局相关

## Basic

- 通常使用 landscape orientation
- 屏幕上的导航通常放在屏幕左右两侧，节省上下方向的空间
- 使用 Fragments 创建 UI，使用 GridView，在横向上有更好的体验
- 使用 RelativeLayout 或者 LinearLayout 来布局视图
- 添加足够的 margin

## 主题

使用 `@style/Theme.Leanback` 主题，该主题不带 ActionBar，如果使用 BrowseSupportFragment，则必须配套使用 FragmentActivity。

如果不使用以上主题，需要使用 NoTitleBar 主题。

[参考](https://developer.android.com/guide/topics/ui/themes.html)

## 安全区

左右 48dp，上下 27dp （5%）。

leanback 库的组件不用设置安全区，默认有。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   >

   <!-- Screen elements that can render outside the overscan safe area go here -->

   <!-- Nested RelativeLayout with overscan-safe margin -->
   <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:layout_marginTop="27dp"
       android:layout_marginBottom="27dp"
       android:layout_marginLeft="48dp"
       android:layout_marginRight="48dp">

      <!-- Screen elements that need to be within the overscan safe area go here -->

   </RelativeLayout>
</RelativeLayout>
```

## 文字

- 文本分成小块方便使用
- 使用深色北京，浅色文字
- 使用 sans-serif fonts and anti-aliasing 字体
- 使用标准 Android 的字体大小
- 确保组件足够大

```java
<TextView
      android:id="@+id/atext"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:gravity="center_vertical"
      android:singleLine="true"
      android:textAppearance="?android:attr/textAppearanceMedium"/>
```

## 资源

使用高质量 1080p 屏幕 的图片，9-patch 图片。

[参考](https://developer.android.com/studio/write/draw9patch.html)

## 禁用

- 复用移动端的布局，TV 布局需要简单
- ActionBar，控制器不好控制
- ViewPage，整屏切换内容在 TV 上不可行

[设计建议](https://developer.android.com/design/tv/index.html)

## 处理大的位图

- 只在使用时加载图片，如 listView 中的 getView()
- 不使用时调用 `recycle()`
- 使用 WeakReference 来在内存中存储图片缓存
- 使用线程加载图片，并存储到设备中
- 下载时缩小大图片到合适尺寸，否则会在下载中造成内存溢出

[参考](https://developer.android.com/topic/performance/graphics/index.html)