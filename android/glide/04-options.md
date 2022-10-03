# Options

## RequestOptions

Glide中的大部分设置项都可以通过 `RequestOptions` 类和 `apply()` 方法来应用到程序中。

使用 `request options` 可以实现（包括但不限于）：

- 占位符(Placeholders)
- 转换(Transformations)
- 缓存策略(Caching Strategies)
- 组件特有的设置项，例如编码质量，或Bitmap的解码配置等。

例如，要应用一个 CenterCrop 转换，你可以使用以下代码：

```java
import static com.bumptech.glide.request.RequestOptions.centerCropTransform;

Glide.with(fragment)
    .load(url)
    .apply(centerCropTransform(context))
    .into(imageView);
// 可以在 xml 中设置，Glide 会根据 xml 中的属性来设置缩放方式
```

在这个例子中使用了对 `RequestOptions` 类里的方法的静态import，这会让代码看起来更简洁流畅。

如果你想让你的应用的不同部分之间共享相同的加载选项，你也可以初始化一个新的 RequestOptions 对象，并在每次加载时传入这个对象：

```java
RequestOptions cropOptions = new RequestOptions().centerCrop(context);
...
Glide.with(fragment)
    .load(url)
    .apply(cropOptions)
    .into(imageView);
```

`apply()` 方法可以被调用多次，因此 `RequestOption` 可以被组合使用。如果 `RequestOptions` 对象之间存在相互冲突的设置，那么只有最后一个被应用的 `RequestOptions` 会生效。

### Generated API

如果你正在使用 Generated API , 所有的 请求选项 方法都会被内联并可以直接使用：

GlideApp.with(fragment)
    .load(url)
    .centerCrop()
    .into(imageView);

## TransitionOptions

`TransitionOptions` 用于决定你的加载完成时会发生什么。

使用 TransitionOption 可以应用以下变换：

- View淡入
- 与占位符交叉淡入
- 或者什么都不发生

如果不使用变换，你的图像将会“跳入”其显示位置，直接替换掉之前的图像。为了避免这种突然的改变，你可以淡入view，或者让多个Drawable交叉淡入，而这些都需要使用 `TransitionOptions` 完成。

```java
import static com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions.withCrossFade;

Glide.with(fragment)
    .load(url)
    .transition(withCrossFade())
    .into(view);
```

不同于 `RequestOptions`，`TransitionOptions` 是特定资源类型独有的，你能使用的变换取决于你让 `Glide` 加载哪种类型的资源。

这样的结果是，假如你请求加载一个 Bitmap ，你需要使用 `BitmapTransitionOptions` ，而不是 `DrawableTransitionOptions` 。同样，当你请求加载 Bitmap 时，你只需要做简单的淡入，而不需要做复杂的交叉淡入。

## RequestBuilder

`RequestBuilder` 是 Glide 中请求的骨架，负责携带请求的 url 和你的设置项来开始一个新的加载过程。

使用 `RequestBuilder` 可以指定：

- 你想加载的资源类型(Bitmap, Drawable, 或其他)
- 你要加载的资源地址(url/model)
- 你想最终加载到的View
- 任何你想应用的（一个或多个）`RequestOption` 对象
- 任何你想应用的（一个或多个）`TransitionOption` 对象
- 任何你想加载的缩略图 `thumbnail()`
- 你可以通过调用 `Glide.with()` 来构造一个 `RequestBuilder` 对象，然后使用某一个 as 方法：

`RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();`

或使用 `Glide.with()` 然后 `load()`：

`RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).load(url);`

### 选择资源类型

`RequestBuilders` 是特定于它们将要加载的资源类型的。默认情况下你会得到一个 `Drawable RequestBuilder` ，但你可以使用 as... 系列方法来改变请求类型。例如，如果你调用了 `asBitmap()` ，你就将获得一个 `Bitmap RequestBuilder` 对象，而不是默认的 `Drawable RequestBuilder`。

`RequestBuilder<Bitmap> requestBuilder = Glide.with(fragment).asBitmap();`

### 应用 RequestOptions

可以通过 `apoply()` 方法来应用 `RequestOptions`，使用 `transition()` 方法来应用 `TransitionOptions`

