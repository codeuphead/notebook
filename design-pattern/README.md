# 设计模式 Design Pattern

参考

- 在线电子书 Refactoring[https://refactoringguru.cn/](https://refactoringguru.cn/)
- 《图解设计模式》

## UML 一览

### 创建型模式 creational

#### Chain Of Responsibility Pattern

```mermaid
classDiagram
    class Handler{
        Handler next
        request()
    }
    class ConcreteHandler1{
        request()
    }
    class ConcreteHandler2{
        request()
    }
    class Client{

    }
    Client --> Handler: use
    Handler o-- Handler
    ConcreteHandler1 ..|> Handler
    ConcreteHandler2 ..|> Handler
```

#### Command Pattern

#### Iterator Pattern

#### Mediator Pattern

#### Memento Pattern

#### Observer Pattern

#### State Pattern

#### Strategy Pattern

#### Template Method Pattern

#### Visitor Pattern

#### Interpreter Pattern

### 行为型模式 behavioral

#### Adapter Pattern

- 类适配器模式

```mermaid
classDiagram
    class Client
    class Target{
        <<interface>>
        targetMethod()
    }
    class Adapter{
        targetMethod()
    }
    class Adapee{
        method()
    }

    Client --> Target : use
    Adapter ..|> Target
    Adapter --|> Adapee
```

- 对象适配器模式

```mermaid
classDiagram
    class Client
    class Target{
        targetMethod()
    }
    class Adapter{
        Adapee adapee
        targetMethod()
    }
    class Adapee{
        method()
    }

    Client --> Target : use
    Adapter --|> Target
    Adapter o-- Adapee
```

#### Bridge Pattern

```mermaid
classDiagram
    class Abstraction{
        <<interface>>
        impl
        method1()
        method2()
    }
    class RefinedAbstraction{
        refinedMethodA()
        refinedMethodB()
    }
    class Implementor{
        <<interface>>
        implMethodX()
        implMethody()
    }
    class ConcreteImplementor{
        implMethodX();
        implMethodY();
    }

    Abstraction o-- Implementor
    Abstraction <|-- RefinedAbstraction
    Implementor <|-- ConcreteImplementor
```

#### Composite Pattern

```mermaid
classDiagram
    class Client{

    }
    class Component{
        <<interface>>
        add(Component child)
        remove(Component child)
        getChild() List~Component~
        method1()
        method2()
    }
    class Leaf{
        method1();
        method2();
    }
    class Composite{
        Component children
        add(Component child)
        remove(Component child)
        getChild() List~Component~
        method1()
        method2()
    }
    Client --> Component : use
    Component <|.. Leaf
    Component <|.. Composite
    Component --o Composite
```

#### Decorator Pattern

```mermaid
classDiagram
    class Component{
        <<interface>>
        method()
    }
    class ConcreteComponent{
        method()
    }
    class Decorator{
        <<interface>>
        Component component
    }
    class ConcreteDecorator{
        method()
    }
    Component <|.. ConcreteComponent
    Component <|.. Decorator
    Decorator <|.. ConcreteDecorator
    Decorator o-- Component
```

#### Facade Pattern

```mermaid
classDiagram
    class Client{

    }
    class Facade{

    }
    class ClassA{

    }
    class ClassB{
        
    }
    class ClassC{

    }
    class ClassD{

    }
    Client --> Facade : use
    Facade --> ClassA
    Facade --> ClassB
    Facade --> ClassC
    Facade --> ClassD
    ClassB --> ClassC
    ClassB <-- ClassC
```

#### Flyweight Pattern

```mermaid
classDiagram
    class Flyweight{
        method()
    }
    class FlyweightFactory{
        pool
        getFlyweight() Flyweight
    }
    class Client{

    }

    Client --> FlyweightFactory : use
    FlyweightFactory o-- Flyweight
```

#### Proxy Pattern

```mermaid
classDiagram
    class Subject{
        <<interface>>
        method()
    }
    class RealSubject{
        method()
    }
    class Proxy{
        method()
    }
    class Client{

    }
    Subject <|.. Proxy
    Subject <|.. RealSubject
    Client --> Subject : use
    Proxy o-- RealSubject    
```

### 创建型模式 structural

#### Factory Method Pattern

#### Abstract Factory Pattern

#### Builder Pattern

#### Prototype Pattern

#### Singleton Pattern
