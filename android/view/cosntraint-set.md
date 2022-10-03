# ConstraintSet

推荐使用 ConstraintSet 来动态调整 View 位置大小等。

同时可以添加动画。

## 创建

创建 ConstraintSet 对象，调用方法创建保存约束，应用于 ConstraintLayout。

```java
//手动创建
c = new Constraint();
c.connect(....);

//从一个 ConstraintLayout 资源对象中获取
c.clone(context, R.layout.layout1);

//从一个 ConstraintLayout 中获取
c.clone(clayout);
```

```java
public class MainActivity extends AppCompatActivity {
    ConstraintSet mConstraintSet1 = new ConstraintSet(); // create a Constraint Set
    ConstraintSet mConstraintSet2 = new ConstraintSet(); // create a Constraint Set
    ConstraintLayout mConstraintLayout; // cache the ConstraintLayout
    boolean mOld = true;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Context context = this;
        mConstraintSet2.clone(context, R.layout.state2); // get constraints from layout
        setContentView(R.layout.state1);
        mConstraintLayout = (ConstraintLayout) findViewById(R.id.activity_main);
        mConstraintSet1.clone(mConstraintLayout); // get constraints from ConstraintSet
    }

    public void foo(View view) {
        TransitionManager.beginDelayedTransition(mConstraintLayout);
        if (mOld = !mOld) {
            mConstraintSet1.applyTo(mConstraintLayout); // set new constraints
        }  else {
            mConstraintSet2.applyTo(mConstraintLayout); // set new constraints
        }
    }
}
```

## 可调用的公开方法：

```
applyTo();

clear() //清楚约束
connect() //创建约束 连接 view 与 目标 view

constraintWidth() constraintHeight()
createHorizontalChain() createVerticalChain()
setHorizontalBias() setVerticalBias()
setVisibility()
setMargin()
centerHorizontally()
...
```

## 动态调整 demo

```java
set.clone(rootLayout);

set.connect(mViewSwitcher.getId(), ConstraintSet.TOP, mTitleView.getId(), ConstraintSet.BOTTOM, 50)

set.centerHorizontally(mTimerView.getId(), ConstraintSet.PARENT_ID);

set.constrainHeight(mStateLine.getId(), 2);

set.applyTo(rootLayout);
```