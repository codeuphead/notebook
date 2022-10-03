# Kotlin 协程基本要素

- [Kotlin 协程基本要素](#kotlin-协程基本要素)
  - [挂起函数](#挂起函数)
    - [Continuation](#continuation)
  - [挂起函数的类型](#挂起函数的类型)
  - [将回调转成挂起函数](#将回调转成挂起函数)
  - [协程的创建](#协程的创建)
  - [启动协程](#启动协程)
  - [协程上下文](#协程上下文)
  - [拦截器](#拦截器)
  - [Continuation 执行示意](#continuation-执行示意)
  - [拦截 Continuation](#拦截-continuation)
  - [Java 模拟 Kotlin 协程代码](#java-模拟-kotlin-协程代码)
  - [协程挂起恢复要点](#协程挂起恢复要点)
  - [协程的线程调度](#协程的线程调度)

## 挂起函数

- 以`suspend`修饰的函数
- 只能在其他的**挂起函数**或**协程**中调用
- 挂起函数调用时，包含了协程挂起的语义
- 挂起函数返回时，包含了协程的恢复语义

## Continuation

```kotlin
interface Continuation<in T>{
  fun resumeWith(result: Result<T>)
}
```

## 挂起函数的类型

```kotlin
suspend fun foo(){}
```

函数类型：suspend () -> Unit

```kotlin
suspend fun bar(a: Int): String{
  return "Hello"
}
```

函数类型：suspend (Int) -> String

以上两个函数反编译后实际是：(包含了 continuation 对象)

```kotlin
fun foo(continuation: Continuation<Unit>): Any{}
```

```kotlin
fun bar(a:Int, continuation: Continuation<String>): Any{
  return "Hello"
}
```

如果函数没有挂起，Any 代表返回真正的返回值类型。如果函数有真正挂起，则 Any 代表挂起标志对象：`COROUTINE_SUSPENDED`

## 将回调转成挂起函数

使用 suspendCoroutine 获取挂起函数的 Continuation

回调成功的分支使用 Continuation.resume(value)

回调失败则使用 Contiuation.resumeWithException(e)

```kotlin
suspend fun getUserSuspend(name: String) = suspendCoroutine<User> {
  continuation -> 
    githubApi.getUserCallback(name).enqueue(object: Callback<User>{
      override fun onFailure(call: Call<User>, t: Throwable)  
        = continuation.resumeWithException(t)
      override fun onResponse(call: Call<User>, response: Response<User>)
        = response.takeIf{ it.isSuccessful }?.body()?.let(continuation::resume)
          ?:continuation.resumeWithException(HttpException(response))
    })
}
```

真正的挂起：必须异步调用 resume

## 协程的创建

协程是一段可执行的程序，通常需要一个函数，协程的创建需要一个 API。标准库中提供了一个函数 createCoroutine，通过它我们可以矿建协程。

```kotlin
fun <T> (suspend () -> T).createCoroutine(
  completion: Continuation<T>): Continuation<Unit>

fun <R, T> (suspend R.() -> T).createCoroutine(
  receiver: R,
  completion: Continuation<T>): Continuation<Unit>
)
```

createCoroutine 函数的 Receiver 是一个 suspend 修饰的挂起函数

创建协程传递的 continuation 和 返回的 contination 是两个东西

- suspend 函数本身执行需要一个 Continuation 实例在执行完成后调用，既此处参数中的 completion，实际是协程的完成回调
- 返回值 Contiunation 则是创建出来的协程的载体，可以通过这个值触发协程的启动。Receiver suspend 函数会被传递给该实例作为协程的实际执行体

创建协程

```kotlin
val continuation = suspend {
  println("In Coroutine")
  5
}.createCoroutine(object: Continuation<Int>{
  override fun resumeWith(result: Result<Int>){
    log("Coroutine End with $reuslt")
  }

  override val context = EmptyCorotinueContext
})

```

## 启动协程

调用 continuation.resume(Unit) 之后，协程体会立即开始执行

continuation 是 SafeContinuation 的实例，他有一个 delegate 的属性，这个属性才是 COntinuation 的本体。

用以创建协程的 suspend Lambda 表达式（协程体），编译后，编译器生成了一个内部类，这个类继承自 SuspendLambda 类，且是 Continuation 接口的实现类。

SuspendLambda 有一个抽象函数 invokeSuspend（在父类的 BaseContinuationImpl 中声明），编译生成的匿名内部类中这个函数的实现，就是协程体。

创建协程返回的 Continuation 实例就是套了几层马甲的协程体，因而调用它的 resume 就可以触发协程体的执行。

一般来讲，创建协程后，会立即执行，因此标准库提供了一个 API，startCoroutine，其与 createCoroutine 返回值不一样。

```kotlin
fun <T> (suspend () -> T).startCoroutine(
  completion: Continuation<T>
)
```

```kotlin
suspend{
  ...
}startCoroutine(object: Continuation<Unit>{
  override val context = EmptyCoroutineContext
  override fun resumeWith(result: Result<Unit>){
    log("Coroutine End with $result")
  }
})
```

## 协程上下文

协程执行过程中需要携带数据

索引是 CoroutineContext.Key

元素是 CoroutineContext.Element

## 拦截器

拦截器 ContinuationInterceptor 是一类协程上下文元素

可以对协程上下文所在协程的 Continuation 进行拦截

```kotlin
interface ContinuationInterceptor: CoroutineContext.Element {
  fun <T> interceptContinuation(
    continuation: Continuation<T>): Continuation<T>
}
```

## Continuation 执行示意

suspend{
  a()
}.startCoroutine(...)

SafeContinuation
  SuspendLambda

```kotlin
suspend fun a() = suspendCoroutine<Unit> {
  thread{
    it.resume(Unit)
  }
}
```

SafeContinuation 的作用就是确保：

- resume 只被调用一次
- 如果在当前线程调用栈上直接调用则不会挂起

## 拦截 Continuation

SafeContinuation
  Intercepeted
      SuspendLambda

SafeContinuation 仅在挂起点时出现

拦截器在每次（包括恢复）执行协程体时调用

SuspendLambda 是协程函数体

## Java 模拟 Kotlin 协程代码

`https://git.imooc.com/coding-398/Kotlin-Tutorials/src/master/CoroutineBasics/src/main/java/com/bennyhuo/kotlin/coroutinebasics/javaimpl/ContinuationImpl.java`

## 协程挂起恢复要点

- 协程体内的代码都是通过 Continuation。resumeWith 调用
- 每调用一次 label 加1，每一个挂起点对应于一个 case 分支
- 挂起函数在返回 COROUTINE_SUSPENDED 时才会挂起

## 协程的线程调度

```kotlin
suspend{
  ...
}
```
