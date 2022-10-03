# Composite 模式

## 容器与内容的一致性

能够使容器与内容具有一致性，创造出递归结构的模式

```java
public abstract class Entry {
    public abstract String getName();                               // 获取名字
    public abstract int getSize();                                  // 获取大小
    public Entry add(Entry entry) throws FileTreatmentException {   // 加入目录条目
        throw new FileTreatmentException();
    }
    public void printList() {                                       // 为一览加上前缀并显示目录条目一览
        printList("");
    }
    protected abstract void printList(String prefix);               // 为一览加上前缀
    public String toString() {                                      // 显示代表类的文字
        return getName() + " (" + getSize() + ")";
    }
}

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
    protected void printList(String prefix) {
        System.out.println(prefix + "/" + this);
    }
}

// Directory 类的 getSize() 方法、printList() 方法体现了容器和内容具有一致性的特点
public class Directory extends Entry {
    private String name;                    // 文件夹的名字
    private ArrayList directory = new ArrayList();      // 文件夹中目录条目的集合
    public Directory(String name) {         // 构造函数
        this.name = name;
    }
    public String getName() {               // 获取名字
        return name;
    }
    public int getSize() {                  // 获取大小
        int size = 0;
        Iterator it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = (Entry)it.next();
            size += entry.getSize();
        }
        return size;
    }
    public Entry add(Entry entry) {         // 增加目录条目
        directory.add(entry);
        return this;
    }
    protected void printList(String prefix) {       // 显示目录条目一览
        System.out.println(prefix + "/" + this);
        Iterator it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = (Entry)it.next();
            entry.printList(prefix + "/" + name);
        }
    }
}

public class FileTreatmentException extends RuntimeException {
    public FileTreatmentException() {
    }
    public FileTreatmentException(String msg) {
        super(msg);
    }
}

public class Main {
    public static void main(String[] args) {
        try {
            System.out.println("Making root entries...");
            Directory rootdir = new Directory("root");
            Directory bindir = new Directory("bin");
            Directory tmpdir = new Directory("tmp");
            Directory usrdir = new Directory("usr");
            rootdir.add(bindir);
            rootdir.add(tmpdir);
            rootdir.add(usrdir);
            bindir.add(new File("vi", 10000));
            bindir.add(new File("latex", 20000));
            rootdir.printList();

            System.out.println("");
            System.out.println("Making user entries...");
            Directory yuki = new Directory("yuki");
            Directory hanako = new Directory("hanako");
            Directory tomura = new Directory("tomura");
            usrdir.add(yuki);
            usrdir.add(hanako);
            usrdir.add(tomura);
            yuki.add(new File("diary.html", 100));
            yuki.add(new File("Composite.java", 200));
            hanako.add(new File("memo.tex", 300));
            tomura.add(new File("game.doc", 400));
            tomura.add(new File("junk.mail", 500));
            rootdir.printList();
        } catch (FileTreatmentException e) {
            e.printStackTrace();
        }
    }
}
```

## 角色

- Leaf

表示“内容”角色，该角色不能放入其他对象

- Composite

表示容器的角色，可以在其中放入 Leaf 和 Composite 角色

- Component

使 Leaf 和 Composite 具有一致性的角色，使他们的父类

- Client

使用 Composite 模式的角色

## 要点

- 多个和单个的一致性

将多个对象结合在一起，当作一个对象处理

- Add 方法
  - 定义在 Entry 中，报错。需要实现的类进行重写
  - 定义在 Entry 中，什么都不做，需要实现的类进行重写
  - 声明在 Entry 中，但不实现
  - 只定义在 Directory 中，需要进行转型

- 递归结构

树结构的数据结构都适用 Composite 模式

## 相关

- Command 模式

适用 Command 模式编写宏命令时使用了 Composite 模式

- Visitor 模式

可以用 Visitor 模式访问 Composite 模式中的递归结构

- Decorator 模式

该模式使装饰框和内容具有一致性；本模式通过 Component 使容器和内容具有一致性
