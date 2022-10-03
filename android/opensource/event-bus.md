# EventBus

## 引入工程

```java
compile "org.greenrobot:eventbus:3.1.1"
annotationProcessor "org.greenrobot:eventbus-annotation-processor:3.1.1"
```

与广播或者 Intent 不同，EventBus 使用的是标准 Java 类库。提供更便捷的 API 和更少的资源开销。

## 使用步骤

### 第一步 创建 Event

创建的类属性即为需要进行传递的消息或数据，也可以没有属性，为一个单纯的命令。

```java
public class MessageEvent {
    public final String message;
    public MessageEvent(String message) {
        this.message = message;
    }
}
```

### 第二步 配置订阅者

订阅者实现了事件的处理方法（订阅方法），当事件发送时，该方法会被调用。这些方法以 `@Subscribe` 注解进行定义。

```java
// This method will be called when a MessageEvent is posted (in the UI thread for Toast)
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessageEvent(MessageEvent event) {
    Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
}

// This method will be called when a SomeOtherEvent is posted
@Subscribe
public void handleSomethingElse(SomeOtherEvent event) {
    doSomethingWith(event);
}
```

订阅者需要进行注册、注销

通常在 onStart/onStop 进行都能正常工作

```java
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

@Override
public void onStop() {
    EventBus.getDefault().unregister(this);
    super.onStop();
}
```

### 第三部 发送事件

```java
EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
```

## 线程切换

### ThreadMode.POSTING

推荐使用场景为：耗时小的无需主线程的简单任务。

默认的方式。

订阅者被调用的线程和发布事件的线程一致。

事件传递是同步的，事件发布完成后，所有的订阅者都会被调用。

该方式开销最小，因为没有线程切换。

事件处理的方法不能处理耗时任务，防止阻塞发布的线程，发布线程有可能是主线程。

```java
// Called in the same thread (default)
// ThreadMode is optional here
@Subscribe(threadMode = ThreadMode.POSTING)
public void onMessage(MessageEvent event) {
    log(event.message);
}
```

### ThreadMode.MAIN

订阅者被调用的线程是主线程。

如果发布线程是主线程，事件处理方法会同步直接调用。

事件处理方法不能处理耗时任务，防止阻塞主线程。

```java
// Called in Android UI's main thread
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessage(MessageEvent event) {
textField.setText(event.message);
}
```

### ThreadMode.MAIN_ORDERED

订阅者被调用的线程是主线程。

将事件放入队列进行处理。

### ThreadMode.BACKGROUND

订阅者被调用线程是后台线程。

如果发布线程不是主线程，事件处理方法会直接在发布线程调用。

如果发布线程是主线程，将会使用一个独立的后台线程来按队列传递所有的事件。

事件处理方法要尽量快速返回，防止阻塞后台线程。

```java
// Called in the background thread
@Subscribe(threadMode = ThreadMode.BACKGROUND)
public void onMessage(MessageEvent event){
    saveToDisk(event.message);
}
```

### ThreadMode.ASYNC

订阅者被调用线程是一个异步的线程。

该线程是与发布线程和主线程独立的线程。

此模式，发布线程不会等待事件处理方法。

如果事件处理方法比较耗时，应该此模式。

不要触发大量的长时间运行的异步事件处理方法，因为会产生大量的并发线程。

EventBus 使用线程池来提高异步处理通知的效率。

```java
// Called in a separate thread
@Subscribe(threadMode = ThreadMode.ASYNC)
public void onMessage(MessageEvent event){
    backend.send(event.message);
}
```

## 配置

### 通过 EventBusBuilder 构建

默认 EventBus 会捕获从订阅者方法中抛出的异常，并发送一个`SubscriberExceptionEvent`。

```java
EventBus eventBus = EventBus.builder().throwSubscriberException(true).build();
```

EventBus 的配置很多，例如一个保持静默的 EventBus，对应的是发布的事件没有订阅者。

```java
EventBus eventBus = EventBus.builder()
    .logNoSubscriberMessages(false)
    .sendNoSubscriberEvent(false)
    .build();
```

### 配置默认 EventBus 实例

在任何地方，使用`EventBug.getDefault()`获取共享实例。

通过 `installDefaultEventBus()`来设置默认的实例。

例如，配置将订阅方法中抛出异常再次抛出。

```java
EventBus.builder().throwSubscriberException(BuildConfig.DEBUG).installDefaultEventBus();
```

必须在 EventBus 实例第一次使用之前进行设置。初始化在 Application 类中。

## Sticky Events

### 发布

EventBus 可以保持最后的 sticky event 在内存中。这个 sticky event 可以被传送给订阅者或者显式查询到。

```java
EventBus.getDefault().postSticky(new MessageEvent("Hello everyone!"));
```

当新的组件注册成为订阅者后，所有 sticky 订阅者方法都活立即获取到之前发布的 sticky event。

```java
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

// UI updates must run on MainThread
@Subscribe(sticky = true, threadMode = ThreadMode.MAIN)
public void onEvent(MessageEvent event) {
    textField.setText(event.message);
}

@Override
public void onStop() {
    EventBus.getDefault().unregister(this);
    super.onStop();
}
```

### 可以手动检查 sticky event

```java
MessageEvent stickyEvent = EventBus.getDefault().getStickyEvent(MessageEvent.class);
// Better check that an event was actually posted before
if(stickyEvent != null) {
    // "Consume" the sticky event
    EventBus.getDefault().removeStickyEvent(stickyEvent);
    // Now do something with it
}
```

