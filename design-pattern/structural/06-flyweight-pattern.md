# Flyweight 模式

## 共享对象，避免浪费

通过尽量共享实例来避免 new 出实例

```java
public class BigChar {
    // 字符名字
    private char charname;
    // 大型字符对应的字符串(由'#' '.' '\n'组成)
    private String fontdata;
    // 构造函数
    public BigChar(char charname) {
        this.charname = charname;
        try {
            BufferedReader reader = new BufferedReader(
                new FileReader("big" + charname + ".txt")
            );
            String line;
            StringBuffer buf = new StringBuffer();
            while ((line = reader.readLine()) != null) {
                buf.append(line);
                buf.append("\n");
            }
            reader.close();
            this.fontdata = buf.toString();
        } catch (IOException e) {
            this.fontdata = charname + "?";
        }
    }
    // 显示大型字符
    public void print() {
        System.out.print(fontdata);
    }
}

public class BigCharFactory {
    // 管理已经生成的BigChar的实例
    private HashMap pool = new HashMap();
    // Singleton模式
    private static BigCharFactory singleton = new BigCharFactory();
    // 构造函数
    private BigCharFactory() {
    }
    // 获取唯一的实例
    public static BigCharFactory getInstance() {
        return singleton;
    }
    // 生成（共享）BigChar类的实例
    public synchronized BigChar getBigChar(char charname) {
        BigChar bc = (BigChar)pool.get("" + charname);
        if (bc == null) {
            bc = new BigChar(charname); // 生成BigChar的实例
            pool.put("" + charname, bc);
        }
        return bc;
    }
}

public class BigString {
    // “大型字符”的数组
    private BigChar[] bigchars;
    // 构造函数
    public BigString(String string) {
        bigchars = new BigChar[string.length()];
        BigCharFactory factory = BigCharFactory.getInstance();
        for (int i = 0; i < bigchars.length; i++) {
            bigchars[i] = factory.getBigChar(string.charAt(i));
        }
    }
    // 显示
    public void print() {
        for (int i = 0; i < bigchars.length; i++) {
            bigchars[i].print();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println("Usage: java Main digits");
            System.out.println("Example: java Main 1212123");
            System.exit(0);
        }
        BigString bs = new BigString(args[0]);
        bs.print();
    }
}
```

## 角色

- Flyweight

使用共享对象

- FlyweightFactory

生成 Flyweight 的工厂

- Client

请求者，使用 FlyweightFactory 生成 Flyweight

## 要点

- 对多个地方产生影响

一个实例的改变会反映到所有使用该实例的地方

- Intrinsic Extrinsic

应当共享的信息被称作 Intrinsic 信息。不应当共享的信息被称作 Extrinsic 信息

- 不要让共享的实例被回收

## 相关

- Proxy 模式

如果生成实例的处理需要花费较长实践，可以使用 Flyweight 模式来提高程序处理速度

- Composite 模式

有时可以是用 Flyweight 模式共享 Composite 模式中的 Leaf 角色

- Singleton 模式

在 Flyweight 角色中有时会使用 Singleton 模式

如果使用了 Singleton 模式，由于只会生成一个 Singleton 角色，因此所有使用该实例的地方都共享同一个实例。在 Singleton 实例中只持有 intrinsic 信息