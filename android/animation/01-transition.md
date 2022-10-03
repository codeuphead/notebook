# Transition 框架

三个核心类 Scene Transition TransitionManager。

## Scene

用于保存布局中所有 View 的属性值，不同的 scene 代表了不同的 view 层次关系。

```java
public static Scene getSceneForLayout(ViewGourp SceneRoot, int layoutId, Context context)

public Scene(ViewGroup sceneRoot, View layout)
```

## Transition

Transition 是框架提供的包含了动画信息的对象。作用是获取属性值，然后基于获取的属性值的变化进行动画。

```
java.lang.Object
android.transition.Transition

public abstract class Transition

//直接子类
ChangeBounds
ChangeClipBounds
ChangeImageTransform
ChangeScroll
ChangeTransform
TransitionSet
Visibility

//非直接子类
AutoTransition
Explode
Fade
Slide
```

### 注意

在 SurfaceView 和 TextureView 中不能正常工作，因为他们的显示方式。

### 内置 transition 类型

|Class|Tag|Attributes|Effect|
|-----|---|----------|------|
|AutoTransition|`<autoTransition>`|-|默认过渡。淡出、移动和缩放、淡入的顺序进行|
|Fade|`<fade>`|android:fadingMode="" `[fade_in\|fadeout\|fade_in_out]`|淡入淡出，默认为先淡入后淡出|
|ChangeBounds|`<changeBounds>`|-|移动和缩放|

### 创建 Transition

```java
//XML 生成
// /res/transition/ 目录下创建 XML 资源文件
<fade xmlns:android="http://schemas.android.com/apk/res/android" />

Transition mFadeTransition =
        TransitionInflater.from(this).
        inflateTransition(R.transition.fade_transition);

//代码生成
Transition mFadeTransition = new Fade();
```

### TransitionSet

```java
// /res/transitions/
// 示例为 AutoTransition

<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
    android:transitionOrdering="sequential">
    <fade android:fadingMode="fade_out" />
    <changeBounds />
    <fade android:fadingMode="fade_in" />
</transitionSet>

TransitionInflater.from()....

<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
     android:transitionOrdering="sequential">
    <changeBounds/>
    <fade android:fadingMode="fade_out" >
        <targets>
            <target android:targetId="@id/grayscaleContainer" />
        </targets>
    </fade>
</transitionSet>
```

## TransitionManager

将 Scene 和 Transition 联系起来，主要方法有：

```java
public static void go(Scene scene)
public static void go(Scene scene, Transition transition)
public static void beginDelayedTransition(final ViewGroup sceneRoot)
public static void beginDelayedTransition(final ViewGroup sceneRoot, Transition transition)
public static void endTransitions(final ViewGroup sceneRoot)
```

go() 开始 scene 是上一次 transition 的结束 scene。如果没有则从目前的状态自动创建。

如果不设置 Transition，会设置 automatic transition。

可以通过 `Transition.addTarget()` 和 `removeTarget()` 选择执行动画的 View。

### 不使用 Scene

在当前 View 树下进行改变。

从当前 View 树的状态开始，记录 View 的改变，并应用变换来对改变进行动画，在系统界面重绘的时候

`beginDelayedTransition()` 提供给它想改变换的 View 的父 View，框架会保存所有子 View 的状态和属性值。当系统因为改变属性而重绘时，会进行动画。

```java
private TextView mLabelText;
private Fade mFade;
private ViewGroup mRootView;
...

// Load the layout
this.setContentView(R.layout.activity_main);
...

// Create a new TextView and set some View properties
mLabelText = new TextView();
mLabelText.setText("Label").setId("1");

// Get the root view and create a transition
mRootView = (ViewGroup) findViewById(R.id.mainLayout);
mFade = new Fade(IN);

// Start recording changes to the view hierarchy
TransitionManager.beginDelayedTransition(mRootView, mFade);

// Add the new TextView to the view hierarchy
mRootView.addView(mLabelText);

// When the system redraws the screen to show this update,
// the framework will animate the addition as a fade in
```

## Transition 生命周期回调

通过 `TransitionListener` 接口来定义回调，从 TransitionManager.go() 方法到动画结束的时候。

```java
TransitionListener.onTransitionCancel(Transition tansition)
TransitionListener.onTransitionEnd(Transition tansition)
TransitionListener.onTransitionPause(Transition tansition)
TransitionListener.onTransitionResume(Transition tansition)
TransitionListener.onTransitionStart(Transition tansition)
```

## PathMotion

Transition 辅助工具，以 path 的方式指定过渡效果，两个具体实现类 ArchMotion 和 PatterPathMotion

```java
mTransition = new AutoTransition();
mTransition.setPathMotion(new ArcMotion());
TransitionManager.beginDelayedTransition(mRoot, mTransition);
FrameLayout.LayoutParams lp = (FrameLayout.LayoutParams) mTarget.getLayoutParams();
if ((lp.gravity & Gravity.START) == Gravity.START) {
    lp.gravity = Gravity.END | Gravity.BOTTOM;
} else {
    lp.gravity = Gravity.START | Gravity.TOP;
}
mTarget.setLayoutParams(lp);
```

