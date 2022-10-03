# Creating Catalog Browser

使用 BrowseFragment

## Set UI Elements

- setBadgeDrawable() 放置右上角的图标，如果 setTitle() 也调用，该图标会替换 title 字符串。图标资源应该高 52dp
- setTitle() 设置右上角的文字
- setHeaderState() 和 setHeadersTransitionOnBackEnable() 隐藏或者显示 Headers
- setBrandColor() 设置 UI elements 的背景色
- setSearchAffordancColor() 设置搜索图标颜色

## 自定义 Header Views

创建布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/header_icon"
        android:layout_width="32dp"
        android:layout_height="32dp" />
    <TextView
        android:id="@+id/header_label"
        android:layout_marginTop="6dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>
```

使用 Presenter 并实现抽象方法，对 view holder 进行创建和绑定

```java
public class IconHeaderItemPresenter extends Presenter {
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup viewGroup) {
        LayoutInflater inflater = (LayoutInflater) viewGroup.getContext()
                .getSystemService(Context.LAYOUT_INFLATER_SERVICE);

        View view = inflater.inflate(R.layout.icon_header_item, null);

        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, Object o) {
        HeaderItem headerItem = ((ListRow) o).getHeaderItem();
        View rootView = viewHolder.view;

        ImageView iconView = (ImageView) rootView.findViewById(R.id.header_icon);
        Drawable icon = rootView.getResources().getDrawable(R.drawable.ic_action_video, null);
        iconView.setImageDrawable(icon);

        TextView label = (TextView) rootView.findViewById(R.id.header_label);
        label.setText(headerItem.getName());
    }

    @Override
    public void onUnbindViewHolder(ViewHolder viewHolder) {
    // no op
    }
}
```

需要将 view 设置为 focusable

### Hide or Disable Headers

`HEADERS_ENABLED`

`HEADERS_HIDDEN`

`HEADERS_DISABLED`
