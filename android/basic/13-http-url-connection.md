# HttpURLConnection

## 使用

1. 通过 `URL.openConnection()` 并转型，获取 `HttpURLConnection`。
2. 准备请求，uri、请求头等。
3. 可以上传请求体，需要使用 `setDoOutput(true)`，通过 `URLConnection.getOutputStream()` 获取输出流来写入数据。
4. 读取响应。响应头通常包含 metadata 信息。响应体通过 `URLConnection.getInputStream()` 后去输入流来读取。如果没有响应体，该方法返回红的输入流。
5. 断开连接。当响应读取完成时，调用 `disconnect()` 来关闭连接，释放资源。

```java
URL url = new URL("http://www.google.com");
HttpURLConnection connection = (HttpURLConnection) url.openConnection();

try{
  connection.setRequestMethod("GET");
  connection.setConnectTimeout(5000);
  connection.setReadTimeout(10000);
  InputStream in = connection.getInputStream();
  //处理数据
} finally {
  connection.disconnect();
}

// Post 请求，带请求体
connection.setRequestMethod("POST");
DataOutputStream out = new DataOutputStream(connection.getOutputStream());
out.writeBytes("username=alkdj&id=1231");
```

## MIME TYPE 常见

http 请求返回中的 Content-Type:xxx

- text/html 文本方式的html
- text/plain 纯文本
- text/xml 文本方式的xml
- application/x-www-from-urlencoded 表单提交（普通表单，非上传）
- application/json 数据以json形式编码
- application/xml 数据以xml形式编码
- multipart/form-data 表单上传图片、文件类型等附件时必须用该类型

## HTTPS

如果 URL 是 https 开头的，调用 `URL.openConnection()` 会返回 `HttpsURLConnection`。可以重写默认的 `HostnameVerifier`、`SSLSocketFactory`。可以通过程序提供的 SSLSocketFactory 获取 X509Trustmanager，从而验证证书链，获取 X509KeyManager 来支持客户端的证书。详情参见 `HttpsURLConnection`

## 处理响应

HttpURLConnection 最多支持5个 HTTP 重定向。遵循从一个原服务器到另一个服务器的重定向。不遵循 HTTPS 到 HTTP 的重定向，反之亦然。

如果响应出错，`HttpURLConnection.getInputStream()` 会抛出 `IOException`。调用 `getErrorStream()` 读取错误响应。响应头可以正常访问，通过 `URLConnection.getHeaderFields()`。

## 发送数据

调用 `setDoOutput(true)` 后才可以发送数据。

为提高性能，可调用 `setFixedLengthStreamingMode(int)` 或者 `setChunkedStreamingMode(int)` 来指定请求体的长度或者不指定长度。不设置的话，HttpURLConnection 会强制缓冲所有的请求体至内存中，然后进行传输，如此会增加延迟、消耗内存。

```java
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
try {
  urlConnection.setDoOutput(true);
  urlConnection.setChunkedStreamingMode(0);

  OutputStream out = new BufferedOutputStream(urlConnection.getOutputStream());
  writeStream(out);

  InputStream in = new BufferedInputStream(urlConnection.getInputStream());
  readStream(in);
} finally {
  urlConnection.disconnect();
}
```

## 性能

HttpURLConnection 通过 API 获取的输入输出流是没有缓冲的，最好使用 `BufferedInputStream` `BufferedOutputStream`。只有批量读取或写入的时候，才考虑忽略缓存。

传输大量数据时，使用流。

为了减少延迟，对于多个请求/响应对，Socket 可能会重用，因此 HTTP 连接会长时间被持有。调用 `disconnect()` 会将 socket 置于连接池中。

默认的 HttpURLConnection 请求实现，使用 gzip 压缩，并且默认解压缩从 `URLConnection.getInputStream()` 中读取的数据。这种情况下 Content-Encoding 和 Content-Length 响应头会被清除。Gzip 可通过在请求头中设置可接受的编码来禁用：`urlConnection.setRequestProperty("Accept-Encoding", "identity");`。显式设置 Accept-Encoding 请求头会禁用自动压缩，保持原始响应头。因为 Content-Encoding 响应头的存在，还是在需要的情况下处理解压缩。

`URLConnection.getContentLength()` 返回传输的字节数，不能用于预测从压缩过 InputStream 中读取的字节数。而是直到读取值为 -1.

## 处理网络登录

一些 Wi-Fi 网络会拦截网络访问，直到用户点击进入登录页面。这类登录界面通常使用 HTTP 转发来完成。使用 `URLConnection.getURL()` 来测试是否被转发。只有当响应头收到后，方法才有效，可以通过 `URLConnection.getHeadrFields()` 或 `URLConnection.getInputStream()` 来触发，例如检查响应是否重定向到一个未知的主机：

