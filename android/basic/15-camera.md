# Camera

## 调用Camera拍照

```java
File outputImage = new File(getExternalCacheDir(), "out_put_image.jpg");

if(outputImage.exists()){
    outputImage.delete();
}
outputImage.createNewFile();

if(Buile.VERSION.SDK_INT >= 24){
    //7.0开始，直接使用文件的真实路径 Uri 是不安全的，会抛出一个 FileUriExposedException，FileProvider 是一种特殊的内容提供器。
    //需要在 AndroidManifest 进行注册该 provider
    iamgeUri = FileProvider.getUriFromFile(MainActivity.this, "com.example.cameraalbumtest.fileprovider", outputImage);
}else{
    iamgeUri = Uri.fromFile(outputImage);
}

Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
startActivityForResult(intent, reqCode);

//onActivityResult 回掉中，进行显示等操作。
BitmapFactory.decodeStream(getContentResolver.openInputStream(imageUri));
```

AndroidManifest 中注册 provider

```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.example.cameraalbumtest.fileprovider"
    android:exported="false"
    adnroid:grantUriPermissions="true">
    <meta-data
        android:name="android.supprot.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths"/>
</provider>
```

使用 meta-data 标签来指定 Uri 的共享路径，这里引用一个 xml 资源，xml 通常为

```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android"
    <external-path name="my_images" path=""/>
</paths>
```

external-path 就是用来指定 Uri 共享的，name 属性可以随便填，path 属性表示共享的具体路径。设置空置表示将整个 SD 卡进行共享。当然也可以仅共享我们存放图片的路径

## 从相册选图片

首先是 openAlbum() 方法

```java
private void Album(){
    Intent intent = new Intent("android.intent.action.GET_CONTENT");
    intent.setType("image/*");
    startActivityForResult(intent, CHOOSE_PHOTO);
}
```

回调处理，4.4之后选取相册中的图片，不再返回真实的 Uri 了，而是封装的 Uri 需要进行解析。

```java
if(resultCode == RESULT_OK){
    if(Build.VERSION.SDK_INT >= 19){
        //4.4以上处理
        handleImageOnKitkat(data);
    }else{
        //4.4以下处理
        handleImageBeforeKitkat(data);
    }
}

@TargetApi(19)
private void handleImageOnKitkat(Intent data){
    String imagePath = null;
    Uri uri = data.getData();
    if(DocumentsContract.isDocumentUri(this, uri)){
        //如果是 document 类型的 Uri 则通过 document id 处理
        String docId = DocumentsContract.getDocumentId(uri);
        if("com.android.provider.media.documents".equals(uri.getAuthority())){
            String id = docId.split(":")[1];//解析数字格式的id
            String selection = MediaStore.Image.Media._ID + "=" + id;
            iamgePath = getImagePath(MediaStore.Image.Media.EXTERNAL_CONTENT_URI, selection);
        }else if("com.android.provider.downloads.documents".equals(uri.getAuthority())){
            Uri contentUri = ContentUris.withAppendedId(Uri.parse("content://downloads/public_downlaods",
                Long.valueOf(docId)));
            imagePath = getImagePath(contentUri, null);
        }
    }else if("content".equalsIgnoreCase(uri.getScheme())){
        //如果 content 类型 Uri，使用普通方式处理
        imagePath = getImagePath(uri, null);
    }else if("file".equalsIgnoreCase(uri.getScheme())){
        //file 类型，直接获取路径
        imagePath = uri.getPath();
    }
    displayImage(imagePath);
}
```

如果返回的 Uri 是 document 类型的话，那么取出 document id 进行处理，否则进行普通方式进行处理。

另外，如果 Uri 的 Authority 是 media 格式的话 document id 还需要进行一次解析，要通过字符串分割方式，取出后半部分才能得到真正的数字 id。

```java
private void handleImageBeforeKitkat(Intent data){
    Uri uri = data.getData();
    String imagePath = getImagePath(uri, null);
    displayImage(imagePath);
}
```

最后获取路径，显示图片

```java
private String getImagePath(Uri uri, String selection){
    String path = null;
    //通过 uri 和 selection 来获取真实图片路径
    Cursor cursor = getContentResolver().query(uri, null, selection, null, null);
    if(cursor != null){
        if(cursor.moveToFirst()){
            path = cursor.getString(cursor.getColumIndex(MediaStore.Images.Media.DATA));
        }
        cursor.close();
    }
    return path;
}

private void displayImage(String imagePath){
    if(imagePath != null){
        Bitmap bitmap = BitmapFactory.decodeFile(imagePath);
        picture.setImageBitmap(bitmap);
    }else{
        //something like toast
    }
}
```
