# Maunual Dependency Injection

## 手动管理依赖注入

```kotlin
class UserRepository(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) {...}

class UserLocalDataSource {...}
class UserRemoteDataSource(
    private val loginService: LoginRetrofitService
) {...}

class LoginActivity: Activity() {
    private lateinit var loginViewModel: LoginViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
        //为了满足 LoginViewModel 这个依赖，还必须递归地满足依赖的依赖。

        val retrofit = Retorfit.Builder()
            .baseUrl("https://example.com")
            .build()
            .create(LoginService::class.java)
        val remoteDataSource = UserRemoteDataSource(retrofit)
        val localDataSource = UserLocalDataSource()

        val userRepository = UserRepository(localDataSource, remoteDataSource)

        loginViewModel = LoginViewModel(userRepository)
    }
}
```

这种方式的问题：

- 很多模板代码。
- 依赖必须依次声明。
- 对象很难重用。

## 容器管理依赖注入

```kotlin
class AppContainer {
    private val retrofit = Retrofit.Builder()
                            .baseUrl("https://example.com")
                            .build()
                            .create(LoginService::class.java)
    private val remoteDataSource = UserRemoteDataSource(retrofit)
    private val localDataSource = UserLocalDataSource()

    val userRepository = UserRepository(localDataSource, remoteDataSource)   
}

class MyApplication: Application() {
    val appContainer = AppContainer()
}

class LoginActivity: Activity() {
    private lateinit var loginViewModel: LoginViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val appContainer = (application as MyApplication).appContainer
        loginViewModel = LoginViewModel(appContainer.userRepository)
    }
}
```

还可以将 LoginViewModel 放在集中的容器中进行管理

```kotlin
interface Factory<T> {
    fun Create(): T
}

class LoginViewModelFactory(private val userRepository: UserRepository): Factory {
    override fun create(): LoginViewModel {
        return LoginViewModel(userRepository)
    }
}

class AppContainer {
    val userRepository = UserRepository(localDataSource, remoteDataSource)

    val loginViewModelFactory = LoginViewModelFactory(userRepository)
}

class LoginActivity: Activity() {
       override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val appContainer = (application as MyApplication).appContainer
        loginViewModel = appContainer.loginViewModelFactory.create()
    }
}
```

这种方式的问题：

- 需要管理 AppContainer，手动创建依赖对象。
- 仍然有许多模板代码。

## 在程序工作流中管理依赖

```kotlin
class LoginContainer(val userRepository: UserRepository) {
    val loginData = LoginUserData()

    val loginViewModelFactory = LoginViewModelFactory(userRepository)
}

class AppContainer {
    val userRepository = UserRepository(localDataSource, remoteDataSource)

    val loginContainer: LoginContainer? = null
}

class LoginActivity: Activity() {
    private lateinit var loginViewModel: LoginViewModel
    private lateinit var loginData: LoginUserData
    private lateinit var appContainer: AppContainer

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        appContainer = (application as MyApplication).appContainer

        appContainer.loginContainer = LoginContainer(appContainer.userRepository)

        loginViewModel = appContainer.loginContainer.loginViewModelFactory.create()
        loginData = appContainer.loginContainer.loginData
    }

    override fun onDestroy() {
        appContainer.loginContainer = null
        super.onDestroy()
    }
}
```