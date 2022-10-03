# Memento 模式

## 保存对象状态

保存和恢复实例状态时，有效地防止对象的封装性遭到破坏

```java
public class Memento {
    int money;                              // 所持金钱
    ArrayList fruits;                       // 当前获得的水果
    public int getMoney() {                 // 获取当前所持金钱（narrow interface）
        return money;
    }
    Memento(int money) {                    // 构造函数(wide interface)
        this.money = money;
        this.fruits = new ArrayList();
    }
    void addFruit(String fruit) {           // 添加水果(wide interface)
        fruits.add(fruit);
    }
    List getFruits() {                      // 获取当前所持所有水果（wide interface）
         return (List)fruits.clone();
    }
}

public class Gamer {
    private int money;                          // 所持金钱
    private List fruits = new ArrayList();      // 获得的水果
    private Random random = new Random();       // 随机数生成器
    private static String[] fruitsname = {      // 表示水果种类的数组
        "苹果", "葡萄", "香蕉", "橘子",
    };
    public Gamer(int money) {                   // 构造函数
        this.money = money;
    }
    public int getMoney() {                     // 获取当前所持金钱
        return money;
    }
    public void bet() {                         // 投掷骰子进行游戏
        int dice = random.nextInt(6) + 1;           // 掷骰子
        if (dice == 1) {                            // 骰子结果为1…增加所持金钱
            money += 100;
            System.out.println("所持金钱增加了。");
        } else if (dice == 2) {                     // 骰子结果为2…所持金钱减半
            money /= 2;
            System.out.println("所持金钱减半了。");
        } else if (dice == 6) {                     // 骰子结果为6…获得水果
            String f = getFruit();
            System.out.println("获得了水果(" + f + ")。");
            fruits.add(f);
        } else {                                    // 骰子结果为3、4、5则什么都不会发生
            System.out.println("什么都没有发生。");
        }
    }
    public Memento createMemento() {                // 拍摄快照
        Memento m = new Memento(money);
        Iterator it = fruits.iterator();
        while (it.hasNext()) {
            String f = (String)it.next();
            if (f.startsWith("好吃的")) {         // 只保存好吃的水果
                m.addFruit(f);
            }
        }
        return m;
    }
    public void restoreMemento(Memento memento) {   // 撤销
        this.money = memento.money;
        this.fruits = memento.getFruits();
    }
    public String toString() {                      // 用字符串表示主人公状态
        return "[money = " + money + ", fruits = " + fruits + "]";
    }
    private String getFruit() {                     // 获得一个水果
        String prefix = "";
        if (random.nextBoolean()) {
            prefix = "好吃的";
        }
        return prefix + fruitsname[random.nextInt(fruitsname.length)];
    }
}

public class Main {
    public static void main(String[] args) {
        Gamer gamer = new Gamer(100);               // 最初的所持金钱数为100
        Memento memento = gamer.createMemento();    // 保存最初的状态
        for (int i = 0; i < 100; i++) {
            System.out.println("==== " + i);        // 显示掷骰子的次数
            System.out.println("当前状态:" + gamer);    // 显示主人公现在的状态

            gamer.bet();    // 进行游戏

            System.out.println("所持金钱为" + gamer.getMoney() + "元。");

            // 决定如何处理Memento
            if (gamer.getMoney() > memento.getMoney()) {
                System.out.println("    （所持金钱增加了许多，因此保存游戏当前的状态）");
                memento = gamer.createMemento();
            } else if (gamer.getMoney() < memento.getMoney() / 2) {
                System.out.println("    （所持金钱减少了许多，因此将游戏恢复至以前的状态）");
                gamer.restoreMemento(memento);
            }

            // 等待一段时间
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }
            System.out.println("");
        }
    }
}
```

## 角色

- Originator

角色会在保存自己的最新状态时生成 Memento 角色。当把以前保存的 Memento 角色传递给 Originator 角色时，他会将自己恢复至生成该 Memento 角色时的状态

- Memento

将 Originator 角色的内部信息整合在一起，虽然保存了 Originator 觉得的信息，但不会公开

- Caretaker

当 Caretaker 想要保存当前的 Originator 状态时，会通知 Originator，其收到通知后会生成 Memento 角色并返回给 Caretaker。之后可能会用 Memento 将 Originator 恢复至原来的状态。它只是将 Originator 角色生成的 Memento 角色当作一个黑盒子保存起来

Originator 和 Memento 之间是强关联关系，Caretaker 和 Memento 是弱关联关系，Memento 对 Caretaker 隐藏了自身的内部信息

## 要点

- 对 Caretaker 提供窄接口，半透明

- 需要多少个 Memento

- Memento 的有效期

如果需要保存到文件，那么需要考虑有效期、版本升级等问题

- Caretaker 和 Originator 角色

Caretaker 角色的职责是决定何时进行保存与恢复 Memento 角色

Originator 就是负责生成 Memento 和接收到 Memento 角色来恢复自己的状态

有了这样的职责分担，就可以不用修改 Originator 角色进行需求变更

## 相关

- Command 模式

使用该模式处理命令时，可以使用 Memento 实现撤销功能

- Prototype 模式

该模式生成一个当前实例完全相同的另外一个实例。本模式保存了当前对象的状态

- State 模式

Memento 用的是实例的装填，State 用的是类的状态