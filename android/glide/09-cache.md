# 缓存

## Glide 缓存策略：

- 活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片？
- 内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中？
- 资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存？
- 数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？

前两步检查图片是否在内存中，如果是则直接返回图片。后两步则检查图片是否在磁盘上，以便快速但异步地返回图片。

如果四个步骤都未能找到图片，则Glide会返回到原始资源以取回数据（原始文件，Uri, Url等）。

关于 Glide 缓存的默认大小与它们在磁盘上的位置的更多细节，或如何配置这些参数，请查看 配置 页面。

## Cache Keys

所有缓存 key 至少包含了两种元素：

- 请求的 model（File、Uri、Url）
- 一个可选的签名。

另外，步骤1-3(活动资源，内存缓存，资源磁盘缓存)的缓存键还包含一些其他数据，包括：

- 宽、高
- Transformation（可选）
- 添加的 Options
- 请求的数据类型（Bitmap、GIF）

活动资源和内存缓存使用的键还和磁盘资源缓存略有不同，以适应内存 选项(Options)，比如影响 Bitmap 配置的选项或其他解码时才会用到的参数。

为了生成磁盘缓存上的缓存键名称，以上的每个元素会被哈希化以创建一个单独的字符串键名，并在随后作为磁盘缓存上的文件名使用。

## 缓存配置

### 磁盘缓存策略

`DiskCacheStrategy` 可被 `diskCacheStrategy` 方法应用到每一个单独的请求。目前支持的策略允许你阻止加载过程使用或写入磁盘缓存，选择性地仅缓存无修改的原生数据，或仅缓存变换过的缩略图，或是兼而有之。

默认的策略叫做 AUTOMATIC ，它会尝试对本地和远程图片使用最佳的策略。当你加载远程数据（比如，从URL下载）时，AUTOMATIC 策略仅会存储未被你的加载过程修改过(比如，变换，裁剪–译者注)的原始数据，因为下载远程数据相比调整磁盘上已经存在的数据要昂贵得多。对于本地数据，AUTOMATIC 策略则会仅存储变换过的缩略图，因为即使你需要再次生成另一个尺寸或类型的图片，取回原始数据也很容易。

指定 `DiskCacheStrategy` 非常容易:

```java
GlideApp.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.ALL)
  .into(imageView);
```

### 仅从缓存加载图片

某些情形下，你可能希望只要图片不在缓存中则加载直接失败（比如省流量模式？–译者注）。如果要完成这个目标，你可以在单个请求的基础上使用 `onlyRetrieveFromCache` 方法：

```java
GlideApp.with(fragment)
  .load(url)
  .onlyRetrieveFromCache(true)
  .into(imageView);
```

设置 true 后，如果图片在内存或者磁盘缓存中发现，那么会被加载，如果没有发现，则失败。

### 跳过缓存

如果你想确保一个特定的请求跳过磁盘和/或内存缓存（比如，图片验证码 –译者注），Glide 也提供了一些替代方案。

```java
GlideApp.with(fragment)
  .load(url)
  .skipMemoryCache(true) // 跳过内存缓存
  .into(view);

GlideApp.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE) // 跳过磁盘缓存
  .into(view);

// 两个缓存都不用

GlideApp.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .skipMemoryCache(true)
  .into(view);
```

虽然提供了这些办法让你跳过缓存，但你通常应该不会想这么做。从缓存中加载一个图片，要比拉取-解码-转换成一张新图片的完整流程快得多。

如果你只是想更新缓存中的某个条目，请继续阅读下面关于 invalidation 一节的介绍。

### 可以自己实现 DiskCache

如果内置的选项不满足你的需求，你也可以编写你自己的 DiskCache 实现。请查看 配置 页获得更多信息。

## 缓存的刷新

因为磁盘缓存使用的是哈希键，所以并没有一个比较好的方式来简单地删除某个特定url或文件路径对应的所有缓存文件。如果你只允许加载或缓存原始图片的话，问题可能会变得更简单，但因为Glide还会缓存缩略图和提供多种变换(transformation)，它们中的任何一个都会导致在缓存中创建一个新的文件，而要跟踪和删除一个图片的所有版本无疑是困难的。

在实践中，使缓存文件无效的最佳方式是在内容发生变化时（url，uri，文件路径等）更改你的标识符。

### 定制刷新策略

