# 责任链模式

将多个对象组成一条职责链，按照他们在职责链上的顺序一个一个的找出由谁来处理

可以弱化请求方和处理放的关系，让双方都可以独立成为组件，还可以应对其他需求，如不同情况负责处理的对象变化的需求

```java
public class Trouble {
    private int number;             // 问题编号
    public Trouble(int number) {    // 生成问题
        this.number = number;
    }
    public int getNumber() {        // 获取问题编号
        return number;
    }
    public String toString() {      // 代表问题的字符串
        return "[Trouble " + number + "]";
    }
}

public abstract class Support {
    private String name;                    // 解决问题的实例的名字
    private Support next;                   // 要推卸给的对象
    public Support(String name) {           // 生成解决问题的实例
        this.name = name;
    }
    public Support setNext(Support next) {  // 设置要推卸给的对象
        this.next = next;
        return next;
    }
    public void support(Trouble trouble) {  // 解决问题的步骤
        if (resolve(trouble)) {
            done(trouble);
        } else if (next != null) {
            next.support(trouble);
        } else {
            fail(trouble);
        }
    }
    public String toString() {              // 显示字符串
        return "[" + name + "]";
    }
    protected abstract boolean resolve(Trouble trouble); // 解决问题的方法
    protected void done(Trouble trouble) {  // 解决
        System.out.println(trouble + " is resolved by " + this + ".");
    }
    protected void fail(Trouble trouble) {  // 未解决
        System.out.println(trouble + " cannot be resolved.");
    }
}

public class NoSupport extends Support {
    public NoSupport(String name) {
        super(name);
    }
    protected boolean resolve(Trouble trouble) {     // 解决问题的方法
        return false; // 自己什么也不处理
    }
}

public class LimitSupport extends Support {
    private int limit;                              // 可以解决编号小于limit的问题
    public LimitSupport(String name, int limit) {   // 构造函数
        super(name);
        this.limit = limit;
    }
    protected boolean resolve(Trouble trouble) {    // 解决问题的方法
        if (trouble.getNumber() < limit) {
            return true;
        } else {
            return false;
        }
    }
}

public class OddSupport extends Support {
    public OddSupport(String name) {                // 构造函数
        super(name);
    }
    protected boolean resolve(Trouble trouble) {    // 解决问题的方法
        if (trouble.getNumber() % 2 == 1) {
            return true;
        } else {
            return false;
        }
    }
}

public class SpecialSupport extends Support {
    private int number;                                 // 只能解决指定编号的问题
    public SpecialSupport(String name, int number) {    // 构造函数
        super(name);
        this.number = number;
    }
    protected boolean resolve(Trouble trouble) {        // 解决问题的方法
        if (trouble.getNumber() == number) {
            return true;
        } else {
            return false;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Support alice   = new NoSupport("Alice");
        Support bob     = new LimitSupport("Bob", 100);
        Support charlie = new SpecialSupport("Charlie", 429);
        Support diana   = new LimitSupport("Diana", 200);
        Support elmo    = new OddSupport("Elmo");
        Support fred    = new LimitSupport("Fred", 300);
        // 形成职责链
        alice.setNext(bob).setNext(charlie).setNext(diana).setNext(elmo).setNext(fred);
        // 制造各种问题
        for (int i = 0; i < 500; i += 33) {
            alice.support(new Trouble(i));
        }
    }
}

```

## 角色

- Handler

定义了处理请求的接口。Handler 知道下一个处理者的是谁，如果自己无法处理，会交给下一个

- ConcreteHandler

- Client

向第一个 ConcreteHandler 发送请求的角色

## 要点

- 弱化了发出请求的人和处理请求的人之间的关系

- 可以动态改变职责链

- 专注于自己的工作，无法处理传递出去

如果不使用该模式，那么就需要编写一个“决定谁来处理”的方法，或者让每个 ConcreteHandler 自己负责任务分配的工作

- 会有延迟

如果请求和处理者之间的关系时确定的，而且需要非常快的处理速度时，不适用本模式会更好

## 相关

- Composite 模式

Handler 角色经常会使用 Composite 模式

- Command 模式

有时会使用 Command 模式向 Handler 角色发出请求
