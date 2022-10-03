# Strategy 模式

## 整体地替换算法

Hand 类被此模式使用，但并非此模式中担任角色

```java
public class Hand {
    public static final int HANDVALUE_GUU = 0;  // 表示石头的值
    public static final int HANDVALUE_CHO = 1;  // 表示剪刀的值
    public static final int HANDVALUE_PAA = 2;  // 表示布的值
    public static final Hand[] hand = {         // 表示猜拳中3种手势的实例
        new Hand(HANDVALUE_GUU),
        new Hand(HANDVALUE_CHO),
        new Hand(HANDVALUE_PAA),
    };
    private static final String[] name = {      // 表示猜拳中手势所对应的字符串
        "石头", "剪刀", "布",
    };
    private int handvalue;                      // 表示猜拳中出的手势的值
    private Hand(int handvalue) {
        this.handvalue = handvalue;
    }
    public static Hand getHand(int handvalue) { // 根据手势的值获取其对应的实例
        return hand[handvalue];
    }
    public boolean isStrongerThan(Hand h) {     // 如果this胜了h则返回true
        return fight(h) == 1;
    }
    public boolean isWeakerThan(Hand h) {       // 如果this输给了h则返回true
        return fight(h) == -1;
    }
    private int fight(Hand h) {                 // 计分：平0, 胜1, 负-1
        if (this == h) {
            return 0;
        } else if ((this.handvalue + 1) % 3 == h.handvalue) {
            return 1;
        } else {
            return -1;
        }
    }
    public String toString() {                  // 转换为手势值所对应的字符串
        return name[handvalue];
    }
}
```

```java
public interface Strategy {
    public abstract Hand nextHand();
    public abstract void study(boolean win);
}

public class WinningStrategy implements Strategy {
    private Random random;
    private boolean won = false;
    private Hand prevHand;
    public WinningStrategy(int seed) {
        random = new Random(seed);
    }
    public Hand nextHand() {
        if (!won) {
            prevHand = Hand.getHand(random.nextInt(3));
        }
        return prevHand;
    }
    public void study(boolean win) {
        won = win;
    }
}

public class ProbStrategy implements Strategy {
    private Random random;
    private int prevHandValue = 0;
    private int currentHandValue = 0;
    private int[][] history = {
        { 1, 1, 1, },
        { 1, 1, 1, },
        { 1, 1, 1, },
    };
    public ProbStrategy(int seed) {
        random = new Random(seed);
    }
    public Hand nextHand() {
        int bet = random.nextInt(getSum(currentHandValue));
        int handvalue = 0;
        if (bet < history[currentHandValue][0]) {
            handvalue = 0;
        } else if (bet < history[currentHandValue][0] + history[currentHandValue][1]) {
            handvalue = 1;
        } else {
            handvalue = 2;
        }
        prevHandValue = currentHandValue;
        currentHandValue = handvalue;
        return Hand.getHand(handvalue);
    }
    private int getSum(int hv) {
        int sum = 0;
        for (int i = 0; i < 3; i++) {
            sum += history[hv][i];
        }
        return sum;
    }
    public void study(boolean win) {
        if (win) {
            history[prevHandValue][currentHandValue]++;
        } else {
            history[prevHandValue][(currentHandValue + 1) % 3]++;
            history[prevHandValue][(currentHandValue + 2) % 3]++;
        }
    }
}

public class Player {
    private String name;
    private Strategy strategy;
    private int wincount;
    private int losecount;
    private int gamecount;
    public Player(String name, Strategy strategy) {         // 赋予姓名和策略
        this.name = name;
        this.strategy = strategy;
    }
    public Hand nextHand() {                                // 策略决定下一局要出的手势
        return strategy.nextHand();
    }
    public void win() {                 // 胜
        strategy.study(true);
        wincount++;
        gamecount++;
    }
    public void lose() {                // 负
        strategy.study(false);
        losecount++;
        gamecount++;
    }
    public void even() {                // 平
        gamecount++;
    }
    public String toString() {
        return "[" + name + ":" + gamecount + " games, " + wincount + " win, " + losecount + " lose" + "]";
    }
}

public class Main {
    public static void main(String[] args) {
        if (args.length != 2) {
            System.out.println("Usage: java Main randomseed1 randomseed2");
            System.out.println("Example: java Main 314 15");
            System.exit(0);
        }
        int seed1 = Integer.parseInt(args[0]);
        int seed2 = Integer.parseInt(args[1]);
        Player player1 = new Player("Taro", new WinningStrategy(seed1));
        Player player2 = new Player("Hana", new ProbStrategy(seed2));
        for (int i = 0; i < 10000; i++) {
            Hand nextHand1 = player1.nextHand();
            Hand nextHand2 = player2.nextHand();
            if (nextHand1.isStrongerThan(nextHand2)) {
                System.out.println("Winner:" + player1);
                player1.win();
                player2.lose();
            } else if (nextHand2.isStrongerThan(nextHand1)) {
                System.out.println("Winner:" + player2);
                player1.lose();
                player2.win();
            } else {
                System.out.println("Even...");
                player1.even();
                player2.even();
            }
        }
        System.out.println("Total result:");
        System.out.println(player1.toString());
        System.out.println(player2.toString());
    }
}
```

## 角色

- Strategy

角色负责决定实现策略所必须地接口

- ConcreteStrategy

角色负责实现 Strategy 角色地接口，即具体地策略

- Context

负责使用 Strategy 角色。Context 角色中保存了 ConcreteStrategy 角色地实例

## 要点

- 特意编写 Strategy 类

将算法与具体方法分离，以委托地方式来使用算法。委托这种弱关联关系可以方便地整体替换算法

- 程序运行中也可以切换策略

- 还可以用某种算法去“验算”另外一种算法

## 相关

- Flyweight 模式

有时会使用 Flyweight 模式让多个地方可以公用 ConcreteStrategy 角色

- Abstract Factory 模式

该模式可以整体地替换具体工厂、零件、产品；本模式可以整体地替换算法

- State 模式

这两种模式都可以替换被委托对象。

该模式中，ConcreteState 角色使表示状态的类，每次变化是，被委托对象的类必定被替换

本模式中，ConcreteStrategy 角色使表示算法的类，是否替换由程序自主控制