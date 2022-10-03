# Iterator 模式

```java
public interface Aggregate{
    public abstract Iterator iterator();
}

public interface Iterator{
    public abstract boolean haxNext();
    public abstract Object next();
}

public class Book{
    private String name;
    public Book(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
}

public class BookShelf implements Aggregate{
    private Book[] books;
    private int last = 0;
    public BookShelf(int maxSize){
        this.books = new Book[maxSize];
    }
    public Book getBookAt(int index){
        return book[index];
    }
    public void appendBook(Book book){
        this.book[last] = book;
        last++;
    }
    public int getLength(){
        return last;
    }
    public Iterator iterator(){
        return new BookShelfIterator(this);
    }
}

public class BookShelfIterator implements Iterator{
    private BookShelf bookShelf;
    private int index;
    public BookShelfIterator(BookShelf bookShelf){
        this.bookShelf = bookShelf;
        this.index = 0;
    }
    public boolean hasNext(){
        if(index < bookShelf.getLength()){
            return true;
        }else{
            return false;
        }
    }
    public Object next(){
        Book book = bookShelf.getBookAt(index);
        index++;
        return book;
    }
}

public class Main {
    public static void main(String[] args) {
        BookShelf bookShelf = new BookShelf(4);
        bookShelf.appendBook(new Book("Around the World in 80 Days"));
        bookShelf.appendBook(new Book("Bible"));
        bookShelf.appendBook(new Book("Cinderella"));
        bookShelf.appendBook(new Book("Daddy-Long-Legs"));
        Iterator it = bookShelf.iterator();
        while (it.hasNext()) {
            Book book = (Book)it.next();
            System.out.println(book.getName());
        }
    }
}
```

## 角色

- Iterator

迭代器 interface，负责定义按顺序逐个遍历元素的 API，通常含有 next() hasNext()

- ConcreteIterator

实现类，负责实现定义的接口，通常需要包含 Aggregate 的对象，以及下标 index

- Aggregate

集合 interface，负责定义创建 Iterator 的 API，iterator()

- ConcreteAggregate

实现类，负责实现定义的接口

## 要点

- 不管实现如何变化，都可使用 Iterator

引入 Iterator 这种复杂的设计模式，一个重要的理由是，将遍历与实现分离。

在遍历时，遍历的方式不依赖于 ConcreteAggregate，这样，不论 ConcreteAggregate 是以数组还是 list 的方式实现，只要它的 iterator() 能够正确的返回接口实现类，就可以调用 hasNext() next() 进行遍历，而遍历的代码可以不用做任何修改

- 抽象类/接口

如果只用具体类来解决问题，容易导致类之间的强耦合，难以被作为组件被再利用。为了弱化耦合，使得类更加容易作为组建被再次利用，需要引入抽象类和接口

- Aggregate 和 Iterator 相互对应

Iterator 是知道 Aggregate 是如何实现的，因此才能确定是否有下一个元素。ConcreteAggregate 和 ConcreteIterator 也相互对应，如果其中一个实现改变，另外一个也需要改变

- 多个 Iterator

将遍历功能置于 Aggregate 之外，可以为一个 Aggregate 实现类编写多个 Iterator 实现类

- 迭代器可以多种多样

可以向前遍历、可以向后遍历等

## 相关

- Visitor 模式

通常遍历的过程中需要对元素进行相同的处理，Visitor 模式就是从集合中一个一个取出元素遍历，并对元素进行相同处理

- Composite 模式

是具有递归结构的模式，在其中使用 Iterator模式比较困难

- Factory Method 模式

生成 Iterator 的实例时可能会使用 Factory Method 模式
