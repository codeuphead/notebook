# Retrofit

- 一个 RESTful 的 HTTP 请求框架，基于 OkHttp
- 基于 OkHttp，遵循 RESTful API 设计风格
- 通过注解配置网络请求参数
- 支持同步/异步请求
- 支持多种数据的解析/序列化格式 Gson Json XML Protobuf
- 提供 RxJava 支持

## 引入

```
compile 'com.squareup.retrofit2:retrofit:2.3.0'
```

proguard

```
# Platform calls Class.forName on types which do not exist on Android to determine platform.
-dontnote retrofit2.Platform
# Platform used when running on Java 8 VMs. Will not be used at runtime.
-dontwarn retrofit2.Platform$Java8
# Retain generic type information for use by reflection by converters and adapters.
-keepattributes Signature
# Retain declared checked exceptions for use by a Proxy instance.
-keepattributes Exceptions
```

## 介绍

Retrofit 将 HTTP 接口 转换成为 Java interface

```java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

Retrofit 生成 Java interface 的实现

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

## 请求方式

每个方法必须有一个 HTTP 注解，提供请求方式和请求 URL。有5个内置的注解：`GET、POST、PUT、DELETE、HEAD`。资源 URL 定义在注解中。

```
@GET("users/list")
@GET("users/list?sort=desc")
```

也可通过 `@HTTP` 设置所有请求注解功能以及更过拓展

## URL 操作

请求的 URL 可以动态更新

替换块是大括号包裹的数字和字母字符串。相应的参数需要被 `@Path` 注解，并使用相同的字符串。

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```

可以添加 Query 参数

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```

复杂的 Query 参数组合，使用 Map

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```

## 请求 BODY

`@Body` 注解来使用 HTTP 请求 body 指定为一个对象。

```java
@POST("users/new")
Call<User> createUser(@Body User user);
```

该对象也可以使用一个 Retrofit 对象定义的 converter 来进行转换。如果 converter 没有添加，只能使用 RequestBody。

## form encoded 和 multipart data

方法也可以声明为发送 form-encoded 和 multipart data。

`@FormUrlEncoded` 出现在方法中的时候，Form-encoded 数据会被发送，每一个键值对用 `@Field` 注解，包含了名称和值（对象提供）。

```java
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```

`@Multipart` 出现在方法中的时候，会进行 Multipart 请求。每个部分使用 `@Part` 注解。

```java
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```

使用 Retrofit 的一种 converters 或者自己实现 RequestBody 来处理自己的序列化。

## Header

使用 `@Headers` 注解来设置

```java
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();

@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```

headers 不会被覆盖，相同 headers 都会包含在请求中。

可以进行动态更新 header，使用 `@Header` 注解。

```java
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```

## 同步和异步

`Call` 实例可以进行同步或者异步的执行。每个实例只能使用一次，可以通过 clone() 创建新的实例。

Android，回调会在主线程执行。JVM，回调会在 HTTP 请求线程。

## 配置选项

### Converters
