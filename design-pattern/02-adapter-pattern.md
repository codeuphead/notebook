# Adapter 模式 / Wrapper 模式

- 类适配器模式（使用继承的适配器）
- 对象适配器模式（使用委托的适配器）

## 类适配器模式

- 继承具体实现方式，实现需求接口
- 使用接口编程，调用接口方法，对于调用者来说，Banner 类、即其方法都是透明的。

```java
//扮演实际情况
public class Banner {
    private String string;
    public Banner(String string) {
        this.string = string;
    }
    public void showWithParen() {
        System.out.println("(" + string + ")");
    }
    public void showWithAster() {
        System.out.println("*" + string + "*");
    }
}

//扮演需求接口
public interface Print{
    public abstract void printWeak();
    public abstract void printStrong();
}

//扮演适配器
public class PrintBanner extends Banner implement Print {
    public PrintBanner(String string) {
        super(string);
    }
    public void printWeak() {
        showWithParen();
    }
    public void printStrong() {
        showWithAster();
    }
}

public class Main {
    public static void main(String[] args) {
        Print p = new PringBanner("Hellow");
        p.printWeak();
        p.pringString();
    }
}

```

## 对象适配器模式，委托关系，将任务交给其他实例处理

Print 不是接口而是类

```java
public abstract class Print {
    public abstract void printWeak();
    public abstract void printStrong();
}

public class PrintBanner extends Print {
    private Banner banner;
    public PrintBanner(String string) {
        this.banner = new Banner(string);
    }
    public void printWeak() {
        banner.showWithParen();
    }
    public void printStrong() {
        banner.showWithAster();
    }
}

```

## 角色

- Target

负责定义所需方法，通常由接口或者抽象类担任

- Client

负责使用 Target 角色定义的方法进行具体处理，例子中为 Main 类

- Adaptee

被适配着，是一个持有既定方法的角色

- Adapter

使用 Adaptee 角色的方法来满足 Target 的需求。通过继承或委托来使用 Adaptee 角色

## 要点

- 对现有的类进行适配，生成新的类

现有的类 Bug 少稳定，出现问题时，只用调查 Adapter 角色即可，不用考虑 Adaptee 的问题。

- 让现有的类，适配新的 API 时，通常只需要修改一下就可以了，这样就会修改现有代码。Adapter 模式可以在不改变现有代码的情况下，适配新的 API。

- 版本升级与兼容

新版本扮演 Adaptee 角色，旧版本扮演 Target 角色，编写一个 Adapter 类，让其使用新版本的类实现旧版本类中哦你的方法。

- Adaptee 和 Target 角色的功能不同时，该模式是无法使用的

## 相关

- Bridge

Bridge 模式用于连接类的功能层次结构与实现层次结构。Adapter 用于连接 API 不同的类

- Decorator

Decorator 模式是在不改变 API 的前提下，增加功能。Adapter 用于填补不同 API 的缝隙