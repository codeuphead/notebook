# TV 开发

电视应用与手机平板应用的架构相同，因此可以修改当前的代码使其可以运行在电视设备上。

推荐使用一个 APP 来支持移动设备和电视设备。

使用 24.0.0 以上的 SDK tools；使用 SDK 21 以上；创建或升级现有的项目。

## Manifest 相关

### UI

#### 声明 TV Activity

使用含有 `CATEGORY_LEANBACK_LAUNCHER` 的 intent filter。

```xml
<activity
  android:name="com.example.android.TvActivity"
  android:label="@string/app_name"
  android:theme="@style/Theme.Leanback">

  <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
  </intent-filter>

</activity>
```

#### banner

home 屏 banner 是在主屏中 app 列表中，我们程序的入口，5.0 以上生效。

application 标签或者 activity 标签进行添加。

资源类型：xhdip 320 x 180 px。

```xml
android:banner="@drawable/banner"
```

### 声明 Leanback 支持

```xml
<uses-feature android:name="android.software.leanback"
    android:required="false" />

<uses-feature android:name="android.hardware.touchscreen"
          android:required="false" />
```

### 设置启动颜色

在 style 中设置 colorPrimary，可以设置给 activity。

```xml
<style ... >
  <item name="android:colorPrimary">@color/primary</item>
  <item name="android:windowAllowReturnTransitionOverlap">true</item>
  <item name="android:windowAllowEnterTransitionOverlap">true</item>
</style>
```

## 硬件相关

[参考](https://developer.android.com/guide/topics/manifest/uses-feature-element.html#features-reference)

### 检查 TV

```java
public static final String TAG = "DeviceTypeRuntimeCheck";

UiModeManager uiModeManager = (UiModeManager) getSystemService(UI_MODE_SERVICE);
if (uiModeManager.getCurrentModeType() == Configuration.UI_MODE_TYPE_TELEVISION) {
    Log.d(TAG, "Running on a TV Device");
} else {
    Log.d(TAG, "Running on a non-TV Device");
}
```

### 检查硬件功能

```xml
<uses-feature android:name="android.hardware.touchscreen"
        android:required="false"/>
<uses-feature android:name="android.hardware.faketouch"
        android:required="false"/>
<uses-feature android:name="android.hardware.telephony"
        android:required="false"/>
<uses-feature android:name="android.hardware.camera"
        android:required="false"/>
<uses-feature android:name="android.hardware.nfc"
        android:required="false"/>
<uses-feature android:name="android.hardware.location.gps"
        android:required="false"/>
<uses-feature android:name="android.hardware.microphone"
        android:required="false"/>
<uses-feature android:name="android.hardware.sensor"
        android:required="false"/>
```

### 动态判断是否拥有功能

```java
// Check if the telephony hardware feature is available.
if (getPackageManager().hasSystemFeature("android.hardware.telephony")) {
    Log.d("HardwareFeatureTest", "Device can make phone calls");
}

// Check if android.hardware.touchscreen feature is available.
if (getPackageManager().hasSystemFeature("android.hardware.touchscreen")) {
    Log.d("HardwareFeatureTest", "Device has a touch screen.");
}
```

### 声明权限隐含的硬件功能

`RECORD_AUDIO` android.hardware.microphone

`CAMERA` android.hardware.camera and android.hardware.camera.autofocus

`ACCESS_COARSE_LOCATION` android.hardware.location android.hardware.location.network(TargetAPI 20 or lower only)

`ACCESS_FINE_LOCATION` android.hardware.location android.hardware.location.gps(TargetAPI 20 or lower only)

### GPS 定位相关

可以请求静态的定位，如 zip code 等。

```java
// Request a static location from the location manager
LocationManager locationManager = (LocationManager) this.getSystemService(
        Context.LOCATION_SERVICE);
Location location = locationManager.getLastKnownLocation("static");

// Attempt to get postal or zip code from the static location object
Geocoder geocoder = new Geocoder(this);
Address address = null;
try {
  address = geocoder.getFromLocation(location.getLatitude(),
          location.getLongitude(), 1).get(0);
  Log.d("Zip code", address.getPostalCode());

} catch (IOException e) {
  Log.e(TAG, "Geocoder error", e);
}
```

### Camera

除了在 manifest 中设置 uses-feature 为 false 外，有必要时在运行时添加以下判断。

```java
// Check if the camera hardware feature is available.
if (getPackageManager().hasSystemFeature("android.hardware.camera")) {
    Log.d("Camera test", "Camera available!");
} else {
    Log.d("Camera test", "No camera available. View and edit features only.");
}
```

## 控制器相关

控制器断开、重连，配置 configChanges `android:configChanges="keyboard|keyboardHidden|navigation|screenSize"`。

部分设备有省电模式，onStop() 中停止播放。