# IOC - Inversion of Control

## 依赖注入框架

- View 注入：Xutils、Android Annotations、ButterKnife
- 参数注入：Arouter - Intent 参数
- 对象注入：Dagger2、Hilt

## IOC 控制反转

- 控制：指的是对象的创建、管理的权力
- 反转：控制权交给外部环境（Spring 框架，IoC 容器）

Android 常用的解决方案是，使用注解（@inject），通过 annotation 和 abstractProcessor 编译时生成相关实现类，运行时动态注入。

## DI 依赖注入 

Dependency Injection

IOC 是一种软件设计的思想，DI 是这种软件设计思想的一个实现，相比于控制反转，依赖注入更容易理解。

依赖注入指的是对象是通过外部注入的方式实现的。

## IOC 优势

- 对象之间的耦合度/依赖程度降低
- 对象实例的创建变得容易管理，很容易可以实现一个全局或者局部共享的单例对象
- 使用场景：
  - 模板代码创建实例对象
  - 全局或局部对象共享

## IOC 劣势

- 代码可读性降低
- 增加新人学习成本
- 65535 方法数限制

## 引入 Hilt

项目根`build.gradle`文件引入插件

```kotlin
buildscript {
    ...
    ext.hilt_version = '2.35'
    dependencies {
        ...
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```

然后在各个模块应用插件，并引入 hilt lib

```kotlin
plugins {
    kotlin("kapt")
    id("dagger.hilt.android.plugin")
}

android {
    ...
}

dependencies {
    implementation("com.google.dagger:hilt-android:$hilt_version")
    kapt("com.google.dagger:hilt-android-compiler:$hilt_version")
}
```

## 配置

### 配置 Application

在 Application 使用`@HiltAndroidApp`注解，编译阶段会生成顶层的 components 容器，提供全局的 application context，为 application 提供对象注入的能力

```kotlin
@HiltAndroidApp
class MyApplication: Application {
  
}
```