# Prototype 模式

## 根据实例原型、实例模型来生成新的实例

需求：在不指定类名的情况下生成实例

- 对象种类多，如果分别作为一个类，需要编写很多个类文件
- 难以根据类生成实例时，例如根据用户输入的数据生成
- 解耦框架与生成的实例

```java
public interface Product extends Cloneable {
    public abstract void use(String s);
    public abstract Product createClone();
}

public class Manager {
    private HashMap showcase = new HashMap();
    public void register(String name, Product proto) {
        showcase.put(name, proto);
    }
    public Product create(String protoname) {
        Product p = (Product)showcase.get(protoname);
        return p.createClone();
    }
}

public class MessageBox implements Product {
    private char decochar;
    public MessageBox(char decochar) {
        this.decochar = decochar;
    }
    public void use(String s) {
        //具体实现逻辑，例子中是打印字符框
    }
    public Product createClone() {
        Product p = null;
        try{
            p = (Product)clone;
        } catch (CloneNotSupportException e) {
            e.printStackTrace();
        }
        return p;
    }
}

public class UnderlinePen implements Product {
    private char ulchar;
    public UnderlinePen(char ulchar) {
        this.ulchar = ulchar;
    }
    public void use(String s) {
        //具体实现逻辑，例子中是打印下划线
    }
    public Product createClone() {
        Product p = null;
        try{
            p = (Product)clone;
        } catch (CloneNotSupportException e) {
            e.printStackTrace();
        }
        return p;
    }
}

public class Main {
    public static void main(String[] args) {
        Manager manager = new Manager();
        UnderlindPen upen = new UnderlinePen('~');
        MessageBox mbox = new MessageBox('*');
        MessageBox mbox = new MessageBox('/');
        manager.register("strong message", upen);
        manager.register("warning box", mbox);
        manager.register("slash box", sbox);

        Product p1 = manager.create("strong message");
        p1.use("Hello, world");
        Product p2 = manager.create("warning box");
        p2.use("Hello, world");
        Product p3 = manager.create("slash box");
        p3.use("Hello, world");
    }
}
```

## 角色

- Prototype

原型角色，负责定义用于复制现有实例来生成新实例的方法。

- ConcretePrototype

具体的原型，负责实现复制现有实例并生成新实例的方法。

- Client

使用复制实例的方法生成新的实例

## 要点

- 不根据类来生成实例（不掉用构造方法）
  - 对象种类繁多，无法将他们整合到一个类中时
  - 难以根据类生成实例
  - 想解耦框架与生成的实例
- 一旦在代码中出现要使用的类的名字，就无法与该类分离开来，也就无法实现复用

## 相关

- Flyweight 模式

在不同的地方使用同一个实例，而本模式是生成一个和当前实例状态完全相同的实例

- Memento 模式

保存当前实例的状态，已实现快照和撤销功能，而本模式是生成一个和当前实例状态完全相同的实例

- Composite 模式

Decorator 模式，经常使用这两种模式时，需要能够动态创建复杂结构的实例，可以使用 Prototype 模式，以帮助我们方便地生成实例

- Command 模式

想要复制 Command 模式中出现的命令时，可以使用本模式

## clone

- clone 方法定义在 java.lang.Object 中
- Cloneable 接口没有声明任何方法，只是用来标记可以使用 clone 方法进行复制。这样的接口被称为`标记接口`
- clone 方法进行的是浅复制。只是将被复制的实例字段值直接复制到新的实例中，对于引用来说，复制的也是引用。对于字段到字段的复制(field-to-field-copy)被称为浅复制(shallow copy)
- 当浅复制无法满足需求时，可以重写 clone 方法，来实现自己需要的复制，需要调用 super.clone()
- 不会调用构造方法