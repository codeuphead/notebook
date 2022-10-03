# Observer 模式

发送状态变化通知

```java
public interface Observer {
    public abstract void update(NumberGenerator generator);
}

public abstract class NumberGenerator {
    private ArrayList observers = new ArrayList();        // 保存Observer们
    public void addObserver(Observer observer) {    // 注册Observer
        observers.add(observer);
    }
    public void deleteObserver(Observer observer) { // 删除Observer
        observers.remove(observer);
    }
    public void notifyObservers() {               // 向Observer发送通知
        Iterator it = observers.iterator();
        while (it.hasNext()) {
            Observer o = (Observer)it.next();
            o.update(this);
        }
    }
    public abstract int getNumber();                // 获取数值
    public abstract void execute();                 // 生成数值
}

public class RandomNumberGenerator extends NumberGenerator {
    private Random random = new Random();   // 随机数生成器
    private int number;                     // 当前数值
    public int getNumber() {                // 获取当前数值
        return number;
    }
    public void execute() {
        for (int i = 0; i < 20; i++) {
            number = random.nextInt(50);
            notifyObservers();
        }
    }
}

public class DigitObserver implements Observer {
    public void update(NumberGenerator generator) {
        System.out.println("DigitObserver:" + generator.getNumber());
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
    }
}

public class GraphObserver implements Observer {
    public void update(NumberGenerator generator) {
        System.out.print("GraphObserver:");
        int count = generator.getNumber();
        for (int i = 0; i < count; i++) {
            System.out.print("*");
        }
        System.out.println("");
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
    }
}
```

## 角色

- Subject

表示观察对象，定义了注册观察者和删除观察者的方法，还声明了“获取现在状态”的方法

- ConcreteSubject

表示具体被观察的对象，当自身状态发生变化后，会通知所有已经注册的 Observer 角色

- Observer

负责接收来自 Subject 角色的状态变化的通知。声明了 update 方法

- ConcreteObserver

表示具体的 Observer，当它的 update 方法被调用后，会获取被观察者的最新状态

## 要点

- 这里也出现了可替换性

ConcreteSubject 无需知道正在观察自己的类具体实现。ConcreteObserver 无需知道被观察对象的具体实现

利用抽象类和接口从具体类中抽出抽象方法

在实例中作为参数传递至类中，或者在类的字段中保存实例时，不适用具体类型，而是使用抽象类型和接口

- Observer 的顺序

需要注意不能因为调用顺序改变而产生问题

- 当 Observer 的行为会对 Subject 产生影响时

容易造成循环调用

- 传递更新信息的方式

- 从观察变为通知

Observer 角色是被动地接收 Subject 的通知。因此也被称为 Publish-Subscribe 模式

- Model/View/Controller mvc

## 相关

- Mediator 模式

有时会使用 Observer 模式来实现 Mediator 与 Colleague 之间的通信

Mediator 模式中的通知，是为了对 Colleague 进行仲裁

Observer 模式中的通知，是为了使 Subject 和 Observer 同步