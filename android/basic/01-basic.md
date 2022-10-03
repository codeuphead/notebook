# Android Base

2003 年 Andy Rubin 等创办公司，2005 年 8 月谷歌收购。2008 年推出第一个版本

## 系统架构

- Linux 内核层。为硬件提供底层驱动
- 系统运行库和运行时库层。系统运行库是通过 C/C++ 库，提供主要特性支持。运行时库提供核心库，允许开发者使用 Java 开发应用。运行时库还包括虚拟机
- 应用框架层。提供构建应用时的各种 API
- 应用层。各种应用程序

## Android Studio 项目结构

- .gradle .idea 是 IDE 自动生成的一些文件
- build 编译生成文件
- app 项目中的代码
- build 编译生成文件
- libs 第三方 jar 库文件
- src 源代码文件
  - androidTest 编写 Android Test 测试用例
  - main/java java 代码
  - main/res 资源文件
  - main/AndroidManifest.xml 项目清单文件
  - test 单元测试用例
  - proguard-ruls.pro 代码混淆规则
- gradle 包含 gradle wrapper 的配置文件
- .gitignore 忽略文件
- build.gradle 构建脚本
- gradle.properties 全局 gradle 配置文件
- gradlew gradlew.bat 命令行中执行 gradle 命令用的
- xxx.iml IDEA 自动生成文件，用于标识。
- local.properties 指定 SDK 路径等本地信息
- settings.gradle 指定项目模块的引入，默认只有一个 app 模块
