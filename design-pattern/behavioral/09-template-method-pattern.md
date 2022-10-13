# Template Method 模式

## 模板

组成模板的方法被定义在父类中，不同的子类实现不同的处理，处理流程依照父类中定义的那样进行

```java
public abstract class AbstractDisplay{
    public abstract void open();
    public abstract void print();
    public abstract void close();
    public final void display(){
        open();
        for(int i = 0; i < 5; i++){
            print();
        }
        close();
    }
}

public class CharDisplay extends AbstractDisplay{
    private char ch;
    public CharDisplay(char ch){
        this.ch = ch;
    }
    public void open(){
        System.out.print("<<");
    }
    public void print(){
        System.out.print(ch);
    }
    public void close(){
        System.out.print(">>");
    }
}

public class StringDisplay extends AbstractDisplay{
    private String string;
    private int width;
    public StringDisplay(String string){
        this.string = string;
        this.width = string.getBytes().length;
    }
    public void open(){
        printLine();
    }
    public void print(){
        System.out.println("|" + string + "|");
    }
    public void close(){
        printLine();
    }
    private void printLine(){
        System.out.print("+");
        for(int i = 0; i < width; i++){
            System.out.print("-");
        }
        System.out.println("+");
    }
}

public class Main{
    public static void main(String[] args){
        AbstractDisplay d1 = new CharDisplay('H');
        AbstractDisplay d2 = new StringDisplay("Hello, world");
        AbstractDisplay d3 = new StringDisplay('adfadfzc');
        d1.display();
        d2.display();
        d3.display();
    }
}
```

## 角色

- AbstractClass

不仅负责实现模板方法，还负责声明模板方法中使用的抽象方法，这些抽象方法由子类负责实现

- ConcreteClass

实现抽象类中定义的抽象方法，

## 要点

- 使逻辑处理通用化
- 父类、子类紧密联系共同工作，在看不到父类源代码的情况下，想要编写子类是非常困难的
- LSP，里氏替换，进行编程，是通用的继承原则

## 相关

- Factory Method 模式

将 Template Method 模式用于生成实力的典型例子

- Strategy 模式

Template Method 模式中，使用继承改变程序行为，父类决定处理流程。Strategy 模式，使用委托改变程序行为，Strategy 模式用于替换整个算法。

- 父类实现更多的方法，降低了子类的灵活性

父类实现的方法过少，子类就会臃肿，各个子类可能重复代码。

- 处理流程定义在父类中，具体实现交给子类