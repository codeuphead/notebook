# OkHttp

Square 开源库之一

```java
OKHttpClient client = new OkHttpClient();
Request request = new Request.Builder()
                        .url("http://www.google.com")
                        .build();
Response response = client.newCall(request).execute();
String responseData = response.body().string();

RequestBody requestBody = new FormBody.Builder()
                            .add("username", "name")
                            .add("password", "123")
                            .build();
Request request = new Request.Builder()
                    .url("http://www.google.com")
                    .post(requestBody)
                    .build();

```