# Intent

## 属性

- Uri data
  - 主要用于指定当前 Intent 正在操作的数据
  - manifest 中，`<data>` 标签中可以配置
    - android:scheme 指定数据的协议
    - android:host 指定数据的主机
    - android:port 指定端口
    - android:path 指定路径
    - android:mimeType 指定处理的数据类型

## 显式 Intent

Intent(Context packageContext, Class<?> cls)

## 隐式 Intent

Intent(String action)

指定一系列抽象的 action 和 category，只有 action category 同时匹配，组件才会响应。

android.intent.category.DEFAULT 是默认的 category，调用 startActivity() 的时候会自动加上。

一个 Intent 只能指定一个 action，但是可以指定多个 category。

Data 也会完全匹配