```java
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
try {
  InputStream in = new BufferedInputStream(urlConnection.getInputStream());
  if (!url.getHost().equals(urlConnection.getURL().getHost())) {
    // we were redirected! Kick the user out to the browser to sign on?
  }
  ...
} finally {
  urlConnection.disconnect();
}
```

## HTTP 认证

HttpURLConnection 支持 `HTTP basic authentication`，使用 `Authenticator` 设置 VM 范围的认证处理：

```java
Authenticator.setDefault(new Authenticator() {
  protected PasswordAuthentication getPasswordAuthentication() {
    return new PasswordAuthentication(username, password.toCharArray());
  }
});
```

并不安全。

## Sessiongs 和 Cookies

HttpURLConnection 内置一个可扩展的 cookie manager。使用 `CookieHandler` 和 `CookieHandler` 来启动 VM 范围的 cookie 管理。

```java
CookieManager cookieManager = new CookieManager();
CookieHandler.setDefault(cookieManager);
```

`CookieManager` 默认只接受原始服务器的 cookie。同时包含两种其他的策略：`CookiePolicy.ACCEPT_ALL` 和 `CookiePolicy.ACCEPT_NONE`。实现 `CookiePolicy` 来进行自定义。

默认的 `CookieManager` 将 cookies 保存在内存中。VM 退出后，会清除。实现 `CookieStore` 自定义。

除了 HTTP 响应设置的 cookie 外，可通过代码设置 cookie。包含在 HTTP 请求头中，必须包含 域名和路径的属性。

默认 `HttpCookie` 只对于支持 RFC 2965 的服务器有效。许多服务器只支持 RFC 2109 较老的规范。为了支持这些，设置 cookie version 至 0：

```java
HttpCookie cookie = new HttpCookie("lang", "fr");
cookie.setDomain("twitter.com");
cookie.setPath("/");
cookie.setVersion(0);
cookieManager.getCookieStore().add(new URI("http://twitter.com/"), cookie);
```

## HTTP Method

默认使用 GET 方法。调用 `setDoOutput(true)` 后，使用 POST 方法。

其他的方法，通过调用 `setRequestMethod(String)` 使用：OPTIONS、HEAD、PUT、DELETE、TRACE。

## 代理

可通过 HTTP 或者 SOCKS 代理。使用 `URL.openConnection(Proxy)`。

## IPv6

支持

## 响应缓存

查看 `android.net.http.HttpResponseCache` ，从 Android 4.0 开始支持。

## 2.2 之前有 bug

## 设置请求头

```java
connection.setRequestMethod("POST");
connection.setRequestProperty("version", "1.2.3");//设置请求头
connection.setRequestProperty("token", token);//设置请求头
connection.connect();
```

## 上传文件参考

```java
try {
    URL url = new URL(getUrl);
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setRequestMethod("POST");
    connection.setDoOutput(true);
    connection.setDoInput(true);
    connection.setUseCaches(false);
    connection.setRequestProperty("Content-Type", "file/*");//设置数据类型
    connection.connect();

    OutputStream outputStream = connection.getOutputStream();
    FileInputStream fileInputStream = new FileInputStream("file");//把文件封装成一个流
    int length = -1;
    byte[] bytes = new byte[1024];
    while ((length = fileInputStream.read(bytes)) != -1){
        outputStream.write(bytes,0,length);//写的具体操作
    }
    fileInputStream.close();
    outputStream.close();

    int responseCode = connection.getResponseCode();
    if(responseCode == HttpURLConnection.HTTP_OK){
        InputStream inputStream = connection.getInputStream();
        String result = is2String(inputStream);//将流转换为字符串。
        Log.d("kwwl","result============="+result);
    }

} catch (Exception e) {
    e.printStackTrace();
}
```

## 上传表单文件

