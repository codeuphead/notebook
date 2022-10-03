# WebView

## 设置属性

`WebView.getSetting()` 可设置一些 WebView 的属性。

`setJavaScriptEnabled()` 支持 JavaScript 脚本

## setWebViewClient()

设置了 WebViewClient 那么就会在该 WebView 打开新的重定向的链接，如果不设置，那么默认使用 ActionView 选择合适的应用程序打开链接。

shouldOverrideUrlLoading 默认返回 false，不去拦截url 的加载，返回true，则由程序处理 url 的加载，webView不再处理。

## loadUrl()

## JS 调用 Java 方法

1. 允许 WebView 加载 JS `webView.getSettings().setJavaScriptEnabled(true);`
2. 编写 JS 接口类，编写供 JS 调用的方法，给方法添加注解 `@JavascriptInterface`
3. 给 WebView 添加 JS 接口，调用 `webView.addJavaScriptInterface(obj, name);`，obj 参数为需要创建的第二部中 JS 接口类的对象，name 参数为创建的对象，在 JS 中对象的名称。
4. JS 中调用：
    ```
    if(windows.name){
        //接口存在
        name.xxxx
    }else{
        //接口不存在
    }
    ```
5. JS 会在非主线程中调用 Android 方法（JS Bridge）

## Java 调用 JS 方法

webView.loadUrl("javascript:jsString");

jsString：js 代码

```
"javascript:if(window.test){window.test('sdfsdf')}"
```