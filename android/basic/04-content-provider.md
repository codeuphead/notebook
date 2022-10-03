# ContentProvider

## Basic

- Context 类 getContentResolver()
- 通过 URI 标识数据。主要组成部分 authority 和 path
  - authority 用于对不同应用程序做区分
  - path 用于对同一应用不同的表做区分
- query、delete、insert、update，方法，相当于数据库的操作

## 创建自己的 ContentProvider

- 继承 ContentProvider，需要重写6个方法，onCreate、query、insert、update、delete、getType
- onCreate() 初始化组件的时候会调用，通差个你完成对数据库的创建和升级操作，返回true，表示初始化成功。只有当 ContentResolver 尝试访问我们的程序的时候，才会初始化。
- getType() 根据传入的内容的 URI，返回相应的 MIME 类型
- 需要对 URI 进行解析，来进行期望的访问
- URI 匹配相关
  - URI 标准为 content://com.example.app.provider/tableA，表示期望访问的是 com.example.app 的应用，tableA 表中的数据，还可以在 URI 后面加一个 id。content：//com.example.app.provider/tableA/1。以路径结尾表示想要访问所有数据，以 id 结尾表示想要访问该 id 的数据，可以使用通配符来匹配这两种格式。
  - `#` 表示匹配任意长度字符
  - `*` 表示匹配任意长度数字
  - UriMatcher 辅助进行匹配。其提供的 addURI() 方法，接受3个参数，authority、path 和一个自定义代码。当调用它的 match() 方法时，可以将一个 URI 对象传入，返回值是某个能够匹配这个 URI 对象所对应的自定义代码，利用这个代码，我们就可以判断出调用方期望访问的是哪张表中的数据
- getType() 用于获取 Uri 对象所对应的 MIME 类型。通常 MIME 字符串主要由3个部分组成
  - 必须由 `vnd` 开头
  - 如果内容 URI 以路径结尾，则后接`android.cursor.dir/`，如果以 id 结尾`android.cursor.item/`
  - 最后接上 `vnd.<authority>.<path>`
  - URI 为 content://com.example.app.provider/table1 的 MIME 为 vnd.android.cursor.dir/vnd.com.example.app.provider.table1
  - URI 为 content://com.example.app.provider/table1/1 的 MIME 为 vnd.android.cursor.item/vnd.com.example.app.provider.table1
  - Uri 对象的 getPathSegments() 方法，会将 URI 权限之后的部分以`/`进行分割，将分割后的结果放到一个字符串列表中，列表中第一条数据是 path，第二条数据是 id 等。
  - AndroidManifest 中进行注册，需要填写的有`authorities`、enable=true、export=true

## 跨进程访问
