# View Animation

## Tween Animation

### Set

- android:fillAfter ture 保持最后的动画状态
- android:fillBefore true 将控件还原到开始状态
- android:fillEnabled true 将控件还原到开始状态
- android:repeatCount
- android:repeatMode restart/reverse
- android:interpolator
- android:duration

### Scale

- android:fromXScale float
- android:toXScale float
- android:fromYScale float
- android:toYScale float
- android:pivotX 缩放锚点，50，50%，50%p，分别代表：相对自己50px，自己50%，左上角+父50%
- android:pivotY

### Alpha

- android:fromAlpha float 0.0-1.0
- android:toAlpha float 0.0-1.0

### Rotate

- android:fromDegress 正代表顺时针方向度数
- android:toDegress
- android:pivotX
- android:pivotY

### Translate

- android:fromXDelta 50，50%，50%p
- android:fromYDelta
- android:toXDelta
- android:toYDelta

### Code

AnimationUtils

`public static Animation loadAnimation(Context context, @AnimRes int id)`

View

`public void startAnimation(Animation animation)`

## Animator

### ValueAnimator

负责对指定的数字区间进行动画运算，我们需要对运算进行监听，然后操作控件

```java
public static ValueAnimator ofInt(int... values)

public static ValueAnimator ofArgb(int... values)

public static ValueAnimator ofFloat(float... values)

public ValueAnimator setDuration(long duration)

public Object getAnimatedValue()

public void setRepeatCount(int value) // ValueAnimator.INFINITE

public void setRepeatMode(@RepeatMode int value) // ValueAnimator.RESTART ValueAnimator.REVERSE

public void start()

public void cancel()

public void addUpdateListener(AnimatorUpdateListener listener)

public void addListener(AnimatorListener listener)

public void removeUpdateListener(AnimatorUpdateListener listener)

public void removeAllUpdateListeners()

public void removeListener(AnimatorListener listener)

public void removeAllListeners()
```

---

#### ofObject()

`public static ValueAnimator ofObject(TypeEvaluator evaluator, Object... values)`

对于自定义的 Object 需要有专门的 TypeEvaluator 来进行计算

---

#### hook method 类

`public abstract class AnimatorListenerAdapter implements Animator.AnimatorListener, Animator.AnimatorPauseListener`

### Interpolator

自定义加速器，实现 Interpolator 接口

Interpolator 接口实现了 TimeInterpolator，其只有一个方法`float getInterpolation(float input)`

input 取值范围是 0-1，表示当前动画的时间进度

返回值表示当前数值的进度，0-1，超过1表示超过目标值

### Evaluator

将当前数值进度，从小数转化成实际数值，其返回的数值

自定义 Evaluator，实现 TypeEvaluator 接口

### ObjectAnimator

修改某个属性，需要有 setter 方法，必须是驼峰命名的方法

`alpha、rotation、rotationX、rotationY、tanslationX、translationY、scaleX、scaleY`

通过反射确定调用的方法，因此参数的类型需要正确，如 ofFloat、ofInt 的使用

对于可变长的数值参数，如果只给了一个值，那么会调用 get 方法来获取值，如果 get 方法不存在，则使用默认值，过渡值传什么类型，get 方法就需要返回什么类型，否则反射找不到方法

## PropertyValuesHolder

```java
public static ValueAnimator ofPropertyValuesHolder(PropertyValuesHolder... values)

public static ObjectAnimator ofPropertyValuesHolder(Object target,PropertyValuesHolder... values)
```

---

PropertyValuesHolder

```java
public static PropertyValuesHolder ofFloat(String propertyName, float... values)

public static PropertyValuesHolder ofInt(String propertyName, int... values)

public static PropertyValuesHolder ofObject(String propertyName, TypeEvaluator evaluator,Object... values)

public static PropertyValuesHolder ofKeyframe(String propertyName, Keyframe... values)
```

### Keyframe

```java
public static Keyframe ofFloat(float fraction, float value) // faction 为当前动画进度，getInterpolation() 中返回的值，value 为当前该位置的值

public static Keyframe ofFloat(float fraction) // 使用此方法生成的 Keyframe 可以通过其他方法设置属性
```

Keyframe 的 fraction 和 value 必须被设置，interpolator 可以不设置，其使用的是默认线性的

```java
public void setFraction(float fraction)

public void setValue(Object value)

public void setInterpolator(TimeInterpolator interpolator) // 给当前 Keyframe 添加插值器，影响的是上一个 Keyframe 到本 Keyframe 过程中的计算
```

### 其他方法

```java
public void setEvaluator(TypeEvaluator evaluator) // 如果使用 obObject() 来创建 PropertyValuesHolder，需要显式调用

public void setFloatValues(float... values)

public void setIntValues(int... values)

public void setKeyframes(Keyframe... values)

public void setObjectValues(Object... values)

public void setPropertyName(String propertyName)
```

## AnimatorSet

`new AnimatorSet()`

### playSequentially

```java
public void playSequentially(Animator... items);

public void playSequentially(List<Animator> items);
```

前一个动画没有结束，下一个动画不会执行

### playTogether

```java
public void playTogether(Animator... items);

public void playTogether(Collection<Animator> items);
```

### 设置顺序 AnimatorSet.Builder

```java
public Builder play(Animator anim) // 生成 AnimatorSet.Builder 的方法，表示要播放哪个动画

// 都是以 play 中的动画为基准的
public Builder with(Animator anim) // 一起执行

public Builder before(Animator anim) // 执行 play 的动画后才执行该动画

public Builder after(Animator anim) // 执行先执行这个动画再执行 play 的动画

public Builder after(long delay) // 延迟n毫秒之后执行动画
```

### AnimatorListener

AnimatorSet 的监听器，只进行动画集合状态的监听，与其中单个动画无关

AnimatorSet 无法设置循环次数

### 其他

```java
public AnimatorSet setDuration(long duration); // 设置单次动画时长

public void setInterpolator(TimeInterpolator interpolator) // 设置加速器

public void setTarget(Object target) // 设置ObjectAnimator动画目标控件
```

调用后会覆盖单个动画中的设置

`public void setStartDelay(long startDelay) // 该方法不会覆盖单个动画的延时，而是整个动画集合的激活时间`

AnimatorSet 激活的真正延时是设置了 startDelay 的时间加上第一个动画的延时时间

## xml 实现 Animator

```xml
<animator> 代表 valueAnimator

<objectAnimator> 代表 ObjectAnimator

<set> 代表 AnimatorSet
```

### animator

- android:duration
- android:valueFrom
- android:valueTo
- android:startOffset 动画延时
- android:repeatCount
- android:repeatMode repeat/reverse
- android:valueType intType/floatType
- android:interpolator @android:interpolator/xxx

```java
ValueAnimator valueAnimator = (ValueAnimator) AnimatorInflater.loadAnimator(MyActivity.this, R.animator.animator);
valueAnimator.start();
```

### objectAnimator

- andorid:propertyName

```java
ObjectAnimator animator = (ObjectAnimator) AnimatorInflater.loadAnimator(MyActivity.this,
        R.animator.object_animator);
animator.setTarget(mTv1);
animator.start();
```

### set

- android:ordering together/sequentially

```java
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(MyActivity.this,
        R.animator.set_animator);
set.setTarget(mTv1);
set.start();
```