```java
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment);
requestBuilder.apply(requestOptions);
requestBuilder.transition(transitionOptions);
```

RequestBuilder 也可以被复用于开始多个请求：

```java
RequestBuilder<Drawable> requestBuilder =
        Glide.with(fragment)
            .asDrawable()
            .apply(requestOptions);

for (int i = 0; i < numViews; i++) {
   ImageView view = viewGroup.getChildAt(i);
   String url = urls.get(i);
   requestBuilder.load(url).into(view);
}
```

### 缩略图 (Thumbnail) 请求

Glide 的 `thumbnail` API 允许你指定一个 `RequestBuilder` 以与你的主请求并行启动。缩略图 会在主请求加载过程中展示。如果主请求在缩略图请求之前完成，则缩略图请求中的图像将不会被展示。`thumbnail` API 允许你简单快速地加载图像的低分辨率版本，并且同时加载图像的无损版本，这可以减少用户盯着加载指示器的时间。

`thumbnail()` API 对本地和远程图片都适用，尤其是当低分辨率缩略图存在于 Glide 的磁盘缓存时，它们将很快被加载出来。

`thumbnail` API 使用起来相对简单：

```java
Glide.with(fragment)
  .load(url)
  .thumbnail(Glide.with(fragment)
    .load(thumbnailUrl))
  .into(imageView);
```

只要你的 `thumbnailUrl` 指向的图片比你的主 `url` 的分辨率更低，它将会很好地工作。

大量图片加载 API 提供了不同的指定图片尺寸的方法，它们尤其适用于 `thumbnail` API。

如果你仅仅想加载一个本地图像，或者你只有一个单独的远程 URL， 你仍然可以从缩略图 API 受益。请使用 Glide 的 `override` 或 `sizeMultiplier` API 来强制 Glide 在缩略图请求中加载一个低分辨率图像：

```java
int thumbnailSize = ...;
Glide.with(fragment)
  .load(localUri)
  .thumbnail(Glide.with(fragment)
    .load(localUri)
    .override(thumbnailSize))
  .into(view);
```

`thumbnail` 方法有一个简化版本，它只需要一个 `sizeMultiplier` 参数。如果你只是想为你的加载相同的图片，但尺寸为 View 或 Target 的某个百分比的话特别有用：

```java
Glide.with(fragment)
  .load(localUri)
  .thumbnail(/*sizeMultiplier=*/ 0.25f)
  .into(imageView);
```

### 在失败时开始新的请求

从 Glide 4.3.0 开始，你现在可以使用 `error` API 来指定一个 `RequestBuilder`，以在主请求失败时开始一次新的加载。例如，在请求 `primaryUrl` 失败后加载 `fallbackUrl`：

```java
Glide.with(fragment)
  .load(primaryUrl)
  .error(Glide.with(fragment)
      .load(fallbackUrl))
  .into(imageView);
```

如果主请求成功完成，这个 `error RequestBuilder` 将不会被启动。如果你同时指定了一个 `thumbnail` 和一个 `error RequestBuilder`，则这个后备的 `RequestBuilder` 将在主请求失败时启动，即使缩略图请求成功也是如此。

## 组件 Options

`Option` 类是给 Glide 组件添加参数的通用方法，包括 `ModeLoaders`、`ResourceDecoders`、`ResourceEncorders`、`Encoders` 等。一些 Glide 的内置组件提供了设置项，自定义的组件也可以添加设置项。

`Option` 通过 `RequestOptions` 类应用到请求上：

```java
import static com.bumptech.glide.request.RequestOptions.option;

Glide.with(context)
  .load(url)
  .apply(option(MyCustomModelLoader.TIMEOUT_MS, 1000L))
  .into(imageView);


// 创建新的对象

RequestOptions options = new RequestOptions()
  .set(MyCustomModelLoader.TIMEOUT_MS, 1000L);

Glide.with(context)
  .load(url)
  .apply(options)
  .into(imageView);

// 使用生成的 API

GlideApp.with(context)
  .load(url)
  .set(MyCustomModelLoader.TIMEOUT_MS, 1000L)
  .into(imageView);
```