```java
try {

    String BOUNDARY = java.util.UUID.randomUUID().toString();
    String TWO_HYPHENS = "--";
    String LINE_END = "\r\n";

    URL url = new URL(URLContant.CHAT_ROOM_SUBJECT_IMAGE);
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setRequestMethod("POST");
    connection.setDoOutput(true);
    connection.setDoInput(true);
    connection.setUseCaches(false);

    //设置请求头
    connection.setRequestProperty("Connection", "Keep-Alive");
    connection.setRequestProperty("Charset", "UTF-8");
    connection.setRequestProperty("Content-Type","multipart/form-data; BOUNDARY=" + BOUNDARY);
    connection.setRequestProperty("Authorization","Bearer "+UserInfoConfigure.authToken);
    connection.connect();

    DataOutputStream outputStream = new DataOutputStream(connection.getOutputStream());
    StringBuffer strBufparam = new StringBuffer();
    //封装键值对数据一
    strBufparam.append(TWO_HYPHENS);
    strBufparam.append(BOUNDARY);
    strBufparam.append(LINE_END);
    strBufparam.append("Content-Disposition: form-data; name=\"" + "groupId" + "\"");
    strBufparam.append(LINE_END);
    strBufparam.append("Content-Type: " + "text/plain" );
    strBufparam.append(LINE_END);
    strBufparam.append("Content-Lenght: "+(""+groupId).length());
    strBufparam.append(LINE_END);
    strBufparam.append(LINE_END);
    strBufparam.append(""+groupId);
    strBufparam.append(LINE_END);

    //封装键值对数据二
    strBufparam.append(TWO_HYPHENS);
    strBufparam.append(BOUNDARY);
    strBufparam.append(LINE_END);
    strBufparam.append("Content-Disposition: form-data; name=\"" + "title" + "\"");
    strBufparam.append(LINE_END);
    strBufparam.append("Content-Type: " + "text/plain" );
    strBufparam.append(LINE_END);
    strBufparam.append("Content-Lenght: "+"kwwl".length());
    strBufparam.append(LINE_END);
    strBufparam.append(LINE_END);
    strBufparam.append("kwwl");
    strBufparam.append(LINE_END);

    //拼接完成后，一块写入
    outputStream.write(strBufparam.toString().getBytes());


    //拼接文件的参数
    StringBuffer strBufFile = new StringBuffer();
    strBufFile.append(LINE_END);
    strBufFile.append(TWO_HYPHENS);
    strBufFile.append(BOUNDARY);
    strBufFile.append(LINE_END);
    strBufFile.append("Content-Disposition: form-data; name=\"" + "image" + "\"; filename=\"" + file.getName() + "\"");
    strBufFile.append(LINE_END);
    strBufFile.append("Content-Type: " + "image/*" );
    strBufFile.append(LINE_END);
    strBufFile.append("Content-Lenght: "+file.length());
    strBufFile.append(LINE_END);
    strBufFile.append(LINE_END);

    outputStream.write(strBufFile.toString().getBytes());

    //写入文件
    FileInputStream fileInputStream = new FileInputStream(file);
    byte[] buffer = new byte[1024*2];
    int length = -1;
    while ((length = fileInputStream.read(buffer)) != -1){
        outputStream.write(buffer,0,length);
    }
    outputStream.flush();
    fileInputStream.close();

    //写入标记结束位
    byte[] endData = (LINE_END + TWO_HYPHENS + BOUNDARY + TWO_HYPHENS + LINE_END).getBytes();//写结束标记位
    outputStream.write(endData);
    outputStream.flush();

    //得到响应
    int responseCode = connection.getResponseCode();
    if(responseCode == HttpURLConnection.HTTP_OK){
        InputStream inputStream = connection.getInputStream();
        String result = is2String(inputStream);//将流转换为字符串。
        Log.d("kwwl","result============="+result);
    }

} catch (Exception e) {
    e.printStackTrace();
}
```

## POST 参数

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            URL url = new URL(getUrl);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("POST");
            connection.setDoOutput(true);
            connection.setDoInput(true);
            connection.setUseCaches(false);
            connection.connect();

            String body = "userName=zhangsan&password=123456";
            BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(connection.getOutputStream(), "UTF-8"));
            writer.write(body);
            writer.close();

            int responseCode = connection.getResponseCode();
            if(responseCode == HttpURLConnection.HTTP_OK){
                InputStream inputStream = connection.getInputStream();
                String result = is2String(inputStream);//将流转换为字符串。
                Log.d("kwwl","result============="+result);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}).start();
```

## POST JSON

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            URL url = new URL(getUrl);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("POST");
            connection.setDoOutput(true);
            connection.setDoInput(true);
            connection.setUseCaches(false);
            connection.setRequestProperty("Content-Type", "application/json;charset=utf-8");//设置参数类型是json格式
            connection.connect();

            String body = "{userName:zhangsan,password:123456}";
            BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(connection.getOutputStream(), "UTF-8"));
            writer.write(body);
            writer.close();

            int responseCode = connection.getResponseCode();
            if(responseCode == HttpURLConnection.HTTP_OK){
                InputStream inputStream = connection.getInputStream();
                String result = is2String(inputStream);//将流转换为字符串。
                Log.d("kwwl","result============="+result);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}).start();
```