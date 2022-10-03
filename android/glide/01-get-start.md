# Glide

- dependencies

```
compile 'com.github.bumptech.glide:glide:4.7.1'
annotationProcessor 'com.github.bumptech.glide:compiler:4.7.1'
```

- ProGuard

```
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public class * extends com.bumptech.glide.AppGlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
  **[] $VALUES;
  public *;
}
```

- DexGuard only

```java
-keepresourcexmlelements manifest/application/meta-data@value=GlideModule
```

## 基本用法

```java
// 加载图片
Glide.with(fragment)
  .load(url)
  .into(imageView);

// 取消加载
Glide.with(fragment).clear(imageView);
```

当 `Glide.with()` 中传入的 `Activity` 或 `Fragment` 实例销毁时，`Glide` 会自动取消加载并回收资源。

## AppGlideModule

在应用程序中，可创建一个 `@GlideModule` 注解的类，该类需要继承 `AppGlideModule`。此类可生成一个流式 API，内联了多种选项，集成库中自定义的选项：

```java
package com.example.myapp;

import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}
```

生成的 API 默认名为 `GlideApp` ，其与创建的 `AppGlideModule` 的子类的包名相同。

在程序中中将 `Glide.with()` 替换为 `GlideApp.with()`，即可使用该 API 去完成加载工作。

```java
GlideApp.with(fragment)
   .load(myUrl)
   .placeholder(placeholder)
   .fitCenter()
   .into(imageView);
```

## ListView & RecyclerView

在 `ListView` 或 `RecyclerView` 中加载图片的代码和在单独的 `View` 中加载完全一样。Glide 已经自动处理了 `View` 的复用和请求的取消：

```java
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    String url = urls.get(position);
    Glide.with(fragment)
        .load(url)
        .into(holder.imageView);
}
```

对 url 进行 null 检验并不是必须的，如果 url 为 null，Glide 会清空 View 的内容，或者显示 `placeholder Drawable` 或 `fallback Drawable` 的内容。

Glide 唯一的要求是，对于任何可复用的 `View` 或 `Target` ，如果它们在之前的位置上，用 Glide 进行过加载操作，那么在新的位置上要去执行一个新的加载操作，或调用 `clear()` API 停止 Glide 的工作。

```java
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    if (isImagePosition(position)) {
        String url = urls.get(position);
        Glide.with(fragment)
            .load(url)
            .into(holder.imageView);
    } else {
        Glide.with(fragment).clear(holder.imageView);
        holder.imageView.setImageDrawable(specialDrawable);
    }
}
```

对 View 调用 `clear()` 或 `into(View)`，表明在此之前的加载操作会被取消，并且在方法调用完成后，Glide 不会改变 view 的内容。如果你忘记调用 `clear()`，而又没有开启新的加载操作，那么就会出现这种情况，你已经为一个 view 设置好了一个 Drawable，但该 view 在之前的位置上使用 Glide 进行过加载图片的操作，Glide 加载完毕后可能会将这个 view 改回成原来的内容。

这里的代码以 `RecyclerView` 的使用为例，但规则同样适用于 `ListView`。

## 非 View 目标

除了将 `Bitmap` 和 `Drawable` 加载到 `View` 之外，你也可以开始异步加载到你的自定义 `Target` 中：

```java
Glide.with(context)
  .load(url)
  .into(new SimpleTarget<Drawable>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<Drawable> transition) {
      // Do something with the Drawable here.
    }
  });
```

使用自定义 `Target` 有一些陷阱，所以请务必阅读详细内容。

## 后台线程

在后台线程加载图片直接使用 `submit(int, int)`：

```java
FutureTarget<Bitmap> futureTarget =
  Glide.with(context)
    .asBitmap()
    .load(url)
    .submit(width, height);

Bitmap bitmap = futureTarget.get();

// Do something with the Bitmap and then when you're done with it:
Glide.with(context).clear(futureTarget);
```

如果你不想让 `Bitmap` 和 `Drawable` 自身在后台线程中，你也可以使用和前台线程一样的方式来开始异步加载：

```java
Glide.with(context)
  .asBitmap()
  .load(url)
  .into(new Target<Bitmap>() {
    ...
  });
```