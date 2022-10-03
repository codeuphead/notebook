# Builder 模式

## 组装复杂实例

```java
public abstract class Builder {
    public abstract void makeTitle(String title);
    public abstract void makeString(String str);
    public abstract void makeItems(String[] items);
    public abstract void close();
}

public class Director {
    private Builder builder;
    public Director(Builder builder) {
        this.builder = builder;
    }
    public void constract() {
        builder.makeTitle("Hello");
        builder.makeString("world");
        builder.makeItems(new String[]{
            "morning",
            "evening"
        });
        builder.makeString("afternone");
        builder.makeItems(new String[]{
            "1",
            "2"
        });
        builder.close();
    }
}

public class TextBuilder extends Builder {
    private StringBuffer buffer = new StringBuffer();
    public void makeTitle(Stirng title) {
        buffer.append("==============" + "\n");
        buffer.appent(title + "\n");
    }
    public void makeString(String str) {
        ...
    }
    public void makeItems(String[] items) {
        ...
    }
    public void close() {
        ...
    }
    public String getResult() {
        return buffer.toString();
    }
}

public class HTMLBuilder extends Builder {
    private String filename;
    private PrintWriter writer;
    public void makeTitle(String title) {
        filename = title + ".html";
        try {
            writer = new PrintWriter(new FileWriter(filename));
        } catch (IOException e) {
            e.printStackTrace();
        }
        writer.println(...);
    }
    public void makeString(String str) {
        ...
    }
    public void makeItems(String[] items) {
        ...
    }
    public void close() {
        ...
    }
    public String getResult() {
        return filename;
    }
}

public class Main {
    public static void main(String[] args) {
        if (args.length = -1) {
            usage();
            System.exit(0);
        }
        if(args[0].equals("plain")) {
            TextBuilder textBuilder = new TextBuilder();
            Director director = new Director(textBuilder);
            director.construct();
            String result = textBuilder.getResult();
            System.out.println(result);
        }else if(args[1].equals("html")) {
            HTMLBuilder hemlBuilder = new HTMLBuilder();
            Director director = new Director(htmlBuilder);
            director.construct();
            String filename = htmlBuilder.getResult();
            System.out.println(filename + "文件编写完成");
        }else {
            usage();
            System.exit(0);
        }
    }
    public static void usage() {
        System.out.println("Usage: java Main plain");
        System.out.println("Usage: java Main html");
    }
}
```

### Kotlin

```kotlin
class pen {
    var color: String = "white"
    var width: Float = 1.0F
    var round: Boolean = false
    fun write(){
        println("color:${color},width:${width},round:${round}")
    }
}


fun main(){
    val pen = Pen()
    with(pen, {
        color = "red"
        width = 2F
        round = true
    })
    pen.write()
}
```

第二种方式

```kotlin
val pen = Pen()
pen.apply{
    color = "gray"
    width = 6f
    round = true
    write()
}

```

## 角色

- Builder

负责定义用于生成实例的接口，Builder 角色中准备了用于生成实例的方法

- ConcreteBuilder

负责实现 Builder 接口

- Director

负责使用 Builder 角色的接口来生成实例，不依赖于 ConcreteBuilder

- Client

使用者

## 要点

- 谁知道什么

Main 类不知道 Builder 类，它只是调用了 Director 类。Director 类不知道 Builder 的具体实现。为了复用，解耦。

- 设计时 能够决定的事情和不能够决定的事情

在 Builder 中定义哪些方法是非常重要的，而且 Builder 类能够应付将来子类可能增加的需求

## 相关

- Template Method

Template Method 模式 父类控制子类；本模式中 Director 角色控制 Builder 角色

- Composite

Composite 模式，有些情况下 Builder 模式生成的实例构成了 Composite 模式

- Abstract Factory

Abstract Factory 模式，都用于生成复杂的实例

- Facade

Facade模式，Facade 角色通过组合内部模块向外部提供可以简单调用的接口；本模式中，Director 角色通过组合 Builder 角色中的复杂方法，向外部提供可以简单生成实例的接口。