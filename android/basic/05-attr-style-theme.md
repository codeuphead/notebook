# 属性、式样、主题

## 属性

```xml
<declare-styleable name="ViewGroup_Layout">
    <attr name="layout_width" format="dimension">
        <enum name="fill_parent" value="-1" />
        <enum name="match_parent" value="-1" />
        <enum name="wrap_content" value="-2" />
    </attr>
    ...
</declare-styleable>

<attr name="textStyle">
    <flag name="normal" value="0" />
    <flag name="bold" value="1" />
    <flag name="italic" value="2" />
</attr>
```

enum 只能选择一个值，flag 可以进行‘或’运算，format 也可以进行运算

获取属性

```java
Theme theme = context.getTheme();
TypedArray array = theme.obtainStyledAttributes(attrs, R.styleable.xxx, defStyleAttr, defStyleRes);

String name = array.getXXX(R.styleable.XXX_XXX);

array.recycle();
```

## 式样、主题

Theme 是一个应用给整个 Activity 或 APP 的 Style

### 定义式样

继承自定义的式样，可以使用 parent，也可以在 name 属性中用 `.` 来进行分割，可以进行链式继承

```xml
<style name="CodeFont.Red">
    <item name="android:textColor">#FF0000</item>
</style>
```

所有的 XML 属性都可以使用 Style 文件进行定义，对于不支持的属性，会进行忽略

查看 R.attr 文档

以 window 开始的属性不是针对某一个 View 而是整个窗口

### 应用式样

- xml 中指定
- 构造方法中指定（5.0）
- 给 Activity/Application 元素使用 theme

通常控件都有默认的式样，可以设置其他可选式样，使用 Widget.xxx.xxx

可以查看 R.style 文档中 Widget 开头的

所有主题，可以查看 R.style.Theme 开头的

覆盖一部分的 Theme，针对某一个控件使用某一 ThemeOverlay，R.style.ThemeOverlay

### 主题兼容

Theme.AppCompat

android:Theme
