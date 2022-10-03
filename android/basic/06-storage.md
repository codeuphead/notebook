# Storage

文件存储、SharedPreferences、数据库

## 文件存储

- Context 类 openFileOutput() openFileInput()
  - 默认存储在 /data/data/[packagename]/files/ 下
  - mode 有  MODE\_PRIVATE 和 MODE\_APPEND。

## SharedPreferences 键值对存储

- Context 类 getSharedPreferences(String fileName, int mode)
  - 默认在 /data/data/[packagename]/shared_prefs/ 下
  - 只有 MODE_PRIVATE
- Activity 类 getSharedPreferences()
  - 默认文件名为 Activity 的类名
- PreferenceManager.getDefaultSharedPreferences()
  - 默认文件名为包名前缀。

## 数据库

- LitePal 框架
