# Visitor 模式

## 访问数据结构并处理数据

数据结构与处理分离

主要角色

```java
public abstract class Visitor {
    public abstract void visit(File file);
    public abstract void visit(Directory directory);
}

public interface Element {
    public abstract void accept(Visitor v);
}

public abstract class Entry implements Element {
    public abstract String getName();                                   // 获取名字
    public abstract int getSize();                                      // 获取大小
    public Entry add(Entry entry) throws FileTreatmentException {       // 增加目录条目
        throw new FileTreatmentException();
    }
    public Iterator iterator() throws FileTreatmentException {    // 生成Iterator
        throw new FileTreatmentException();
    }
    public String toString() {                                          // 显示字符串
        return getName() + " (" + getSize() + ")";
    }
}
```

```java
public class File extends Entry {
    private String name;
    private int size;
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    public String getName() {
        return name;
    }
    public int getSize() {
        return size;
    }
    public void accept(Visitor v) {
        v.visit(this);
    }
}

public class Directory extends Entry {
    private String name;                    // 文件夹名字
    private ArrayList dir = new ArrayList();      // 目录条目集合
    public Directory(String name) {         // 构造函数
        this.name = name;
    }
    public String getName() {               // 获取名字
        return name;
    }
    public int getSize() {                  // 获取大小
        int size = 0;
        Iterator it = dir.iterator();
        while (it.hasNext()) {
            Entry entry = (Entry)it.next();
            size += entry.getSize();
        }
        return size;
    }
    public Entry add(Entry entry) {         // 增加目录条目
        dir.add(entry);
        return this;
    }
    public Iterator iterator() {      // 生成Iterator
        return dir.iterator();
    }
    public void accept(Visitor v) {         // 接受访问者的访问
        v.visit(this);
    }
}

public class FileTreatmentException extends RuntimeException {
    public FileTreatmentException() {
    }
    public FileTreatmentException(String msg) {
        super(msg);
    }
}
```

visitor 实现

```java
public class ListVisitor extends Visitor {
    private String currentdir = "";                         // 当前访问的文件夹的名字
    public void visit(File file) {                  // 在访问文件时被调用
        System.out.println(currentdir + "/" + file);
    }
    public void visit(Directory directory) {   // 在访问文件夹时被调用
        System.out.println(currentdir + "/" + directory);
        String savedir = currentdir;
        currentdir = currentdir + "/" + directory.getName();
        Iterator it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = (Entry)it.next();
            entry.accept(this);
        }
        currentdir = savedir;
    }
}
```

## 角色

- Visitor

负责对数据结构中每个具体的元素（ConcreteElement）声明一个用于访问 xxx 的 visit(xxx) 方法。

- ConcreteVisitor

负责实现 Visitor 角色所定义的接口。需要实现 visit() 方法，即实现如何处理每个 ConcreteElement 角色。

- Element

表示 Visitor 角色访问的对象。声明了接受访问者的 accept 方法，accept 方法接收的参数是 Visitor 角色

- ConcreteElement

负责实现 Element 角色所定义的接口

- ObjectStructure

负责处理 Element 角色的集合，Directory 类扮演此角色，为了让 ConcreteVisitor 能够遍历处理每个角色，Directory 类中实现了 iterator 方法

## 要点

- 双重分发

accept 方法的调用为：element.accept(visitor);

visit 方法调用为：visitor.visit(element);

element 接收 visitor，visitor 又访问 element

在 Visitor 模式始终，ConcreteElement 和 ConcreteVisitor 这两个角色共同决定了实际进行的处理，这种消息分发的方式一般被称为“双重分发”

- 为什么这么复杂

Visitor 模式目的是，将处理从数据结构中分离出来。我们可以编写其他 ConcreteVisitor 进行数据处理，ConcreteVisitor 的开发可以独立于 ConcreteElement 类。Visitor 模式提高了 ConcreteElement 作为组件的独立性

如果将处理的方法定义在 ConcreteElement 中，那么当每次要扩展功能时，增加新的处理时，就不得不去修改 ConcreteElement 类。

- 开闭原则，对扩展开放，对修改关闭

- 易于增加 ConcreteVisitor 角色

- 难于增加 ConcreteElement 角色

如果增加一个新的 ConcreteElement 角色，那么就需要在 Visitor 类中声明一个新的 visit(Device d) 的方法，并在所有的子类中都去实现这个方法

- Visitor 工作所需要的条件

对数据结构中的元素进行处理的任务被分离出来，交给 Visitor 类负责，但是有个条件就是 Element 角色必须要向 Visitor 角色公开足够多的信息，如果公开了不应公开的信息，将来对数据结构的改良就会变得非常困难

## 相关

- Iterator 模式

两种模式都是在某种数据结构上处理

Iterator 模式用于逐个遍历保存在数据结构中的元素

Visitor 模式用于对保存在数据结构中的元素进行某种特定的处理

- Composite 模式

有时访问者所访问的数据结构会使用 Composite 模式

- Interpreter 模式

有时会使用 Visitor 模式。