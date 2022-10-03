# SQLite

SQLiteDatabase 对象通过 SQLiteOpenHelper 进行管理，一个 SQLiteOpenHelper 对象持有一个数据库连接

创建 SQLiteOpenHelper 使用 Application 作为 Context 的参数

SQLite 每一行都有列 64位有符号整型 ROWID，当没有一个 INTEGER 主键时候，且需要提供一个 _id 给 CursorAdapter，可以`SELECT ROWID AS _id`进行替换