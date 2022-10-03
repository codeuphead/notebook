# Butter Knife

## 引入

```groovy
dependencies {
  implementation 'com.jakewharton:butterknife:10.0.0'
  annotationProcessor 'com.jakewharton:butterknife-compiler:10.0.0'
}
```

## `@BindView`

对 field 使用 `@BindView` 注解，指定 view id，ButterKnife 会自动寻找并进行转型

```java
class ExampleActivity extends Activity {
  @BindView(R.id.title) TextView title;
  @BindView(R.id.subtitle) TextView subtitle;
  @BindView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

框架通过 bind 方法会，生成对应的代码，如 `findViewById` 等。

## 资源绑定

`@BindBool @BindColor @BindDimen @BindDrawable @BindInt @BindString`

```java
class ExampleActivity extends Activity {
  @BindString(R.string.title) String title;
  @BindDrawable(R.drawable.graphic) Drawable graphic;
  @BindColor(R.color.red) int red; // int or ColorStateList field
  @BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
  // ...
}
```

## 非 Activity 的绑定

可在任意提供了根 view 的对象中进行绑定

```java
public class FancyFragment extends Fragment {
  @BindView(R.id.button1) Button button1;
  @BindView(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }
}
```

也可在 ViewHolder 模式中使用，进行简化操作

```java
static class ViewHolder {
    @BindView(R.id.title) TextView name;
    @BindView(R.id.job_title) TextView jobTitle;

    public ViewHolder(View view) {
      ButterKnife.bind(this, view);
    }
  }
```

## 其他绑定方式

- 将一个 Activity 作为根 View，来绑定任意对象。

例如使用 MVC 模式，可以绑定 `ButterKnife.bind(this, activity)`

- 绑定 View 的子 view 使用 `ButterKnife.bind(this)`

如果在布局中使用 `<merge>` 标签，那么可以在自定义 view 的构造方法中，inflate 该 view 之后立即调用。

通过 XML 创建的自定义 view，可以在 onFinishInflate() 回调中调用该方法。

## View 列表

- 可以将多个 View 放到列表中

```java
@BindViews({ R.id.first_name, R.id.middle_name, R.id.last_name })
List<EditText> nameViews;
```

- apply 方法可以一次对 list 中的所有 View 进行操作。

```java
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);
```

- 通过 Action 和 Setter 接口，可以进行一些简单行为的定制

```java
static final ButterKnife.Action<View> DISABLE = new ButterKnife.Action<View>() {
  @Override public void apply(View view, int index) {
    view.setEnabled(false);
  }
};

static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
  @Override public void set(View view, Boolean value, int index) {
    view.setEnabled(value);
  }
};
```

- Android 属性可以直接通过 apply 方法使用

```java
ButterKnife.apply(nameViews, View.ALPHA, 0.0f);
```

## 监听器 绑定

- 监听器可以通过在方法上加上注解自动配置。

```java
@OnClick(R.id.submit)
public void submit(View view) {
  // TODO submit data to server...
}
```

- 所有参数都是可选的

```java
@OnClick(R.id.submit)
public void submit() {
  // TODO submit data to server...
}
```

- 可以定义特殊的类型，会自动进行转型。

```java
@OnClick(R.id.submit)
public void sayHi(Button button) {
  button.setText("Hello!");
}
```

- 可以给多个 ID 进行绑定单独的一个处理方法

```java
@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
  if (door.hasPrizeBehind()) {
    Toast.makeText(this, "You win!", LENGTH_SHORT).show();
  } else {
    Toast.makeText(this, "Try again", LENGTH_SHORT).show();
  }
}
```

- 自定义 View 可以不通过 ID 进行绑定

```java
public class FancyButton extends Button {
  @OnClick
  public void onClick() {
    // TODO do something!
  }
}
```

## 绑定重置

fragment 有和 activity 不同的生命周期，当在 onCreateView 中进行绑定的时候，需要在 onDestroyView 中进行重置

ButterKnife 返回一个 UnBinder 实例帮助我们实现这个方法。

```java
public class FancyFragment extends Fragment {
  @BindView(R.id.button1) Button button1;
  @BindView(R.id.button2) Button button2;
  private Unbinder unbinder;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    unbinder = ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }

  @Override public void onDestroyView() {
    super.onDestroyView();
    unbinder.unbind();
  }
}
```

### 可选的绑定

如果目标 View 无法找到，会抛出一个异常。

可以使用 @Nullable(field) 或者 @Optional(method) 来阻止这个特性。

任何 @Nullable 的注解都可以对 fields 进行使用。鼓励使用 Android 的 support-annotations library。

```java
@Nullable @BindView(R.id.might_not_be_there) TextView mightNotBeThere;

@Optional @OnClick(R.id.maybe_missing) void onMaybeMissingClicked() {
  // TODO ...
}
```

### 多方法监听器

对有多个回调的监听器的情况下的方法注解，每个注解有一个默认的绑定的回调。

使用 callback 注解参数，指定一个另外可选的。

```java
@OnItemSelected(R.id.list_view)
void onItemSelected(int position) {
  // TODO ...
}

@OnItemSelected(value = R.id.maybe_missing, callback = NOTHING_SELECTED)
void onNothingSelected() {
  // TODO ...
}
```
