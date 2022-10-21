# Denpendency Injection With Hilt

- [Denpendency Injection With Hilt](#denpendency-injection-with-hilt)
  - [添加项目依赖](#添加项目依赖)
  - [Hilt Application 类](#hilt-application-类)
  - [注入依赖](#注入依赖)
  - [定义 Hilt binding](#定义-hilt-binding)
  - [Hilt modules](#hilt-modules)
    - [@Binds 注入 interface 实例](#binds-注入-interface-实例)
    - [@Provides 注入实例](#provides-注入实例)
    - [为一个类型提供多个 binding](#为一个类型提供多个-binding)
    - [Hilt 中 预定义的修饰符](#hilt-中-预定义的修饰符)
  - [为 Android 类生成的组件](#为-android-类生成的组件)
    - [组件生命周期](#组件生命周期)
    - [组件作用域](#组件作用域)
    - [组件层级](#组件层级)
    - [组件默认 binding](#组件默认-binding)
  - [为不支持 Hilt 的类注入依赖](#为不支持-hilt-的类注入依赖)
  - [Hilt 和 Dagger](#hilt-和-dagger)
  - [其他资源](#其他资源)

Hilt 帮助减少手动依赖注入导致的大量模板代码。手动依赖注入需要开发者自己创建类及其依赖，使用容器来管理和复用依赖。

Hilt 能够提供一个标准的方式进行依赖注入，通过提供容器从而自动管理依赖的生命周期并向整个应用程序提供依赖。

Hilt 基于 Dagger，相较于Dagger，Hilt 在编译时正确性，运行时的性能，扩展性和 Android Studio 的支持方面更加优秀。

## 添加项目依赖

添加 hilt-android-gralde-glugin

```kotlin
plugins {
    id("com.google.dagger.hilt.android") version "2.44" apply false
}
```

然后添加依赖

```kotlin
plugins {
    kotlin("kapt")
    id("com.google.dagger.hilt.android")
}

android {
    ...
}

dependencies {
    implementation("com.google.dagger:hilt-android:2.44")
    kapt("com.google.dagger:hilt-android-compiler:2.44")
}

kapt {
    correctErrorTypes = true
}

```

Hilt 使用了 Java 8 的特性，需要在项目中启用 Java 8。

```kotlin
android {

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompativitlity = JavaVersion.VERSION_1_8
    }
}
```

## Hilt Application 类

使用 Hilt 的项目必须有用 @HiltAndroidApp 注解的 Application。

这个注解会触发 Hilt 进行模板代码的生成，其中有一个基础类提供了 application-level 的依赖容器。

```kotlin
@HiltAndroidApp
class ExampleApplication: Application() {...}
```

生成的 Hilt 组件与 Application 的生命周期关联并提供依赖。它是 app 的父组件，因此其他组件可以从这里获取依赖。

## 注入依赖

使用 `@AndroidEntryPoint` 注解

```kotlin
@AndroidEntryPoint
class ExampleActivity: AppCompatActivity() {...}
```

Hilt 目前支持以下的 Android 类：

- Application (@HiltAndroidApp)
- ViewModel (@HiltViewModel)
- Activity
- Fragment
- View
- Service
- BroadcastReceiver

如果一个类被 @AndroidEntryPoint 注解修饰，依赖它的其他类也需要注解修饰。例如注解修饰了一个 Fragment，那么必须用注解修饰使用了该 Fragment 的Activity。

@AndroidEntryPoint 注解会为使用它的类生成一个独立的 Hilt 组件，这个组件可以收到他们在组件树上的父类的依赖。

从一个组件获取依赖，使用 @Inject 注解

```kotlin
@AndroidEntryPoint
class ExampleActivity: AppCompatActivity() {
    @Inject lateinit var analytics: AnalyticsAdapter
}
```

Hilt 注入的类可以使用其他被注入的类，并且如果他们是抽象的话，不需要使用 @AndroidEntryPoint 注解。

## 定义 Hilt binding

为了对字段进行注入，Hilt 需要知道怎么向提供需要的依赖注入对象。binding 提供了相关的信息。

其中提供 binding 信息的一种方式是构造方法注入。在构造方法上使用 @Inject 注解，向 Hilt 描述如何提供这个类的对象。

```kotlin
class AnalyticsAdapter @Inject constructor(
    private val service: AnalyticsService
) {...}
```

被注解的构造方法的参数，是这个类的依赖。因此 Hilt 必须知道如何提供该依赖的实例，例如上例中的 AnalyticsService。

## Hilt modules

一些类无法使用构造方法注入，可以使用 Hilt modules 向 Hilt 提供 binding 信息。

Hilt modules 是使用了 @Module 注解的类。与 Dagger 不同的是，必须使用 @InstallIn 注解来告诉 Hilt 哪些类会被使用或引入。

在 Hilt modules 中提供的依赖可以在所有的生成的组件中使用，这些组件是与安装了 Hilt module 的类关联的。

### @Binds 注入 interface 实例

如果依赖是一个 interface，那么需要用 @Binds 注解修饰一个 Hilt module 创建的一个抽象方法。

- 该方法的返回类型告诉了 Hilt 该方法提供了什么样的接口的实例。
- 该方法的参数告诉 Hilt 需要提供哪些实现。

```kotlin
interface AnalyticsService {
    fun analyticsMethods()
}

class AnalyticsServiceImpl @Inject constructor(
    ...
): AnalyticsService {...}

@Module
@InstallIn(ActivityComponent::class)
abstract class AnalyticsModule {
    @Binds
    abstract fun bindAnalyticsService(
        analyticsServiceImpl: AnalyticsServiceImpl
    ): AnalyticsService
}
```

AnalyticsMoudle 使用了 @InstallIn(ActivityComponent::class) 这个注解，因为我们希望 Hilt 将其注入到 ExampleActivity 中。这个注解使用后，AnalyticsMoudle 中的所有依赖可以被应用中的 activity 使用。

### @Provides 注入实例

如果并不拥有创建实例的权限，也是不能使用构造方法注入的情况之一，如外部的库的实例，Retrofit、Room 之类的，或者 Builder 模式下的实例。

通过在 Hilt module 内创建一个方法，并使用 @Provides 注解修饰，告诉 Hilt 如何提供这种类型的实例。

注解的方法提供了如下的信息

- 返回值告诉 Hilt 该方法提供了什么样的接口的实例。
- 方法的参数告诉 Hilt 相应的依赖类型。
- 方法体告诉了 Hilt 如何提供相应类型的实例。Hilt 在每次需要提供实例时，会执行方法体。

```kotlin
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

    @Provides
    fun provideAnalyticsService(
        ...
    ): AnalyticsService {
        return Retrofit.Builder()
                .BaseUrl("https://example.com")
                .build()
                .create(AnalyticsService::class.java)
    }
}
```

### 为一个类型提供多个 binding

使用 qualifier 对同一类型提供多种 binding

定义限定词

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptorOkHttpClient

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class OtherInterceptorOkHttpClient
```

在 Hilt module 中，可以使用 @Provides 或者 @Binds，同时使用创建的 Qualifier 注解进行修饰。

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @AuthInterceptorOkHttpClient
    @Provides
    fun provideAuthInterceptorOkHttpClient(
        authInterceptor: AuthInterceptior
    ): OkHttpClient {
        return OkHttpClient.Builder()
                .addInterceptor(authInterceptor)
                .build()
    }

    @OtherInterceptorOkHttpClient
    @Provides
    fun provideAuthInterceptorOkHttpClient(
        otherInterceptor: OtherInterceptor
    ): OkHttpClient {
        return OkHttpClient.Builder()
                .addInterceptor(otherInterceptor)
                .build()
    }
}
```

接着就是使用对应的限定符进行注入

```kotlin

//作为其他类的依赖
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

    @Provides
    fun provideAnalyticsService(
        @AuthInterceptorOkHttpClient okHttpClient: OkHttpClient
    ): AnalyticsService {
        return Retrofit.Builder()
                .baseUrl("https://example.com")
                .client(okHttpClient)
                .build()
                .create(AnalyticsService::class.java)
    }
}

//作为构造注入类的依赖
class ExampleServiceImpl @Inject constructor(
    @AuthInterceptorOkHttpClient private val okHttpClient: OkHttpClient
): ...

//作为属性注入
@AndroidEntryPoint
class ExampleActivity: AppCompatActivity() {
    @AuthInterceptorOkHttpClient
    @Inject lateinit var okHttpClient: OkHttpClient
}
```

### Hilt 中 预定义的修饰符

@ApplicationContext @ActivityContext

```kotlin
class AnalyticsAdapter @Inject constructor(
    @ActivityContext private val context: Context,
    private val service: AnalyticsService
) {...}
```

## 为 Android 类生成的组件

为任意可以执行属性注入的 Android 类，有一系列可以用于 @InstallIn 注解的 Hilt 组件。每一个 Hilt 组件的责任是为注入对应的绑定找到对应的 Android 类。

| Hilt Component            | Injector for                              |
| ------------------------- | ----------------------------------------- |
| SingletonComponent        | Application                               |
| ActivityRetainedComponent | N/A                                       |
| ViewModelComponent        | ViewModel                                 |
| ActivityComponent         | Activity                                  |
| FragmentComponent         | Fragment                                  |
| ViewComponent             | View                                      |
| ViewWithFragmentComponent | View annotated with @WithFragmentBindings |
| ServiceComponent          | Service                                   |

### 组件生命周期

| Generated component       | Created at             | Destroyed at          |
| ------------------------- | ---------------------- | --------------------- |
| SingletonComponent        | Application#onCreate() | Application destroyed |
| ActivityRetainedComponent | Activity#onCreate()    | Activity#onDestroy()  |
| ViewModelComponent        | ViewModel created      | ViewModel destroyed   |
| ActivityComponent         | Activity#onCreate()    | Activity#onDestroy()  |
| FragmentComponent         | Fragment#onAttach()    | Fragment#onDestroy()  |
| ViewComponent             | View#super()           | View destroyed        |
| ViewWithFragmentComponent | View#super()           | View destroyed        |
| ServiceComponent          | Service#onCreate()     | Service#onDestroy()   |

### 组件作用域

所有 binding 默认是没有作用域的，每一次请求 binding 时，Hilt 都会创建一个新的实例。

| Android class | Generated component                                            | Scope                   |
| ------------- | -------------------------------------------------------------- | ----------------------- |
| Application   | SingletonComponent                                             | @Singleton              |
| Activity      | ActivityRetainedComponent                                      | @ActivityRetainedScoped |
| ViewModel     | ViewModelComponent                                             | @ViewModelScoped        |
| Activity      | ActivityComponent                                              | @ActivityScoped         |
| Fragment      | FragmentComponent                                              | @FragmentScoped         |
| View          | ViewComponent                                                  | @ViewScoped             |
| View          | annotated with @WithFragmentBindings ViewWithFragmentComponent | @ViewScoped             |
| Service       | ServiceComponent                                               | @ServiceScoped          |

```kotlin
@ActivityScoped
class AnalyticsAdapter @Inject constructor(
    private val service: AnalyticsService
) {...}
```

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class AnalyticsModule {

    @Singleton
    @Binds
    abstract fun bindAnalyticsService(
        analyticsServiceImpl: AnalyticsServiceImpl
    ): AnalyticsService
}

@Moudle
@InstallIn(SingletonComponent::class)
object AnalyticsModule {

    @Singleton
    @Provides
    fun provideAnalyticsService(): AnalyticsService {
        return Retrofit.builder()
                .baseUrl("https://example.com")
                .build()
                .create(AnalyticsService::class.java)
    }
}
```

[更多：Scoping in Android and Hilt](https://medium.com/androiddevelopers/scoping-in-android-and-hilt-c2e5222317c0)

### 组件层级

### 组件默认 binding

| Android component         | Default bindings                      |
| ------------------------- | ------------------------------------- |
| SingletonComponent        | Application                           |
| ActivityRetainedComponent | Application                           |
| ViewModelComponent        | SavedStateHandle                      |
| ActivityComponent         | Application, Activity                 |
| FragmentComponent         | Application, Activity, Fragment       |
| ViewComponent             | Application, Activity, View           |
| ViewWithFragmentComponent | Application, Activity, Fragment, View |
| ServiceComponent          | Application, Service                  |

application context binding 可以使用 @ApplicationContext

```kotlin
class analyticsServiceImpl @Inject constructor(
    @ApplicationContext context: Context
): AnalyticsService {...}

// The Application binding is available without qualifiers.
class AnalyticsServiceImpl @Inject constructor(
    application: Application
): AnalyticsService {...}

class AnalyticsAdapter @Inject constructor(
  @ActivityContext context: Context
) { ... }

// The Activity binding is available without qualifiers.
class AnalyticsAdapter @Inject constructor(
  activity: FragmentActivity
) { ... }
```

## 为不支持 Hilt 的类注入依赖

使用 @EntryPoint 创建一个入口。

```kotlin
class ExampleContentProvider : ContentProvider() {

    @EntryPoint
    @InstallIn(SingletonComponent::class)
    interface ExampleContentProviderEntryPoint {
        fun analyticsService(): AnalyticsService
    }
}
```

访问 entry point 需要使用 EntryPointAccessors的静态方法，参数可以是组件的实例或者使用@AndroidEntryPoint注释的对象（作为组件的持有者）。需要传入的参数和静态方法都与@InstallIn注解修饰的@EntryPoint 类匹配。

```kotlin
class ExampleContentProvider: ContentProvider() {

    override fun query(...): Cursor {
        val appContext = context?.applicationContext ?: throw IllegalStateException()
        val hiltEntryPoint = 
            EntryPointAccessors.fromApplication(appContext, ExampleContentProviderEntryPoint::class.java)

        val analyticsService = hiltEntryPoint.analyticsService()
    }
}
```

在这个例子中，必须使用 ApplicationContext 来获取 entry point，因为 entry point 安装在了 SingletonComponent 中。如果需要的 binding 是在 ActivityComponent 中，那么应该使用 ActivityContext。

## Hilt 和 Dagger

Hilt 基于 Dagger，提供了一个标准的方式将 Dagger 应用到 Android 中。

基于 Dagger， Hilt 的目标是：

- 简化 Dagger 相关的基础构建。
- 制定component 集合和作用域的标准，从而简化配置，提高可读性，代码的在其他APP的可复用性
- 提供一个简单的方式去供应不同的 binding 给各种类型的 build types，例如 testing debug 或 release

因为 Android 系统会实例化许多framework 中的类，使用 Dagger 时，需要写很多模板代码。Hilt 减少模板代码，自动生成并提供以下：

- 与 Android 框架集成的组件
- 定义域注解的组件
- 预定义的 binding，例如 Application Activity
- 预定义的 修饰符 例如 @ApplicationContext @ActivityContext

## 其他资源

samples

https://github.com/android/architecture-samples/tree/views-hilt

https://github.com/android/architecture-samples

codelabs

https://codelabs.developers.google.com/codelabs/android-hilt/

https://codelabs.developers.google.com/codelabs/android-dagger-to-hilt/

blogs

https://medium.com/androiddevelopers/dependency-injection-on-android-with-hilt-67b6031e62d

https://medium.com/androiddevelopers/scoping-in-android-and-hilt-c2e5222317c0

https://medium.com/androiddevelopers/hilt-adding-components-to-the-hierarchy-96f207d6d92d

https://medium.com/androiddevelopers/migrating-the-google-i-o-app-to-hilt-f3edf03affe5