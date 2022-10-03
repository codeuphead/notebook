# Facade 模式

## 简单窗口

可以为相互关联在一起的错综复杂的类，整理出高层接口。Facade 角色可以让系统对外只有一个简单的接口，而且 Facade 角色还会考虑到系统内部各个类之间的责任关系和依赖关系，按照正确的顺序调用各个类

```java
public class Database {
    private Database() {    // 防止外部new出Database的实例，所以声明为private方法
    }
    public static Properties getProperties(String dbname) { // 根据数据库名获取Properties
        String filename = dbname + ".txt";
        Properties prop = new Properties();
        try {
            prop.load(new FileInputStream(filename));
        } catch (IOException e) {
            System.out.println("Warning: " + filename + " is not found.");
        }
        return prop;
    }
}

public class HtmlWriter {
    private Writer writer;
    public HtmlWriter(Writer writer) {  // 构造函数
        this.writer = writer;
    }
    public void title(String title) throws IOException {    // 输出标题
        writer.write("<html>");
        writer.write("<head>");
        writer.write("<title>" + title + "</title>");
        writer.write("</head>");
        writer.write("<body>\n");
        writer.write("<h1>" + title + "</h1>\n");
    }
    public void paragraph(String msg) throws IOException {  // 输出段落
        writer.write("<p>" + msg + "</p>\n");
    }
    public void link(String href, String caption) throws IOException {  // 输出超链接
        paragraph("<a href=\"" + href + "\">" + caption + "</a>");
    }
    public void mailto(String mailaddr, String username) throws IOException {   //  输出邮件地址
        link("mailto:" + mailaddr, username);
    }
    public void close() throws IOException {    // 结束输出HTML
        writer.write("</body>");
        writer.write("</html>\n");
        writer.close();
    }
}

public class PageMaker {
    private PageMaker() {   // 防止外部new出PageMaker的实例，所以声明为private方法
    }
    public static void makeWelcomePage(String mailaddr, String filename) {
        try {
            Properties mailprop = Database.getProperties("maildata");
            String username = mailprop.getProperty(mailaddr);
            HtmlWriter writer = new HtmlWriter(new FileWriter(filename));
            writer.title("Welcome to " + username + "'s page!");
            writer.paragraph("欢迎来到" + username + "的主页。");
            writer.paragraph("等着你的邮件哦！");
            writer.mailto(mailaddr, username);
            writer.close();
            System.out.println(filename + " is created for " + mailaddr + " (" + username + ")");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        PageMaker.makeWelcomePage("hyuki@hyuki.com", "welcome.html");
    }
}
```

## 角色

- Facade

代表构成系统的许多其他角色的简单窗口，Facade 角色向系统外部提供高层接口

- 构成系统的其他角色

各自完成自己的工作，他们并不知道 Facade 角色

- Client

负责调用 Facade 角色

## 要点

- Facade 角色的工作

让复杂的东西看起来简单。接口变少了，程序与外部的关联关系弱化了，更容易是我们的包作为组件复用

还需要考虑将那些方法、字段设为 public。如果公开的过多，会导致类的内部修改变得困难

- 递归使用 Facade 模式

## 相关

- Abstract Factory 模式

可以将此模式看作生成复杂实例时的 Facade 模式。

- Singleton 模式

有时会使用 Singleton 模式创建 Facade 角色

- Mediator 模式

本模式单方面使用其他角色来提供高层接口；Mediator 模式中，Mediator 角色作为 Colleague 角色间的仲裁者负责调停。Facade 模式是单向的，Mediator 角色是双向的