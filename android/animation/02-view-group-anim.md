# View Group Animation

## LayoutAnimation

```xml
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
                 android:delay="1"
                 android:animationOrder="normal"
                 android:animation="@anim/slide_in_left"/>
```

- android:delay 每个 item 的动画开始的延时，取值是 - android:animation 所指定的动画时长的倍数，可以是 float 类型，也可以是百分数
- android:order viewGroup 中控件动画开始的顺序，normal、reverse、random
- android:animation 动画 xml 文件

```java
public LayoutAnimationController(Animation animation)

public LayoutAnimationController(Animation animation, float delay)
```

```java
public void setAnimation(Animation animation)

public void setDelay(float delay)

public void setOrder(int order)
```

示例

```java
Animation animation= AnimationUtils.loadAnimation(this,R.anim.slide_in_left);   //得到一个LayoutAnimationController对象；
        LayoutAnimationController controller = new LayoutAnimationController(animation);   //设置控件显示的顺序；
        controller.setOrder(LayoutAnimationController.ORDER_REVERSE);   //设置控件显示间隔时间；
        controller.setDelay(0.3f);   //为ListView设置LayoutAnimationController属性；
        mListView.setLayoutAnimation(controller);
        mListView.startLayoutAnimation();
```

## gridLayoutAnimation

- rowDelay
- columnDelay
- directionPriority 方向优先级，row、column、none
- xxdirection gridView 的动画方向，`left_to_right right_to_left top_to_bottom bottom_to_top`

```java
<gridLayoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
                     android:rowDelay="75%"
                     android:columnDelay="60%"
                     android:directionPriority="none"
                     android:direction="bottom_to_top|right_to_left"
                     android:animation="@anim/slide_in_left"/>
```

```java
public GridLayoutAnimationController(Animation animation)

public GridLayoutAnimationController(Animation animation, float columnDelay, float rowDelay)
```

```java
Animation animation = AnimationUtils.loadAnimation(MyActivity.this,R.anim.slide_in_left);
GridLayoutAnimationController controller = new GridLayoutAnimationController(animation);
controller.setColumnDelay(0.75f);
controller.setRowDelay(0.5f);
controller.setDirection(GridLayoutAnimationController.DIRECTION_BOTTOM_TO_TOP|GridLayoutAnimationController.DIRECTION_LEFT_TO_RIGHT);
controller.setDirectionPriority(GridLayoutAnimationController.PRIORITY_NONE);
grid.setLayoutAnimation(controller);
grid.startLayoutAnimation();
```

## android:animateLayoutChanges

设置`android;animateLayoutChanges="true"`，就可以让 ViewGroup 在添加删除 View 的时候加上动画，所有 ViewGroup 都可以设置，大部分 ViewGroup 都有默认动画

### 自定义 LayoutTransition

```java
mTransitioner = new LayoutTransition();
//入场动画:view在这个容器中消失时触发的动画
ObjectAnimator animIn = ObjectAnimator.ofFloat(null, "rotationY", 0f, 360f,0f);
mTransitioner.setAnimator(LayoutTransition.APPEARING, animIn);
//出场动画:view显示时的动画
ObjectAnimator animOut = ObjectAnimator.ofFloat(null, "rotation", 0f, 90f, 0f);
mTransitioner.setAnimator(LayoutTransition.DISAPPEARING, animOut);
layoutTransitionGroup.setLayoutTransition(mTransitioner);
```

LayoutTransition

public void setAnimator(int transitionType, Animator animator)

transitionType:应用动画的对象范围

- APPEARING 元素在容器中出现时的动画
- DISAPPEARING 元素在容器中消失时的动画
- CHANGE_APPEARING 元素的出现导致需要变化的元素的动画
- CHANGE_DISAPPEARING

ViewGroup

public void setLayoutTransition(LayoutTransition transition)

#### `CHANGE_APPEARING` `CHANGE_DISAPPEARING`

```java
PropertyValuesHolder pvhLeft = PropertyValuesHolder.ofInt("left",0,100,0);
PropertyValuesHolder pvhTop = PropertyValuesHolder.ofInt("top",1,1);
Animator changeAppearAnimator = ObjectAnimator.ofPropertyValuesHolder(layoutTransitionGroup, pvhLeft,pvhBottom,pvhTop,pvhRight);
mTransitioner.setAnimator(LayoutTransition.CHANGE_APPEARING,changeAppearAnimator);
layoutTransitionGroup.setLayoutTransition(mTransitioner);
```

对于这两种情况，必须使用 PropertyValuesHolder 构造动画

left、top 属性的变动是必写的，如果不需要变动，可以设置为 0，0

构造 PropertyValuesHolder 时，所使用的可变长参数，第一个和最后一个必须相同，如果所有值相同，也不会有动画效果

#### 其他方法

`mTransition.setStagger(LayoutTransition.CHANGE_APPEARING, 30)`

设置每个 item 的时间间隔，否则就是一起进行动画