### 移除 sticky event

removeStickyEvent 有重载方法。

参数是 class 的时候，会返回之前持有的 sticky event

```java
MessageEvent stickyEvent = EventBus.getDefault().removeStickyEvent(MessageEvent.class);
// Better check that an event was actually posted before
if(stickyEvent != null) {
    // Now do something with it
}
```

## 优先级

订阅者在注册的时候提供一个优先级。

在同一个传送线程（ThreadMode）中，高优先级的订阅者会比其他低优先级订阅者先收到事件。

不同 ThreadMode 中的订阅者，优先级没用。

默认优先级是 0。

```java
@Subscribe(priority = 1);
public void onEvent(MessageEvent event) {
    ...
}
```

## 取消事件

在一个订阅者的事件处理方法中，调用`cancelEventDelivery(Object event)`，可以将事件的传递中止。

```java
// Called in the same thread (default)
@Subscribe
public void onEvent(MessageEvent event){
    // Process the event
    ...
    // Prevent delivery to other subscribers
    EventBus.getDefault().cancelEventDelivery(event) ;
}
```

事件通常会在更高优先级的订阅者中进行取消。ThreadMode.PostTread 执行的事件处理方法中，是不可以取消的。

## 订阅者索引

EventBus 3.0 后的新功能。

这是一个可选的优化设置，可以提高订阅者注册的初始化过程。

订阅者索引可以在编译期进行创建，使用的是 EventBus annotation processor。

订阅者索引不是必须的，但是可以提高很多的性能。

### 条件

只有 @Subscribe 方法可以被索引。包括订阅者和事件的 class，并且必须是 public。

@Subscribe 注解不可在匿名类中使用。

当不能使用注解的时候，会自动使用运行时反射进行处理，速度会有所下降。

### 生成索引

#### 使用 annotationProcessor

Android Gradle Plugin 2.2.0 及以上

需要添加 EventBus annotation processor 到 build.gradle。设置一个变量 eventBusIndex 来指定一个全限定名的类，用来作为生成的索引类。

```java
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [ eventBusIndex : 'com.example.myapp.MyEventBusIndex' ]
            }
        }
    }
}

dependencies {
    compile 'org.greenrobot:eventbus:3.1.1'
    annotationProcessor 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
```

#### 使用 kapt

使用 Kotlin

```java
apply plugin: 'kotlin-kapt' // ensure kapt plugin is applied

dependencies {
    compile 'org.greenrobot:eventbus:3.1.1'
    kapt 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}

kapt {
    arguments {
        arg('eventBusIndex', 'com.example.myapp.MyEventBusIndex')
    }
}
```

#### 使用 android-apt

Android Gradle Plugin 2.2.0 以下

```java
buildscript {
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}

apply plugin: 'com.neenbedankt.android-apt'

dependencies {
    compile 'org.greenrobot:eventbus:3.1.1'
    apt 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}

apt {
    arguments {
        eventBusIndex "com.example.myapp.MyEventBusIndex"
    }
}
```

#### 使用索引

编译工程后，eventBusIndex 指定的类就会自动生成。

接着设置 EventBus

```java
EventBus eventBus = EventBus.builder().addIndex(new MyEventBusIndex()).build();
```

或者使用默认实例

```java
EventBus.builder().addIndex(new MyEventBusIndex()).installDefaultEventBus();
// Now the default instance uses the given index. Use it like this:
EventBus eventBus = EventBus.getDefault();
```

可以将同样的规则，应用给 library 中的 code。这种情况下，可能会有多个索引类。

```java
EventBus eventBus = EventBus.builder()
    .addIndex(new MyEventBusAppIndex())
    .addIndex(new MyEventBusLibIndex()).build();
```

## ProGuard

```java
-keepattributes *Annotation*
-keepclassmembers class ** {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }

# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
```

## AysncExecutor

### 使用

AysncExecutor 是非核心工具类。帮助在处理后台线程异常时节省时间。

类似线程池，提供异常处理。

抛出的异常，AsyncExecutor 将其包装到一个 event 中，并自动发布。

通常可以调用`AsyncExecutor.create()`创建实例，并在 Application 中持有。

需要执行任务的时候，实现`RunnableEx`接口，并传递给`AsyncExecutor`的 execute 方法。

如果`RunnableEx`的实现抛出了异常，那么将会被捕捉并包装到`ThrowableFailureEvent`进行发布。

```java
AsyncExecutor.create().execute(
    new AsyncExecutor.RunnableEx() {
        @Override
        public void run() throws LoginException {
            // No need to catch any Exception (here: LoginException)
            remote.login();
            EventBus.getDefault().postSticky(new LoggedInEvent());
        }
    }
);
```

异常处理部分

```java
@Subscribe(threadMode = ThreadMode.MAIN)
public void handleLoginEvent(LoggedInEvent event) {
    // do something
}

@Subscribe(threadMode = ThreadMode.MAIN)
public void handleFailureEvent(ThrowableFailureEvent event) {
    // do something
}
```

### AsyncExecutor Builder

`AsyncExecutor.builder()`可以设置 EventBus 实例、threadPool、failure event

`execution scope`一个种设置方式，可以给予 failure event 上下文信息。例如相关的 Activity 类。

如果自定义的失败事件类实现的是`HasExectuionScope`接口，那么 AsyncExecutor 会自动设置 execution scope。

订阅者可以查看失败事件的 exectuion scope 并依此作出反映。