# Proxy 模式

## 只在必要时生成实例

```java
public class Printer implements Printable {
    private String name;
    public Printer() {
        heavyJob("正在生成Printer的实例");
    }
    public Printer(String name) {                   // 构造函数
        this.name = name;
        heavyJob("正在生成Printer的实例(" + name + ")");
    }
    public void setPrinterName(String name) {       // 设置名字
        this.name = name;
    }
    public String getPrinterName() {                // 获取名字
        return name;
    }
    public void print(String string) {              // 显示带打印机名字的文字
        System.out.println("=== " + name + " ===");
        System.out.println(string);
    }
    private void heavyJob(String msg) {             // 重活
        System.out.print(msg);
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }
            System.out.print(".");
        }
        System.out.println("结束。");
    }
}

public interface Printable {
    public abstract void setPrinterName(String name);   // 设置名字
    public abstract String getPrinterName();            // 获取名字
    public abstract void print(String string);          // 显示文字（打印输出）
}

public class PrinterProxy implements Printable {
    private String name;            // 名字
    private Printer real;           // “本人”
    public PrinterProxy() {
    }
    public PrinterProxy(String name) {      // 构造函数
        this.name = name;
    }
    public synchronized void setPrinterName(String name) {  // 设置名字
        if (real != null) {
            real.setPrinterName(name);  // 同时设置“本人”的名字
        }
        this.name = name;
    }
    public String getPrinterName() {    // 获取名字
        return name;
    }
    public void print(String string) {  // 显示
        realize();
        real.print(string);
    }
    private synchronized void realize() {   // 生成“本人”
        if (real == null) {
            real = new Printer(name);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Printable p = new PrinterProxy("Alice");
        System.out.println("现在的名字是" + p.getPrinterName() + "。");
        p.setPrinterName("Bob");
        System.out.println("现在的名字是" + p.getPrinterName() + "。");
        p.print("Hello, world.");
    }
}
```

## 角色

- Subject

角色定义了使 Proxy 角色 和 RealSubject 角色之间具有一致性的接口，因此 Client 不必关心究竟使用的使 Proxy 还是 RealSubject

- Proxy

角色会尽量处理来自 Client 角色的请求，只有当自己不能处理是，才会将工作交给 RealSubject 角色。只有在必要时才会生成 RealSubject 角色。Proxy 角色实现了 Subject 定义的接口

- RealSubject

在 Proxy 角色无法胜任工作的时候出厂，实现了 Subject 定义的接口

- Client

使用 Proxy 模式的角色

## 要点

- 使用代理人来提升处理速度

- 有必要划分代理人和本人

可以独立成组件，修改使不会相互影响

- 代理与委托

代理中遇到无法处理的问题，就会通过委托的方式进行处理

- 透明性

使用接口 Printable 对于 Client

- 各种 Proxy 模式

Virtual Proxy

Remote Proxy

Access Proxy

## 相关

- Adapter 模式

Adapter 适配了两种不同接口的对象，使他们可以一同工作；Proxy 与 RealSubject 的接口使相同的

- Decorator 模式

两者实现上很相似，不过使用目的不同

Decorator 目的使增加新功能；Proxy 更注重通过代理人的方式来减轻本人的工作负担