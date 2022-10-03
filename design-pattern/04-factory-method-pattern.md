# Factory Method 模式

## Template Method 模式用于生成实例

父类决定生成实例的方式，但并不决定所要生成的具体的类，具体处理过程交给子类，将生产实例的框架与实际负责生产实例的类解耦

```java
public abstract class Product{
    public abstract void use();
}

public abstract class Factory{
    public final Product create(String owner){
        Product p = createProduct(owner);
        registerProduct(p);
        return p;
    }
    protected abstract Product createProduct(String owner);
    protected abstract void registerProduct(Product product);
}

public class IDCard extends Product{
    private String owner;
    IDCard(String owner){
        System.out.println("制作" + owner + "的 ID 卡");
        this.owner = owner;
    }
    public void use(){
        System.out.println("使用" + owner + "的 ID 卡");
    }
    public String getOwner(){
        return owner;
    }
}

public class IDCardFactory extends Factory{
    private List owners = new ArrayList();
    protected Product createProduct(String owner){
        return newIDCasrd(owner);
    }
    protected void registerProduct(Product product){
        owners.add(((IDCard)product).getOwner());
    }
    public List getOwners(){
        return owners;
    }
}

public class Main{
    public static void main(String[] args){
        Factory factory = new IDCardFactory();
        Product card1 = factory.create("abc");
        Product card2 = factory.create("abc2");
        Product card3 = factory.create("abc3");
        card1.user();
        card2.user();
        card3.user();
    }
}
```

## 角色

- Product

框架层，抽象类，定义生成的实例所持有的接口，具体处理由实现类决定

- Creator

框架层，抽象类，负责生成 Product，具体处理由实现类决定。其生成实例方法不用 new 关键字，而是使用专用方法来生成实例，实现解耦

- ConcreteProduct

具体加工方，决定了具体产品

- ConcreteCreator

具体加工坊，负责生成具体产品

## 要点

- 框架与具体加工分包管理

相互不引用解耦，框架可以用于生成其他产品，framework 包 不依赖于 idcard 包

- 生成实例通常由三种方式
  - 通过抽象方法
  - 默认处理，使用 new 生成 Product，此时 Product 不能定义成抽象类
  - 在默认方法中抛出异常，表示默认方法必须由子类重写，变相的抽象

## 相关

- Template Method

Template Method 模式，Factory Method 模式是其典型应用

- Singleton

Singleton 模式，多数情况下，可以将 Singleton 模式用于 Creator 角色

- Composite

Composite 模式，有时可以将 Composite 模式用于 Product 角色

- Iterator

Iterator 模式，可以使用 Factory Method 模式生成 Iterator 实例