因为通常改变标识符比较困难或者根本不可能，所以Glide也提供了 签名 API 来混合（你可以控制的）额外数据到你的缓存键中。签名(signature)适用于媒体内容，也适用于你可以自行维护的一些版本元数据。

- MediaStore 内容 - 对于媒体存储内容，你可以使用Glide的 MediaStoreSignature 类作为你的签名。MediaStoreSignature 允许你混入修改时间、MIME类型，以及item的方向到缓存键中。这三个属性能够可靠地捕获对图片的编辑和更新，这可以允许你缓存媒体存储的缩略图。
- 文件 - 你可以使用 ObjectKey 来混入文件的修改日期。
- Url - 尽管最好的让 url 失效的办法是让 server 保证在内容变更时对URL做出改变，你仍然可以使用 ObjectKey 来混入任意数据（比如版本号）。

将签名传入加载请求很简单：

```java
// 传入 signatures
GlideApp.with(yourFragment)
    .load(yourFileDataModel)
    .signature(new ObjectKey(yourVersionMetadata))
    .into(yourImageView);

// media store signature 也可以是用 MediaStore 的直接数据
GlideApp.with(fragment)
    .load(mediaStoreUri)
    .signature(new MediaStoreSignature(mimeType, dateModified, orientation))
    .into(view);
```

也可以通过 `Key` 接口来定义自己的 signature，确保要实现 `equals() hashcode() updateDiskCacheKey()` 方法

```java
public class IntegerVersionSignature implements Key {
    private int currentVersion;

    public IntegerVersionSignature(int currentVersion) {
         this.currentVersion = currentVersion;
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof IntegerVersionSignature) {
            IntegerVersionSignature other = (IntegerVersionSignature) o;
            return currentVersion = other.currentVersion;
        }
        return false;
    }

    @Override
    public int hashCode() {
        return currentVersion;
    }

    @Override
    public void updateDiskCacheKey(MessageDigest md) {
        messageDigest.update(ByteBuffer.allocate(Integer.SIZE).putInt(signature).array());
    }
}
```

请记住，为了避免降低性能，您将需要在后台批量加载任何版本元数据，以便在要加载图像时即已处于可用状态。

如果这些努力都无法奏效，您不能更改标识符，也不能跟踪任何合理的版本元数据的情况下，也可以使用 `diskCacheStrategy()` 和 `DiskCacheStrategy.NONE` 来完全禁用磁盘缓存。

## 资源管理

Glide 的磁盘和内存缓存都是 LRU ，这意味着在达到使用限制或持续接近限制值之前，它们将占用持续增加的内存或磁盘空间。为了增加额外的灵活性，Glide 提供了一些额外的方式来让你可以管理你的应用使用的资源。

请记住，更大的内存缓存、位图池和磁盘缓存通常能提供更好的性能，或者至少在同等级别。如果你改变了缓存的大小， 你应该小心地测量一下你改动之前和之后的性能对比，以确保你的修改带来的性解蔽是可以接受的。

### 内存缓存

默认情况下 Glide 的内存缓存和 `BitmapPool` 会响应 `ComponentCallback2` ，并根据 Android framework 提供的级别自动清理内容。 因此你通常不需要尝试动态监视或清理你的缓存或 `BitmapPool。然而，如果必要的话，Glide 确实提供了几个手动选项。

#### 永久尺寸调整

要改变你应用中 Glide 的可用 RAM 大小，请查看 配置。

#### 暂时尺寸调整

要在你应用的特定部分暂时允许 Glide 使用更多或更少的内存，你可以使用 setMemoryCategory:

```java
// This method must be called on the main thread.
Glide.get(context).clearMemory();
```

清理所有内存并非特别经济，并且应该尽可能避免，以避免出现抖动和增加加载时间。

### 磁盘缓存

Glide 在运行时仅提供对磁盘缓存的有限控制，但是其大小和配置可以在 AppGlideModule 中改变。

#### 永久尺寸修改

要改变你应用中 Glide 可用的 sdcard 可用空间，请查看 配置。

#### 清理磁盘缓存

要尝试清理所有磁盘缓存条目，你可以使用 clearDiskCache。

```java
new AsyncTask<Void, Void, Void> {
  @Override
  protected Void doInBackground(Void... params) {
    // This method must be called on a background thread.
    Glide.get(applicationContext).clearDiskCache();
    return null;
  }
}
```