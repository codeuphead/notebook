# AbstractFactory 模式

## 将关联零件组成产品

抽象 factory 包

```java
public abstract class Item {
    protected String caption;

    public Item(String caption) {
        this.caption = caption;
    }
    public abstract String makeHTML();
}

public abstract class Link extends Item {
    protected String url;
    public Link(String caption, String url) {
        super(caption);
        this.url = url;
    }
}

public abstract class Tray extends Item {
    protected ArrayList tray = new ArrayList();
    public Tray(String caption) {
        super(caption);
        public void add(Item item) {
            tray.add(item);
        }
    }
}

public abstract class Page {
    protected String title;
    protected String author;
    protected ArrayList content = new ArrayList();
    public Page(String title, String author) {
        this.title = title;
        this.author = author;
    }
    public void add(Item item) {
        content.add(item);
    }
    public void output(){
        try {
            String filename = title + ".html";
            Writer writer = new FileWriter(filename);
            writer.write(makeHTML());
            writer.close();
            System.out.println(filename + "编写完成");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public abstract String makeHTML();
}

public abstract class Factory {
    public abstract Factory getFactory(String classname) {
        Factory factory = null;
        try {
            factory = (Factory)Class.forName(classname).newInstance();
        } catch (ClassNotFoundException e) {
            System.err.pringln("没有找到" + classname);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return factory;
    }
    public abstract Link createLink(String caption, String url);
    public abstract Tray createTray(String caption);
    public abstract Page createPage(String title, String author);
}
```

```java
public static void main(String[] args) {
    if (args.length != 1) {
        System.out.println("Usage: java Main class.name.of.ConcreteFactory");
        System.out.println("Example 1: java Main listFactory.ListFactory");
        System.out.println("Example 2: java Main tableFactory.TableFactory");
        System.exit(0);
    }
    Factory factory = Factory.getFactory(args[0]);
    Link people = factory.createLink("人民日报"， "http://www.people.com.cn/");
    Link gmw = factory.createLink("光明日报", "http://www.gmw.cn/");
    Link us_yahoo = factory.createLink("Yahoo!", "http://www.yahoo.com/");
    Link jp_yahoo = factory.createLink("Yahoo!Japan", "http://www.yahoo.co.jp/");
    Link excite = factory.createLink("Excite", "http://www.excite.com/");
    Link google = factory.createLink("Google", "http://www.google.com/");

    Tray traynews = factory.createTray("日报");
    traynews.add(people);
    traynews.add(gmw);

    Tray trayyahoo = factory.createTray("Yahoo!");
    trayyahoo.add(us_yahoo);
    trayyahoo.add(jp_yahoo);

    Tray traysearch = factory.createTray("搜索引擎");
    traysearch.add(trayyahoo);
    traysearch.add(excite);
    traysearch.add(google);

    Page page = factory.createPage("LinkPage", "xxx");
    page.add(traynews);
    page.add(traysearch);
    page.output();
```

实现包

ListFactory

```java
public class ListFactory extends Factory {
    public Link createLink(String caption, String url) {
        return new ListLink(caption, url);
    }
    public Tray createTray(String caption) {
        return new ListTray(caption);
    }
    public Page createPage(String title, String author) {
        return new ListPage(title, author);
    }
}

public class ListLink extends Link {
    public ListLink(String caption, String url) {
        super(caption, url);
    }
    public String maktHTML() {
        return " <li><a href=\"" + url + "\">" + caption + "</a></li>\n";
    }
}

public class ListTray extends Tray {
    public ListTray(String caption) {
        super(caption);
    }
    public String makeHTML() {
        StringBuffer buffer = new StringBuffer();
        buffer.append("<li>\n");
        buffer.append(caption + "\n");
        buffer.append("<ul>\n");
        Iterator it = tray.iterator();
        while (it.hasNext()) {
            Item item = (Item)it.next();
            buffer.append(item.makeHTML());
        }
        buffer.append("</ul>\n");
        buffer.append("</li>\n");
        return buffer.toString();
    }
}

public class ListPage extends Page {
    public ListPage(String title, String author) {
        super(title, author);
    }
    public String makeHTML() {
        StringBuffer buffer = new StringBuffer();
        buffer.append("<html><head><title>" + title + "</title></head>\n");
        buffer.append("<body>\n");
        buffer.append("<h1>" + title + "</h1>\n");
        buffer.append("<ul>\n");
        Iterator it = content.iterator();
        while (it.hasNext()) {
            Item item = (Item)it.next();
            buffer.append(item.makeHTML());
        }
        buffer.append("</ul>\n");
        buffer.append("<hr><address>" + author + "</address>");
        buffer.append("</body></html>\n");
        return buffer.toString();
    }
}
```