## 自定义 Transition

```java
public class CustomTransition extends Transition {
    @Override
    public void captureStartValues(TransitionValues values) {}
    @Override
    public void captureEndValues(TransitionValues values) {}
    @Override
    public Animator createAnimator(ViewGroup sceneRoot,
                                   TransitionValues startValues,
                                   TransitionValues endValues) {}
}
```

使用属性动画系统，需要继承 Transition，在 captureStartValues 和 captureEndValues 中分别记录 View 的属性值。（官网建议确保属性值不冲突，属性值的命名格式参考）

`captureStartValues(TransitionValues v)` 方法在开始 scene 的时候会为每个 View 调用。 TransitionValues 对象包含了 View 的引用和 一个 Map 用来存储属性。使用以下 scheme 来作为属性值的 key `package_name:transition_name:property_name`。

```java
private void captureValues(TransitionValues transitionValues) {
    // Get a reference to the view
    View view = transitionValues.view;
    // Store its background property in the values map
    transitionValues.values.put(PROPNAME_BACKGROUND, view.getBackground());
}
```

`captureEndValues(TransitionValues v)` 方法在结束每个 View 的 scene 时候调用。调用与 captureStartValues 进行相同的实现

在 `createAnimator()` 中创建动画，对比属性值的改变执行动画效果，如自定义修改颜色动画效果。方法调用次数基于从开始到结束的改变次数，改变了多少个 View 就会调用多少次。

如果 View 从开始到结束都存在，那么 startValue 和 endValue 都会存在。如果只存在于某一个 Scene 中，那么另外不存在的情况下的 TransitionValues 为 null。

```java
@Override public Animator createAnimator(ViewGroup sceneRoot, TransitionValues startValues,
        TransitionValues endValues) {

      //两者都存在的时候，才进行动画
      if (null == startValues || null == endValues) {
        return null;
      }
      final View view = endValues.view;
      Drawable startBackground = (Drawable) startValues.values.get(PROPNAME_BACKGROUND);
      Drawable endBackground = (Drawable) endValues.values.get(PROPNAME_BACKGROUND);
      if (startBackground instanceof ColorDrawable && endBackground instanceof ColorDrawable) {
        ColorDrawable startColor = (ColorDrawable) startBackground;
        ColorDrawable endColor = (ColorDrawable) endBackground;
        if (startColor.getColor() != endColor.getColor()) {
          ValueAnimator animator = ValueAnimator.ofObject(new ArgbEvaluator(),
              startColor.getColor(), endColor.getColor());
          animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
              Object value = animation.getAnimatedValue();
              if (null != value) {
                view.setBackgroundColor((Integer) value);
              }
            }
          });
          return animator;
        }
      }
      return null;
    }
```

### 使用

代码中像 Transition 一样使用

xml 中 `<transition class="my.app.transition.CustomTransition"/>`

从 XML 中加载自定义 Transition 类，需要有含有 Context 和 AttributeSet 参数的构造。

## 自定义 Activity Transitions

```xml
<style name="BaseAppTheme" parent="android:Theme.Material">
  <!-- enable window content transitions -->
  <item name="android:windowActivityTransitions">true</item>

  <!-- specify enter and exit transitions -->
  <item name="android:windowEnterTransition">@transition/explode</item>
  <item name="android:windowExitTransition">@transition/explode</item>

  <!-- specify shared element transitions -->
  <item name="android:windowSharedElementEnterTransition">
    @transition/change_image_transform</item>
  <item name="android:windowSharedElementExitTransition">
    @transition/change_image_transform</item>
</style>

<!-- res/transition/change_image_transform.xml -->
<!-- (see also Shared Transitions below) -->
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
  <changeImageTransform/>
</transitionSet>
```

开启 Activity Transition

```java
//可以在 theme 中开启
// inside your activity (if you did not enable transitions in your theme)
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);

// set an exit transition
getWindow().setExitTransition(new Explode());
```

设置 transition

```java
Window.setEnterTransition()
Window.setExitTransition()
Window.setSharedElementEnterTransition()
Window.setSharedElementExitTransition()
```

立即使用，调用 `Window.setAllowEnterTransitionOverlap()`。

使用：

`startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this).toBundle());`

### 共享元素

1. 在 theme 开启 window content transitions
1. 在 style 定义 分享元素 transition
1. 定义 xml transition
1. 给两个 layout 中分配 android:transitionName 一致，动态生成的 View 使用 `View.setTransitionName()`
1. 使用 `ActivityOptions.makeSceneTransitionAnimation()`
1. `Activity.finishAfterTransition()` 进行反转

```java
Intent intent = new Intent(this, Activity2.class);
        // create the transition animation - the images in the layouts
        // of both activities are defined with android:transitionName="robot"
        ActivityOptions options = ActivityOptions
            .makeSceneTransitionAnimation(this, androidRobotView, "robot");
        // start the new activity
        startActivity(intent, options.toBundle());
```

对于多个共享元素，需要设置不同的名字

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(this,
        Pair.create(view1, "agreedName1"),
        Pair.create(view2, "agreedName2"));
```
