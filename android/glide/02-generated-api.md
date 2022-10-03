# Generated API

## 简介

Glide v4 使用注解处理器 (Annotation Processor) 来生成出一个 API，在应用中可使用该流式 API 一次性调用到 `RequestBuilder`， `RequestOptions` 和集成库中所有的选项。

Generated API 模式的设计出于以下两个目的：

- 集成库可以为 Generated API 扩展自定义选项
- 在应用中可将常用的选项组打包成一个选项在 Generated API 中使用

虽然以上所说的工作均可以通过手动创建 `RequestOptions` 子类的方式来完成，但使用起来更复杂，并且降低了 API 流式调用。

## 有效使用范围

Generated API 目前仅可以在应用内使用。这一限制可以让我们仅持有一份 Generated API，而不是各个 Library 和 Application 中均有自己定义出来的 Generated API。这一做法会让 Generated API 的调用更简单，并确保应用中 Generated API 调用的选项在各处行为一致。这一限制在接下来的版本中也许会被取消（以实验性或其他的方式给出）。

## 生成

- 添加 Glide 注解处理器的依赖：

```java
dependencies {
  annotationProcessor 'com.github.bumptech.glide:compiler:4.6.1'
}
```

- 在应用中包含一个 `AppGlideModule` 的实现：

```java
package com.example.myapp;

import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}
```

你不必去重写 `AppGlideModule` 中的任何一个方法。子类中完全可以不用写任何东西，它只需要继承 `AppGlideModule` 并且添加 `@GlideModule` 注解。

`AppGlideModule` 的实现必须使用 `@GlideModule` 注解标记。如果注解不存在，该 module 将不会被 Glide 发现，并且在日志中收到一条带有 Glide tag 的警告，表示 module 未找到。

- 程序库 (Library) 不 应该包含 AppGlideModule 实现，详见 配置。

## 使用 Generated API

Generated API 默认名为 `GlideApp`，与应用中 `AppGlideModule` 的子类包名相同。在应用中将 `Glide.with()` 替换为 `GlideApp.with()`，即可使用该 API 去完成加载工作：

```java
GlideApp.with(fragment)
   .load(myUrl)
   .placeholder(R.drawable.placeholder)
   .fitCenter()
   .into(imageView);
```

与 `Glide.with()` 不同，诸如 `fitCenter()` 和 `placeholder()` 等选项在 `Builder` 中直接可用，并不需要再传入单独的 `RequestOptions` 对象。

## GlideExtension

Glide Generated API 可在 Application 和 Library 中被扩展。扩展使用被注解的静态方法来添加新的选项、修改现有选项、甚至添加额外的类型支持。

`@GlideExtension` 注解用于标识一个扩展 Glide API 的类。任何扩展 Glide API 的类都必须使用这个注解来标记，否则其中被注解的方法就会被忽略。

被 `@GlideExtension` 注解的类应以工具类的思维编写。这种类应该有一个私有的、空的构造方法，应为 final 类型，并且仅包含静态方法。被注解的类可以含有静态变量，可以引用其他的类或对象。

在应用中可以根据需求实现任意多个被 `@GlideExtension` 注解的类，在 Library 模块中同样如此。当 `AppGlideModule` 被发现时，所有有效的 Glide 扩展类会被合并，所有的选项在 API 中均可以被调用。合并冲突会导致 Glide 的 Annotation Processor 抛出编译错误。

被 @GlideExtention 注解的类有两种扩展方式：

1. `GlideOption` - 为 `RequestOptions` 添加一个自定义的选项。
1. `GlideType` - 添加对新的资源类型的支持(GIF，SVG 等等)。

### GlideOption

用 `@GlideOption` 注解的静态方法用于扩展 `RequestOptions` 。

`GlideOption` 可以：

- 定义一个在应用中频繁使用的选项集合。
- 创建新的选项，通常与 Glide 的 `Option` 类一起使用。

```java
@GlideExtension
public class MyAppExtension {
  // Size of mini thumb in pixels.
  private static final int MINI_THUMB_SIZE = 100;

  private MyAppExtension() { } // utility class

  @GlideOption
  public static void miniThumb(RequestOptions options) {
    options
      .fitCenter()
      .override(MINI_THUMB_SIZE);
  }
```

这将会在 RequestOptions 的子类中生成一个方法，类似这样：

```java
public class GlideOptions extends RequestOptions {

  public GlideOptions miniThumb() {
    MyAppExtension.miniThumb(this);
  }

  ...
}
```

你可以为方法任意添加参数，但要保证第一个参数为 `RequestOptions`。

```java
@GlideOption
public static void miniThumb(RequestOptions options, int size) {
  options
    .fitCenter()
    .override(size);
}

// 在自动生成的方法中新添的参数同样被加了进来：
public GlideOptions miniThumb(int size) {
  MyAppExtension.miniThumb(this);
}
```

之后你就可以使用生成的 `GlideApp` 类调用你的自定义方法：

```java
GlideApp.with(fragment)
   .load(url)
   .miniThumb(thumbnailSize)
   .into(imageView);
```

使用 `@GlideOption` 标记的方法应该为静态方法，并且返回值为空。

请注意，这些生成的方法在一般的 `Glide` 和 `RequestOptions` 类里不可用。

### GlideType

被 `@GlideType` 注解的静态方法用于扩展 `RequestManager` 。被 `@GlideType` 注解的方法允许你添加对新的资源类型的支持，包括指定默认选项。

例如，为添加对 GIF 的支持，你可以添加一个被 `@GlideType` 注解的方法：

```java
@GlideExtension
public class MyAppExtension {
  private static final RequestOptions DECODE_TYPE_GIF = decodeTypeOf(GifDrawable.class).lock();

  @GlideType(GifDrawable.class)
  public static void asGif(RequestBuilder<GifDrawable> requestBuilder) {
    requestBuilder
      .transition(new DrawableTransitionOptions())
      .apply(DECODE_TYPE_GIF);
  }
}
```

这样会生成一个包含对应方法的 `RequestManager` ：

```java
public class GlideRequests extends RequesetManager {

  public RequestBuilder<GifDrawable> asGif() {
    RequestBuilder<GifDrawable> builder = as(GifDrawable.class);
    MyAppExtension.asGif(builder);
    return builder;
  }

  ...
}
```

之后你可以使用生成的 `GlideApp` 类调用你的自定义类型：

```java
GlideApp.with(fragment)
  .asGif()
  .load(url)
  .into(imageView);
```

被 `@GlideType` 标记的方法必须使用 `RequestBuilder <T>` 作为其第一个参数，这里的泛型 <T> 对应 `@GlideType` 注解中传入的类。

该方法应为静态方法，且返回值为空。方法必须定义在一个被 `@GlideExtension` 注解标记的类中。