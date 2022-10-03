# LeakCanary

```
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:1.5.4'
  releaseImplementation 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.4'
}
```

```java
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      // This process is dedicated to LeakCanary for heap analysis.
      // You should not init your app in this process.
      return;
    }
    LeakCanary.install(this);
    // Normal app init code...
  }
}
```

在 Unit Test 中关闭：在 build.gradle 中添加

```
configurations.all { config ->
  if (config.name.contains('UnitTest')) {
    config.resolutionStrategy.eachDependency { details ->
      if (details.requested.group == 'com.squareup.leakcanary' && details.requested.name == 'leakcanary-android') {
        details.useTarget(group: details.requested.group, name: 'leakcanary-android-no-op', version: details.requested.version)
      }
    }
  }
}
```

如果需要在 instrumentation test 中关闭，添加 `|| config.name.contains('AndroitTest')`
