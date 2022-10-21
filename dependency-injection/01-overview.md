# DI - Dependency Injection

基于 Inversion of Control 概念提出

## 总览

### DI 的优势

- 重用代码
- 易扩展
- 易测试

### 没有使用依赖注入

```kotlin
class Car {
    private val engine = Engine()
    
    fun start(){
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.start()
}
```

Car 和 Engine 是强耦合的。如果想要创建两种不同 engine 的 Car，就无法重用 Car 类。

因为使用了自己创建的 Engine 对象，从而使得测试更加困难。

### 使用依赖注入

```kotlin
class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val engine = Engine()
    val car = Car(engine)
    car.start()
}
```

针对不同类型的 Engine，Car 类也可以重用。

Car 变得可测。

### Android 中的依赖注入

- Constructor 注入，如上例子。
- Field 注入（或者叫 setter 注入）。

```kotlin
class Car {
    lateinit var engine: Engine
    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.engine = Engine()
    car.start()
}
```

### 自动注入

- 运行时注入，基于反射。
- 静态注入，在编译时生成特定代码进行注入。

Dagger 是静态注入，相较于 Guice 等运行时注入框架，有更好的开发便利和性能指标。

## 依赖注入替代品 IOC

service locator 设计模式（类似 IOC）

```kotlin
object ServiceLocator {
    fun getEngine(): Engine = Engine()
}

class Car {
    private val engine = ServiceLocator.getEngine()

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.start()
}
```

该设计模式下，Car 类会请求需要注入的类。DI 则是更主动的注入对象。

### 优势

- 对象之间的耦合度/依赖程度降低
- 对象实例的创建变得容易管理，很容易可以实现一个全局或者局部共享的单例对象
- 使用场景：
  - 模板代码创建实例对象
  - 全局或局部对象共享

### 劣势

- 将依赖存储在 service locator 中，所有的测试需要与之进行互动，因此使得测试困难。
- 依赖对象在类的实现中进行创建，而不是 API 层面，因此外部很难知道需要什么，并且容易在运行时或测试时因为改变了 Car 或者依赖的实现而导致错误。
- 管理依赖的生命周期变得复杂。

## 引入 Hilt

Hilt 是 Jetpack 推荐的依赖注入库。基于 Dagger。

## 总结 DI 优势

- 重用类，解耦依赖。
- 容易重构扩展。
- 容易测试。

## 相关链接

[Test Double](https://en.wikipedia.org/wiki/Test_double)
