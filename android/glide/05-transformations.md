# 变换

在 Glide 中，Transformations 可以获取资源并修改它，然后返回被修改后的资源。通常变换操作是用来完成剪裁或对位图应用过滤器，但它也可以用于转换 GIF 动画，甚至自定义的资源类型。

## 内置类型

Glide 有一系列的内置变换，包括：

- CenterCrop
- FitCenter
- CircleCrop

## 应用变换

使用 `RequestOptions` 类来进行

### 默认变换

```java
RequestOptions options = new RequestOptions();
options.centerCrop(context);

Glide.with(fragment)
    .load(url)
    .apply(options)
    .into(imageView);
```

许多内置变换可以使用静态引入来使调用更简化

```java
import static com.bumptech.glide.request.RequestOptions.fitCenterTransform;


Glide.with(fragment)
    .load(url)
    .apply(fitCenterTransform(context))
    .into(imageView);

// 生成 API
GlideApp.with(fragment)
    .load(url)
    .fitCenter()
    .into(imageView);
```

## 多个变换

默认情况下，每个 `transform()` 调用，或任何特定转换方法(fitCenter(), centerCrop(), bitmapTransform() etc)的调用都会替换掉之前的变换。

如果你想在单次加载中应用多个变换，请使用 `MultiTransformation` 类。

```java
Glide.with(fragment)
    .load(url)
    .transform(new MultiTransformation(new Fitcenter(), new CustomTransformation())
    .into(imageView);

GlideApp.with(fragment)
  .load(url)
  .transforms(new FitCenter(), new YourCustomTransformation())
  .into(imageView);
```

传入 `MultiTransformation` 构造中的变换顺序，是变换应用的顺序。

## 定制变换

尽管 Glide 提供了各种各样的内置 `Transformation` 实现，如果你需要额外的功能，你也可以实现你自己的 `Transformation`。

### BitmapTransformation

如果你只需要变换 Bitmap，最好是从继承 `BitmapTransformation` 开始。`BitmapTransformation` 处理了一些基础的东西，包括提取和回收原始的 Bitmap，如果你的变换返回了一个新修改的 Bitmap 的话。

一个简单的实现看起来可能像这样：

```java
public class FillSpace extends BitmapTransformation {
    private static final String ID = "com.bumptech.glide.transformations.FillSpace";
    private static final String ID_BYTES = ID.getBytes(STRING_CHARSET_NAME);

    {@literal @Override}
    public Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        if (toTransform.getWidth() == outWidth && toTransform.getHeight() == outHeight) {
            return toTransform;
        }

        return Bitmap.createScaledBitmap(toTransform, outWidth, outHeight, /*filter=*/ true);
    }

    {@literal @Override}
    public void equals(Object o) {
      return o instanceof FillSpace;
    }

    {@literal @Override}
    public int hashCode() {
      return ID.hashCode();
    }

    {@literal @Override}
    public void updateDiskCacheKey(MessageDigest messageDigest)
        throws UnsupportedEncodingException {
      messageDigest.update(ID_BYTES);
    }
}
```

尽管你的 `Transformation` 将几乎确定比这个示例更复杂，但它应该包含了相同的基本元素和复写方法。

### 必需的方法

请特别注意，对于任何 `Transformation` 子类，包括 `BitmapTransformation`，你都有三个方法你 必须 实现它们，以使得磁盘和内存缓存正确地工作：

- equals()
- hashCode()
- updateDiskCacheKey

如果你的 `Transformation` 没有参数，通常使用一个包含完整包限定名的 `static final String` 来作为一个 ID，它可以构成 `hashCode()` 的基础，并可用于更新 `updateDiskCacheKey()` 传入的 `MessageDigest`。如果你的 `Transformation` 需要参数而且它会影响到 Bitmap 被变换的方式，它们也必须被包含到这三个方法中。

例如，Glide 的 `RoundedCorners` 变换接受一个 int，它决定了圆角的弧度。它的equals(), hashCode() 和 updateDiskCacheKey 实现看起来像这样：

```java
  @Override
  public boolean equals(Object o) {
    if (o instanceof RoundedCorners) {
      RoundedCorners other = (RoundedCorners) o;
      return roundingRadius == other.roundingRadius;
    }
    return false;
  }

  @Override
  public int hashCode() {
    return Util.hashCode(ID.hashCode(),
        Util.hashCode(roundingRadius));
  }

  @Override
  public void updateDiskCacheKey(MessageDigest messageDigest) {
    messageDigest.update(ID_BYTES);

    byte[] radiusData = ByteBuffer.allocate(4).putInt(roundingRadius).array();
    messageDigest.update(radiusData);
  }
```

原来的 String 仍然保留，但 `roundingRadius` 被包含到了三个方法中。这里，`updateDiskCacheKey` 方法还演示了你可以如何使用 ByteBuffer 来包含基本参数到你的 `updateDiskCacheKey` 实现中。

### 不要忘记 equals() / hashCode()

值得重申的一点是，为了让内存缓存正常地工作你是否必须实现 `equals()` 和 `hashCode()` 方法。很不幸，即使你没有复写这两个方法，`BitmapTransformation` 和 `Transformation` 也能通过编译，但这并不意味着它们能正常工作。我们正在探索一些方案，以使在 Glide 的未来版本中，使用默认的 `equals()` 和 `hashCode` 方法将抛出一个编译时错误。

## Glide 的特殊行为

### 重用变换

`Transforamtions` 的设计初衷是无状态的。因此重用 `Transformations` 对象是安全的。创建一次 Transformation 并在多个加载中使用它，通常是很好的实践。

### ImageViews 的自动变换

在 Glide 中，当你为一个 `ImageView` 开始加载时，Glide 可能会自动应用 `FitCenter` 或 `CenterCrop` ，这取决于 view 的 `ScaleType` 。如果 scaleType 是 `CENTER_CROP` , Glide 将会自动应用 `CenterCrop` 变换。如果 scaleType 为 `FIT_CENTER` 或 `CENTER_INSIDE` ，Glide会自动使用 `FitCenter` 变换。

当然，你总有权利覆写默认的变换，只需要一个带有 `Transformation` 集合的 `RequestOptions` 即可。另外，你也可以通过使用 `dontTransform()` 确保不会自动应用任何变换。

### 自定义资源

因为 Glide 4.0 允许你指定你将解码的资源的父类型，你可能无法确切地知道将会应用何种变换。例如，当你使用 asDrawable() (或就是普通的 with() ，因为 asDrawable() 是默认情形)来加载 Drawable 资源时，你可能会得到 BitmapDrawable 子类，也有可能得到 GifDrawable 子类。

为了确保你添加到 `RequestOptions` 中的任何变换都会被使用，Glide将 `Transformation` 添加到一个 Map 中保存，其Key为你提供变换的资源类型。当资源被成功解码时，Glide 使用这个 Map 来取回对应的 `Transformation`。

Glide 可以将 `Bitmap Transformation` 应用到 `BitmapDrawable`, `GifDrawable`, 以及 `Bitmap` 资源上，因此通常你只需要编写和应用 `Bitmap Transformation` 。然而，如果你添加了额外的资源类型，你可能需要考虑派生 `RequestOptions` 类，并让你的资源类型能应用 Glide 内置的 `Bitmap Transformation`。