TableFactory

```java
public class TableFactory extends Factory {
    public Link createLink(String caption, String url) {
        return new TableLink(caption, url);
    }
    public Tray createTray(String caption) {
        return new TableTray(caption);
    }
    public Page createPage(String title, String author) {
        return new TablePage(title, author);
    }
}

public class TableLink extends Link {
    public TableLink(String caption, String url) {
        super(caption, url);
    }
    public String makeHTML() {
        return "<td><a href=\"" + url + "\">" + caption + "</a></td>\n";
    }
}

public class TableTray extends Tray {
    public TableTray(String caption) {
        super(caption); // 使用super(...)表达式
    }
    public String makeHTML() {
        StringBuffer buffer = new StringBuffer();
        buffer.append("<td>");
        buffer.append("<table width=\"100%\" border=\"1\"><tr>");
        buffer.append("<td bgcolor=\"#cccccc\" align=\"center\" colspan=\""+ tray.size() + "\"><b>" + caption + "</b></td>");
        buffer.append("</tr>\n");
        buffer.append("<tr>\n");
        Iterator it = tray.iterator();
        while (it.hasNext()) {
            Item item = (Item)it.next();
            buffer.append(item.makeHTML());
        }
        buffer.append("</tr></table>");
        buffer.append("</td>");
        return buffer.toString();
    }
}

public class TablePage extends Page {
    public TablePage(String title, String author) {
        super(title, author);
    }
    public String makeHTML() {
        StringBuffer buffer = new StringBuffer();
        buffer.append("<html><head><title>" + title + "</title></head>\n");
        buffer.append("<body>\n");
        buffer.append("<h1>" + title + "</h1>\n");
        buffer.append("<table width=\"80%\" border=\"3\">\n");
        Iterator it = content.iterator();
        while (it.hasNext()) {
            Item item = (Item)it.next();
            buffer.append("<tr>" + item.makeHTML() + "</tr>");
        }
        buffer.append("</table>\n");
        buffer.append("<hr><address>" + author + "</address>");
        buffer.append("</body></html>\n");
        return buffer.toString();
    }
}
```

## 角色

- AbstractProduct

负责定义 AbstractFactory 角色生成的抽象零件和产品的接口

- AbstractFactory

负责定义用于生成抽象产品的接口

- Client

角色会调用 AbstractFactory 和 AbstractProduct 角色的接口来进行工作，对于工厂、产品的实现细节是不知道的

- ConcreteProduct

负责实现 AbstractProduct 接口

- ConcreteFactory

负责实现 ConcreteFactory 接口

## 要点

- 易于增加具体的工厂

- 难以增加新的零件

如果增加新的零件，需要修改所有具体的工厂才行

## 相关

- Builder 模式

分阶段的制作复杂的实例；本模式通过调用抽象产品的接口来组装抽象产品，生成复杂结构的实例

- Factory Method 模式

有时 Abstract Factory 模式在制作产品时，会使用 Factory Method 模式

- Composite 模式

有时 Abstract Factory 模式在制作产品时，会使用 Composite 模式

- Singleton 模式

有时 Abstract Factory 模式在制作产品时，会使用 Singleton 模式