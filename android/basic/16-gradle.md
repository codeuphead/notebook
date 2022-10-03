# Gradle

Gradle Wrapper 由三部分组成

- 脚本文件 `gradlew.bat gradlew`
- JAR 文件 `gradle/wrapper`
- 配置文件 `gradle/wrapper/gradle-wrapper.properties`

配置文件的 `distributionUrl` 决定 Android Studio 下载 Gradle 的地方

## rule

- **只使用信任的 distributionUrl** 例如：`services.gradle.org`
- **只使用信任的 gradlew 脚本**

## 环境变量

如果是自己安装 Gradle，需要设置的环境变量

- GRADLE_HOME
- GRADLE_USER_HOME
- PATH 下添加 bin 路径

## Gradle 文件

### 项目级 文件

- 项目根目录下的 `build.gradle` 文件

`buildscript` 闭包，配置 JARs 和 Gradle 用于解释剩下文件内容的物料

`buildscript` 中的 `repositories` 指定插件的来源，通常为 `jcenter() 和 google()`

`buildscript` 中的 `dependencies` 指定运行剩下的构建脚本所需要的东西。其中 `classpath 'com.android.tools.build:gradle:3.0.0'` 为 google 提供的 gradle 构建插件

`allprojects` 闭包为所有模块都使用的配置，通常为 `repositories`，并设置为 `jcenter() 和 google()`，用来告诉程序模块中的依赖在哪里下载

### 模块级 文件

- 模块目录下的 `build.gradle` 文件

`dependencies` 闭包下的库，是模块所使用的

`android` 闭包下包含所有 Android 配置

顶部 `apply plugin: 'com.android.application`

## Gradle 和 Manifest

Manifest 的 package 控制一些自动生成的文件位置，必须是一个有效的位置，也是默认的 applicationID。Gradle 文件中的 android 闭包 的 defaultConfig 闭包中，通过 applicationId 语句可以重写
