# Mediator 模式

要调整多个对象之间的关系时，就需要用到 Mediator 模式，将控制显示的逻辑处理交给仲裁者负责

```java
public interface Mediator {
    public abstract void createColleagues();
    public abstract void colleagueChanged();
}

public interface Colleague {
    public abstract void setMediator(Mediator mediator);
    public abstract void setColleagueEnabled(boolean enabled);
}
```

## 角色

- Mediator

负责定义与 Colleague 角色进行通信和做出决定的接口

- ConcreteMediator

负责实现 Mediator 角色的接口，负责实际做出决定

- Colleague

负责定义与 Mediator 角色进行通信的接口

- ConcreteColleague

负责实现 Colleague 角色接口

## 要点

- 发生分散灾难

将控制集中起来，如果需求变更，集中控制的方法容易产生 bug，不过因为集中起来，容易处理。如果分散在每个 ConcreteColleague 中，无论编写代码、调试代码、修改代码都非常困难。

面向对象编程让我们分散处理，避免处理过于集中。本模式中，分散处理时不明智的，会导致灾难

- 通信线路增加

本模式是为通信线路增加做好准备

- 那些角色可以复用

ConcreteColleague 可以复用；ConcreteMediator 很难复用

## 相关

- Facade 模式

一个是单向交互，一个是双向交互

- Observer 模式

有时会使用 Observer 模式来实现 Mediator 角色与 Colleague 角色的的通信