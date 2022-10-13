# State 模式

## 用类表示状态

```java
public interface State {
    public abstract void doClock(Context context, int hour);    // 设置时间
    public abstract void doUse(Context context);                // 使用金库
    public abstract void doAlarm(Context context);              // 按下警铃
    public abstract void doPhone(Context context);              // 正常通话
}

public class DayState implements State {
    private static DayState singleton = new DayState();
    private DayState() {                                // 构造函数的可见性是private
    }
    public static State getInstance() {                 // 获取唯一实例
        return singleton;
    }
    public void doClock(Context context, int hour) {    // 设置时间
        if (hour < 9 || 17 <= hour) {
            context.changeState(NightState.getInstance());
        }
    }
    public void doUse(Context context) {                // 使用金库
        context.recordLog("使用金库(白天)");
    }
    public void doAlarm(Context context) {              // 按下警铃
        context.callSecurityCenter("按下警铃(白天)");
    }
    public void doPhone(Context context) {              // 正常通话
        context.callSecurityCenter("正常通话(白天)");
    }
    public String toString() {                          // 显示表示类的文字
        return "[白天]";
    }
}

public class NightState implements State {
    private static NightState singleton = new NightState();
    private NightState() {                              // 构造函数的可见性是private
    }
    public static State getInstance() {                 // 获取唯一实例
        return singleton;
    }
    public void doClock(Context context, int hour) {    // 设置时间
        if (9 <= hour && hour < 17) {
            context.changeState(DayState.getInstance());
        }
    }
    public void doUse(Context context) {                // 使用金库
        context.callSecurityCenter("紧急：晚上使用金库！");
    }
    public void doAlarm(Context context) {              // 按下警铃
        context.callSecurityCenter("按下警铃(晚上)");
    }
    public void doPhone(Context context) {              // 正常通话
        context.recordLog("晚上的通话录音");
    }
    public String toString() {                          // 显示表示类的文字
        return "[晚上]";
    }
}

public interface Context {

    public abstract void setClock(int hour);                // 设置时间
    public abstract void changeState(State state);          // 改变状态
    public abstract void callSecurityCenter(String msg);    // 联系警报中心
    public abstract void recordLog(String msg);             // 在警报中心留下记录
}
```

## 角色

- State

表示状态，定义了根据不同状态进行不同处理的接口。该接口是那些处理内容依赖于状态的方法的集合

- ConcreteState

表示各个具体的状态

- Context

角色持有当前状态的 ConcreteState 角色。还定义了共外部调用者使用的 State 模式接口

## 要点

- 分而治之

非常适用大规模的复杂处理。将一个复杂的大问题分解为多个小问题，然后逐个解决。State 模式用，适用类来表示状态，问题就被分解了

- 依赖于状态的处理

在 State 接口中声明的所有方法都是“依赖于状态的处理”，状态不同，处理也不同

定义接口，声明抽象方法

定义多个类，实现具体方法

- 谁来管理状态迁移

示例中，changeState() 方法是 Context 实现的，但实际调用的是 ConcreteState 角色。

优点是将状态迁移的信息集中在一个类中，比如从 DayState 迁移到其他状态时，只要阅读 DayState 的代码即可；缺点是每个 ConcreteState 都需要知道其他 ConcreteState 角色

也可以将所有迁移交给 Context 角色负责，可以提高 ConcreteState 的独立性，Context 就必须知道所有 ConcreteState 角色，可以适用 Mediator 模式

- 不会自相矛盾

如果不使用 State 模式，则需要使用多个变量来表示状态，这是必须小心，不让他们之间相互矛盾。State 模式中，是用类来表示状态的，这样我们只需要一个表示系统状态的变量即可

- 易于增加新的状态

修改状态迁移部分的代码，需要小心，因为是与其他 ConcreteState 角色关联的部分

增加其他以来状态的处理是很困难的，因为需要在 State 接口中增加新的方法，而所有实现类都要去实现

## 相关

- Singleton 模式

Singleton 模式会经常出现在 ConcreteState 角色中

- Flyweight 模式

在表示状态的类中，没有定义任何实例字段，有时我们可以使用 Flyweight 模式在多个 Context 角色之间共享 ConcreteState 角色